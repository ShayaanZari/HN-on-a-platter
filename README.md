# HN on a Platter
An n8n-based system that utilizes the HN API, JavaScript, Gemini, SQLite, and the Discord API to publish articles of interest to me in a Discord channel.

**Goal**: Use an LLM to routinely <u>classify<u/> and share math, or physics related articles from Hacker news to a dedicated <u>Discord</u> channel in <u>beautiful embeds</u>.

## Workflow Diagram
<img width="720" height="692" alt="image" src="https://github.com/user-attachments/assets/402ae257-734d-4f6a-9fde-f5debdb39fc4" />

## Explanation

1. Data Collection and Filtering: The workflow fetches the IDs of the top 500 HN posts, narrows to the top 30 (making up the frontpage), and cross-references these IDs with a lightweight SQLite database to skip previously processed content. For new articles, the system then retrieves full metadata (title, URL< author, post date, etc.) via the HN API.

2. Intelligent classification: Gemini accurately classifies titles as math/physics related via <u>few-shot prompting</u>.

3. Deduplication and persistent storage: All processed IDs are stored in SQLite to prevent duplicate classification and publishing in future runs.

4. Automated Publishing: Curated articles are formatted into rich Discord embeds and posted.
