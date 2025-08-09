# HN on a Platter
An n8n-based system that utilizes the HN API, JavaScript, Gemini, SQLite, and the Discord API to publish articles of interest to me in a Discord channel.

**Goal**: Use an LLM to routinely classify and share math or physics related articles from Hacker news to a dedicated Discord channel in beautiful embeds.

## Workflow Diagram
<p align="center">
<img width="720" height="692" alt="image" src="https://github.com/user-attachments/assets/402ae257-734d-4f6a-9fde-f5debdb39fc4" />
</p>

## Explanation

1. Data Collection and Filtering: The workflow fetches the IDs of the top 500 HN posts, narrows to the top 30 (making up the frontpage), and cross-references these IDs with a lightweight SQLite database to skip previously processed content. For new articles, the system then retrieves full metadata (title, URL author, post date, etc.) via the HN API.

2. Intelligent Classification: Gemini accurately classifies titles as math/physics related via few-shot prompting.

3. Deduplication and Persistent Storage: All processed IDs are stored in SQLite to prevent duplicate classification and publishing in future runs.

4. Automated Publishing: Curated articles are formatted into rich Discord embeds and posted.

## Screenshot of Discord

<p align="center">
<img width="508" height="697" alt="image" src="https://github.com/user-attachments/assets/e5525354-cfd5-4471-86ad-863f8fd5abbb" />
</p>
