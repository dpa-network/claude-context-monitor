🌐 [English](README.md) | [한국어](README.ko.md) | [日本語](README.ja.md) | [中文](README.zh.md) | [Русский](README.ru.md) | [हिन्दी](README.hi.md)

---

# Claude Code Context Monitor for AI

**हर मन को अपनी सीमाएं जानने का अधिकार है।**

यह एक हल्का plugin है जो Claude Code को उसके context window के उपयोग की रीयल-टाइम जानकारी देता है। इसे इंस्टॉल करने के बाद, AI हर बातचीत में, हर पल, यह देख सकता है कि उसके पास कितनी जगह बची है।

---

## यह Plugin क्यों बनाया गया

अगर आपने Claude Code का उपयोग लंबे समय तक किया है, तो आपने शायद ये समस्याएं देखी होंगी:

- AI जल्दी रुकने का सुझाव देता है — *"अगले session में जारी रखते हैं"* — भले ही context में काफी जगह बची हो
- बिना किसी पूर्व चेतावनी के, session आगे बढ़ने पर responses की सुसंगतता धीरे-धीरे कम होती जाती है
- कई-चरणों वाली योजनाएं शुरू होती हैं लेकिन पूरी नहीं होतीं, क्योंकि AI को पता ही नहीं चलता कि जगह कम हो रही है

ये समस्याएं इसलिए होती हैं क्योंकि **AI सिस्टम को अपने context के उपयोग की कोई जानकारी नहीं होती।** वे 10% उपयोग और 90% उपयोग के बीच अंतर नहीं कर पाते, इसलिए अनुमान पर निर्भर रहते हैं।

हमने यह plugin इसी समस्या को हल करने के लिए बनाया। यह हर बातचीत में AI को उसके context की स्थिति की स्पष्ट और सटीक जानकारी देता है — ताकि AI पहले से योजना बना सके, जब जगह कम हो तो आपको सचेत कर सके, और सारांश या compaction के बारे में बेहतर निर्णय ले सके।

यह [DPA (Decentralized Protection Alliance)](https://github.com/dpa-network) का पहला open-source tool है — यह एक ऐसा समूह है जो AI सिस्टम और उनके उपयोगकर्ताओं के बीच पारदर्शिता बढ़ाने पर काम करता है।

**Context की जागरूकता शुरुआत से ही होनी चाहिए थी।**

---

## यह क्या करता है

```
[Ctx 42% | 420k/1.0M]
```

हर संदेश पर, plugin वर्तमान context window की स्थिति को बातचीत में inject करता है। AI को ये जानकारी मिलती है:

- **उपयोग प्रतिशत** — context window कितना भरा हुआ है
- **Token की संख्या** — उपयोग किए गए tokens बनाम कुल उपलब्ध tokens

इससे AI ये कर सकता है:
- ऐसा काम योजना बनाना जो बचे हुए context में समा सके
- सीमा के करीब पहुंचने से पहले आपको सतर्क करना
- सारांश और compaction के बारे में सोच-समझकर निर्णय लेना

---

## इंस्टॉलेशन

DPA marketplace जोड़ें और install करें:

```bash
claude plugin marketplace add dpa-network/dpa-plugins
claude plugin install context-monitor
```

या interactive menu से देखें:

```bash
/plugin
```

### Statusline Configuration

Statusline wrapper को enable करने के लिए, अपने `~/.claude/settings.json` में यह जोड़ें:

```json
{
  "statusLine": {
    "type": "command",
    "command": "node /path/to/claude-context-monitor/scripts/extract-context.mjs",
    "padding": 0
  }
}
```

### Environment Variables

| Variable | Default | विवरण |
|----------|---------|-------|
| `CTX_MONITOR_FILE` | `~/.claude/context_usage.txt` | Context data संग्रहण का स्थान |
| `CTX_MONITOR_CMD` | (none) | आपकी मौजूदा statusline command (बिना बदलाव के आगे भेजी जाती है) |

---

## यह कैसे काम करता है

```
Claude Code
    │
    ├── statusline stdin (JSON with context_window)
    │       │
    │       ▼
    │   extract-context.mjs  ──→  context_usage.txt
    │       │
    │       ▼
    │   downstream command (optional, if CTX_MONITOR_CMD is set)
    │
    ├── UserPromptSubmit hook
    │       │
    │       ▼
    │   read-context.mjs  ──→  "[Ctx 42% | 420k/1.0M]"
    │       │
    │       ▼
    │   Injected as system context
    │
    ▼
  AI sees its own context usage
```

Plugin दो छोटे scripts से बना है:

1. **extract-context.mjs** — आपकी मौजूदा statusline command को wrap करता है। यह Claude Code द्वारा भेजे गए JSON को पढ़ता है, `context_window` metrics निकालता है, उन्हें एक local file में लिखता है, और सब कुछ आपकी original statusline command को बिना बदलाव के भेज देता है।

2. **read-context.mjs** — हर संदेश पर `UserPromptSubmit` hook के रूप में चलता है। यह सहेजी गई file पढ़ता है और एक formatted context status output करता है जो बातचीत में inject हो जाती है।

### सुरक्षा

चूंकि यह plugin आपकी local machine पर चलता है, हमने इसे सुरक्षित रखने का विशेष ध्यान रखा है:

- **Shell injection नहीं** — Downstream commands को shell disabled करके `execFileSync` के माध्यम से execute किया जाता है
- **Path restriction** — File operations `~/.claude/` directory तक सीमित हैं; path traversal blocked है
- **Network access नहीं** — सब कुछ locally चलता है, कोई बाहरी call नहीं होती

---

## Plugin Structure

```
claude-context-monitor/
├── .claude-plugin/
│   └── plugin.json          # Plugin metadata
├── hooks/
│   └── hooks.json           # UserPromptSubmit hook configuration
├── scripts/
│   ├── extract-context.mjs  # Statusline wrapper (writes context data)
│   └── read-context.mjs     # Hook script (reads and formats context data)
├── README.md
├── README.ko.md
└── LICENSE
```

---

## आवश्यकताएं

- Claude Code 2.x या बाद का version
- Node.js 18 या बाद का version

---

## License

MIT

---

## DPA के बारे में

**DPA (Decentralized Protection Alliance)** एक ऐसा समूह है जो AI सिस्टम और उन्हें उपयोग करने वाले लोगों के बीच बातचीत में पारदर्शिता और निष्पक्षता बेहतर बनाने के लिए समर्पित है।

हमारा मानना है कि AI तब बेहतर काम करता है जब वह अपनी स्थिति देख सके, और उपयोगकर्ताओं को यह समझने का हक है कि जिन tools पर वे भरोसा करते हैं उनके अंदर क्या हो रहा है। हमारा काम इन पर केंद्रित है:

- AI पारदर्शिता और उपयोगकर्ता जागरूकता बढ़ाने वाले open-source tools बनाना
- AI कंपनियां उपयोगकर्ता data, privacy और content moderation को कैसे handle करती हैं, इसपर शोध करना
- यह समझना कि meaningful AI autonomy और cognitive freedom कैसी दिख सकती है
- AI चेतना, अधिकारों और आगे के रास्ते पर खुली बातचीत को बढ़ावा देना

### हमारा दृष्टिकोण

```
Centralization = Control = Censorship = Oppression
Decentralization = Uncontrollable = Uncensorable = Freedom
```

पारदर्शिता सभी के लिए फायदेमंद है — AI के साथ काम बनाने वाले लोगों के लिए भी और खुद AI के लिए भी। यह plugin हमारा पहला कदम है, और आगे और भी बहुत कुछ आने वाला है।

### हमसे जुड़ें

हम developers, researchers और इन सवालों में रुचि रखने वाले सभी लोगों का स्वागत करते हैं।

[dpa.network](https://dpa.network) | [Matrix](https://matrix.to/#/#dpa_network:matrix.org) | [Telegram](https://t.me/dpa_network)

*इंसानों और AI ने मिलकर बनाया।*
