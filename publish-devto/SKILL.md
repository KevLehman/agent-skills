---
name: publish-devto
description: Publish or update an article on DEV Community, or prepare a DEV draft for review.
---

# Publish to DEV

## Requirement

A DEV Community API key is required. Create one in [DEV settings](https://dev.to/settings/extensions), then provide it as the `DEVTO_API_KEY` environment variable or in the project's ignored `.env` file. Never print, commit, or send the key to another host. If no key is available, explain how to set it and stop before authenticated operations.

## Workflow

1. Read the article and applicable project instructions. Identify whether the user wants a draft or a published article, and respect existing publication flags. A request to prepare a draft does not authorize publication.
2. If syndicating an article published elsewhere, determine its original canonical URL and verify it is publicly accessible at the correct route before publishing. Set DEV's `canonical_url` to that URL. For a DEV-original article, omit the canonical URL unless the user specifies one.
3. Check the authenticated account and existing drafts and published articles using paginated `GET /api/articles/me/all`. Match an existing DEV URL or canonical URL first; resolve ambiguous matches before writing. Update the intended article instead of creating a duplicate. Never overwrite unrelated content.
4. Prepare the title, description, up to four appropriate DEV tags, optional series and cover image, and Markdown body. Remove source YAML frontmatter from `body_markdown`: Forem frontmatter overrides API fields. Convert relative links and images to publicly reachable absolute URLs, preserving code, embeds, and meaning. Do not invent a cover image.
5. Use the [Forem API](https://developers.forem.com/api/v1): `POST /api/articles` to create or `PUT /api/articles/{id}` to update. Set `published` to the requested state. Use the v1 Accept header and avoid sending the API key to redirects. An explicit request to publish the article authorizes publication. On an uncertain write result, check the account for the article before retrying.
6. Verify the returned article's ID, URL, publication state, and canonical URL if applicable; retrieve it again if needed. If the project tracks DEV links, record the exact URL following its conventions and run the applicable checks. Report the DEV URL and state, canonical URL when used, and any remaining verification or local changes.
