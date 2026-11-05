# Auto Article Writer Agent

**Project:** SEO Blog Generation Agent (n8n workflow)

Enter a topic, brand and writing style. The AI researches keywords, optionally learns the writing style of a reference website, and writes a complete SEO-optimized blog post delivered as a styled, ready-to-publish HTML page.

## What it does

Auto Article Writer Agent is an end-to-end blog generation workflow. It collects the brief (topic, brand story, style, audience, format), optionally analyzes a reference website to copy its blog style, generates SEO keywords and a content outline with Gemini, then writes the full article. The markdown output is converted into a themed HTML document with meta tags, TL;DR, comparison table, conclusion and FAQs, and is sent back to your app.

## Workflow

```
Webhook (POST /form-submit) -> Prepare Data -> Has Reference Link?
     -> Yes: Link Analyzer (Gemini) -> Parse Link Analysis --+
     -> No:  No Link Skip -----------------------------------+
                                                             v
     Generate Keywords (Gemini) -> Parse Keyword -> Build prompt
     -> Blog Writer (Gemini) -> Results (Markdown -> HTML)
     -> data forward (HTTP POST to your backend)
```

| Phase | Nodes | Description |
|-------|-------|-------------|
| Input | Webhook, On form submission | Receives the blog brief (see input fields below) from your app or the built-in n8n form |
| Prepare | Prepare Data | Cleans the fields and maps the selected writing style to tone, structure, keyword strategy and FAQ style |
| Style Learning | Has Reference Link?, Link Analyzer, Parse Link Analysis, No Link Skip | If a reference website is given, Gemini reads its blog and extracts tone words, sentence style, heading style, key phrases and top keywords. Otherwise this step is skipped |
| Keywords | Generate Keywords, Parse Keyword | Creates the primary keyword, secondary keywords, LSI keywords, meta title (under 60 chars), meta description (under 160 chars), suggested H2s, hook question and tip box text. Your own keywords are used as the base if provided, with a safe fallback if the AI output is invalid |
| Prompt | Build prompt | Combines everything into one writing prompt: format, word count, TL;DR rule, table rule, conclusion-before-FAQs rule, brand mentions in FAQs and target audience |
| Write | Blog Writer | Gemini writes the complete article in the selected language and style |
| Format | Results | Converts markdown to a themed HTML page, restores links, adds the product reference link in body paragraphs and processes the FAQ section |
| Deliver | data forward | Sends `jobId`, `blogContent` and `productRefLink` to your backend API |

## Input fields

| Field | Type | Notes |
|-------|------|-------|
| Blog Topic | Text | Required |
| Short Brand Story | Text | Required, used to match tone in the article and FAQs |
| Writing Style | Dropdown | Spiritual & Healing, Luxury Editorial, Technical Expert or Conversational Smart |
| Brand / Business Name | Text | Required, mentioned naturally in at least 2 FAQ answers |
| Blog Format | Dropdown | Standard Article, Listicle, How-To Guide, Product Review or Ultimate Guide |
| Reference Website Link | Text | Optional, the blog style of this site is analyzed and blended in |
| Product Reference Link | Text | Optional, inserted into body paragraphs |
| Keywords | Text | Optional, comma separated, the first one becomes the primary keyword |
| Product Category | Text | Optional |
| Language | Dropdown | English, Urdu, Hindi, Arabic, Chinese or Turkish |
| Target Origin, Gender, Age Group | Text / Dropdown | Audience targeting |
| Word Count | Dropdown | 800, 1200, 1500 or 2000 |
| Include Comparison Table? | Dropdown | Yes or No |
| Include TL;DR Summary? | Dropdown | Yes or No |
| Number of FAQs | Dropdown | 4, 6 or 8 |
| Call to Action Text | Text | Optional |
| Additional Prompt | Text | Optional extra instructions |

## Setup

### Requirements
- n8n (cloud or self-hosted)
- Google Gemini API key
- A backend or app that sends the brief to the webhook and receives the finished blog (optional if you only use the n8n form)

### Steps
1. In n8n go to **Workflows -> Import from File** and select `Blogmads_n8n.json`.
2. Connect the **Google Gemini (PaLM)** credential in the three Gemini nodes: **AI Link Analyzer**, **AI Generate Keywords** and **AI Blog Writer**.
3. Open the **Webhook** node and copy the production URL (path: `form-submit`, method: POST). Send the input fields above as the JSON body, plus a `jobId`.
4. In the **data forward** node, replace the URL with your own backend endpoint that receives `jobId`, `blogContent` and `productRefLink`.
5. Optional: change the Gemini model name in the three model nodes.
6. Activate the workflow and send a test request.

## Writing styles

| Style | Voice | Theme |
|-------|-------|-------|
| Spiritual & Healing | Poetic, warm, short flowing paragraphs | Purple and gold |
| Luxury Editorial | Sophisticated, high-end brand voice | Black and gold |
| Technical Expert | Data-driven, spec-heavy, comparison tables | Blue and orange |
| Conversational Smart | Science-backed, warm, accessible | Teal and orange |

## Customization

- Add or edit writing styles in the **Prepare Data** node and the theme colors in the **Results** node.
- Add languages or blog formats by editing the form dropdowns and the `formatGuides` list in **Prepare Data**.
- Change the article rules (FAQ count, conclusion order, brand mentions) in the **Build prompt** node.
- Adjust the style analysis fields in the **Link Analyzer** prompt.

## Notes

- Concurrency lock: the form branch (**form Prepare Data**) uses a static-data flag to block a second request while one is running. The webhook branch used by the main pipeline does not use this lock.
- The execution timeout is 300 seconds. Long articles (2000 words) with reference-site analysis can take a while, so keep this in mind.
- AI output is not fact-checked. Review statistics, claims and links before publishing.
- Do not commit API keys or private backend URLs to a public repository.

## Author

Muhammad Daoud, AI Automation Engineer
