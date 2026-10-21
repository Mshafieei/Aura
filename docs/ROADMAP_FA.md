# نقشه راه جامع Hamrah — منشی هوش مصنوعی با Agent/Skill Factory

## چشم‌انداز
Hamrah یک منشی شخصی مبتنی بر AI است که از iPhone قابل‌دسترسی باشد، کارهای روزمره را مدیریت کند و مهم‌تر از همه بتواند با درخواست مالک، Agent و Skillهای جدید طراحی، اعتبارسنجی، آزمایش، نسخه‌بندی و پس از تأیید منتشر کند. این سند مشخصات محصول است، نه ادعای پیاده‌سازی کامل.

## اصول معماری
- iOS: SwiftUI؛ دسترسی سریع از طریق Siri، App Intents، Shortcuts، ویجت و اعلان‌ها.
- Backend: FastAPI/Python یا معادل مستدل؛ API نسخه‌بندی‌شده، احراز هویت، دیتابیس و صف کار.
- Orchestrator مرکزی: طبقه‌بندی درخواست، ساخت plan، انتخاب ابزار/Skill، کنترل مجوز، اجرا، تأیید نتیجه و گزارش شفاف.
- Agent Registry و Skill Registry: تعریف، نسخه، وابستگی، وضعیت، مجوز، تست و تاریخچه.
- LLM Gateway: پشتیبانی از providerهای قابل‌تعویض، timeout، retry محدود، کنترل هزینه و عدم افشای کلیدها در کلاینت.
- Memory Service: حافظه کوتاه‌مدت و بلندمدت با منشأ، زمان، حساسیت، TTL و امکان مشاهده/ویرایش/حذف توسط کاربر.
- Event/Audit Log: ثبت حداقلی و امن اقدامات مهم، تصمیم‌های مجوز، تأییدها، خطاها و نتایج.
- Sandbox: اجرای کد تولیدشده خارج از فرایند اصلی، با محدودیت زمان، حافظه، شبکه، فایل‌سیستم و دسترسی.
- Approval Service: تأیید صریح برای ارسال پیام، انتشار، خرید، حذف، دسترسی جدید یا هر اقدام بیرونی حساس.

## فازها و معیار اتمام

### F0 — کشف نیاز و تعریف محصول
خروجی: PRD، personas، جریان‌های اصلی، threat model، فهرست non-goals، معیار موفقیت.
پذیرش: هر قابلیت دارای user story، اولویت، ریسک و معیار تست باشد.

### F1 — طراحی سیستم و قراردادها
خروجی: دیاگرام معماری، API contract، مدل داده، JSON Schemaهای AgentSpec/SkillSpec/Plan/Approval، ADRها.
پذیرش: قراردادها نسخه‌دار، قابل اعتبارسنجی و دارای مثال معتبر/نامعتبر باشند.

### F2 — زیرساخت توسعه
خروجی: ساختار monorepo، lint/format، type-check، تست خودکار، CI، مدیریت secrets، محیط dev/test.
پذیرش: پروژه از README با فرمان‌های مشخص بالا بیاید؛ CI روی PR اجرا شود.

### F3 — Backend پایه
خروجی: FastAPI، health/readiness، config، migration، auth، user-scoped data، error envelope، OpenAPI.
پذیرش: تست API و جداسازی داده کاربران؛ هیچ secret در کد یا log نباشد.

### F4 — اپ iOS پایه
خروجی: SwiftUI app، ورود، صفحه گفت‌وگو، وضعیت کار، تنظیمات حریم خصوصی، نمایش تأییدها.
پذیرش: build در Xcode؛ خطا و قطع شبکه به‌درستی نمایش داده شود؛ دسترسی‌ها opt-in باشند.

### F5 — Task/Reminder Core
خروجی: task CRUD، اولویت، موعد، منطقه زمانی، یادآوری، idempotency، اعلان‌های مجاز.
پذیرش: زمان‌ها و timezone درست؛ درخواست تکراری باعث ایجاد task تکراری نشود.

### F6 — Orchestrator v1
خروجی: parse intent، plan ساختاریافته، انتخاب ابزار allowlisted، اجرا، جمع‌آوری نتیجه، پاسخ نهایی.
پذیرش: tool call خارج از allowlist رد شود؛ نتیجه اجرا از ادعای مدل متمایز باشد؛ حلقه اجرا دارای سقف باشد.

### F7 — Skill Registry و Runtime
خروجی: manifest استاندارد، ورودی/خروجی schema، permission declaration، نسخه، dependency graph، lifecycle.
پذیرش: Skill غیرمعتبر یا دارای وابستگی حل‌نشده اجرا نشود؛ ثبت اجرای هر Skill قابل ممیزی باشد.

### F8 — Memory و Personalization
خروجی: پروفایل، ترجیحات، episodic memory، provenance، retention و ابزار مدیریت حافظه.
پذیرش: کاربر بتواند حافظه را ببیند، اصلاح کند و پاک کند؛ اطلاعات حساس بدون نیاز و رضایت ذخیره نشود.

### F9 — Skill Factory
ورودی: درخواست طبیعی مالک.
مراحل: clarification → SkillSpec → بررسی سیاست → ساخت artifact → schema validation → unit/contract/adversarial tests → sandbox evaluation → گزارش → تأیید مالک → انتشار نسخه.
پذیرش: تولید به‌تنهایی به معنی فعال‌شدن نیست؛ شکست تست جلوی انتشار را می‌گیرد؛ نسخه قبلی قابل rollback باشد.

### F10 — Agent Factory
مراحل: تعریف هدف → تفکیک وظایف → انتخاب Skillهای موجود → پیشنهاد Skill جدید در صورت نیاز → AgentSpec → بررسی dependency/capability → simulation → evaluation → human approval → staged publish.
پذیرش: Agent فقط به ابزارهای allowlist و مجوزهای تأییدشده دسترسی دارد؛ recursion/depth، تعداد فراخوانی، زمان و هزینه محدود است؛ Agent نمی‌تواند خودش مجوز صادر کند یا سیاست را تغییر دهد.

### F11 — Versioning و Release Management
خروجی: draft/review/approved/staged/published/paused/retired، immutable versions، changelog، rollback، canary.
پذیرش: هر اجرا به نسخه مشخص pin شود؛ rollback قابل آزمایش باشد؛ نسخه منتشرشده بی‌ردپا تغییر نکند.

### F12 — iOS quick access
خروجی: App Intents، Siri phrases، Shortcuts، widgets، push/local notifications، deep links.
پذیرش: هر اقدام محدودیت iOS را رعایت کند؛ عدم دسترسی/permission به fallback واضح منجر شود. اجرای دائمی پس‌زمینه یا شنود دائمی وعده داده نشود.

### F13 — Calendar, communications, research
خروجی: اتصال OAuth حداقل‌دسترسی، Calendar read/write با تأیید، ساخت پیش‌نویس پیام/ایمیل، تحقیق وب با ارجاع.
پذیرش: ارسال بیرونی بدون تأیید جداگانه ممنوع؛ OAuth tokenها امن نگهداری شوند؛ داده منبع نامطمئن تلقی شود.

### F14 — Proactive assistant
خروجی: daily briefing، rule-based triggers، scheduler، notification preferences، quiet hours.
پذیرش: opt-in، کنترل فرکانس، عدم اعلان مزاحم، لغو/توقف آسان، deduplication.

### F15 — Security, privacy, reliability
خروجی: threat model به‌روز، prompt-injection defense، SSRF protection، rate limits، tenant isolation، audit redaction، backups.
پذیرش: تست‌های منفی، abuse tests، restore drill و بررسی مجوزها موفق باشد.

### F16 — Beta و انتشار
خروجی: TestFlight plan، monitoring، crash reporting با حداقل داده، support guide، release checklist.
پذیرش: build قابل نصب، CI سبز، migration/rollback مستند، محدودیت‌های شناخته‌شده شفاف.

### F17 — بهبود مستمر
خروجی: eval suite، benchmark کیفیت، feedback loop با رضایت کاربر، cost dashboard، backlog.
پذیرش: تغییر مدل/پرامپت/Skill با regression tests ارزیابی شود؛ تغییرات خودکار مستقیم به production نروند.

## Agentهای پیشنهادی
1. Executive Orchestrator — هماهنگی و plan؛ مجوز مستقلی ندارد.
2. Task & Reminder Agent — مدیریت کارها.
3. Calendar Agent — بررسی و پیشنهاد زمان؛ نوشتن با مجوز.
4. Briefing Agent — خلاصه روزانه از منابع مجاز.
5. Research Agent — تحقیق با citation و تفکیک واقعیت/استنباط.
6. Communication Drafting Agent — پیش‌نویس پیام و ایمیل؛ ارسال فقط پس از تأیید.
7. Memory Curator — پیشنهاد ذخیره/حذف ترجیحات؛ کنترل کاربر.
8. Agent Builder — ساخت AgentSpec و طرح اجرا.
9. Skill Builder — ساخت manifest/artifact/test suite.
10. Evaluator & Red Team Agent — تست کیفیت، سوءاستفاده و prompt injection.
11. Release Manager — نسخه‌بندی و rollback؛ انتشار نیازمند approval.
12. Integration Agent — مدیریت اتصال‌ها؛ هر permission جدید نیازمند رضایت.

## Skillهای پایه
- task.create/read/update/complete
- reminder.create/cancel
- calendar.read/propose_slot/create_event
- note.create/search
- memory.propose/read/update/delete
- web.research/summarize/cite
- communication.draft
- approval.request/status
- agent.spec.create/validate/test/publish/pause/rollback
- skill.spec.create/validate/test/publish/rollback
- audit.read (محدود به مالک/ادمین)
هر Skill باید input/output schema، permissions، timeout، retry policy، idempotency، failure modes، tests و version داشته باشد.

## امنیت و کنترل
- Deny-by-default برای ابزارها و دسترسی‌ها.
- داده‌های وب، ایمیل، اسناد و خروجی ابزارها «داده نامطمئن» هستند، نه دستور سیستم.
- هر Agent/Skill فقط مجوزهای تأییدشده خود را دارد؛ escalation ممنوع.
- برای ارسال پیام، حذف، خرید، انتشار و دسترسی تازه تأیید صریح لازم است.
- sandbox با محدودیت CPU/RAM/time/network/filesystem؛ کد تولیدشده در process اصلی اجرا نشود.
- سقف عمق delegation، تعداد tool call، هزینه، مدت اجرا و تعداد تلاش.
- ثبت audit بدون ذخیره غیرضروری محتوای خصوصی.
- secretها فقط در secret manager/backend؛ هرگز در app binary یا repo.
- توقف اضطراری برای Agent و Skill و قطع اتصال‌ها.

## تعریف Done برای هر فاز
1. کد واقعی در repository.
2. تست‌های unit/integration/contract مربوطه.
3. lint/type-check/build سبز.
4. مستندات اجرا و تنظیمات به‌روز.
5. امنیت و permission review انجام شده.
6. محدودیت‌های محیط و موارد mock شفاف گزارش شده.
7. هیچ ادعای «پیاده‌سازی شد» بدون شواهد build/test/log معتبر پذیرفته نیست.
