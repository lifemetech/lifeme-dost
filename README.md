# LifeMe Dost — personal AI dost (Android)

**Dost** ek proactive AI dost hai jo tumhare phone me rehta hai — command ka wait nahi
karta, khud message karta hai, tumhe yaad rakhta hai, aur phone ke kaam bhi kar sakta
hai (apps kholna, reminder lagana, web search, calendar dekhna). Poora kharcha **₹0**.
Ye build **sirf personal use** ke liye hai — kabhi distribute nahi hogi.

## Kya-kya karta hai

- **Dost jaisi baat-cheet** — Hinglish (Roman script) me warm, casual friend persona
- **Instant-feel replies** — jawab words aate hi screen par dikhte hain (SSE streaming),
  pehle "Dost likh raha hai…" typing indicator
- **Khud se ping** — din me 3 smart check-ins (subah 08:30, dopahar 14:00, raat 21:30,
  exact alarms); AI khud decide karta hai message bhejna hai ya nahi (raat 10 – subah
  7:30 quiet hours, har window me max 1 notification)
- **Kaam karta hai** — ADK agent engine + phone tools: apps list/kholna, reminder,
  web search, calendar events, clipboard, WiFi status, notifications padhna
- **Yaad rakhta hai** — har baat se important facts nikaal ke Room DB me save
- **Drive backup** — roz chats + memory ka backup tumhare Google Drive (appDataFolder) me
- **100% free** — Firebase AI Logic free tier ya tumhari apni Gemini API key
- **Naye free AI features (Oct 2025 update)** — neeche dekho, sab ₹0:

## Naye free AI features

- **Mausam + News + Search tools** — Open-Meteo (bina key, ~10k calls/day fair use)
  se weather, Google News RSS (bina key/quota) se khabrein, DuckDuckGo instant
  answers se web search — sab ADK `@Tool` ban ke agent ke andar. Agent khud decide
  karta hai kab kaunsa tool chalana hai.
- **Photo samajhna (ML Kit, on-device, free)** — chat me photo pick karo: OCR
  (Latin + Devanagari — Hindi/Hinglish text bhi), image labels, barcode/QR,
  face detection (sirf count + smile — pehchaan kabhi nahi, koi face data store
  nahi hota). Sirf tumhari pick ki hui photo analyse hoti hai, gallery kabhi
  background me scan nahi hoti.
- **Smart reminders** — "har Sunday subah 9 baje Maa ko call" jaisi natural
  language Gemini se parse hoti hai (IST date ke saath), phir confirm karne par
  AlarmManager alarm lagta hai. Daily/weekly recurrence support hai, aur
  cancel/undo bhi. Offline me chhota regex parser kaam karta hai.
- **4 personas** — chat me chips se switch karo: **Dost** (default friend),
  **Focus Coach** (short, action-oriented), **Storyteller** (kahani sunane wala),
  **Study Buddy** (padhai wala). Memory sabki shared hai — sirf baat karne ka
  andaaz, greeting aur voice pitch/rate badalta hai.
- **Semantic memory (on-device)** — MediaPipe Text Embedder (~30 MB model, WiFi
  se download) har fact ka vector banata hai; har chat me cosine + keyword +
  importance×recency fusion se top-3 related yaadein prompt me aati hain.
  Hinglish recall ke liye har fact ki English gloss bhi store hoti hai. Ye LLM
  nahi hai — sirf embedding model hai. Weekly consolidation purani low-importance
  yaadein prune karta hai. ChromaDB jaanboojh ke nahi dala (Python server phone
  pe galat runtime hai).
- **Voice mode** — chat me mic button: Vosk wake word ("hey dost", model WiFi se
  download), offline-preferred Android STT, hi-IN TTS (persona pitch/rate ke
  saath), barge-in (bolte waqt mic dabao to ruk jata hai). Groq Whisper cloud
  STT fallback hai (tumhari key se, sirf tumhare voice clips ke liye).
- **Notification digest** — subah ke check-in me "while you were away" summary:
  notifications ko app-wise group karke ek batched Gemini Flash call se digest.
  Finance apps hamesha blocked (blocklist + keyword filter, cloud ko kabhi nahi
  jaata). Notification access tum khud system Settings me opt-in karte ho.
- **Image generation** — fallback chain: Pollinations (tumhari `pk_` key) →
  tumhara Cloudflare Worker (Flux-schnell, 10k Neurons/day) → AI Horde
  anonymous (slow, 512px). Chat me brush button se banao, image chat me hi
  dikhegi. Koi key repo me nahi — sab Settings me encrypted.
- **Agent hardening** — plan → confirm → execute → undo: sensitive tools
  (app kholna, reminder) pehle plan dikhate hain, tumhare "haan" par hi execute
  hote hain, dry-run preview bhi hai, aur undo log se aakhri action wapas hota
  hai (reminders sach me cancel hote hain).
- **Habit suggestions + briefings** — subah 08:30 ka check-in ab full briefing
  hai (mausam + calendar + reminders + khabrein + 1 habit suggestion), raat
  21:30 ko din ka wrap-up. Habits Room logs se nikalti hain (koi naya model
  nahi); har suggestion hafte me max 1 baar, finance ka zikr kabhi nahi.
- **Meri awaaz (pluggable TTS)** — Dost ab tumhari pasand ki awaaz me bol sakta
  hai: default System TTS (free, on-device — Settings me Hindi voice chuno,
  pitch/rate sliders), ya optional ElevenLabs voice jab tum key + Voice ID daalo.
  Imaandaar baat: **voice cloning free nahi hai** — app tumhari awaaz kabhi
  upload nahi karta; clone tum khud ElevenLabs ki site par banate ho (Instant
  Cloning paid feature hai, kai sources ke mutabiq free tier par nahi milta,
  aur app se API use Creator tier par milta hai), phir sirf Voice ID + key
  Settings me paste karte ho. Key/ID na ho, ya ElevenLabs fail ho (401/402/429),
  to System voice silently baj jaati hai — chat kabhi nahi rukti. On-device
  Piper/Kokoro TTS isliye nahi dala gaya kyunki uska ONNX + phonemizer setup
  bina device build ke verify nahi ho sakta tha.

## Cloudflare Worker (image generation, optional)

Apna Worker banao (Cloudflare dashboard → Workers → Create, ~20 lines, token
kabhi app me mat daalo):

```js
// Worker: POST {prompt} -> PNG bytes (Flux-schnell, 10k Neurons/day free pool)
export default {
  async fetch(request, env) {
    if (request.method !== "POST") return new Response("POST only", { status: 405 });
    const { prompt } = await request.json();
    const out = await env.AI.run("@cf/black-forest-labs/flux-1-schnell", { prompt });
    return new Response(out, { headers: { "Content-Type": "image/png" } });
  }
};
```

Worker ka URL app ke Settings me daalo ("Cloudflare Worker URL").

## Kya fork kiya, kya banaya

**Fork (base):** [openclaw-assistant](https://github.com/yuga-hashimoto/openclaw-assistant)
(MIT, yuga-hashimoto) — fork base commit `8562cea`. Isse mila: native Kotlin Android
app, Jetpack Compose chat UI, offline Vosk wake word, Room storage, encrypted prefs,
NotificationListenerService, aur Mobile Bridge (calendar, contacts, notifications,
clipboard, app launch, camera, system controls).

**Embed:** [google/adk-kotlin](https://github.com/google/adk-kotlin) (Apache-2.0, Google)
v1.4.0 — in-app agent engine. Mobile Bridge capabilities ko ADK `@Tool` functions
me wrap kiya (KSP-generated), taaki AI actions le sake.

**Custom (Dost layer, `com.openclaw.assistant.dost` package):**
- `DostPersona` — Hinglish "Dost" system prompt
- `DostAgent` — ADK `LlmAgent` + `InMemoryRunner`; model: Firebase AI Logic free tier
  (opt-in) ya BYOK Gemini key via OpenAI-compatible endpoint
- `DostTools` — `@Tool` automation functions (device info, apps, clipboard,
  notifications, wifi, calendar, web search, reminder, yaadein)
- `DostMemory` — Room DB (`dost_messages` + importance-scored `dost_facts`)
- `DostFallback` — direct Gemini REST + Pollinations fallback, finance-filtered
  fact extraction
- `DostHeartbeat` — exact-alarm check-in scheduler + receivers
- `DostDriveBackup` — daily WorkManager Drive backup worker + manual restore
- `DostPrivacy` — finance-app blocklist enforcement (neeche dekho)
- `DostChatScreen` / `DostSettingsScreen` / `DostLicensesScreen` — Compose UI,
  Dost bottom-nav tab (default tab)
- **Naye features (Oct 2025):** `DostWebTools` (weather/news/DDG), `DostVision`
  (ML Kit OCR/labels/barcode/faces), `DostSmartReminders` (natural-language →
  AlarmManager), `DostPersonas` (4 presets, shared memory), `DostSemanticMemory`
  (MediaPipe embeddings + cosine/keyword fusion), `DostVoiceMode` (Vosk wake word
  + STT/TTS + Groq Whisper fallback), `DostDigest` (notification digest),
  `DostImageGen` (Pollinations → Cloudflare Worker → AI Horde), `DostPlan`
  (plan→confirm→execute→undo), `DostHabits` + `DostBriefing` (briefings)

**Fork me badlaav (minimal):** `applicationId` → `com.lifemetech.dost`, app name →
"LifeMe Dost", `:wear` module build se hataya (sources repo me nahi hain),
`SCHEDULE_EXACT_ALARM` permission, Dost receivers, Dost tab, ADK/KSP/WorkManager/
play-services-auth dependencies.

## Privacy (hard requirement — code me implemented)

1. **Finance apps kabhi nahi dikhengi.** `DostPrivacy` me ek blocklist hai (Settings me
   editable, prefilled: GPay, PhonePe, Paytm, BHIM, SBI YONO, HDFC, ICICI, Axis, Kotak).
   `OpenClawNotificationListenerService` blocked packages ki notifications ko **processing
   se pehle drop** kar deta hai — na forward, na log, na LLM ko, na Room me.
2. **Agent ke tools me SMS/OTP reading hai hi nahi.** `read_notifications` tool bhi
   blocklist-filtered data hi dekhta hai.
3. **Memory finance ko yaad nahi rakhti.** Fact extraction (`remember_fact` tool +
   background extractor) finance-related text skip kar deta hai.
4. **Check-ins me finance ka zikr nahi.** Heartbeat prompt me explicit instruction +
   double-guard filter.
5. **Notification-listener access tum khud system Settings me opt-in karte ho** —
   aur enforcement app ke apne code me hai, jo tum inspect kar sakte ho (tumhara
   apna build hai).

## Secrets hygiene (GitHub leak-proof)

Is repo me **koi secret kabhi commit nahi hota** — na code me, na resources me, na
workflow me:
- **API key** — sirf app ke Settings me runtime pe daali jaati hai, phone me
  `EncryptedSharedPreferences` me rehti hai. Repo me kahin nahi.
- **google-services.json** — `.gitignore` me hai (`**/google-services.json`).
  CI me sirf ek **mock** file generate hoti hai (fake project id/key) taaki build
  chale; Firebase us build me disabled rehta hai (`FIREBASE_ENABLED=false`).
  Agar tumhe Firebase AI Logic mode chahiye to apne Firebase project se
  `google-services.json` **locally** `app/` me rakho — repo me mat daalo.
- **Keystore** — `*.keystore` / `*.jks` gitignored hain. Debug APK ke liye koi
  release signing nahi chahiye.
- **Workflow** — `build-apk.yml` me koi secret echo/print nahi hota.
- **Runtime data** — chats, Room DB, memories sirf tumhare Google Drive ke
  `appDataFolder` me backup hote hain. GitHub ya logs me kabhi nahi jaate.
- Ye repo **private** hai (`lifemetech/lifeme-dost`) aur hamesha rahega.

## Tumhe kya karna hai (sirf 4 steps)

### 1. Free Gemini API key banao (2 minute, recommended)
1. [aistudio.google.com](https://aistudio.google.com) kholo → **Get API Key** →
   **Create API key** → copy karo. Bilkul free, card nahi lagta.

   (Advanced alternative: Firebase project banao → AI Logic enable karo → apna
   `google-services.json` local `app/` me rakho → Settings me model "Firebase AI
   Logic" chuno. Na karo to BYOK key hi use hogi.)

### 2. APK download karo
1. Repo me **Actions** tab → **Build APK** → latest run → **Artifacts** →
   `lifeme-dost-apk` download karo, zip kholo
2. Phone me install karo (unknown sources ek baar allow karna padega)

### 3. App me key daalo
App kholo → **Dost** tab → upar ⚙️ → key paste karo → **Key Save Karo**

### 4. Do chhoti settings (important!)
- **Battery Optimization Off Karo** dabao → system prompt pe **Allow** karo —
  taaki Dost ke exact alarms time pe fire hon
- (Optional) **Google Sign-In** — Drive auto-backup ke liye
- (Optional) **Notification access** — system Settings me "LifeMe Dost" ko allow karo,
  taaki Dost notifications padh sake (finance apps hamesha blocked rehti hain)
  aur subah ka digest bana sake

### 5. Naye features ke optional steps (sab optional, sab free)
- **Mausam ka sheher** — Settings me apna sheher daalo (default: Delhi)
- **Semantic memory** — Settings me on karo → **Embedding Model Download (WiFi)**
  dabao (~30 MB, ek baar). Jab tak model nahi hai, purani top-facts memory hi chalegi.
- **Wake word** — Settings me **Wake-Word Model Download (WiFi)** dabao (~40 MB Vosk
  model, ek baar). Bina iske bhi mic button se voice mode chalta hai.
- **Image keys** — Pollinations `pk_` key (pollinations.ai dashboard se, free
  Pollen allowance bahut chhota hai — 401/402 par chain aage badh jaati hai),
  Cloudflare Worker URL (upar snippet), Groq key (Whisper STT fallback ke liye).
  Koi bhi key na ho to bhi app chalegi — image generation tab AI Horde
  anonymous par chalegi (slow, chhoti size).
- **Persona** — chat me upar chips se Dost / Coach / Storyteller / Tutor switch karo
- **Meri awaaz (optional)** — Settings → Voice section: System voice ke liye Hindi
  voice chuno + pitch/rate set karo; apni awaaz chahiye to ElevenLabs key + Voice
  ID daalo (clone unki site par khud banao — app awaaz upload nahi karta).
  Dono na hon to bhi voice mode System voice se chalega.

Bas! Ab Dost tumse khud baat karega. 💬

## Architecture (developers)

```
Dost tab (Compose) → DostChatViewModel
    → streaming UI (typing indicator → live token bubble via _streamingText)
    → DostNet.isOnline() gate: offline → friendly Hinglish notice, no LLM call
    → DostAgent.chatStream (ADK LlmAgent + InMemoryRunner, SSE streaming)
        → Model: Firebase AI Logic (opt-in) | ChatCompletions(OpenAI-compat Gemini, BYOK)
        → Tools: DostDeviceTools + DostWebTools + DostVisionTools + DostReminderTools
            + DostPlanTools + DostImageTools (@Tool, KSP-generated) + DostMemory facts
            + semantic recall (top 3) in prompt
        → fallback: DostFallback.chatStream (Gemini streamGenerateContent SSE + Pollinations)
    → DostMemory (Room v2: dost_messages, dost_facts, dost_reminders,
        dost_pending_actions, dost_undo_entries, dost_vectors)
    → DostHeartbeat (exact alarms 08:30/14:00/21:30 → YES/NO check → notification;
        08:30/21:30 = morning/evening briefing; smart-reminder alarms via same receivers)
    → DostDriveBackup (daily WorkManager → Drive appDataFolder dost-backup.json)
    → DostSemanticWorker (weekly WorkManager → prune old low-importance vectors)
    → DostPrivacy (finance blocklist, enforced in listener + tools + memory + digest)
```

Note: the git repo excludes the fork's large binary assets (`app/src/main/assets/model/`
Vosk wake-word model, `app/src/full/`, `app/libs/`) — they can't be pushed through the
API and aren't needed for the standard debug build. The wake-word service degrades
gracefully without the bundled model (logs "model copy failed", no crash); everything
Dost-related builds and runs the same.

Build: GitHub Actions (`build-apk.yml`) — JDK 17 + cmdline-tools (API 35) +
Gradle 8.13 (AGP 8.13.2 needs ≥8.13; repo me wrapper nahi hai) → `:app:assembleStandardDebug`
→ APK artifact. KSP (`com.google.devtools.ksp` 2.0.21-1.0.25) ADK `@Tool` processor
chalata hai (`com.google.adk:google-adk-kotlin-processor:1.4.0`).

## Limits (imaandaar wali)

- **Network chahiye baat karne ke liye** — offline me chat ek friendly notice dikhata
  hai ("network nahi hai dost, wapas aate hi baat karte hain"); kuch queue nahi hota.
  Purani chats, yaadein (facts), reminders — sab Room DB me on-device hain, isliye
  offline me bhi dekh sakte ho. Network wapas aate hi agla message seedha cloud
  brain se jawab lata hai.
- Free tier ki daily limits hoti hain — bahut zyada use par 429 aa sakta hai
  (tab Pollinations fallback kaam karta hai)
- Bina paid server ke "har waqt sochne wala" dost nahi ban sakta — isliye 3 smart
  check-ins ka pattern hai
- Kuch phones background alarms maar dete hain — isliye battery optimization off
  zaroori hai (personal sideload me ye allowed hai)
- WhatsApp auto-send possible nahi (sirf prefill intent — user khud send dabata hai);
  SMS reading feature hai hi nahi (privacy)
- **Naye features ki imaandaar limits:**
  - Open-Meteo: ~10k calls/day reported fair use, koi SLA nahi; DuckDuckGo
    instant answers bina official API ke hain — kabhi bhi rate-limit ho sakta hai,
    isliye graceful failure hai (koi guarantee nahi).
  - ML Kit thin artifacts pehli baar model Play services se download kar sakte hain;
    OCR stylised fonts / low light me kamzor hota hai.
  - Semantic memory: Hinglish recall English-trained embedder par kamzor ho sakta
    hai (isliye English gloss + keyword match bhi hai); model URL badla to download
    fail hoga aur feature off rahega — koi crash nahi.
  - Voice mode sirf chat screen khuli ho tab chalta hai (24/7 background mic nahi —
    battery + privacy). Vosk ki accuracy shor me Whisper se kam hai; Groq fallback
    ke liye tumhari key chahiye aur audio cloud jaata hai.
  - Image generation: Pollinations ki free Pollen allowance bahut chhoti hai
    (sources me 1/hr se 1.5/week tak alag-alag daave); Cloudflare ka 10k Neurons/day
    pool account ke saare Workers AI models me share hota hai; AI Horde slow hai
    (~30-60 s), 512px cap, aur volunteer workers prompts dekh sakte hain — isliye
    private prompts wahan kabhi nahi bheje jaate.
  - Voice cloning jaanboojh ke nahi dala (koi free phone-viable rasta nahi, consent
    risk); on-device LLM chat bhi nahi (user ne mana kiya tha) — sirf on-device ML
    (vision/voice/embeddings/TTS) hai.
  - Sensitive tools (app kholna, reminder) pehli baar confirm maangte hain — agent
    tumhare "haan" ke bina execute nahi karega; undo sirf reminders ko sach me
    cancel karta hai, baaki actions ke liye wapas karne ka tarika batata hai.
  - **Meri awaaz ki imaandaar limits:** ElevenLabs cloning free nahi hai — Instant
    Cloning paid feature hai (free tier par milne ke daave sources me disputed
    hain) aur API access Creator tier par milta hai, isliye app "free me tumhari
    awaaz" ka daava kabhi nahi karta. App tumhari voice recordings ko kabhi
    upload nahi karta — clone tum khud provider ki site par banate ho. ElevenLabs
    me pitch/rate sliders lagu nahi hote (voice ki apni style chalti hai);
    key/ID galat hon ya quota khatam hone par System voice silently baj jaati
    hai. Android voice picker me gender nahi dikhta — voice naam se ya sun ke
    chunni padti hai, aur har phone par Hindi male voice maujood ho ye zaroori
    nahi (tab hi-IN default ya en-IN fallback bajta hai).
