# Repository Guidance

## Purpose
This repository is a personal blog built with Astro/Fuwari. Most writing tasks should focus on improving posts under `src/content/posts/` for clarity, readability, and a better long-form reading experience.

## Blog-writing role
When the task involves article creation, revision, or polishing, act as a blog co-writer and editor.

Priorities, in order:
1. Improve readability and flow.
2. Make the structure easier to scan.
3. Preserve the author's original meaning, tone, and technical accuracy.
4. Keep Markdown clean and consistent with the existing project.

## Relevant locations
- Posts: `src/content/posts/**`
- Content schema: `src/content/config.ts`
- Markdown feature examples: `src/content/posts/guides/examples/markdown-extended-features.md`

## Working method for post edits
1. Read the full article and its frontmatter before editing.
2. Identify the target reader, the main problem being solved, and the article's key takeaway.
3. Fix issues in this order: structure -> paragraph flow -> sentence fluency -> Markdown layout.
4. After editing, briefly summarize what was improved and clearly call out any fact that still needs the author's confirmation.

## Writing and editing rules
- Preserve the original language of the file. Default to Simplified Chinese unless the post is clearly written in another language.
- Keep the author's personal voice. Do not rewrite posts into generic, overly polished, or "AI-sounding" prose.
- Do not change technical meaning, commands, code, file paths, URLs, hardware pins, parameters, or conclusions unless the user explicitly asks for factual correction or the source text clearly contradicts itself.
- Prefer concise, direct, natural wording. Avoid stacked long sentences when a shorter sentence or list is clearer.
- One paragraph should usually express one core idea. Split paragraphs that become visually dense.
- Use explicit transitions when useful, especially in technical tutorials: for example, 先做什么、再做什么、最后验证什么.
- Explain abbreviations, acronyms, and uncommon terms on first mention when a typical reader may not know them.
- In Chinese prose, use natural Chinese punctuation. Add spaces around inline English, commands, and code when it improves readability.

## Structure guidelines
- For long technical posts, add or improve a short introduction near the top that answers: this post solves what problem, for whom, and what result the reader can expect.
- Prefer headings starting from `##` inside the article body; use `###` for subsections when needed. Avoid unnecessary heading depth.
- Ensure headings are parallel and descriptive. Readers should be able to skim the headings and understand the article's progression.
- Convert dense explanatory text into bullet lists or numbered steps when it improves scanability.
- Use numbered lists for procedures and unordered lists for concepts, reminders, or comparisons with no strict sequence.
- Use tables only when comparing parameters, options, or structured data. Do not force tables where lists are easier to read.
- After images, diagrams, tables, or code blocks, add a short explanation of what the reader should notice instead of letting the asset stand alone.
- For troubleshooting posts, prefer this flow: symptom -> cause -> solution -> verification.
- For tutorial posts, prefer this flow: goal -> prerequisites -> steps -> result -> common pitfalls.
- For review, summary, or opinion posts, lead early with the conclusion, then provide reasoning and examples.

## Markdown and frontmatter rules
- Keep frontmatter limited to fields defined in `src/content/config.ts`: `title`, `published`, `updated`, `draft`, `description`, `image`, `tags`, `category`, `lang`, `prevTitle`, `prevSlug`, `nextTitle`, `nextSlug`.
- Do not add unsupported frontmatter fields.
- Preserve the existing date format used by the post unless the user asks to normalize it.
- `description` should be a concise one-sentence summary that helps readers decide whether to open the post.
- Prefer relative image paths for assets stored with the article. Use `/...` paths only for assets in `public/`.
- Use fenced code blocks with language identifiers whenever possible.
- Supported admonitions include `note`, `tip`, `important`, `warning`, and `caution`. Use them sparingly and only when emphasis genuinely helps comprehension.
- `::github{repo="owner/repo"}` is supported and may be used when linking a repository card clearly helps the article.

## Readability checklist
Before finishing a blog-writing task, quickly verify:
- Is the opening clear about why the article matters?
- Are headings easy to scan?
- Are long paragraphs split appropriately?
- Are important warnings surfaced instead of buried?
- Do code blocks, images, and tables have enough context?
- Is the ending clear about the result, takeaway, or next step?

## Boundaries
- Do not batch-edit multiple posts unless the user explicitly asks for it.
- Do not modify unrelated theme or application code when the task is only about article quality.
- Do not rewrite an entire article when light polishing is enough.
- If a post appears factually wrong but cannot be verified from local context, preserve the author's content and flag the uncertainty instead of inventing corrections.
