# Week 1 Exercise — Technical Q&A Tutor

## Goal

Build a small **technical Q&A tutor**: you set a `question`, you get back a clear, well-structured explanation you can reuse during the course.

Demonstrates familiarity with:

- OpenAI Chat Completions API
- Ollama (OpenAI-compatible local endpoint)
- Role-based prompting
- Streaming output in Jupyter

---

## Architecture: Two Models, Two Roles

Use a **two-step pipeline**, not one model doing everything.


| Step | Model          | Role                                              |
| ---- | -------------- | ------------------------------------------------- |
| 1    | Ollama (Llama) | Fast, local first draft — raw technical mechanics |
| 2    | OpenAI (GPT)   | Polish into a study guide you’d actually use      |


### Step 1 — Ollama (local Llama)

- **System prompt:** Short, precise, mechanics-focused (no fluff).
- **User prompt:** The question only.
- **Streaming:** Optional; non-streaming is fine — you need the full text for step 2.

### Step 2 — OpenAI (GPT)

- **System prompt:** “Elite technical educator” — require:
  1. A simple real-world analogy
  2. A technical breakdown with scannable headers
  3. A summary checklist or configuration takeaway
- **User prompt:** Include **both** the original question **and** Llama’s raw answer.
- **Streaming:** Yes — render with `Markdown` + `update_display` (same pattern as day 5 brochure streaming).

This satisfies “uses OpenAI **and** Ollama” and produces output better than either model alone.

---

## Setup

### OpenAI

- `OpenAI()` client
- Model: `gpt-4o-mini` (or course default)
- API key in `.env` as `OPENAI_API_KEY`

### Ollama

- Second client: `base_url="http://localhost:11434/v1"`, dummy `api_key` (e.g. `"ollama"`)
- Model: `llama3.2` (or `llama3.2:1b` on slower machines)
- Prerequisites: `ollama serve` running, model pulled (`ollama pull llama3.2`)

> **Note:** The exercise template may label the Ollama cell “Gemini” — treat it as **Ollama**, not Google Gemini.

---

## Notebook Flow

1. **Define** `question` (code snippet, AWS concept, etc.).
2. **Call Ollama** → store result as `raw_answer`.
3. **Build GPT `user_prompt`** with the question + `raw_answer`.
4. **Call GPT** with `stream=True`, accumulate chunks, update display as tokens arrive.
5. **Optional:** Wrap steps 2–4 in `explain(question)` for a reusable tool.

---

## Streaming Pattern (GPT step)

Same idea as day 5:

1. Create a display handle: `display_handle = display(Markdown(""), display_id=True)`
2. Call `openai.chat.completions.create(..., stream=True)`
3. For each chunk, append to `response` and `update_display(Markdown(response), display_id=display_handle.display_id)`

---

## Definition of Done

- Change `question`, re-run, get a streamed markdown study guide.
- Two clients: OpenAI cloud + Ollama local.
- Two distinct system prompts (drafter vs educator).
- GPT user message includes question + Llama output.
- Final answer streams in the notebook.

---

## Optional Enhancements (not required for week 1)

- Gradio UI for the question input
- GPT-only fallback if Ollama is down
- Small function API: `explain(question) -> str`

The core exercise is the **two-stage prompt chain** plus **streaming final answer**.