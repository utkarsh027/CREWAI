# 🎬 CrewAI YouTube Blog Automation

<p align="left">
  <img src="https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white"/>
  <img src="https://img.shields.io/badge/CrewAI-FF4B4B?style=flat-square"/>
  <img src="https://img.shields.io/badge/OpenAI_GPT--4-412991?style=flat-square&logo=openai&logoColor=white"/>
  <img src="https://img.shields.io/badge/YouTube_Transcript_API-FF0000?style=flat-square&logo=youtube&logoColor=white"/>
  <img src="https://img.shields.io/badge/Multi--Agent-00D4AA?style=flat-square"/>
</p>

A **multi-agent AI pipeline** that takes a YouTube topic, pulls the video transcript, and produces a complete, publication-ready blog post — fully automated. Two specialised CrewAI agents collaborate sequentially: one researches, one writes.

---

## What It Does

Give it a topic. It finds the relevant YouTube video, extracts the transcript, and outputs a formatted blog post to `new-blog-post.md`. No manual summarising, no copy-pasting.

```bash
# Input
topic = "AI VS ML VS DL vs Data Science"

# Output → new-blog-post.md
# A full blog post written from the YouTube video transcript
```

---

## Agent Crew

Two agents are defined in `agents.py`, each with a distinct role, goal, backstory, and tool access:

### 🔍 Blog Researcher
```python
blog_researcher = Agent(
    role='Blog Researcher from Youtube Videos',
    goal='get the relevant video transcription for the topic {topic} from the provided Yt channel',
    memory=True,
    backstory="Expert in understanding videos in AI, Data Science, Machine Learning and GenAI",
    tools=[yt_tool],
    allow_delegation=True   # can hand off sub-tasks if needed
)
```

### ✍️ Blog Writer
```python
blog_writer = Agent(
    role='Blog Writer',
    goal='Narrate compelling tech stories about the video {topic} from YT video',
    memory=True,
    backstory="With a flair for simplifying complex topics, you craft engaging narratives "
              "that captivate and educate, bringing new discoveries to light in an accessible manner.",
    tools=[yt_tool],
    allow_delegation=False  # terminal agent — produces the final output
)
```

**Key difference:** `blog_researcher` has `allow_delegation=True` — it can break work into sub-tasks. `blog_writer` has `allow_delegation=False` — it is the end of the chain and produces the final content.

---

## Tasks

Defined in `tasks.py` — each task is assigned to a specific agent with an explicit expected output:

```python
# Task 1: Research
research_task = Task(
    description="Identify the video {topic}. Get detailed information from the channel video.",
    expected_output="A comprehensive 3 paragraphs long report based on the {topic} of video content.",
    tools=[yt_tool],
    agent=blog_researcher,
)

# Task 2: Write
write_task = Task(
    description="Get the info from the youtube channel on the topic {topic}.",
    expected_output="Summarize the info from the youtube channel video on the topic {topic} "
                    "and create the content for the blog",
    tools=[yt_tool],
    agent=blog_writer,
    async_execution=False,
    output_file='new-blog-post.md'   # final blog written to disk
)
```

`output_file='new-blog-post.md'` — the writer's output is saved automatically to a markdown file.

---

## Pipeline Architecture

```
crew.kickoff(inputs={'topic': 'AI VS ML VS DL vs Data Science'})
        │
        ▼
┌─────────────────────────────────────────────────┐
│  Process.sequential                             │
│                                                 │
│  Step 1: blog_researcher                        │
│    ├── yt_tool → fetch YouTube transcript       │
│    ├── Analyse transcript for topic             │
│    └── Output: 3-paragraph research report      │
│                    │                            │
│                    ▼  (hands off to writer)     │
│  Step 2: blog_writer                            │
│    ├── Receives research report as context      │
│    ├── yt_tool → access transcript if needed    │
│    └── Output: blog post → new-blog-post.md     │
└─────────────────────────────────────────────────┘
```

---

## Crew Configuration

```python
# crew.py
crew = Crew(
    agents=[blog_researcher, blog_writer],
    tasks=[research_task, write_task],
    process=Process.sequential,  # researcher runs first, writer second
    memory=True,                 # agents remember context across steps
    cache=True,                  # tool results cached — avoids duplicate API calls
    max_rpm=100,                 # rate limit: max 100 requests per minute
    share_crew=True              # agents share context with each other
)
```

- `memory=True` — each agent retains context from previous steps within the run
- `cache=True` — if `yt_tool` is called twice with the same input, the second call is free
- `Process.sequential` — guaranteed order: research completes before writing begins

---

## The YouTube Tool

`yt_tool` (defined in `tools.py`) uses `youtube_transcript_api` to pull the full transcript from a YouTube video by topic or URL. This is what both agents use to ground their outputs in real video content rather than hallucinated summaries.

---

## Tech Stack

| Component | Technology |
|-----------|-----------|
| Agent framework | CrewAI |
| LLM | GPT-4 (`gpt-4-0125-preview`) |
| YouTube transcript | `youtube_transcript_api` |
| Web scraping | `bs4`, `unstructured` |
| Search | `duckduckgo-search` |
| Config | `python-dotenv` |

---

## Setup

```bash
git clone https://github.com/utkarsh027/CREWAI
cd CREWAI
pip install -r requirements.txt
```

**Create `.env`:**
```env
OPENAI_API_KEY=sk-...
```

**Run:**
```bash
python crew.py
```

Output blog post saved to `new-blog-post.md` in the project root.

---

## Changing the Topic

Edit the last line of `crew.py`:

```python
result = crew.kickoff(inputs={'topic': 'Your topic here'})
```

The `{topic}` placeholder is injected into both the agent goals and task descriptions automatically by CrewAI.

---

## Project Structure

```
CREWAI/
├── agents.py          # blog_researcher and blog_writer agent definitions
├── tasks.py           # research_task and write_task definitions
├── crew.py            # Crew assembly and kickoff
├── tools.py           # yt_tool — YouTube transcript fetcher
├── new-blog-post.md   # Auto-generated output (created on run)
├── requirements.txt
└── .env               # OPENAI_API_KEY (not committed)
```

---

## Author

**Utkarsh Tiwari** — MSc Data Science, UC Irvine  
[github.com/utkarsh027](https://github.com/utkarsh027) · [LinkedIn](https://www.linkedin.com/in/utkarsh-tiwari-330857166/)
