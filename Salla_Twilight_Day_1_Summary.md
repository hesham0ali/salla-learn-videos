# Salla Twilight — Day 1 Summary
## Architecture, Repository Orientation, Layouts, Pages, Home Trace, and `twilight.json`

> **هدف اليوم الأول:**  
> بناء Mental Model صحيح لمنظومة Salla Twilight قبل الدخول في كتابة Features حقيقية.
>
> المطلوب بعد اليوم ده إنك لما تشوف Requirement تبدأ تسأل:
>
> - هل ده Store data ولا Theme configuration؟
> - هل التغيير Page-specific ولا Shared؟
> - هل مكانه Layout ولا Page ولا Component؟
> - هل ده Rendering responsibility في Twig؟
> - هل محتاج JavaScript؟
> - هل فيه Merchant configuration وبالتالي `twilight.json` له دور؟
> - هل دي حاجة تخص Theme runtime أصلًا، ولا مجرد Development tooling زي Salla CLI؟

---

# 1. الصورة الكبيرة — Mental Model

الموديل التعليمي الأساسي:

```text
Storefront Request
        ↓
Salla / Twilight
        ↓
Store Data
+
Merchant Configuration
+
Active Theme Files
        ↓
Twig Rendering
        ↓
HTML + Salla Web Components
        ↓
Browser
        ↓
CSS + JavaScript
        ↓
Rendered Storefront
```

وبشكل منفصل:

```text
Developer
   ↓
Theme Repository
   ↓
Salla CLI
   ↓
Preview / Watching / Local Development Workflow
```

## أهم فصل لازم يثبت

```text
Store ≠ Theme
Theme ≠ Twilight
Twilight ≠ Salla CLI
Salla CLI ≠ Storefront Runtime
```

> **ملاحظة دقة:**  
> الرسم الأول Mental Model تعليمي لفهم المسؤوليات والـlayers، وليس وصفًا حرفيًا لكل internal request pipeline داخل Salla.

---

# 2. Salla Store

## ما هو الـStore؟

الـStore هو المتجر نفسه على Salla Platform، بما يحتويه من بيانات وحالة.

Conceptually قد يتضمن:

```text
Products
Categories
Store Information
Customers
Cart
Languages
Store State
Other Platform Data
```

مثال:

```text
Store: Coffee House

Product:
Ethiopian Coffee

Price:
80 SAR
```

هذه البيانات ليست مكتوبة Hardcoded داخل `product.twig`.

نفس الـTheme يمكن أن يعمل على متاجر مختلفة ببيانات مختلفة.

## Mental Model

```text
Store
=
"What does this store actually contain?"
```

مقابل:

```text
Theme
=
"How should this storefront be presented and behave?"
```

---

# 3. Store Data vs Merchant Configuration

دي من أهم الفروق في Twilight.

## Store Data

داتا المتجر الفعلية.

مثال Conceptual:

```text
Product Name
Price
Images
Availability
Store Information
```

## Merchant Configuration

اختيارات التاجر المتعلقة بالـTheme أو Component.

مثال:

```text
Show breadcrumbs?
Primary color
Section title
Banner image
Show/Hide a particular theme option
```

## الفرق

```text
Store Data
"What does the store contain?"

Merchant Configuration
"How did the merchant configure the theme?"
```

### مثال

المتجر عنده:

```text
Product:
Ethiopian Coffee

Price:
80 SAR
```

وفي Theme configuration التاجر اختار:

```text
Show Rating = true
```

الاتنين ممكن يدخلوا في عملية الـrender، لكن مصدرهم ومسؤوليتهم مختلفين.

---

# 4. Twilight Engine

## ما هو Twilight؟

Twilight هو **Theme Engine الخاص بـSalla**.

مش الـTheme نفسه.

ومش Salla CLI.

Conceptually Twilight هو الـlayer اللي يشغّل Theme architecture ويستخدم:

```text
Store Data
+
Merchant Configuration
+
Theme Templates
```

ثم يحصل Rendering للـTwig templates.

## أهم جملة

> **Twilight runs the Theme.**

## ليه Twilight مهم؟

لأن الـBrowser لا يفهم Twig syntax مثل:

```twig
{% extends "layouts.master" %}
```

أو:

```twig
{% if ... %}
```

أو:

```twig
{{ ... }}
```

قبل ما المحتوى يوصل للـBrowser، الـTwig templates يجب أن يتم Render لها إلى output يفهمه المتصفح.

---

# 5. Theme Repository

الـTheme Repository هو الكود الذي يعمل عليه الـTheme Developer.

شكل مبسط:

```text
my-theme/
│
├── twilight.json
│
└── src/
    ├── views/
    │   ├── layouts/
    │   ├── pages/
    │   └── components/
    │
    ├── assets/
    │   ├── js/
    │   └── styles/
    │
    └── locales/
```

الـTheme هنا مسؤول عن presentation layer وstorefront behavior الخاص بالثيم.

---

# 6. Directory Structure — المسؤوليات

فكر في `src/` كده:

```text
src/
│
├── views/       → What gets rendered?
│
├── assets/      → How does it look and behave in the browser?
│
└── locales/     → How are translatable strings handled?
```

ثم:

```text
views/
├── layouts/      → Shared page skeletons
├── pages/        → Page-specific templates
└── components/   → Reusable / organized theme UI pieces
```

---

# 7. `src/views/layouts/`

## المسؤولية

```text
layouts
→ Shared page skeleton
```

الـLayout يحتوي الـstructure المشترك بين Pages متعددة.

مثال أساسي:

```text
src/views/layouts/master.twig
```

Conceptually:

```text
┌─────────────────────────────┐
│ <head>                      │
├─────────────────────────────┤
│ Header                      │
├─────────────────────────────┤
│                             │
│      PAGE CONTENT           │
│                             │
├─────────────────────────────┤
│ Footer                      │
├─────────────────────────────┤
│ Global Scripts              │
└─────────────────────────────┘
```

## ليه موجود؟

بدون Layout قد نكرر:

```text
Header
Footer
Global assets
Shared HTML shell
```

داخل كل Page.

الـLayout يقلل الـduplication ويعطي shared architecture واضحة.

---

# 8. `src/views/pages/`

## المسؤولية

```text
pages
→ Templates for specific storefront pages
```

أمثلة:

```text
Home
Product Single
Product Listing
Cart
Blog
Customer Pages
...
```

مثال معروف:

```text
src/views/pages/index.twig
```

وهو Home Page template.

## نقطة مهمة

Twilight لديه predefined Pages بأسماء ومسارات معروفة.

لذلك لا تتعامل مع أسماء ومسارات predefined page templates كأنها أسماء اعتباطية تستطيع تغييرها بدون تأثير.

Mental Model:

```text
Twilight:
"I expect a predefined page template at a known location."

Theme:
"I provide/customize that template."
```

---

# 9. `src/views/components/`

## المسؤولية

```text
components
→ Reusable / organized Theme UI pieces
```

أمثلة Conceptual:

```text
Header
Footer
Product Card
Home Section
Reusable UI Fragment
```

الهدف هو عدم تحويل Layouts وPages إلى ملفات ضخمة مليئة بتكرار الـmarkup.

---

# 10. Theme Component ≠ Salla Web Component

دي نقطة لازم تفضل واضحة.

## Theme / Twig Component

مرتبط بملفات الثيم مثل:

```text
src/views/components/...
```

ويشارك في template rendering.

## Salla Web Component

Browser Custom Element توفره Salla/Twilight كجزء من storefront component ecosystem.

Mental Model:

```text
Theme/Twig Component
=
Template-side reusable UI

Salla Web Component
=
Browser-side Custom Element provided by Salla
```

لا تخلط الاثنين لمجرد أن كلمة Component موجودة في الاسم.

---

# 11. `src/assets/js/`

## المسؤولية

```text
assets/js
→ Browser-side Theme behavior
```

أمثلة:

```text
Event listeners
Menu interactions
Drawers
Modals
DOM interactions
UI state
Page-specific behavior
Custom interactions
```

### Concept

Twig مسؤول عن template/rendering concerns.

JavaScript مسؤول عن client-side behavior عندما الصفحة وصلت للـBrowser.

---

# 12. `src/assets/styles/`

## المسؤولية

```text
assets/styles
→ Theme styling and visual presentation
```

أمثلة:

```text
Typography
Spacing
Colors
Layout styling
Responsive behavior
Buttons
Product styles
RTL/LTR styling concerns
Animations
```

لو Requirement أساسه Visual presentation، الـstyles layer غالبًا واحدة من أول layers التي يجب فحصها.

---

# 13. `src/locales/`

## المسؤولية

```text
locales
→ Translation strings / localization
```

مثال:

```text
src/locales/ar.json
src/locales/en.json
```

الفكرة:

بدل Hardcoding النصوص التي يجب ترجمتها، يكون عندك localization architecture.

لكن مهم:

```text
Translation
→ locales

RTL layout/styling
→ markup/styles concern أيضًا
```

يعني `locales` وحدها ليست مسؤولة عن كل ما يخص RTL.

---

# 14. ملخص Directory Responsibilities

```text
layouts
→ Shared skeleton used by multiple theme pages.

pages
→ Templates for specific storefront pages.

components
→ Reusable / organized Theme UI pieces.

assets/js
→ Client-side interactions and Theme behavior.

assets/styles
→ Theme styling and visual presentation.

locales
→ Translation strings and localization.
```

## الفكرة الأهم

الـDirectory Structure لا تعني:

```text
Feature = One File
```

Feature حقيقية قد تشمل أكثر من layer.

مثلاً Mega Menu قد تشمل:

```text
Header Component
+
JavaScript
+
Styles
+
Possibly Theme Settings
+
Possibly Localization
```

الهدف هو أن تعرف **كل Responsibility مكانها الطبيعي فين**.

---

# 15. Layout vs Page

أهم Concept:

```text
Page
 │
 │ extends
 ▼
Layout
```

أو من زاوية تركيب النتيجة:

```text
Layout
│
├── Shared Header
│
├── [Page Content Slot]
│
└── Shared Footer
```

والـPage تملأ الـslot.

---

# 16. `master.twig`

الـdefault Layout الأساسي:

```text
src/views/layouts/master.twig
```

Conceptually:

```twig
<!DOCTYPE html>
<html>
<head>
    ...
</head>

<body>

    Header

    <main>
        {% block content %}
        {% endblock %}
    </main>

    Footer

</body>
</html>
```

الجزء الأهم:

```twig
{% block content %}
{% endblock %}
```

ده Named customization point داخل Layout.

---

# 17. يعني إيه Twig Block؟

الـBlock هو:

> **Named slot / customization point داخل Template Inheritance system.**

مثلاً:

```twig
{% block content %}
{% endblock %}
```

الـPage التي تعمل `extend` للـLayout يمكنها أن تملأ أو override هذا الـblock.

مهم:

```text
Block ≠ Component
```

الـBlock Slot.

الـComponent قطعة UI.

---

# 18. Page Extending a Layout

Home Page مثلًا:

```text
src/views/pages/index.twig
```

قد تعمل:

```twig
{% extends "layouts.master" %}

{% block content %}

    {% component home %}

{% endblock %}
```

المعنى:

1. استخدم `layouts.master` كأساس.
2. عندما تصل إلى `content` block:
3. استخدم الـcontent الذي حددته Home Page.

---

# 19. Visualization — Layout + Page

قبل تركيب الـPage:

```text
master.twig

Header

[ CONTENT BLOCK ]

Footer
```

Home Page تقول:

```text
CONTENT BLOCK
=
Home Content
```

فتصبح النتيجة:

```text
Header

Home Content

Footer
```

Product Page قد تملأ نفس الـblock بمحتوى مختلف:

```text
Header

Product Content

Footer
```

المتغير:

```text
Page Content
```

المشترك:

```text
Layout Shell
```

---

# 20. لماذا Template Inheritance مهم؟

بدونه قد يصبح عندنا:

```text
Home
├── Header
├── Content
└── Footer

Product
├── Header
├── Content
└── Footer

Cart
├── Header
├── Content
└── Footer
```

مع inheritance:

```text
               Master Layout
              /      |      \
           Home   Product   Cart
```

الفوائد:

```text
Less duplication
Clear shared structure
Easier global changes
Better separation of responsibilities
Consistency
```

---

# 21. مش كل Page لازم تستخدم `master.twig`

النقطة دي مهمة.

الـMental Model الصح:

```text
Page
↓ extends
Some Layout
```

مش:

```text
Every Page must extend master.twig
```

ممكن يكون عندك Layout إضافي لمجموعة Pages تحتاج shell مختلف.

مثلاً Conceptually:

```text
layouts/master.twig
layouts/customer.twig
```

وده يسمح بفصل page families ذات structures مختلفة.

---

# 22. Header وFooter في Theme Raed

في `master.twig` يظهر Conceptually:

```twig
{% component 'header.header' %}
```

ثم:

```twig
{% block content %}{% endblock %}
```

ثم:

```twig
{% component 'footer.footer' %}
```

فـArchitecture:

```text
Layout
│
├── Header Component
│
├── Content Block
│       ↑
│       Page
│
└── Footer Component
```

وده يعلمك أن Layout لا يجب أن يحتوي كل UI بالتفصيل؛ يمكنه تنظيم Components.

---

# 23. `app.js` داخل الـLayout

Theme Raed يحمل global/broad JavaScript entry من الـmaster layout.

Mental Model:

```text
master.twig
→ app.js
→ broad/global theme behavior
```

لكن Page معينة قد تضيف JavaScript خاص بها من خلال block مخصص للـscripts.

---

# 24. Home Page Trace

الملف:

```text
src/views/pages/index.twig
```

Conceptually:

```twig
{% extends "layouts.master" %}

{% block content %}

    {% component home %}

{% endblock %}
```

وفي scripts block يتم تحميل Home-specific JavaScript مثل:

```text
home.js
```

---

# 25. Home Trace — خطوة بخطوة

```text
User opens Home
        ↓
Twilight resolves the Home page template
        ↓
src/views/pages/index.twig
        ↓
index.twig extends layouts.master
        ↓
master.twig provides shared shell
        ↓
index.twig fills:
content → Home content
scripts → Home-specific JS
        ↓
Twig Rendering
        ↓
HTML output
        ↓
Browser
```

---

# 26. دور `{% component home %}`

الـHome Page نفسها لا يجب بالضرورة أن تكتب كل Home Sections Hardcoded.

Conceptually:

```twig
{% component home %}
```

تحدد مكان الـHome component/section rendering mechanism.

تخيل التاجر عامل Home:

```text
Hero
Featured Products
Offers
Brands
Testimonials
```

`index.twig` قد تظل بسيطة لأن الـHome components نفسها تُدار كطبقة مستقلة.

Mental Model:

```text
index.twig
=
Home Page Template

{% component home %}
=
Where Home components/content are rendered
```

---

# 27. `app.js` vs `home.js`

مثال مهم على scope:

```text
master.twig
→ app.js
→ broad/global behavior
```

مقابل:

```text
index.twig
→ home.js
→ Home-specific behavior
```

دي طريقة organization مستخدمة في Theme Raed، وليست قاعدة أن كل Theme في العالم يجب أن يستخدم نفس أسماء الملفات.

الفكرة هي Scope.

---

# 28. الصفحة النهائية التي تصل للـBrowser

المتصفح لا يرى Twig tags.

Conceptually:

```text
master.twig
+
index.twig
+
Home components
+
Store data/configuration
        ↓
Twig Rendering
        ↓
HTML
```

ثم الـBrowser يتعامل مع:

```text
HTML
Salla Web Components
CSS
JavaScript
```

وليس:

```twig
{% extends %}
{% block %}
{{ variable }}
```

---

# 29. `twilight.json` — أول Mental Model

لا تحتاج في Day 1 حفظ الـschema.

المطلوب تعرف أنه موجود في:

```text
Theme Root
```

مثال:

```text
my-theme/
├── twilight.json
└── src/
```

وإنه Theme configuration/setup file مهم.

---

# 30. أهم أجزاء `twilight.json`

في اليوم الأول ركز على:

```text
Theme Information
Theme Settings
Theme Features
Theme Components
```

---

# 31. Theme Information

Conceptually يعرّف معلومات تخص الثيم نفسه.

Mental Model:

```text
twilight.json
↓
"What is this Theme?"
```

مش:

```text
"Render this page's HTML"
```

---

# 32. Theme Settings

## المسؤولية

تعريف Theme-level configurable values.

Conceptually:

```text
Developer defines setting
        ↓
Salla understands the setting/configuration UI
        ↓
Merchant selects a value
        ↓
Theme can use the configured value
```

مثال Conceptual:

```text
Show something?
Primary style option?
Theme-level visual/configuration choice?
```

الـimportant distinction:

```text
Setting definition
≠
UI markup
```

---

# 33. Theme Features

في Twilight، `features` مرتبطة بالـpredefined features/components التي يوفرها Twilight ويعلن الثيم عن دعمها/استخدامها وفق الـschema الرسمي.

Mental Model:

```text
Twilight has predefined capability
        ↓
Theme declares it in features
```

`features` ليست المكان الذي تكتب فيه HTML implementation.

هي configuration/declaration layer.

---

# 34. Custom Theme Components في `twilight.json`

هنا أهم فكرة في اليوم.

قد يكون عندك Component Definition داخل:

```json
"components": [...]
```

ويكون عندك Twig implementation داخل:

```text
src/views/components/...
```

الاثنان ليسا نفس الشيء.

---

# 35. Component Definition vs Twig File

تخيل Component اسمها:

```text
Promo Banner
```

Twig implementation:

```text
src/views/components/home/promo-banner.twig
```

مسؤوليتها:

```text
How should the component render?
```

لكن Salla تحتاج تعرف أيضًا:

```text
What is this component?
Where is its Twig implementation?
What configurable fields does it expose?
How can the merchant configure it?
```

هنا يدخل `twilight.json`.

---

# 36. `path` كحلقة ربط

Conceptually:

```text
twilight.json

Component:
Promo Banner

path:
home.promo-banner

fields:
title
image
        │
        │ links to
        ▼
src/views/components/home/promo-banner.twig
```

أهم Mental Model:

```text
twilight.json
=
Configuration / Schema / Declaration

Twig Component
=
Rendering Implementation
```

---

# 37. Merchant-configurable Component

نفترض التاجر عنده Component:

```text
Promo Banner
```

ويدخل:

```text
Title:
Summer Sale

Image:
summer.jpg
```

Conceptually:

```text
twilight.json
defines:
"What can be configured?"
        ↓
Merchant fills configuration
        ↓
Salla/Twilight provides values
        ↓
Twig component renders values
        ↓
HTML
```

---

# 38. Theme Settings vs Component Fields

لا تخلطهم.

## Theme Settings

```text
Theme-level/global configuration
```

## Component Fields

```text
Configuration specific to a component
```

Mental Model:

```text
settings
→ Theme-level configuration

components[].fields
→ Component-specific configuration
```

التفاصيل الدقيقة للـschema تأتي لاحقًا.

---

# 39. Features vs Custom Components

Mental Model اليوم الأول:

```text
features
→ Predefined Twilight theme capabilities/components

components
→ Custom components defined by the Theme Developer
```

مهم تعرف الفرق Conceptually قبل دخول تفاصيل الـschema.

---

# 40. `twilight.json` في صورة واحدة

```text
twilight.json
│
├── Theme Information
│      ↓
│   "What is this theme?"
│
├── settings
│      ↓
│   "What theme-level options can be configured?"
│
├── features
│      ↓
│   "Which predefined Twilight features are declared?"
│
└── components
       ↓
    "Which custom merchant-configurable components exist?"
       │
       ├── path
       │     ↓
       │   Twig implementation
       │
       └── fields
             ↓
          Merchant configuration
```

---

# 41. ربط `twilight.json` بالـHome Page

أنت رأيت:

```twig
{% component home %}
```

والآن الصورة تتوسع:

```text
twilight.json
       ↓
declares supported/custom Home components
and their configuration
       ↓
Merchant configures Home
       ↓
pages/index.twig
       ↓
{% component home %}
       ↓
Configured Home content/components
       ↓
Twig rendering
       ↓
HTML
```

وده سبب مهم لدراسة `index.twig` قبل التعمق في `twilight.json`.

---

# 42. Practical Experiment — الهدف

الـExperiment ليس لاختبار `<div>`.

الهدف:

> تشوف Scope الفرق بين Page-level وLayout-level changes.

---

# 43. Experiment 1 — Home Page Change

داخل:

```text
src/views/pages/index.twig
```

قبل:

```twig
{% component home %}
```

ضع مؤقتًا:

```html
<div>
  DAY 1 — HOME PAGE TEST
</div>
```

Conceptually:

```text
master.twig
    ↓
content block
    ↑
index.twig provides Home-specific content
```

المتوقع:

```text
Home ✅
Other unrelated Pages ❌
```

لأن التغيير داخل Home Page template.

---

# 44. Experiment 2 — Layout Change

بعد إزالة التعديل السابق، ضع نفس الـmarkup داخل:

```text
src/views/layouts/master.twig
```

قبل:

```twig
{% block content %}
```

Conceptually:

```text
master.twig

Header

LAYOUT TEST

[Page Content]

Footer
```

كل Page تعتمد على `layouts.master` ستأخذ shared markup الموجود في الـLayout.

---

# 45. الفرق بين التجربتين

## Page-level

```text
                 master.twig
                     │
       ┌─────────────┼─────────────┐
       ↓             ↓             ↓
     Home         Product         Other
       │
       └── TEST

Scope:
Home only
```

## Layout-level

```text
                 master.twig
                     │
                   TEST
                     │
       ┌─────────────┼─────────────┐
       ↓             ↓             ↓
     Home         Product         Other

Scope:
Pages extending this Layout
```

---

# 46. لماذا لا نقول "كل صفحات المتجر"؟

لأن ليس كل Page بالضرورة تستخدم نفس Layout.

التعبير الأدق:

> **التغيير داخل `master.twig` يؤثر على الصفحات التي تعتمد على هذا الـLayout.**

لو Page تستخدم Layout آخر، لا تفترض أنها ستأخذ نفس التغيير بدون tracing.

---

# 47. Salla CLI في الـExperiment

Salla CLI هنا Development tooling.

Mental Model:

```text
VS Code
   ↓
Edit Theme File
   ↓
Salla CLI Preview
   ↓
Preview / Reload / Development Workflow
   ↓
Browser
```

لكن Production storefront لا يعتمد على أن `salla theme preview` يعمل على جهاز المطور.

---

# 48. أهم درس من الـExperiment

```text
Code location
→ determines responsibility/scope
```

مش كل مكان يقدر تقنيًا يستقبل markup هو المكان المعماري الصحيح.

---

# 49. لا تحول Layout إلى Dumping Ground

لو Feature Global، ده لا يعني أن كل implementation يوضع داخل `master.twig`.

مثلاً:

```text
Global Mega Menu
```

قد يتطلب:

```text
master.twig
→ tells you where Header is included

Header Component
→ owns markup

JavaScript
→ interaction

Styles
→ presentation

Theme Settings
→ if merchant-configurable
```

الـLayout يساعدك تفهم **Scope وComposition**، وليس أن ترمي كل الكود فيه.

---

# 50. Architecture Preflight — عقلية اليوم الأول

قبل أي Feature، اسأل:

```text
1. Where does this feature belong?
2. What data does it require?
3. What is Twig responsible for?
4. What is JavaScript responsible for?
5. Do we need a Salla Web Component?
6. Does it need twilight.json / Merchant Settings?
7. What is the styling responsibility?
8. How broad is the scope?
9. What files depend on the thing I will change?
10. How will I verify/debug it?
```

في Day 1 مش مطلوب تجاوب كل سؤال بدقة كاملة، لكن مطلوب تبدأ تفكر بهذه الطريقة.

---

# 51. Architecture Challenge — Assessment

**لا تقرأ حلول جاهزة قبل ما تحاول.**

لكل Requirement اكتب:

```text
Requirement #X

Primary layer/file:
Why:
Other layers possibly involved:
```

## Requirements

### #1
عايز أغير HTML structure للـHeader في كل المتجر.

### #2
عايز أغير شكل صفحة Product Single فقط.

### #3
عايز أضيف Section جديدة للـHome والـmerchant يقدر يضيفها ويضبط محتواها.

### #4
عايز أغير styling عالمي للـButtons.

### #5
عايز JavaScript interaction خاصة بالـHome فقط.

### #6
عايز أضيف نص ثابت يظهر في كل Pages.

### #7
عايز Merchant Setting اسمها:

```text
show_sale_badge
```

### #8
عايز أعدل شكل Product Card المستخدم في أكثر من مكان.

---

# 52. كيف تفكر في الـChallenge؟

بدل ما تسأل:

> "إيه الملف اللي أحط فيه الكود؟"

اسأل:

```text
Is it shared or page-specific?
Is it markup or behavior or styling?
Is it reusable UI?
Does merchant control it?
Is there already a component that owns this UI?
What is the broadest layer that should know about this concern?
```

---

# 53. Common Mistakes — أخطاء لازم تتجنبها

## خطأ 1

```text
Theme = Store
```

الصحيح:

```text
Store
→ Data / platform state

Theme
→ Presentation / storefront behavior
```

---

## خطأ 2

```text
Twilight = Theme
```

الصحيح:

```text
Twilight
→ Theme Engine

Theme
→ Files consumed/executed within Twilight architecture
```

---

## خطأ 3

```text
Salla CLI = Runtime
```

الصحيح:

```text
CLI
→ Development tooling

Storefront runtime
→ Salla / Twilight + Active Theme
```

---

## خطأ 4

```text
Page = Layout
```

الصحيح:

```text
Layout
→ Shared shell

Page
→ Specific page content/template
```

---

## خطأ 5

```text
Block = Component
```

الصحيح:

```text
Block
→ Template inheritance customization slot

Component
→ Reusable/organized UI unit
```

---

## خطأ 6

```text
Theme Component = Salla Web Component
```

الصحيح:

```text
Theme/Twig Component
→ template-side

Salla Web Component
→ browser Custom Element provided by Salla
```

---

## خطأ 7

```text
twilight.json = UI file
```

الصحيح:

```text
twilight.json
→ Theme configuration/schema/declarations

Twig
→ Rendering implementation
```

---

## خطأ 8

```text
Global requirement
→ Put everything in master.twig
```

الصحيح:

```text
Global scope
→ Inspect shared composition

Then place each responsibility in its owning layer:
Component / JS / CSS / Settings / etc.
```

---

# 54. Day 1 Mental Model — النهائي

```text
Salla Store
│
├── Store Data
│
└── Merchant / Theme Configuration
        ↓
Twilight Engine
        ↓
Active Theme Repository
│
├── twilight.json
│
├── views/
│   ├── layouts/
│   ├── pages/
│   └── components/
│
├── assets/
│   ├── js/
│   └── styles/
│
└── locales/
        ↓
Twig Rendering
        ↓
HTML + Salla Web Components
        ↓
Browser
        ↓
CSS + JavaScript
        ↓
Interactive Storefront
```

Development side:

```text
Developer
   ↓
Theme Repository
   ↓
Salla CLI
   ↓
Preview / Watch / Local Workflow
```

---

# 55. Day 1 — مسؤولية كل Layer في جملة

```text
Store
→ The actual merchant store and its platform data/state.

Twilight
→ The Salla Theme Engine that processes/renders the theme architecture.

Theme Repository
→ The developer-owned theme files that define storefront presentation and behavior.

Salla CLI
→ Development tooling for creating, previewing and working with the theme.

layouts
→ Shared page skeletons.

pages
→ Specific storefront page templates.

components
→ Reusable/organized Theme UI pieces.

assets/js
→ Browser-side interactions and behavior.

assets/styles
→ Styling and visual presentation.

locales
→ Translation strings/localization.

twilight.json
→ Theme setup/configuration/schema/declarations.

Twig
→ Template rendering logic/markup.

Salla Web Components
→ Salla-provided browser Custom Elements used in the storefront.
```

---

# 56. Day 1 Gate — قبل ما تنتقل لليوم التاني

لا تعتبر اليوم مكتمل لمجرد إنك قرأت الـDocs.

لازم تقدر تجاوب من غير فتح مصادر:

## سؤال 1
إيه الفرق بين:

```text
Store
Twilight
Theme
Salla CLI
```

## سؤال 2
ليه Salla CLI مش جزء من Production Storefront Runtime؟

## سؤال 3
إيه الفرق بين:

```text
Layout
Page
Component
```

## سؤال 4
إيه معنى:

```twig
{% extends "layouts.master" %}
```

Conceptually؟

## سؤال 5
إيه وظيفة:

```twig
{% block content %}
```

داخل الـLayout؟

## سؤال 6
ليه تعديل `index.twig` غالبًا يكون Home-specific بينما تعديل `master.twig` قد يؤثر على Pages متعددة؟

## سؤال 7
ليه مينفعش تقول إن تغيير `master.twig` سيظهر بالضرورة في **كل** صفحة؟

## سؤال 8
إيه الفرق بين:

```text
Theme Component
Salla Web Component
```

## سؤال 9
إيه وظيفة `twilight.json` بشكل عام؟

## سؤال 10
إيه الفرق بين:

```text
twilight.json Component Definition
```

و:

```text
Twig Component File
```

## سؤال 11
إيه الفرق بين:

```text
Theme Settings
Component Fields
```

Conceptually؟

## سؤال 12
Trace الـHome Page بطريقتك:

```text
User opens Home
        ↓
?
        ↓
?
        ↓
?
        ↓
Browser
```

---

# 57. Ready-to-Move-On Criteria

أنت جاهز تنتقل من Day 1 لما تقدر:

- تفتح Theme Repository ومتبقاش شايف Folders عشوائية.
- تحدد وظيفة `layouts`, `pages`, `components`, `assets`, `locales`.
- تشرح `Store`, `Twilight`, `Theme`, `Salla CLI` بدون تعريفات محفوظة.
- تشرح ليه Browser لا يفهم Twig مباشرة.
- تشرح Layout inheritance.
- تعمل Trace للـHome Page من `index.twig` إلى `master.twig` إلى output.
- تشرح وظيفة `twilight.json` من غير ما تحتاج تعرف كل field.
- تفرق بين configuration schema وTwig rendering.
- تتوقع Scope التغيير قبل ما تعمل Preview.
- تبدأ تقسّم Requirement إلى Responsibilities بدل ما تسأل "أعدل أنهي ملف وخلاص؟"
- تحاول Architecture Challenge بدون Copy/Paste solution.

---

# 58. Mini Glossary

## Store
المتجر الفعلي وبياناته وحالته على Salla.

## Twilight
Theme Engine الخاص بـSalla.

## Theme
مجموعة الملفات المسؤولة عن storefront presentation/behavior.

## Salla CLI
Command-line development tooling.

## Layout
Shared page skeleton.

## Page
Template خاص بصفحة storefront معينة.

## Block
Named slot في Twig inheritance.

## Theme Component
Reusable/organized template UI piece.

## Salla Web Component
Browser Custom Element توفره Salla.

## Twig
Template language/engine syntax المستخدمة داخل theme rendering.

## Rendering
تحويل Templates + Data/Configuration إلى output markup.

## Merchant Configuration
اختيارات التاجر القابلة للضبط في Theme/Component configuration.

## `twilight.json`
Theme setup/configuration/schema/declaration file.

## `assets/js`
Client-side Theme behavior.

## `assets/styles`
Theme styling/presentation.

## `locales`
Translation/localization strings.

---

# 59. الملفات التي يجب أن تكون قادرًا على فتحها بعد Day 1

مش مطلوب تحفظ المشروع، لكن لما تشوف الملفات دي تعرف **لماذا هي موجودة**:

```text
twilight.json

src/
├── views/
│   ├── layouts/
│   │   └── master.twig
│   │
│   ├── pages/
│   │   └── index.twig
│   │
│   └── components/
│
├── assets/
│   ├── js/
│   └── styles/
│
└── locales/
```

والسؤال لكل ملف/Folder:

```text
Why does this exist?
Who uses it?
What responsibility belongs here?
What depends on it?
How broad is the impact if I change it?
```

---

# 60. Source Reading for Day 1

المصادر الأساسية المستخدمة في اليوم الأول:

- Salla Developers / Twilight Documentation  
  https://docs.salla.dev/

- Theme Raed — Official Salla Theme Reference  
  https://github.com/SallaApp/theme-raed

- Twilight — Directory Structure  
  https://docs.salla.dev/twilight-engine/directory-structure

- Twilight — Layouts  
  راجع Layouts Overview من Salla Developers Docs.

- Twilight — Pages  
  راجع Pages Overview من Salla Developers Docs.

- Twilight — `twilight.json`  
  راجع صفحة `twilight.json` الرسمية في Salla Developers Docs.

- Twig Documentation  
  https://twig.symfony.com/doc/3.x/

> **طريقة استخدام Theme Raed:**  
> Reference implementation لفهم كيف طبقت Salla concepts داخل Theme حقيقي.  
> لا تحفظه ولا تنسخ Patterns منه Blindly.  
> اسأل دائمًا: لماذا هذا الملف موجود؟ من يستخدمه؟ ما الذي يعتمد عليه؟

---

# 61. الخلاصة النهائية

لو خرجت من اليوم الأول بفكرة واحدة، تكون:

> **Salla Theme Development ليس "أين أضع HTML؟".  
> هو فهم Data + Rendering + Configuration + Scope + Ownership.**

كل Requirement ستتحول تدريجيًا من:

```text
"عايز أعمل Feature"
```

إلى:

```text
Where does it belong?
        ↓
What data/config does it need?
        ↓
Which Twig/Page/Layout/Component owns the markup?
        ↓
What belongs to JavaScript?
        ↓
What belongs to CSS?
        ↓
Does the Merchant configure it?
        ↓
Does twilight.json need to declare/configure it?
        ↓
How broad is the change?
        ↓
How will I verify it?
```

وده هو أساس الاستقلال الحقيقي في بناء Salla Twilight Themes.

---

## Day 1 Final Self-Test

قبل Day 2، حاول تشرح بصوتك بدون الرجوع للملخص:

```text
Store
→ Twilight
→ Theme
→ Layout
→ Page
→ Component
→ Twig Rendering
→ Browser
```

ثم جاوب:

> **لو عندي Requirement جديدة، إزاي أحدد مين الـlayer اللي "يمتلك" المسؤولية؟**

لو تقدر تجاوب بالمفاهيم مش بحفظ أسماء الملفات، يبقى Day 1 حقق هدفه.
