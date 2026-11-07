# Ideal Backend Blueprint

**A mandatory architectural blueprint for building secure, reliable, and evolvable backends.**

[English](#english) · [العربية](#العربية)

---

<a name="english"></a>
## English

### What is this?

**Ideal Backend Blueprint** is an architecture-focused skill for AI coding agents. It provides a consistent blueprint for building or reviewing backend systems: APIs, servers, databases, authentication, and background work.

Its central principle is simple:

> **The foundations are established from day one; the implementation starts as simple as the project allows and evolves when evidence justifies it.**

The blueprint is intended for projects of every size. Project size changes the implementation of a component, not whether its architectural responsibility is considered.

### Why use it?

Backend decisions made early can become expensive to change after users, data, and business rules accumulate. This blueprint makes the important boundaries and responsibilities explicit from the beginning, while avoiding infrastructure and complexity that the project does not yet need.

It aims to guide decisions across seven architectural pillars:

1. **Security** — protect data and access.
2. **Reliability** — remain operational through failures and recover safely.
3. **Data integrity and concurrency** — preserve correctness when requests run simultaneously.
4. **Performance and efficiency** — use resources effectively and keep responses fast.
5. **Scalability** — grow capacity without redesigning the system.
6. **Maintainability** — make changes and testing safer.
7. **Operability and observability** — deploy, configure, and understand the running system.

Every architectural decision should serve at least one of these pillars. Components that serve none should not be added.

### The architecture

The blueprint defines **12 foundational components** that must be addressed in every backend:

| # | Component | Responsibility |
|---|---|---|
| 1 | Request guard | Protect incoming requests through transport security, rate limits, input validation, authentication, and authorization. |
| 2 | API layer | Define stable, versioned endpoints and consistent response and error contracts. Keep business logic out of controllers. |
| 3 | Business logic | Own domain rules, workflows, state transitions, and transaction boundaries. |
| 4 | Data access | Centralize database access, use parameterized queries, and manage connection and query limits. |
| 5 | Database | Enforce data integrity through constraints, indexes, migrations, least-privilege access, and tested backups. |
| 6 | Concurrency and duplicate prevention | Protect sensitive operations from race conditions and duplicate execution, including through idempotency and atomic updates. |
| 7 | Config and secrets | Load and validate environment-specific configuration securely; keep secrets out of source code, version control, and logs. |
| 8 | Error handling | Handle errors centrally, return safe public responses, and retain diagnostic details in logs. |
| 9 | Observability | Provide structured logs, request IDs, health checks, and route-level performance and saturation metrics. |
| 10 | Process lifecycle | Validate startup dependencies, shut down gracefully, and keep application processes stateless. |
| 11 | Automated tests | Test business logic, critical database paths, authorization boundaries, and concurrency behavior. |
| 12 | Build and deploy | Use reproducible builds, pinned dependencies, CI checks, vulnerability scanning, and rollback capability. |

The blueprint also defines **3 conditional components**. Their architectural boundaries are considered from the beginning, but their full infrastructure is introduced only when the relevant need appears:

| Component | Add the full implementation when… |
|---|---|
| Cache | Repeated reads remain slow after query and indexing improvements, or multiple application instances require a shared cache. |
| Background jobs | Work takes more than about two seconds, depends on a slow external service, or must run on a schedule. |
| External integrations | The backend communicates with providers such as payment, email, SMS, or storage services. |

### Design principles

- **Foundations first:** address all 12 foundational responsibilities, even in small projects.
- **Simple implementations:** use the simplest implementation that satisfies the relevant architectural requirement.
- **Stable boundaries:** define replaceable boundaries for the rate-limit store, cache, background-job dispatch, file storage, and external providers.
- **No premature infrastructure:** do not introduce microservices, Kubernetes, queues, or Redis without a measured reason.
- **Protect irreversible decisions:** establish sound database constraints and migrations, API contracts and versioning, authentication and object-level authorization, ID formats, stateless processes, idempotency, and structured request logging early.
- **Measure before upgrading:** use observability signals to determine when an implementation needs to evolve.
- **Resolve conflicts deliberately:** when a blueprint rule conflicts with a project requirement or the right choice is uncertain, choose the simplest solution that still protects the relevant architectural pillar and explain the decision and reason to the user.

### Build workflow

Before writing backend code, the agent should provide a summary of no more than ten lines covering:

- The main entities.
- The key API endpoints.
- Whether the conditional components are needed, and why.
- Any conflicts between the project requirements and the blueprint.

Then it can proceed with implementation, following the project's language, framework, folder structure, and coding conventions. This blueprint governs **architecture**; it does not prescribe those implementation choices.

### Upgrade ladder

Infrastructure should evolve in response to measured signals, following this order:

1. Improve indexes and tune queries.
2. Eliminate N+1 queries.
3. Introduce a cache when measurements justify it.
4. Adjust connection pooling when connection limits become a problem.
5. Move long-running work to background jobs.
6. Add application instances when CPU saturation justifies horizontal scaling.
7. Scale the database, then consider read replicas.
8. Treat sharding as a last resort and require user approval because it is an irreversible architectural decision.

### Pre-delivery review

Before delivery, verify at least the following:

- Protected routes enforce authentication and object-level ownership checks.
- Inputs are validated and database queries are parameterized.
- Rate limiting is enabled, especially for authentication endpoints.
- Lists are paginated with a maximum page size.
- Database constraints, indexes, and migrations are present.
- Sensitive operations are protected against duplicate execution and concurrency issues, with tests covering the behavior.
- Secrets and sensitive values are not exposed in source code, version control, or logs.
- Unexpected errors do not expose internal details.
- Structured logs, request IDs, and liveness/readiness checks are available.
- Graceful shutdown works and important state is not stored only in process memory.
- Critical tests pass.
- A Dockerfile and CI pipeline are present.
- Conditional-component decisions and any deviations from the blueprint are explained.

### Scope and limitations

This is an **architectural blueprint**, not a guarantee that every generated backend will be flawless, secure, or infinitely scalable. Results still depend on the quality of implementation, project-specific requirements, testing, deployment configuration, and operational practices.

It does not mandate a programming language, framework, folder layout, or code style. Those should follow the project's own instructions and use common, stable tools.

### Full specification

See [`SKILL.md`](SKILL.md) for the complete rules, implementation requirements, upgrade triggers, and pre-delivery checklist.

---

<a name="العربية"></a>
## العربية

### ما هذه السكل؟

**Ideal Backend Blueprint** هي سكل تركّز على الهندسة المعمارية، ومخصّصة لوكلاء البرمجة بالذكاء الاصطناعي. توفّر مخططًا موحّدًا لبناء الباك إند ومراجعته، بما يشمل واجهات API والخوادم وقواعد البيانات والمصادقة والمهام الخلفية.

تقوم فكرتها الأساسية على مبدأ:

> **تُؤسَّس القواعد المعمارية منذ اليوم الأول، ويبدأ التنفيذ بأبسط صورة مناسبة للمشروع، ثم يتطور عندما تظهر حاجة فعلية تبرر ذلك.**

السكل مخصّصة للمشاريع بمختلف أحجامها. حجم المشروع يغيّر طريقة تنفيذ المكوّن، لا ضرورة التفكير في مسؤوليته المعمارية.

### لماذا تستخدمها؟

قد تصبح القرارات المعمارية المبكرة مكلفة عند تغييرها بعد تراكم المستخدمين والبيانات وقواعد العمل. لذلك تجعل السكل الحدود والمسؤوليات الأساسية واضحة منذ البداية، مع تجنّب فرض البنية التحتية والتعقيد الذي لا يحتاجه المشروع بعد.

توجّه السكل القرارات المعمارية وفق سبعة محاور:

1. **الأمان:** حماية البيانات والتحكم بالوصول.
2. **الموثوقية:** استمرار النظام في العمل والتعافي الآمن عند حدوث الأعطال.
3. **سلامة البيانات والتزامن:** الحفاظ على صحة البيانات عند تنفيذ الطلبات بالتوازي.
4. **الأداء والكفاءة:** الاستفادة الفعالة من الموارد وتقديم استجابات سريعة.
5. **قابلية التوسع:** زيادة القدرة الاستيعابية دون الحاجة إلى إعادة تصميم النظام.
6. **قابلية الصيانة:** تسهيل التعديل والاختبار بأمان.
7. **التشغيل وقابلية المراقبة:** نشر النظام وتهيئته وفهم ما يحدث أثناء تشغيله.

يجب أن يخدم كل قرار معماري محورًا واحدًا على الأقل؛ ولا ينبغي إضافة مكوّن لا يخدم أيًّا منها.

### البنية المعمارية

تحدّد السكل **12 مكوّنًا أساسيًا** يجب التعامل معها في كل مشروع باك إند:

| # | المكوّن | المسؤولية |
|---|---|---|
| 1 | حارس الطلبات (Request Guard) | حماية الطلبات الواردة عبر أمان الاتصال، وتحديد المعدل، والتحقق من المدخلات، والمصادقة، والتفويض. |
| 2 | طبقة API | تحديد مسارات واضحة وثابتة ذات إصدارات، وتوحيد شكل الاستجابات والأخطاء، وإبقاء منطق العمل خارج Controllers. |
| 3 | منطق العمل (Business Logic) | امتلاك قواعد النظام وسير العمليات وانتقال الحالات وحدود المعاملات. |
| 4 | الوصول إلى البيانات (Data Access) | تنظيم الوصول إلى قاعدة البيانات، واستخدام الاستعلامات المعلّمة بالمعاملات، وضبط الاتصالات والاستعلامات. |
| 5 | قاعدة البيانات | فرض سلامة البيانات عبر القيود والفهارس والترحيلات، والصلاحيات الأقل، والنسخ الاحتياطية المختبرة. |
| 6 | التزامن ومنع التكرار | حماية العمليات الحساسة من تعارض الطلبات وتكرار التنفيذ، باستخدام مفاتيح Idempotency والتحديثات الذرّية وغيرها. |
| 7 | الإعدادات والأسرار | تحميل إعدادات كل بيئة والتحقق منها بأمان، ومنع وضع الأسرار في الشيفرة أو Git أو السجلات. |
| 8 | معالجة الأخطاء | معالجة مركزية للأخطاء، واستجابات عامة آمنة، مع حفظ التفاصيل التشخيصية في السجلات. |
| 9 | قابلية المراقبة (Observability) | توفير سجلات منظّمة ومعرّفات للطلبات وفحوصات صحة النظام ومقاييس الأداء واستهلاك الموارد. |
| 10 | دورة حياة العملية | التحقق من متطلبات التشغيل عند البدء، والإغلاق التدريجي، وجعل عمليات التطبيق عديمة الحالة. |
| 11 | الاختبارات الآلية | اختبار منطق العمل والمسارات الأساسية مع قاعدة البيانات وحدود الصلاحيات وسلوك التزامن. |
| 12 | البناء والنشر | توفير بناء قابل للتكرار، واعتماديات محدّدة الإصدارات، وفحوصات CI، وفحص الثغرات، وإمكانية التراجع عن الإصدار. |

كما تحدّد السكل **3 مكوّنات مشروطة**. تُراعى حدودها المعمارية منذ البداية، لكن لا تُضاف بنيتها التحتية الكاملة إلا عند ظهور الحاجة المناسبة:

| المكوّن | متى تُضاف بنيته التنفيذية؟ |
|---|---|
| التخزين المؤقت (Cache) | عندما تبقى القراءات المتكررة بطيئة بعد تحسين الاستعلامات والفهارس، أو عندما تتطلب عدة نسخ من التطبيق مخزنًا مشتركًا. |
| المهام الخلفية (Background Jobs) | عندما تتجاوز العملية نحو ثانيتين، أو تعتمد على خدمة خارجية بطيئة، أو يجب تنفيذها وفق جدول زمني. |
| التكاملات الخارجية (External Integrations) | عندما يتواصل الباك إند مع مزوّد خارجي مثل خدمات الدفع أو البريد الإلكتروني أو الرسائل أو التخزين. |

### المبادئ المعمارية

- **تأسيس القواعد أولًا:** معالجة المسؤوليات الأساسية الاثنتي عشرة حتى في المشاريع الصغيرة.
- **بساطة التنفيذ:** اختيار أبسط تنفيذ يحقق المتطلب المعماري المطلوب.
- **حدود ثابتة:** تعريف واجهات قابلة للاستبدال لمخزن تحديد المعدل، والتخزين المؤقت، وتوزيع المهام الخلفية، وتخزين الملفات، والمزوّدات الخارجية.
- **منع التعقيد المبكر:** عدم إدخال Microservices أو Kubernetes أو Queues أو Redis دون سبب تدعمه مؤشرات فعلية.
- **حماية القرارات صعبة التغيير:** تأسيس قيود قاعدة البيانات وترحيلاتها، وعقود API وإصداراتها، والمصادقة وصلاحيات الوصول إلى كل كائن، ومعرّفات غير قابلة للتخمين، والعمليات عديمة الحالة، ومنع التكرار، والسجلات المنظّمة منذ البداية.
- **الترقية بناءً على القياس:** استخدام مؤشرات المراقبة لتحديد الوقت المناسب لتطوير التنفيذ.
- **معالجة التعارضات بوضوح:** عند تعارض إحدى القواعد مع متطلبات المشروع أو عدم وضوح القرار، يُختار أبسط حل يحافظ على المحور المعماري المعني، مع توضيح القرار وسببه للمستخدم.

### سير العمل

قبل كتابة كود الباك إند، يجب على الوكيل تقديم ملخص لا يتجاوز عشرة أسطر، يتضمن:

- الكيانات الرئيسية.
- مسارات API الأساسية.
- ما إذا كانت المكوّنات المشروطة مطلوبة، ولماذا.
- أي تعارض بين متطلبات المشروع ومبادئ السكل.

بعد ذلك يبدأ التنفيذ، مع الالتزام بلغة المشروع وإطاره البرمجي وهيكل مجلداته وأسلوب كتابة الكود المعتمد فيه. السكل تحكم **الهندسة المعمارية**، ولا تفرض لغة أو إطار عمل أو هيكل مجلدات أو أسلوب كتابة محددًا.

### سلّم التطوير

يجب تطوير البنية التحتية استجابةً لمؤشرات فعلية، وبالترتيب التالي:

1. تحسين الفهارس وضبط الاستعلامات.
2. إزالة استعلامات N+1.
3. إضافة التخزين المؤقت عندما تبرره القياسات.
4. تعديل Connection Pooling عند ظهور مشكلات في حدود الاتصالات.
5. نقل العمليات طويلة التنفيذ إلى مهام خلفية.
6. إضافة نسخ أخرى من التطبيق عندما يبرر تشبّع المعالج التوسع الأفقي.
7. ترقية قاعدة البيانات، ثم التفكير في Read Replicas.
8. اعتبار Sharding الحل الأخير، مع اشتراط موافقة المستخدم لأنه قرار معماري يصعب التراجع عنه.

### مراجعة ما قبل التسليم

قبل التسليم، يجب التحقق على الأقل من الآتي:

- جميع المسارات المحمية تفرض المصادقة وتتحقق من ملكية المستخدم للكائن المطلوب.
- التحقق من جميع المدخلات واستخدام الاستعلامات المعلّمة بالمعاملات.
- تفعيل تحديد معدل الطلبات، خصوصًا لمسارات المصادقة.
- تقسيم القوائم إلى صفحات مع حد أقصى لحجم الصفحة.
- وجود قيود قاعدة البيانات وفهارسها وترحيلاتها.
- حماية العمليات الحساسة من التكرار ومشكلات التزامن، مع وجود اختبارات تثبت ذلك.
- عدم كشف الأسرار أو البيانات الحساسة في الشيفرة أو Git أو السجلات.
- عدم كشف الأخطاء غير المتوقعة للتفاصيل الداخلية.
- توفر السجلات المنظّمة ومعرّفات الطلبات وفحوصات Liveness وReadiness.
- عمل الإغلاق التدريجي وعدم تخزين حالة مهمة داخل ذاكرة العملية فقط.
- نجاح اختبارات المسارات الأساسية.
- وجود Dockerfile وخط CI.
- توضيح قرارات المكوّنات المشروطة وأي اختلاف عن قواعد السكل.

### النطاق والحدود

هذه السكل **مخطط للهندسة المعمارية**، وليست ضمانًا بأن كل باك إند ناتج سيكون خاليًا من العيوب أو آمنًا بالكامل أو قابلًا للتوسع بلا حدود. تظل النتيجة مرتبطة بجودة التنفيذ ومتطلبات المشروع والاختبارات وإعدادات النشر وممارسات التشغيل.

كما أنها لا تفرض لغة برمجة أو إطار عمل أو هيكل مجلدات أو أسلوب كتابة كود؛ بل يُترك ذلك لتعليمات المشروع، مع تفضيل الأدوات الشائعة والمستقرة.

### المواصفات الكاملة

راجع ملف [`SKILL.md`](SKILL.md) للاطلاع على القواعد التفصيلية ومتطلبات التنفيذ ومحفزات الترقية وقائمة التحقق قبل التسليم.

---

## License

Add the license you intend to publish this project under before making the repository public.
