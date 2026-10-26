Add the following persistent documentation rule to the project's root AGENTS.md.
Create the file if missing. Preserve all existing instructions and avoid duplicating
an equivalent rule.

## Persian Markdown writing rule

When creating or editing Persian Markdown documentation:

- Every sentence, paragraph, heading, and list item containing Persian prose MUST
  begin with a Persian word.
- Apply this to each sentence individually, including sentences within the same
  paragraph or list item.
- Ignore Markdown syntax when checking the first word: heading markers, bullet
  markers, numbering, blockquote markers, and emphasis delimiters do not count.
- Never start mixed Persian-English prose with an English word, inline code,
  a command, a file path, a URL, or a technical identifier. Introduce it naturally
  with an appropriate Persian word, such as «برای»، «دستور»، «فایل»،
  «تابع»، «سرویس»، or «پارامتر».
- Preserve technical names, identifiers, paths, URLs, and code exactly.
- Exempt fenced code blocks, indented code blocks, standalone commands, and
  entirely English sections or text.
- Before finishing, review the Markdown you created or modified and rewrite
  any mixed-language sentence that violates this rule.
- Use this rule for all future Persian Markdown documentation in this project.

Examples:

Incorrect: `Redis` برای کش استفاده می‌شود.
Correct: سرویس `Redis` برای کش استفاده می‌شود.

Incorrect: - `DB_HOST` آدرس دیتابیس است.
Correct: - پارامتر `DB_HOST` آدرس دیتابیس است.

Incorrect: ## API برداشت
Correct: ## رابط API برداشت