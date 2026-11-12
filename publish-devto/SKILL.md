---
name: publish-devto
description: Publish or update a syndicated DEV Community article from the author's blog, or prepare a DEV draft for review.
---

# Publish to DEV

Use the blog as the source of truth and DEV as a syndication channel. Read the current project instructions and article before acting. Creating this skill does not authorize publication; follow the user's request for each use.

1. Identify the requested article and its local publication state. For `/Users/kaleman/abcdev`, follow README.md and the article routing conventions. Preserve public slugs and publication flags. Do not syndicate a draft or unlisted article publicly unless explicitly requested; do not change those flags implicitly.
2. For public syndication, verify the original is deployed and publicly accessible on `https://kevinaleman.com` at its actual canonical route. Confirm the final response is HTML at that route, not a redirect to another page. If unavailable, prepare a DEV draft or report the blocker; do not publish with an unverified canonical URL.
3. Check the authenticated account and existing DEV drafts and published articles using paginated `GET /api/articles/me/all`. Match an existing `devtoUrl` or canonical URL first; resolve ambiguous matches before writing. Update the intended article instead of creating a duplicate. Never overwrite unrelated content.
4. Prepare the title, excerpt, up to four appropriate DEV tags, series title if applicable, and Markdown body. Remove the site's YAML frontmatter from `body_markdown`: Forem frontmatter overrides API fields. Convert relative links and images to verified public absolute URLs, preserving code, embeds, and meaning. Use a publicly reachable absolute cover image if available; do not invent one. Respect the requested draft or published state; a request to prepare a draft does not authorize publication.
5. Use the [Forem API](https://developers.forem.com/api/v1): `POST /api/articles` to create or `PUT /api/articles/{id}` to update. Set `canonical_url` to the original URL and `published` to the requested state. Use `DEVTO_API_KEY` from the environment or the project's ignored `.env`; never print, commit, or send it to another host. Use the v1 Accept header and avoid sending the API key to redirects. An explicit request to publish that article authorizes publication. On an uncertain write result, check the account for the article before retrying.
6. Verify the returned article's ID, URL, publication state, and canonical URL; retrieve it again if needed. Record its exact `devtoUrl` in the local article when appropriate, without changing other metadata. Follow project checks for any local edit. Report the DEV URL and state, canonical URL, and any verification or local changes still needed. Do not claim the blog deployment or a push succeeded without checking it.

The existing `npm run devto:sync` command only reconciles canonical URLs for existing posts; it does not create or publish articles. Use its dry-run and apply modes only when canonical synchronization is requested and its documented conditions are met.
