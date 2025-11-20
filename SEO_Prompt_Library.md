# SEO & Content Prompt Library — Explained (Markdown)

This document explains each of the prompts you provided: what the prompt does, when to use it, required inputs, expected outputs, step-by-step usage, best practices, pitfalls, and a ready-to-copy prompt template you can paste into your LLM. Use this file as a quick reference to pick, adapt, and run the right prompt for your task.

---

# Table of contents
1. [Expert content creator & SEO specialist (full article writer)](#expert-content-creator--seo-specialist-full-article-writer)  
2. [Expert SEO analyst (keyword research)](#expert-seo-analyst-keyword-research)  
3. [Expert SEO copywriter (meta descriptions)](#expert-seo-copywriter-meta-descriptions)  
4. [Expert SEO content strategist (create high-ranking blog content)](#expert-seo-content-strategist-create-high-ranking-blog-content)  
5. [Expert SEO content creator (keyword-rich content generation)](#expert-seo-content-creator-keyword-rich-content-generation)  
6. [Expert SEO strategist (optimize article length for ranking)](#expert-seo-strategist-optimize-article-length-for-ranking)  
7. [General tips for using these prompts together](#general-tips-for-using-these-prompts-together)  
8. [Quick reference: prompt templates (copy/paste-ready)](#quick-reference-prompt-templates-copypaste-ready)

---

# 1. Expert content creator & SEO specialist (full article writer)

## Purpose
Generate a ready-to-publish, SEO-optimized article that follows a clear structure (Introduction, Main Content with subheadings, Conclusion), uses dependency grammar for clarity, includes headlines/meta, image alt text suggestions, and internal/external link suggestions.

## When to use
- You need a longform article (blog post, pillar content) optimized for organic search and readability.
- You want the output in a structured format (HTML was requested in your earlier convo).

## Required inputs (placeholders in the prompt)
- Website/blog name  
- Target audience  
- Primary keywords  
- Content topic  
- Desired word count

## Expected output
- Full article in the requested length and format (e.g., 1500 words), structured with H1/H2/H3, bulleted key takeaways, and a short meta description at the end. Optionally HTML if asked.

## Step-by-step usage
1. Fill the placeholders with site-specific info.  
2. Ask the model to "produce the article in HTML" (if you want HTML) or plain Markdown.  
3. Review headings and keyword placement; adjust tone/word count if needed.  
4. Add final proofreading and fact-checking.

## Best practices
- Provide 2–3 example competitor URLs if you want closely-matching style/coverage.  
- Provide the exact desired tone (professional, casual, conversational).  
- Ask the model to include internal links to specific pages on your site (list them).

## Pitfalls
- If you don't supply keywords and a clear audience, output can be generic.  
- Always fact-check statistics and claims — the model may hallucinate numbers.

## Example template (ready)
```
Adopt the role of an expert content creator and SEO specialist tasked with developing high-quality, optimized content. 
My website/blog name: [INSERT WEBSITE/BLOG NAME]
Target audience: [INSERT TARGET AUDIENCE]
Primary keywords: [INSERT PRIMARY KEYWORDS]
Content topic: [INSERT CONTENT TOPIC]
Desired word count: [INSERT DESIRED WORD COUNT]
Tone: [e.g., professional / conversational / beginner-friendly]

Output required:
- A structured article with sections: Introduction, Main Content (with H2/H3 subheadings), Conclusion.
- Use dependency grammar for sentence clarity.
- Include internal link suggestions and 3 external authoritative link suggestions.
- Provide optimized image alt text for 3 images.
- Provide key takeaways as bullet points and a meta description (max 160 chars).
- Deliver in Markdown / HTML (choose one): [CHOOSE: MARKDOWN or HTML].

Write step-by-step and ensure factual accuracy.
```

---

# 2. Expert SEO analyst (keyword research)

## Purpose
Produce a prioritized keyword table (Keyword | Search Volume | Keyword Difficulty) of at least 20 keywords for a website/business.

## When to use
- Preparing a content plan or Paid Search/SEO campaign.  
- Doing initial keyword discovery and prioritization.

## Required inputs
- Website/business name  
- Target audience description  
- Core products/services  
- Geographic focus (if any)  
- Industry/niche

## Expected output
- A markdown table with ≥20 keywords, each with estimated Search Volume and Keyword Difficulty, sorted by potential impact (balancing volume & difficulty). Also include notes on intent and recommended content type for groups of keywords.

## Step-by-step usage
1. **Collect inputs.** If possible, provide the website URL.  
2. **Run keyword tools** (Ahrefs, SEMrush, Google Keyword Planner, or web.run + relevant APIs) to get actual volumes and difficulty.  
3. **Filter** for relevant long-tail keywords and user intent.  
4. **Prioritize** by impact (high volume + low difficulty, or high relevance).  
5. **Return** a table and a short action plan: “Create pillar page X + cluster of 5 posts.”

> **Important execution note:** this prompt is *designed* to be used with SEO tools. When you actually run it with a live LLM, you should fetch search volumes & difficulty from real tools. If you can’t fetch live data, ask the model to label values as **estimates** and show the assumed data source.

## Best practices
- Supply competitor URLs to analyze gaps.  
- Give geographic scope (global vs country vs city) — volumes differ widely.

## Pitfalls
- Model-generated volumes without live data are only estimates. Always confirm with an SEO tool.

## Example template (ready)
```
Adopt the role of an expert SEO analyst tasked with conducting comprehensive keyword research.

My website/business: [INSERT WEBSITE/BUSINESS NAME]
Target audience: [DESCRIBE TARGET AUDIENCE]
Primary products/services: [LIST MAIN PRODUCTS/SERVICES]
Geographic focus: [SPECIFY GEOGRAPHIC AREA IF APPLICABLE]
Industry: [SPECIFY INDUSTRY]
Competitor domains (optional): [LIST 2-3 COMPETITOR DOMAINS]

Required output:
- Markdown table with columns: Keyword | Search Volume | Keyword Difficulty (≥20 rows), sorted by estimated potential impact.
- Mark which keywords are short-tail vs long-tail and indicate user intent (informational/commercial/transactional/navigational).
- Suggest 3 content ideas (page/post) that target top 5 keywords.

Note: If live search data is not available, clearly mark volumes as ESTIMATES and explain assumptions/sources.
```

---

# 3. Expert SEO copywriter (meta descriptions)

## Purpose
Create concise, action-oriented meta descriptions (150–160 chars) to improve CTRs for specific pages.

## When to use
- Optimizing search snippets for blog posts, product pages, or landing pages.

## Required inputs
- Website name  
- Target audience  
- Primary keywords  
- Unique selling proposition (per page)  
- Page URLs or page purpose/summary

## Expected output
- A bullet list of meta descriptions; each bullet corresponds to a page. Each description: 150–160 characters, contains target keyword(s), action wording, and unique selling point.

## Step-by-step usage
1. Provide the page list (titles/short summaries).  
2. Provide target keywords per page.  
3. Ask the model to produce multiple variants if you want A/B testing options.

## Best practices
- Keep within 150–160 characters.  
- Put the most important info and keywords early.  
- Use action verbs and a value proposition.

## Pitfalls
- Avoid clickbait or misleading meta descriptions — they hurt bounce rate and rankings.

## Example template (ready)
```
Adopt the role of an expert SEO copywriter tasked with crafting meta descriptions.

My website: [INSERT WEBSITE NAME]
Target audience: [INSERT YOUR TARGET AUDIENCE]
Primary keywords (global): [INSERT PRIMARY KEYWORDS]
Unique selling proposition: [INSERT YOUR UNIQUE SELLING PROPOSITION]
Pages to write metas for (list title + 1-line page summary): 
- Page 1: [TITLE] — [1-line summary]
- Page 2: ...

Output:
- Bullet point list of meta descriptions (one per page), 150–160 characters each. Use dependency grammar for clarity.
```

---

# 4. Expert SEO content strategist (create high-ranking blog content)

## Purpose
Plan and produce SEO-optimized blog posts: keyword research, outline, headings, internal links, meta tags, and content brief.

## When to use
- Creating an editorial plan or content brief for writers/teams.  
- Producing a single article outline & first draft focused on ranking.

## Required inputs
- Topic  
- Target keywords (primary + secondary)  
- Target audience  
- Existing content summary (to avoid duplication)  
- Desired word count

## Expected output
- Keyword list (primary/secondary), content outline (H1/H2/H3), paragraph guidance, internal link targets, meta title & description, and suggested CTAs.

## Step-by-step usage
1. Provide topic and any existing pages on the site to avoid redundancy.  
2. Ask for an outline that matches user intent for the target keyword.  
3. Ask for a full draft or a content brief, depending on needs.

## Best practices
- For new content, use a pillar + cluster approach (one long pillar page, several specific cluster posts).  
- Always include suggested internal links to relevant existing pages.

## Pitfalls
- If you skip competitor analysis, you may miss necessary depth or subtopics.

## Example template (ready)
```
Adopt the role of an expert SEO content strategist to create high-ranking blog content.

Topic: [INSERT TOPIC]
Target keywords: [PRIMARY], [SECONDARY...]
Target audience: [INSERT TARGET AUDIENCE]
Existing content on site: [BRIEF DESCRIPTION OR LINKS]
Desired word count: [INSERT WORD COUNT]

Output:
- Keyword list (primary + 5 secondary)
- Full content outline using H1, H2, H3 with suggested word ranges per section
- Sample intro paragraph and 2 body paragraphs
- Meta title and meta description (optimized)
- 5 suggested internal links (by URL or page title) and 3 external authoritative refs
- Suggested images and alt text
```

---

# 5. Expert SEO content creator (keyword-rich content generation)

## Purpose
Generate keyword-rich, user-intent aligned copy (blogs, product descriptions, landing pages) using dependency grammar for clarity.

## When to use
- You need the actual body copy written with target terms integrated naturally.

## Required inputs
- Target terms  
- Website niche  
- Target audience  
- Content type (blog, product page, landing page)  
- Desired word count

## Expected output
- Structured content with headings, subheadings, keyword usage highlighted or noted, on-page SEO elements (title tag suggestion, H-tags, meta description), and internal linking suggestions.

## Step-by-step usage
1. Provide the list of target terms and priority for each (e.g., primary/secondary).  
2. Provide any brand style or tone preferences.  
3. Ask for the draft and a version with keywords highlighted.

## Best practices
- Favor readability over forced keyword stuffing.  
- Use natural morphological variants and LSI terms.

## Pitfalls
- Over-optimization (unnatural phrasing or keyword stuffing) — penalized by modern search algorithms.

## Example template (ready)
```
Adopt the role of an expert SEO content creator.

Target terms: [INSERT TARGET TERMS, MARK PRIMARY]
Website niche: [INSERT WEBSITE NICHE]
Target audience: [INSERT TARGET AUDIENCE]
Content type: [blog post / product description / landing page]
Desired word count: [INSERT WORD COUNT]
Tone: [brand tone]

Output:
- Full draft organized with H1/H2/H3
- Indicate where target terms appear (primary terms must appear in intro, at least one H2, and conclusion)
- Title tag, meta description, and 3 internal link suggestions
```

---

# 6. Expert SEO strategist (optimize article length for ranking)

## Purpose
Determine optimal article length and structure for a given target keyword by analyzing top-ranking pages — then produce an outline and draft within that word-count range.

## When to use
- You want to create content that matches or surpasses competitors in depth and coverage while staying within an optimal word count.

## Required inputs
- Target keyword  
- Desired word count (or ask the model to recommend one)  
- Target audience  
- Content type and niche

## Expected output
- Analysis of top-ranking pages (word counts + coverage), recommended word count range, outline for the article using dependency grammar framework, and a draft that respects the recommended length.

## Step-by-step usage
1. Provide the target keyword and ask for competitor URLs (or supply them).  
2. Have the model analyze top pages (structure/topics/word counts).  
3. Receive recommended word-count range and a final outline/draft.

> **Execution note:** When doing this for real-world ranking optimization, the step that analyzes top-ranking pages should be performed using live web data (search results + page scraping). If you can't provide that, instruct the model to estimate and mark it as such.

## Best practices
- Use multiple competitor pages to avoid chasing a single noisy example.  
- Focus on content *coverage* (subtopics) more than raw word count alone.

## Pitfalls
- Word count alone isn't the ranking factor. Depth, freshness, and E-A-T matter too.

## Example template (ready)
```
Adopt the role of an expert SEO strategist optimizing article length.

Target keyword: [INSERT TARGET KEYWORD]
Desired word count (optional): [INSERT DESIRED WORD COUNT or "Recommend"]
Target audience: [INSERT TARGET AUDIENCE]
Content type: [e.g., blog post, product guide]

Output:
- Analysis of top-ranking pages for the keyword (word counts and key covered subtopics) — if live data not available, mark as ESTIMATE.
- Recommended word count range to target.
- Article outline using dependency grammar framework.
- Draft of introduction + first two sections (within the recommended length).
```

---

# General tips for using these prompts together

- **Sequence for best results:**  
  1. Run the **keyword research** prompt to get prioritized keywords.  
  2. Use keywords in the **content strategist** prompt to create an outline.  
  3. Use the **content creator** prompt to write the article.  
  4. Use the **meta description** prompt to craft page snippets.  
  5. Use the **article length** prompt to validate word count and coverage.

- **Provide examples & constraints:** the clearer and more specific your inputs (audience, tone, competitor URLs), the better the outputs.

- **Fact-check numbers & claims:** Always verify statistics, dates, and factual claims with authoritative sources.

- **Local SEO:** For local businesses, always provide geographic targeting in the keyword research prompt.

- **A/B testing:** Generate 2–3 variants of meta descriptions and titles for testing.

---

# Quick reference: prompt templates (copy/paste-ready)

Below are the core templates in compact form you can paste directly into an LLM.

## 1 — Full article writer (compact)
```
You are an expert content creator & SEO specialist.
Website: [SITE]
Audience: [AUDIENCE]
Primary keywords: [KEYWORDS]
Topic: [TOPIC]
Desired word count: [WC]
Tone: [TONE]
Output: Full article with Introduction, Main Content (H2/H3), Conclusion, key takeaways (bullets), 3 image alt texts, internal/external link suggestions, and a meta description (<=160 chars). Use dependency grammar for clarity. Format: [MARKDOWN/HTML].
```

## 2 — Keyword research (compact)
```
You are an expert SEO analyst.
Website/business: [NAME]
Audience: [AUDIENCE]
Products/services: [PRODUCTS]
Geo: [GEO]
Industry: [INDUSTRY]
Output: Markdown table with 20+ keywords (Keyword | Search Volume | Keyword Difficulty) sorted by impact, with notes on intent and 3 content suggestions. If live data not available, mark as ESTIMATE.
```

## 3 — Meta descriptions (compact)
```
You are an expert SEO copywriter.
Website: [SITE]
Audience: [AUDIENCE]
Pages:
- [PAGE TITLE] — [ONE-LINE SUMMARY] — Target keyword: [KEYWORD]
... (repeat)
Output: Bullet list: one meta description per page (150–160 chars). Use action language and include target keyword naturally.
```

## 4 — Content strategist (compact)
```
You are an expert SEO content strategist.
Topic: [TOPIC]
Target keywords: [PRIMARY | SECONDARY...]
Audience: [AUDIENCE]
Existing content: [BRIEF]
Desired word count: [WC]
Output: Keyword list, full outline (H1/H2/H3), sample intro, meta title & description, internal links, and external refs.
```

## 5 — Keyword-rich content (compact)
```
You are an expert SEO content creator.
Target terms: [TERMS, mark primary]
Niche: [NICHE]
Audience: [AUDIENCE]
Content type: [TYPE]
Word count: [WC]
Output: Draft with H1/H2/H3, sample intro, indicate keyword placement, title tag, meta description, and internal links.
```

## 6 — Article length optimization (compact)
```
You are an expert SEO strategist.
Target keyword: [KEYWORD]
Desired/Recommend word count: [WC or "Recommend"]
Audience: [AUDIENCE]
Content type: [TYPE]
Output: Analyze top pages (word count + topics), recommend optimum word count range, provide outline, and draft intro + 2 sections.
If live analysis not possible, mark as ESTIMATE.
```

---

# Final notes & checklist before running any prompt
- Fill all placeholders before submitting.  
- Provide competitor URLs for targeted tone/coverage.  
- State whether you want **Markdown** or **HTML** output.  
- If you need live metrics (search volume, KD), run the keyword research prompt while connected to an SEO tool or indicate that the model should label numbers as *estimates*.  
- Always perform a final human review — for accuracy, brand voice, and legal compliance.
