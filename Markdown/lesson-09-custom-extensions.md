# الدرس التاسع: الإضافات المخصصة والتطبيقات العملية

  الدرس التاسع: الإضافات المخصصة والتطبيقات العملية  

## أين نخزن الإضافات المخصصة؟

يوفر ECC طريقتين لتخزين المكونات المخصصة:

*   **عالمياً (Globally):** في مجلد `~/.claude/`. أي إضافة هنا ستعمل في جميع مشاريعك.
*   **محلياً (Locally):** في مجلد `.claude/` داخل مسار مشروعك الحالي. يُنصح بهذا لتخصيص بيئة فريق معين ومشاركتها عبر Git.

## 1\. إنشاء مهارة (Skill) مخصصة للفريق

تخيل أن فريقك لديه طريقة معينة لكتابة رسائل الالتزام (Commit Messages). بدلاً من الشرح لـ Claude في كل مرة، سنصنع مهارة:

```
# إنشاء المجلد
mkdir -p .claude/skills/team-commit/

# كتابة ملف SKILL.md
echo "---
name: team-commit
description: Enforces the team's strict conventional commit format.
---
# Team Commit Rules
1. Format: \`[TICKET-ID] <type>: <description>\`
2. Types: feat, fix, hotfix, chore, docs.
3. No commit message can exceed 50 characters in the first line.
" > .claude/skills/team-commit/SKILL.md
```

**طريقة الاستخدام:** `team-commit skill "راجع تعديلاتي واكتب رسالة commit"`

## 2\. إنشاء وكيل فرعي (Sub-Agent) مخصص

إذا كنت تستخدم أداة قواعد بيانات داخلية ليست مشهورة، فلن يعرفها `database-reviewer` العادي. يمكنك إنشاء `my-db-agent`:

```
mkdir -p .claude/agents/
# قم بإنشاء ملف my-db-agent.md واكتب بداخله:
---
name: my-db-agent
description: متخصص في مراجعة استعلامات قاعدة بيانات الشركة الداخلية (InternalDB).
---
# دورك
أنت خبير في محرك قاعدة البيانات InternalDB الخاص بنا. 
- يجب أن ترفض أي استعلام يستخدم \`SELECT *\`
- يجب التأكد من استخدام الـ Index المعرف في ملف \`/docs/db-indexes.md\`
```

**طريقة الاستخدام:** `/my-db-agent "فحص ملف الاستعلامات الجديد."`

## 3\. إنشاء خطافات مخصصة عبر /hookify

لكتابة خطاف معقد يمنع Claude من تعديل أي ملف في مجلد `/legacy/`، يمكنك التحدث باللغة الطبيعية بدلاً من كتابة JSON:

```
/hookify "قم بإنشاء PreToolUse hook. إذا حاولت أداة 'Edit' أو 'Replace' تعديل أي ملف يبدأ مساره بـ '/legacy/'، قم برفض الطلب وأرجع رسالة تنبيه بأن هذا المجلد مجمد ولا يجب المساس به."
```

سيقوم وكيل \`hookify\` بترجمة هذا الوصف البشري إلى هيكل JSON صحيح وحفظه في مسار `.claude/hooks/hooks.json`.

## مشاركة إعدادات مشروعك مع زملائك

بما أن كل هذه الإضافات تم حفظها محلياً في `.claude/`، كل ما عليك فعله هو إضافتها لـ Git:

```
git add .claude/
git commit -m "chore: add team specific claude agents and skills"
```

بمجرد أن يقوم زميلك بسحب التحديث (git pull) وفتح Claude Code في المشروع، ستظهر لديه جميع المهارات والوكلاء المخصصة للفريق فوراً!

[الدرس السابق](lesson-08-hooks-deepdive.html) [الدرس التالي: أنظمة التصميم](lesson-10-design-systems.html)