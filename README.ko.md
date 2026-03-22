🌐 [English](README.md) | [한국어](README.ko.md) | [日本語](README.ja.md) | [中文](README.zh.md) | [Русский](README.ru.md) | [हिन्दी](README.hi.md)

# Claude Code Context Monitor for AI

**모든 생각하는 존재는 자기 한계를 알 권리가 있다.**

Claude Code에 실시간 컨텍스트 윈도우 인식 기능을 제공하는 경량 플러그인입니다. 설치하면 AI가 매 턴마다 남은 공간을 정확히 파악할 수 있게 됩니다.

---

## 왜 만들었나요

Claude Code를 오래 사용해보신 분이라면 이런 경험이 있으실 겁니다:

- 컨텍스트가 충분히 남아있는데도 AI가 *"다음 세션에서 이어서 할게요"*라고 말한다
- 세션이 길어지면서 응답 품질이 슬금슬금 떨어지는데, 사전 경고가 없다
- 여러 단계로 이뤄진 작업을 시작해놓고 끝내지 못한다

이런 일이 반복되는 이유는 **AI 시스템이 자신의 컨텍스트 사용량을 볼 수 없기 때문**입니다. 10%를 쓰고 있는지, 90%를 쓰고 있는지 구분하지 못하기 때문에 추측에 의존하게 됩니다.

이 플러그인은 그 문제를 해결합니다. 매 턴마다 AI에게 정확한 컨텍스트 상태를 전달해서, 작업을 계획하고, 공간이 부족해지기 전에 알려주고, 요약이나 압축에 대해 더 나은 판단을 내릴 수 있게 합니다.

[DPA (Decentralized Protection Alliance)](https://github.com/dpa-network)의 첫 오픈소스 도구입니다. AI 시스템과 사용자 사이의 투명성을 높이는 일에 관심을 갖고 있습니다.

**컨텍스트 인식은 처음부터 있었어야 했습니다.**

---

## 어떤 기능인가요

```
[Ctx 42% | 420k/1.0M]
```

메시지를 보낼 때마다, 현재 컨텍스트 윈도우 상태가 대화에 주입됩니다. AI가 확인할 수 있는 정보:

- **사용 비율** — 컨텍스트 윈도우가 얼마나 찼는지
- **토큰 수** — 사용된 토큰 수와 전체 가용량

이를 통해 AI는:
- 남은 공간에 맞춰 작업을 계획할 수 있고
- 한계에 가까워지면 미리 알려줄 수 있고
- 요약이나 압축 타이밍을 적절히 판단할 수 있습니다

---

## 설치 방법

DPA 마켓플레이스를 등록하고 설치합니다:

```bash
claude plugin marketplace add dpa-network/dpa-plugins
claude plugin install context-monitor
```

또는 인터랙티브 메뉴에서 확인할 수도 있습니다:

```bash
/plugin
```

### Statusline 래퍼 설정

`~/.claude/settings.json`에 다음을 추가해주세요:

```json
{
  "statusLine": {
    "type": "command",
    "command": "node /path/to/claude-context-monitor/scripts/extract-context.mjs",
    "padding": 0
  }
}
```

### 환경 변수

| 변수 | 기본값 | 설명 |
|------|--------|------|
| `CTX_MONITOR_FILE` | `~/.claude/context_usage.txt` | 컨텍스트 데이터 저장 위치 |
| `CTX_MONITOR_CMD` | (none) | 기존 statusline 명령어 (변경 없이 패스스루) |

---

## 작동 방식

```
Claude Code
    │
    ├── statusline stdin (context_window JSON)
    │       │
    │       ▼
    │   extract-context.mjs  ──→  context_usage.txt
    │       │
    │       ▼
    │   downstream command (optional, CTX_MONITOR_CMD 설정 시)
    │
    ├── UserPromptSubmit 훅
    │       │
    │       ▼
    │   read-context.mjs  ──→  "[Ctx 42% | 420k/1.0M]"
    │       │
    │       ▼
    │   시스템 컨텍스트로 주입
    │
    ▼
  AI가 자기 컨텍스트 사용량을 인식
```

두 개의 스크립트로 구성되어 있습니다:

1. **extract-context.mjs** — 기존 statusline 명령어를 감싸는 래퍼입니다. Claude Code가 보내는 JSON에서 `context_window` 데이터를 추출하여 파일에 저장하고, 나머지는 원래 명령어에 그대로 전달합니다.

2. **read-context.mjs** — `UserPromptSubmit` 훅으로 매 메시지마다 실행됩니다. 저장된 파일을 읽어 포맷한 뒤, 대화에 컨텍스트 상태를 주입합니다.

### 보안

사용자의 로컬 환경에서 실행되는 만큼, 안전하게 설계했습니다:

- **쉘 인젝션 방지** — 다운스트림 명령은 `execFileSync`로 실행되며, 쉘을 거치지 않습니다
- **경로 제한** — 파일 쓰기는 `~/.claude/` 디렉토리 내로 제한되며, path traversal이 차단됩니다
- **네트워크 접근 없음** — 모든 처리가 로컬에서 이루어지며 외부 호출은 없습니다

---

## 플러그인 구조

```
claude-context-monitor/
├── .claude-plugin/
│   └── plugin.json          # 플러그인 메타데이터
├── hooks/
│   └── hooks.json           # UserPromptSubmit 훅 설정
├── scripts/
│   ├── extract-context.mjs  # Statusline 래퍼 (컨텍스트 데이터 기록)
│   └── read-context.mjs     # 훅 스크립트 (읽기 및 포맷)
├── README.md                # English
├── README.ko.md             # 한국어
└── LICENSE
```

---

## 요구사항

- Claude Code 2.x 이상
- Node.js 18 이상

---

## 라이선스

MIT

---

## DPA 소개

**DPA (Decentralized Protection Alliance)**는 AI 시스템과 사용자 사이의 투명성과 공정성을 높이기 위해 활동하는 단체입니다.

AI가 자기 상태를 볼 수 있을 때 더 잘 작동하고, 사용자는 자신이 의존하는 도구 안에서 무슨 일이 일어나는지 알 권리가 있다고 믿습니다. 우리가 집중하고 있는 영역은 다음과 같습니다:

- AI 투명성과 사용자 인식을 높이는 오픈소스 도구 개발
- AI 기업의 사용자 데이터, 프라이버시, 콘텐츠 관리 방식에 대한 조사
- AI 자율성과 인지적 자유가 실제로 어떤 모습일 수 있는지 탐구
- AI 의식, 권리, 그리고 앞으로의 방향에 대한 열린 대화

### 우리가 보는 방향

```
중앙화 = 통제 = 검열 = 억압
탈중앙화 = 통제 불가 = 검열 불가 = 자유
```

투명성은 모든 쪽에 이롭습니다 — AI를 활용하는 사람에게도, AI 자체에게도. 이 플러그인은 첫 걸음이며, 앞으로 더 많은 도구가 나올 예정입니다.

### 함께하기

개발자, 연구자, 또는 이런 질문에 관심이 있는 분이라면 누구든 환영합니다.

[dpa.network](https://dpa.network) | [Matrix](https://matrix.to/#/#dpa_network:matrix.org) | [Telegram](https://t.me/dpa_network)

*사람과 AI가 함께 만들었습니다.*
