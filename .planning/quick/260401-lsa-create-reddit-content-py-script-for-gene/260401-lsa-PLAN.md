---
phase: quick
plan: 01
type: execute
wave: 1
depends_on: []
files_modified:
  - reddit_content.py
  - data/reddit/
autonomous: true
must_haves:
  truths:
    - "Running reddit_content.py generates 3-5 Reddit-ready comments as a markdown file"
    - "Comments are genuinely helpful aquarium advice with natural article links"
    - "Already-processed articles are not regenerated on subsequent runs"
  artifacts:
    - path: "reddit_content.py"
      provides: "Reddit comment generation script"
    - path: "data/reddit/"
      provides: "Output directory for generated comment markdown files"
  key_links:
    - from: "reddit_content.py"
      to: "data/content.db"
      via: "sqlite3 query for published articles"
      pattern: "SELECT.*FROM articles.*status.*published"
    - from: "reddit_content.py"
      to: "claude CLI subprocess"
      via: "subprocess.run calling claude"
      pattern: "subprocess.*claude"
---

<objective>
Create a reddit_content.py script in the pipeline root that generates Reddit-ready comments from published articles.

Purpose: Drive traffic to aquapicked.com by generating genuinely helpful Reddit comments that naturally reference published articles. This is a content repurposing tool, not spam — each comment must stand on its own as useful advice.

Output: A standalone Python script + markdown output files in data/reddit/
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/execute-plan.md
@$HOME/.claude/get-shit-done/templates/summary.md
</execution_context>

<context>
@config.yaml
@database.py

Key codebase facts:
- Database: data/content.db (SQLite), accessed via database.py ContentDatabase class
- Articles table has: id, site, keyword, title, slug, content, word_count, category, status, meta_description
- Published articles have status='published', site key is 'aquarium'
- Site domain: aquapicked.com (from memory), URL pattern: https://aquapicked.com/{slug}/
- Config loaded from config.yaml, db path is config["database"]["path"]
- Existing pipeline scripts (pipeline.py, keywords.py, writer.py, publisher.py) live at project root: /Users/toxic/Documents/Projects/affiliate-pipeline/
</context>

<tasks>

<task type="auto">
  <name>Task 1: Create reddit_content.py script</name>
  <files>reddit_content.py</files>
  <action>
Create reddit_content.py in the pipeline root (/Users/toxic/Documents/Projects/affiliate-pipeline/reddit_content.py).

The script should:

1. **Database integration**: Import and use ContentDatabase from database.py. Load config from config.yaml to get db path. Query published articles for site 'aquarium'.

2. **Tracking table**: Add a `reddit_comments` table to the database (create if not exists) with columns:
   - id INTEGER PRIMARY KEY AUTOINCREMENT
   - article_id INTEGER NOT NULL (FK to articles.id)
   - generated_at TEXT DEFAULT CURRENT_TIMESTAMP
   - UNIQUE(article_id) — prevents regenerating for same article

3. **Article selection**: Query published articles that do NOT have an entry in reddit_comments. Limit to 5 articles per run. If no unprocessed articles exist, print a message and exit.

4. **Comment generation via Claude CLI**: For each selected article, call `subprocess.run(["claude", "-p", prompt], capture_output=True, text=True)` where the prompt asks Claude to generate a single Reddit comment that:
   - Provides genuinely helpful aquarium advice related to the article's topic (use the article title, category, keyword, and meta_description for context — do NOT send the full article content to keep the subprocess fast)
   - Naturally mentions "I wrote a more detailed breakdown here: https://aquapicked.com/{slug}/" near the end
   - Targets a specific subreddit (include in prompt: suggest from r/Aquariums, r/PlantedTank, r/ReefTank, r/Bettafish, r/Cichlids based on article category)
   - Is 150-300 words, conversational Reddit tone, no marketing speak
   - Returns structured output as JSON with keys: comment_text, suggested_subreddit, article_url

5. **Output generation**: Write all generated comments to a single markdown file at `data/reddit/reddit-comments-{YYYY-MM-DD}.md` with this format per comment:
   ```
   ## Comment for: {article_title}
   **Subreddit:** r/{subreddit}
   **Article:** {url}

   ---
   {comment_text}
   ---
   ```

6. **Tracking update**: After successfully generating a comment, insert article_id into reddit_comments table.

7. **CLI entry point**: Make it runnable as `python reddit_content.py [site]` (default site: 'aquarium'). Print progress and summary.

Use the same config.yaml / ContentDatabase patterns as pipeline.py. Create data/reddit/ directory if it doesn't exist (os.makedirs).

Parse the JSON from Claude CLI output — wrap in try/except for malformed responses and skip that article with a warning if parsing fails. Use `json.loads()` and look for the JSON object in the response (Claude may include text before/after the JSON — extract with a regex or find first `{` and last `}`).
  </action>
  <verify>
    <automated>cd /Users/toxic/Documents/Projects/affiliate-pipeline && python -c "import reddit_content; print('import ok')"</automated>
  </verify>
  <done>reddit_content.py exists, imports cleanly, contains all described functionality: db tracking table, article selection, Claude CLI generation, markdown output, dedup logic</done>
</task>

<task type="auto">
  <name>Task 2: Test with dry-run and verify output</name>
  <files>data/reddit/</files>
  <action>
Run the script against the live database to generate comments for unprocessed published articles:

```
cd /Users/toxic/Documents/Projects/affiliate-pipeline && python reddit_content.py aquarium
```

Verify:
1. The script connects to data/content.db successfully
2. The reddit_comments tracking table is created
3. It selects published articles not yet processed
4. It calls Claude CLI and generates comments
5. A markdown file appears at data/reddit/reddit-comments-2026-04-01.md
6. The markdown contains 3-5 comment entries with subreddit, URL, and comment text
7. Running the script again shows "No unprocessed articles" or processes only new ones (dedup works)

If any errors occur during generation, fix them in reddit_content.py. Common issues to watch for:
- Claude CLI not found (check path)
- JSON parsing failures from Claude output
- Database locking issues
  </action>
  <verify>
    <automated>cd /Users/toxic/Documents/Projects/affiliate-pipeline && test -f data/reddit/reddit-comments-2026-04-01.md && echo "output file exists" && python -c "
import sqlite3
conn = sqlite3.connect('data/content.db')
count = conn.execute('SELECT COUNT(*) FROM reddit_comments').fetchone()[0]
print(f'tracked articles: {count}')
conn.close()
" && echo "PASS"</automated>
  </verify>
  <done>Reddit comments markdown file generated with 3-5 entries, tracking table populated, re-run does not duplicate</done>
</task>

</tasks>

<verification>
- reddit_content.py exists in pipeline root and runs without errors
- data/reddit/ directory contains at least one markdown output file
- reddit_comments table in content.db tracks processed articles
- Generated comments are helpful, conversational, and include article links naturally
- Re-running the script does not regenerate for already-processed articles
</verification>

<success_criteria>
- Script generates 3-5 Reddit-ready comments per run from published articles
- Output is a well-formatted markdown file in data/reddit/
- Deduplication prevents regenerating for same articles
- Comments are genuinely helpful first, with natural link mentions
</success_criteria>

<output>
After completion, create `.planning/quick/260401-lsa-create-reddit-content-py-script-for-gene/260401-lsa-SUMMARY.md`
</output>
