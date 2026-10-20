# EngX — إصلاح مسار بناء Android

## سبب الخطأ

```text
Directory '<repository root>' does not contain a Gradle build.
```

السجل يشغّل قالب Klencod العام من جذر المستودع، بينما مشروع Gradle الحقيقي داخل `android/`. جذر المستودع هو جذر npm ويحتوي `package.json`، وليس `settings.gradle`. استدعاء `./android/gradlew` من الجذر وحده لا يغيّر مجلد المشروع؛ يجب الدخول إلى `android/` أو استخدام `-p android`.

السجل استخدم Java 17 وSDK 34 وGradle النظام 9.8.1؛ أما هذه الحزمة فتستخدم **Node 22 وJDK 21 وSDK platform 35 وbuild-tools 35.0.0 وGradle Wrapper 8.11.1**. لا تشغّل `gradle init` ولا تنقل `android/` وحده؛ يعتمد على `../node_modules`.

## المسار المقترح من الهاتف

1. فك ZIP وارفع **المحتويات كاملة** إلى جذر مستودعك، بما فيها المجلد المخفي `.github`، وليس ZIP نفسه.
2. تأكد من استبدال `.github/workflows/android-apk.yml` بالنسخة الجديدة في الفرع الافتراضي.
3. اختر **Actions → Build EngX Android APK → Run workflow**. لا تختَر workflow عامًا يولّده Klencod؛ تعديل الحزمة لا يغيّر ملفه تلقائيًا. لا تحذف workflows أخرى غير مرتبطة بهذا التطبيق.
4. بعد نجاح جميع الخطوات فقط، نزّل **EngX-debug-APK** من Artifacts وفكّه.

الملف المرفق يثبت npm ويشغّل الفحص والاختبارات وبناء الويب ومزامنة Capacitor من جذر npm، ثم يشغّل Gradle في خطوة مستقلة ذات `working-directory` يشير إلى `android/`. يدعم المصدر في جذر المستودع أو داخل `EngX/`؛ يجب بقاء `.github/workflows/` في جذر المستودع.

## البناء بالأوامر

بعد تثبيت المتطلبات أعلاه، من المجلد الذي يحتوي `package.json`:

```bash
npm ci
npm run android:doctor
npm test
npm run android:sync
cd android
sh ./gradlew --no-daemon :app:assembleDebug
```

على Windows استخدم `gradlew.bat --no-daemon :app:assembleDebug` بدل أمر `sh` الأخير. يتوفر أيضًا الاختصار `npm run android:apk` من جذر npm؛ يزامن Capacitor ثم يشغّل Gradle من `android/` عبر السكربت المرفق.

الناتج المتوقع بعد النجاح، بالنسبة إلى جذر npm:

```text
android/app/build/outputs/apk/debug/app-debug.apk
```

## لو تستخدم Klencod

اختَر **Android (Java)** كتجربة استيراد مشروع موجود، وحدِّد `android/` كمجلد Gradle إن أتاح ذلك. المشروع React/TypeScript + Capacitor، ونقطة دخوله `MainActivity.java`، مع إضافات Kotlin يجب الاحتفاظ بها. ليس Flutter ولا React Native ولا Java خالصًا. اختيار النوع وحده لا يضمن توافق المستورد؛ إن لم يسمح بتجهيز npm واختيار مجلد Gradle، استخدم workflow المرفق أو Android Studio. لا تسمح لقالب جديد باستبدال ملفات المشروع.

## ما اختُبر فعلًا في هذه النسخة

- نجح `npm ci` في نسخة اختبار منفصلة.
- نجح `npm test`: **92 اختبارًا في 20 ملفًا**، بعد تحديث اختبار workflow ليتحقق من فصل المزامنة عن Gradle ومسار `android/`.
- نجح `npm run android:sync`، بما يشمله من فحص TypeScript وبناء Vite ومزامنة إضافات Capacitor وأصول Android.
- نجح Gradle Wrapper الحقيقي 8.11.1 بالأمر `sh ./gradlew --no-daemon projects` من `android/`، وتعرّف على `:app` ومشاريع Capacitor.
- نجح فحص YAML ووجود ملفات البناء ومسار APK، واختبار توجيه خطوات ملف workflow للمصدر في الجذر وداخل `EngX/`. استُخدمت بدائل أوامر لفحص مجلدات التشغيل؛ هذا ليس تشغيلًا على GitHub ولا تجميع APK.
- جرت مقارنة الحزمة بالمصدر الأصلي: التغييرات محصورة في workflow والوثائق واختبار البناء، دون حذف مصادر التطبيق أو تعديلها.

**لم ينجح تجميع APK كامل هنا:** محاولة `:app:assembleDebug` الحقيقية تجاوزت خطأ المجلد ثم توقفت لأن Java المتاحة في بيئة الاختبار لا تحتوي مترجم `JAVA_COMPILER`. كما لا يتوفر Android SDK كامل في هذه البيئة. ملف GitHub المرفق يطلب JDK 21 وSDK 35، لكن **لم يُشغّل workflow على GitHub أو داخل مسار Klencod المرفوع نفسه، ولم يُختبر التطبيق على هاتف**. لذلك لا نضمن خلو المراحل غير المختبرة من أخطاء أخرى، ولا تتضمن الحزمة APK جاهزًا. إصدار debug للاختبار وليس إصدار متجر موقّعًا بمفتاحك.

راجع [دليل اختيار النوع والاستيراد](KLENCOD-IMPORT.ar.md) و[دليل البناء التفصيلي](docs/KLENCOD-APK.ar.md).
