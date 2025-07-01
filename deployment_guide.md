# دليل نشر VSCode Coding Tracker Server على Render

## المقدمة

يهدف هذا الدليل إلى توفير إرشادات خطوة بخطوة لنشر مكون الخادم الخاص بـ VSCode Coding Tracker (`vscode-coding-tracker-server`) على منصة Render السحابية. سيغطي الدليل إعداد بيئة التطوير المحلية، وتكوين دعم التوكنات المتعددة (للمسؤول والموظفين) باستخدام متغيرات البيئة، وتحضير المشروع للنشر، وأخيراً خطوات النشر والاختبار على Render.

VSCode Coding Tracker هو إضافة لـ Visual Studio Code تتيح للمطورين تتبع أنشطتهم البرمجية وإنشاء تقارير مفصلة حول الوقت المستغرق في المشاريع والملفات المختلفة. يعتمد هذا النظام على مكونين رئيسيين: إضافة VSCode التي تعمل كعميل لجمع البيانات، ومكون خادم يستقبل هذه البيانات ويخزنها ويقدم واجهة لإنشاء التقارير وعرضها. هذا الدليل سيركز على نشر مكون الخادم.

## 1. إعداد بيئة التطوير المحلية

قبل النشر على Render، من الضروري إعداد المشروع محلياً لضمان عمله بشكل صحيح وتكوين التوكنات المطلوبة.

### 1.1 استنساخ المستودع

أولاً، قم باستنساخ مستودع `vscode-coding-tracker-server` من GitHub إلى جهازك المحلي. يمكنك القيام بذلك باستخدام الأمر التالي في الطرفية:

```bash
git clone https://github.com/hangxingliu/vscode-coding-tracker-server.git
cd vscode-coding-tracker-server
```

### 1.2 تثبيت Node.js و npm

يتطلب `vscode-coding-tracker-server` بيئة Node.js لتشغيله. يوصى باستخدام Node Version Manager (NVM) لإدارة إصدارات Node.js، حيث قد تتطلب بعض التبعيات إصدارات محددة. في هذا الدليل، وجدنا أن الإصدار 14.x من Node.js يعمل بشكل جيد مع المشروع.

لتثبيت NVM (إذا لم يكن مثبتاً لديك):

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
```

بعد التثبيت، قم بتفعيل NVM في جلستك الحالية (قد تحتاج إلى إعادة تشغيل الطرفية أو تشغيل الأوامر التالية):

```bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"
```

الآن، قم بتثبيت Node.js الإصدار 14 واستخدمه:

```bash
nvm install 14
nvm use 14
```

### 1.3 تثبيت التبعيات

بعد استنساخ المستودع وتثبيت Node.js، انتقل إلى دليل المشروع وقم بتثبيت جميع التبعيات باستخدام npm:

```bash
npm install
```

إذا واجهت أي مشاكل تتعلق بـ `node-sass` أو `gyp`، فقد تحتاج إلى تثبيت أدوات البناء الأساسية (build-essential) وإعادة محاولة تثبيت التبعيات. على أنظمة Debian/Ubuntu، يمكنك القيام بذلك باستخدام:

```bash
sudo apt-get update
sudo apt-get install -y build-essential
npm install
```

## 2. تكوين دعم التوكنات المتعددة باستخدام متغيرات البيئة

يدعم `vscode-coding-tracker-server` الآن تحميل التوكنات من متغيرات البيئة، مما يوفر طريقة أكثر أماناً ومرونة لإدارة التوكنات في بيئات النشر السحابية مثل Render. سنقوم بتعريف توكن المسؤول وتوكنات الموظفين كمتغيرات بيئة.

### 2.1 تعريف متغيرات البيئة للتوكنات

بدلاً من استخدام ملف `token.json`، سنقوم بتعريف التوكنات مباشرة كمتغيرات بيئة. هذه المتغيرات هي:

*   `ADMIN_TOKENS`: توكنات المسؤول، مفصولة بفاصلة (,) إذا كان هناك أكثر من توكن.
*   `UPLOAD_TOKENS`: توكنات تحميل البيانات للموظفين، مفصولة بفاصلة (,) إذا كان هناك أكثر من توكن.
*   `VIEW_REPORT_TOKENS`: توكنات عرض التقارير، مفصولة بفاصلة (,) إذا كان هناك أكثر من توكن.

**مثال:**

لتعريف توكن مسؤول واحد وثلاثة توكنات للموظفين، ستقوم بتعيين متغيرات البيئة كالتالي:

*   `ADMIN_TOKENS=your-admin-token-here`
*   `UPLOAD_TOKENS=employee-token-1,employee-token-2,employee-token-3`
*   `VIEW_REPORT_TOKENS=` (يمكن تركها فارغة إذا لم تكن هناك حاجة لتوكنات عرض تقارير منفصلة)

**ملاحظة هامة:** قم بتغيير `your-admin-token-here` و `employee-token-1` و `employee-token-2` و `employee-token-3` إلى توكنات قوية وفريدة من اختيارك. هذه التوكنات ستستخدم للمصادقة عند الوصول إلى الخادم.

### 2.2 اختبار التكوين محلياً (اختياري)

لاختبار هذا التكوين محلياً، يمكنك تشغيل الخادم مع تعريف متغيرات البيئة في الطرفية:

```bash
ADMIN_TOKENS="your-admin-token-here" UPLOAD_TOKENS="employee-token-1,employee-token-2,employee-token-3" npm start
```

يجب أن ترى في مخرجات الطرفية أن الخادم قد بدأ، وأن التوكنات قد تم تحميلها بنجاح من متغيرات البيئة.

إذا واجهت مشكلة `EADDRINUSE` (المنفذ قيد الاستخدام)، فهذا يعني أن هناك عملية أخرى تستخدم المنفذ 10345. يمكنك إيقافها باستخدام:

```bash
fuser -k 10345/tcp || true
```

ثم أعد تشغيل الخادم.

## 3. تحضير المشروع للنشر على Render

Render هي منصة سحابية تسمح بنشر التطبيقات بسهولة. لكي يتمكن Render من بناء ونشر تطبيقك، تحتاج إلى إنشاء ملف `render.yaml` في جذر مشروعك.

### 3.1 إنشاء ملف `render.yaml`

في الدليل الرئيسي لمشروع `vscode-coding-tracker-server/`، أنشئ ملفاً جديداً باسم `render.yaml` بالمحتوى التالي:

```yaml
services:
  - type: web
    name: vscode-coding-tracker-server
    env: node
    plan: free
    buildCommand: npm install
    startCommand: node app.js
    ports:
      - 10345
    envVars:
      - key: NODE_VERSION
        value: 14.x
      - key: ADMIN_TOKENS
        value: "your-admin-token-here"
      - key: UPLOAD_TOKENS
        value: "employee-token-1,employee-token-2,employee-token-3"
      - key: VIEW_REPORT_TOKENS
        value: ""
```

**شرح ملف `render.yaml`:**

*   `type: web`: يحدد أن هذا الخدمة هي تطبيق ويب.
*   `name: vscode-coding-tracker-server`: اسم الخدمة على Render.
*   `env: node`: يحدد أن البيئة هي Node.js.
*   `plan: free`: يحدد خطة الاستضافة (يمكنك تغييرها إذا كنت بحاجة إلى موارد أكبر).
*   `buildCommand: npm install`: الأمر الذي سيتم تنفيذه لتثبيت تبعيات المشروع أثناء عملية البناء على Render.
*   `startCommand: node app.js`: الأمر الذي سيتم تنفيذه لبدء تشغيل الخادم بعد النشر. **لاحظ أننا لم نعد نمرر مسار ملف `token.json` هنا.**
*   `ports: - 10345`: يحدد المنفذ الذي يستمع إليه الخادم (المنفذ الافتراضي لـ `vscode-coding-tracker-server`).
*   `envVars`: هذا القسم هو الأهم. هنا نقوم بتعريف متغيرات البيئة التي سيستخدمها الخادم لتحميل التوكنات.
    *   `key: NODE_VERSION`, `value: 14.x`: يحدد متغير بيئة لضمان استخدام Render لإصدار Node.js 14.x، وهو الإصدار الذي اختبرنا عليه المشروع محلياً.
    *   `key: ADMIN_TOKENS`, `value: 


"your-admin-token-here"`: توكن المسؤول.
    *   `key: UPLOAD_TOKENS`, `value: "employee-token-1,employee-token-2,employee-token-3"`: توكنات الموظفين.
    *   `key: VIEW_REPORT_TOKENS`, `value: ""`: توكنات عرض التقارير (يمكن تركها فارغة).

## 4. النشر على Render

بعد إعداد المشروع محلياً وتعديل ملف `render.yaml`، يمكنك الآن نشر تطبيقك على Render.

### 4.1 ربط مستودع GitHub الخاص بك بـ Render

1.  قم برفع مشروعك (`vscode-coding-tracker-server` بما في ذلك ملف `render.yaml` والملفات المعدلة في `lib/TokenMiddleware.js`) إلى مستودع GitHub خاص بك.
2.  انتقل إلى لوحة تحكم Render (https://dashboard.render.com/).
3.  انقر على `New Web Service`.
4.  اختر مستودع GitHub الذي يحتوي على مشروعك. قد تحتاج إلى منح Render الإذن للوصول إلى مستودعاتك.
5.  بعد اختيار المستودع، سيقوم Render تلقائياً باكتشاف ملف `render.yaml` الخاص بك وتعبئة معظم الإعدادات.

### 4.2 تكوين الخدمة على Render

تأكد من أن الإعدادات التي تم اكتشافها من `render.yaml` صحيحة. يجب أن تكون كالتالي:

*   **Name**: `vscode-coding-tracker-server` (أو أي اسم تختاره)
*   **Root Directory**: (اتركه فارغاً إذا كان `render.yaml` في جذر المستودع، وإلا حدد المسار الصحيح)
*   **Runtime**: Node.js
*   **Build Command**: `npm install`
*   **Start Command**: `node app.js`
*   **Port**: `10345`

**متغيرات البيئة (Environment Variables):**

تأكد من أن متغيرات البيئة التالية مضافة في قسم `Environment` في إعدادات خدمة Render:

*   **Key**: `NODE_VERSION`
*   **Value**: `14.x`

*   **Key**: `ADMIN_TOKENS`
*   **Value**: `your-admin-token-here` (استبدلها بالتوكن الفعلي)

*   **Key**: `UPLOAD_TOKENS`
*   **Value**: `employee-token-1,employee-token-2,employee-token-3` (استبدلها بالتوكنات الفعلية)

*   **Key**: `VIEW_REPORT_TOKENS`
*   **Value**: (اتركها فارغة أو أضف توكنات عرض التقارير مفصولة بفاصلة)

### 4.3 بدء النشر

بعد مراجعة جميع الإعدادات، انقر على `Create Web Service`. سيبدأ Render عملية البناء والنشر. يمكنك مراقبة السجلات في لوحة تحكم Render لمتابعة التقدم وأي أخطاء قد تحدث.

### 4.4 الوصول إلى الخادم المنشور

بمجرد اكتمال النشر بنجاح، سيوفر لك Render عنوان URL عاماً لخدمتك. سيكون هذا العنوان هو نقطة النهاية (endpoint) لخادم `vscode-coding-tracker-server` الخاص بك.

مثال على عنوان URL: `https://your-service-name.onrender.com`

## 5. استخدام التوكنات بعد النشر

بعد نشر الخادم، يمكنك استخدام التوكنات التي قمت بتعريفها كمتغيرات بيئة.

### 5.1 توكن المسؤول (Admin Token)

توكن المسؤول (`your-admin-token-here`) يمنحك صلاحيات كاملة (تحميل، عرض تقارير، تعديل الخادم). يمكنك استخدامه للوصول إلى واجهة الإدارة أو لأي عمليات تتطلب صلاحيات كاملة.

مثال على الوصول إلى التقرير باستخدام توكن المسؤول:

`https://your-service-name.onrender.com/report?token=your-admin-token-here`

### 5.2 توكنات الموظفين (Upload Tokens)

توكنات الموظفين (`employee-token-1`, `employee-token-2`, `employee-token-3`) تستخدم لتحميل بيانات التتبع من إضافة VSCode Coding Tracker. كل موظف سيستخدم التوكن الخاص به في إعدادات الإضافة في VSCode.

في إضافة VSCode Coding Tracker، ستحتاج إلى تكوين نقطة نهاية الخادم (Server Endpoint) والتوكن (Token) الخاص بالموظف. على سبيل المثال:

*   **Server Endpoint**: `https://your-service-name.onrender.com`
*   **Token**: `employee-token-1` (أو `employee-token-2`، `employee-token-3`)

### 5.3 توكنات عرض التقارير (View Report Tokens)

إذا قمت بتعريف `VIEW_REPORT_TOKENS`، فيمكنك استخدام هذه التوكنات للسماح للمستخدمين بعرض التقارير فقط دون صلاحيات تحميل أو إدارة.

مثال على الوصول إلى التقرير باستخدام توكن عرض التقارير:

`https://your-service-name.onrender.com/report?token=your-view-report-token`

## 6. استكشاف الأخطاء وإصلاحها

*   **مشاكل في النشر**: تحقق من سجلات البناء والنشر في لوحة تحكم Render. الأخطاء الشائعة تشمل مشاكل في تثبيت التبعيات (تأكد من صحة `buildCommand`) أو مشاكل في بدء تشغيل الخادم (تأكد من صحة `startCommand` والمنفذ).
*   **الخادم لا يستجيب**: تأكد من أن الخادم يستمع على المنفذ الصحيح (10345) وأن هذا المنفذ مكشوف في `render.yaml`. تحقق أيضاً من أن `startCommand` لا يحتوي على `--local`، حيث أن هذا الخيار يقيد الوصول إلى `127.0.0.1` فقط.
*   **مشاكل في التوكنات**: تأكد من أن متغيرات البيئة `ADMIN_TOKENS`, `UPLOAD_TOKENS`, و `VIEW_REPORT_TOKENS` تم تعريفها بشكل صحيح في Render وأن التوكنات المستخدمة في الإضافة تتطابق تماماً مع تلك الموجودة في متغيرات البيئة.

## الخاتمة

باتباع هذا الدليل، يجب أن تكون قادراً على نشر VSCode Coding Tracker Server بنجاح على Render، مع إعداد نظام توكنات متعدد يلبي احتياجاتك للمسؤول والموظفين باستخدام متغيرات البيئة. تذكر دائماً الحفاظ على توكناتك آمنة وعدم مشاركتها علناً.

---

