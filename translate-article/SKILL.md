---
name: translate-article
description: Use whenever translating or localizing an article, blog post, or technical essay, including requests to translate a published article or draft into Spanish or another language.
---

# Translate an article

Translate the author's article, not just its sentences. Preserve its meaning, voice, qualifications, technical accuracy, and useful details. Default to Latin American Spanish unless the user specifies another language or locale.

## Review the original first

1. Read the article, its metadata, relevant project instructions, and a recent writing sample or existing translation. Identify the intended audience, voice, links, images, code, and publication state.
2. Check for errors, outdated claims, broken examples, or writing that could be improved. Verify technical claims against primary sources; distinguish historical context from a current claim.
3. If English-source changes are warranted, explain the proposed changes and ask for approval before editing the English article, unless the user has already approved them. Make approved changes before translating affected passages. Do not silently correct the source or introduce disputed corrections only in the translation. If approval is pending, continue independent work and wait before dependent translation work.

## Translate

- Use natural, conversational language that sounds like the author. Translate literally when it carries the meaning; translate freely when a literal rendering would obscure the intent. Preserve emphasis, uncertainty, examples, and the scope of claims. Do not invent facts, personal experiences, or humor.
- Write titles and headings as natural, specific Spanish, not word-for-word translations. Prefer everyday phrasing that fits the section; for example, “Piensa antes de subir tu PR” instead of “Piensa antes de enviar código”. Avoid awkward calques such as “más fácil en lo mecánico”.
- When the site numbers the table of contents automatically, omit redundant numbers from translated headings. Preserve any substantive numbering and existing public anchors.
- Localize jokes, idioms, and cultural references where needed. Prefer a natural equivalent; remove the joke when it does not work in the target language, while preserving the point.
- Keep code, identifiers, commands, output, and behavior intact; translate human-language comments in code. Check that comment changes do not affect runnable examples or expected output.
- Translate titles, excerpts, captions, alt text, and other reader-facing metadata. Retain technical terms that are clearer in the original language. Preserve links, attribution, and existing public URLs and anchors.

## Images and links

- Inspect the images. If an article has none, or an image is unattractive or low resolution, generate suitable replacement or new imagery in the site's established style. Use the available image-generation workflow. Keep important information from technical diagrams and respect attribution; don't present generated imagery as a real event or screenshot. If image generation is unavailable, report that gap.
- Add oEmbed cards for key references where the site supports them, following its existing embed conventions. Keep ordinary links as links. Check that cards render useful titles and descriptions; if a card shows only the domain or fails, retain a descriptive text link instead.

## Publishable output

- Follow the site's language routing and translation metadata. For this blog, store Spanish articles under `src/content/articles/es/`, set `lang: es` and `translationOf` to the English public slug, and preserve existing slugs and publication flags. Paired listed articles share the English path with `?lang=es` for Spanish. Do not publish, commit, or push unless authorized.
- Review the translation against the final source, including code comments, image descriptions, links, and embedded cards. Verify the site's required checks, build output, language switcher, metadata, and localized exports as relevant; never claim live deployment is verified without checking it. Report checks performed and any remaining gaps.
