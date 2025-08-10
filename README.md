# HN on a Platter
An n8n-based system that utilizes the HN API, JavaScript, Gemini, SQLite, and the Discord API to publish articles of interest to me in a Discord channel.

**Goal**: Use an LLM to routinely classify and share math or physics related articles from Hacker news to a dedicated Discord channel in beautiful embeds.

Please see development.md for a writeup of the development process.

## Workflow Diagram
<p align="center">
<img width="720" height="692" alt="image" src="https://github.com/user-attachments/assets/402ae257-734d-4f6a-9fde-f5debdb39fc4" />
</p>

## Explanation

1. Data Collection: The workflow fetches the IDs of the top 500 HN posts and narrows to the top 30 (making up the frontpage).

2. Filtering: The 30 IDs are cross-referenced with a lightweight SQLite database to skip previously processed content.

3. Retrieval: For new articles, full metadata (title, URL author, post date, etc.) is retrieved via the HN API.

5. Intelligent Classification: Gemini accurately classifies titles as math/physics related via few-shot prompting.

6. Deduplication and Persistent Storage: All processed IDs are stored in SQLite to prevent duplicate classification and publishing in future runs.

7. Automated Publishing: Curated articles are formatted into rich Discord embeds and posted.

## Screenshot of Discord

<p align="center">
<img width="508" height="697" alt="image" src="https://github.com/user-attachments/assets/e5525354-cfd5-4471-86ad-863f8fd5abbb" />
</p>

## Future Improvements
- Placing an "glance" at the article in the embed. if it's a paper, then its abstract. If it's an open article, then the blurb. But the cases diverge. If it's a blog, often the first couple lines contains a quote or a look at what's coming. If it's a noble prize announcement, the first couple lines will announce who the prize is being awarded to and for what. And so on. There might be a way for an LLM to do this intelligently instead of hardcoding different variations.
- Embedding the first picture in the URL. Requires less intelligence than the above.
- List of big ideas/tags/keywords of the article, kind of like an expanded title. This could be naive word counting or intelligent context analysis.
