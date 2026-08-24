# Weekly IG Post Generator

An n8n workflow that automatically creates a ready-to-post Instagram draft — caption and image — from the latest property/real-estate news, every week.

## How It Works

1. **Trigger** — Runs every Monday at 9am via a schedule trigger.
2. **Read Property News** — Pulls the latest articles from the HousingWire RSS feed (`https://www.housingwire.com/feed/`).
3. **Filter Out Used Articles** — Removes articles already used in previous runs, tracked via n8n's built-in workflow static data (no external database needed).
4. **Pick One Article** — Selects the single most recent unused article and extracts its title, summary, and source link.
5. **Write Instagram Caption** — An OpenAI-powered agent (GPT-5 mini) writes a scroll-stopping caption: a punchy hook, a short body with context, a call-to-action, the source link, and 3-5 relevant hashtags.
6. **Generate Image Prompt** — A second AI agent reads the finished caption plus the article context and writes a concrete visual scene description grounded in the actual story (not just the raw headline).
7. **Generate Post Image** — Uses `gpt-image-1-mini` to generate a clean, editorial-style square image based on that visual description — bright, professional, real-estate themed, no text overlays.
8. **Email Draft Post** — Emails the caption and generated image as a ready-to-review draft.
9. **Mark Article As Used** — Records the article's link in static data so it's excluded from future runs.

## Why the Two-Step Image Prompt?

Early versions generated the image prompt directly from the raw RSS headline, which is often vague, clickbait-y, or missing visual context (e.g., "Rates Just Did Something Nobody Expected"). Routing the image prompt through an AI step that reads the *finished caption and article summary* instead produces visuals that actually match the story — mortgage rate charts vs. new housing developments vs. rental market scenes, depending on what the article is really about.

## Tech Stack

- **n8n** — workflow orchestration
- **OpenAI GPT-5 mini** — caption writing and visual prompt generation
- **OpenAI gpt-image-1-mini** — image generation
- **RSS Feed Read** — news ingestion
- **Gmail node** — draft delivery

## Setup

1. Import `workflow.json` into your n8n instance.
2. Connect your OpenAI API credentials to both `OpenAI Chat Model` nodes and the `Generate Post Image` node.
3. Connect your Gmail OAuth2 credentials to the `Email Draft Post` node and update the `sendTo` address.
4. Activate the workflow so it runs on its schedule and static data persists correctly across weekly runs.

## Notes

- Deduplication relies on n8n's workflow static data, which persists reliably when the workflow is **Active** and run via its own trigger. Manual test executions in the editor may not always save state the same way.
- The RSS source can be swapped for any other real-estate/property news feed by changing the URL in `Read Property News`.
