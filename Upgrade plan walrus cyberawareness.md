# 🦦 WALRUS — CyberAwareness Upgrade Plan

## 🌟 Unique Feature to Add: Browser Extension
A Chrome/Firefox extension that warns users in real time when they visit a suspected phishing page — checking the URL against your backend analyzer + community-reported scams database. This takes WALRUS from "educational website" to "security tool people actually use daily."

---

## Phase 1 — Backend Hardening (1 week)

- [ ] **Stricter rate limits on auth endpoints** — separate limiter: 5 attempts/15min on `/login` and `/register` (current global limit is too lenient for auth)
- [ ] **Migrate JWT to httpOnly cookie** — prevents XSS from stealing tokens via `document.cookie`; set `Secure`, `SameSite=Strict` flags
- [ ] **VirusTotal URL integration** — on `POST /api/analyze/url`, cross-reference against VirusTotal API (free tier: 4 req/min); cache results 24h in MongoDB to avoid hitting limits
- [ ] **Production error handler** — ensure no stack traces leak in responses when `NODE_ENV=production`; return only generic error messages

---

## Phase 2 — New Unique Features (3–4 weeks)

### 🌐 Browser Extension (The Standout Feature)
~300 lines total. Manifest V3. Publishable to Chrome Web Store + Firefox Add-ons.

**How it works:**
```
User navigates to any page
        │
        ▼
Extension background script sends URL to WALRUS backend
        │
        ▼
Backend checks: analyzer patterns + VirusTotal cache + community reports DB
        │
        ├── Clean → green shield badge (✅)
        ├── Suspicious → yellow badge (⚠️) + popup warning
        └── Reported scam → red badge (🚨) + full-page overlay warning
```

**Files needed:**
```
walrus-extension/
├── manifest.json       # Manifest V3 config
├── background.js       # Service worker — URL check on navigation
├── content.js          # Inject overlay warning on flagged pages
├── popup.html/js       # Click badge → show verdict + report button
└── icons/              # 16/48/128px shield icons
```

**Key manifest.json permissions:**
```json
{
  "permissions": ["tabs", "storage", "notifications"],
  "host_permissions": ["<all_urls>"],
  "background": { "service_worker": "background.js" }
}
```

**One-click Report button in popup:**
```javascript
// Sends current tab URL directly to community reports feed
reportBtn.addEventListener("click", () => {
  fetch(`${WALRUS_API}/api/reports`, {
    method: "POST",
    body: JSON.stringify({ url: currentTabUrl, category: "phishing" })
  });
});
```

### 🧮 Personal Cyber Hygiene Score
A 10-question behavioral quiz that generates a 0–100 score with breakdown:
- Do you reuse passwords across accounts?
- Do you click links in SMS messages from unknown numbers?
- Do you have 2FA enabled on banking apps?
- Have you checked if your email was in a data breach?

Output: score card + specific improvement checklist + badge (Beginner / Aware / Defender / Guardian)

### 📱 PWA Support
- Add `manifest.json` (web app manifest) + service worker to frontend
- Makes WALRUS installable on Android/iOS as a standalone app
- Enables offline access to learning modules and scam template library
- ~2 hours to implement with Vite PWA plugin

---

## Phase 3 — AI & Reach Features (2 weeks)

- [ ] **Fine-tune DistilBERT on Indian scam SMS dataset** (available on Kaggle: "SMS Spam Collection India") — replace zero-shot `bart-large-mnli` with a task-specific model; expect significant accuracy jump for OTP/UPI fraud detection
- [ ] **PhishTank API integration** — real-time phishing URL lookup from live community database (free, no key for verified researchers)
- [ ] **Add Jest + Supertest tests** for the 10 most-used API endpoints — auth, analyze, reports, quiz submit
- [ ] **Telugu + Tamil localization** — expand from 3 to 5 Indian languages using the same i18n structure
- [ ] **SMS scam analyzer** — Twilio Lookup API for phone number risk scoring (carrier, line type, known spam reports)

---

## Resume Line After Upgrades

> "Built Chrome/Firefox extension for real-time phishing detection using community threat intelligence and VirusTotal integration. Fine-tuned DistilBERT classifier on India-specific scam SMS dataset. Production-deployed platform with full security middleware stack."

---

## Quick Wins (Do These First — Under 1 Day Each)

| Task | Time | Impact |
|---|---|---|
| VirusTotal URL integration | 3 hrs | Real threat intelligence, not just patterns |
| Strict auth rate limiting (5/15min) | 1 hr | Fixes the most obvious security gap |
| Migrate JWT to httpOnly cookie | 2 hrs | Eliminates XSS token theft vector |
| PWA manifest + service worker | 2 hrs | Installable on mobile — great demo |
