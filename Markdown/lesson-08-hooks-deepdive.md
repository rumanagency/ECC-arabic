# الدرس الثامن: الفهم العميق للخطافات (Hooks) والتطبيقات العملية

  الدرس الثامن: الفهم العميق للخطافات (Hooks) والتطبيقات العملية  

## حالة استخدام 1: الحماية من تسريب الأسرار (PreToolUse Hook)

من أسوأ الكوارث البرمجية أن يقوم الذكاء الاصطناعي (أو أنت) بكتابة مفتاح API سري داخل الكود ودفعه (Push) إلى GitHub.

يمكننا إنشاء خطاف يمنع Claude من تنفيذ أمر الـ `git push` إذا وجد كلمات مثل "API\_KEY" في الملفات المعدلة. يتم تشغيل هذا الخطاف **قبل** استخدام أداة Bash (PreToolUse).

### كود الـ Hook:

```
{
  "PreToolUse": [
    {
      "matcher": "tool == \"Bash\" && tool_input.command matches \"git push\"",
      "hooks": [
        {
          "type": "command",
          "command": "#!/bin/bash\nif git diff HEAD --name-only | xargs grep -iE 'api[_-]?key|secret'; then echo '[Hook] WARNING: Potential secrets detected! Aborting push.' >&2; exit 1; fi"
        }
      ]
    }
  ]
}
```

**ماذا يحدث عملياً؟** إذا طلب Claude تنفيذ `git push`، سيقوم النظام بتشغيل أمر الـ grep أولاً. إذا وجد أي سر مكشوف، سيتوقف الأمر، وسيظهر لـ Claude رسالة خطأ تخبره بضرورة إزالة المفتاح السري قبل المحاولة مجدداً.

## حالة استخدام 2: التنسيق التلقائي (PostToolUse Hook)

لنفترض أن مشروعك يستخدم لغة Python وتود التأكد من أن الكود منسق دائماً باستخدام أداة `black`. لا تريد أن تطلب من Claude تنسيق الكود في كل مرة يكتب فيها دالة جديدة.

سنستخدم خطافاً يعمل **بعد** قيام Claude بتعديل أي ملف Python (PostToolUse).

### كود الـ Hook:

```
{
  "PostToolUse": [
    {
      "matcher": "tool == \"Edit\" && tool_input.file_path matches \"\\\\.py$\"",
      "hooks": [
        {
          "type": "command",
          "command": "black \"$file_path\" && echo '[Hook] Formatted file with Black' >&2"
        }
      ]
    }
  ]
}
```

**النتيجة العملية:** يقوم الذكاء الاصطناعي بكتابة دالة فوضوية، يغلق الملف، وفوراً يعمل الخطاف في الخلفية لتنسيق الملف. سيقرأ Claude الرد ليعرف أن الملف قد تم تنسيقه لصالحه.

## حالة استخدام 3: تذكير التشغيل قبل الاختبار (PreToolUse Hook)

في العديد من مشاريع الـ Backend، لا يمكن تشغيل الاختبارات (Tests) إلا إذا كانت خدمة Docker (مثلاً قاعدة بيانات Redis) تعمل في الخلفية. غالباً ما ينسى المطورون ذلك.

### كود الـ Hook:

```
{
  "PreToolUse": [
    {
      "matcher": "tool == \"Bash\" && tool_input.command matches \"(npm test|pytest)\"",
      "hooks": [
        {
          "type": "command",
          "command": "if ! docker ps | grep -q 'redis'; then echo '[Hook] ⚠️ REMINDER: Redis container is NOT running. Tests may fail. Run docker-compose up -d redis first.' >&2; fi"
        }
      ]
    }
  ]
}
```

**النتيجة العملية:** قبل أن ينفذ Claude أمر `npm test`، سيتحقق الخطاف من وجود حاوية Redis. إذا لم يجدها، سيرسل تحذيراً فورياً لـ Claude لكي يشغل الحاوية أولاً، مما يوفر وقتاً كبيراً في تتبع الأخطاء الغبية.

## أين نضع هذه الخطافات؟

يتم تخزين هذه الإعدادات في `hooks/hooks.json`. استخدم الإضافة الرسمية `/hookify` لإنشائها بسهولة عبر الدردشة دون الحاجة لكتابة JSON معقد يدوياً.

[الدرس السابق](lesson-07-skills-mastery.html) [الدرس التالي: الإضافات المخصصة](lesson-09-custom-extensions.html)