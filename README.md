# hybrid-rag
A hybrid and traceable retrieval system with memory and output checks

## My Goal(s) - What I'm building

In this notebook-system, I'm putting together a **small retrieval-augmented generation prototype** for research-style questions.

I'm not trying to make this **production**-ready yet. I'm deliberately keeping it small and understandable so I can experiment with the moving parts:

- fetching source material from a few URLs
- cleaning and chunking the text
- retrieving relevant chunks with both keyword and embedding search
- remembering what worked in earlier runs
- generating a structured answer
- checking whether the answer follows my evidence rules
- repairing the answer if it breaks those rules

The main point of this notebook is not scale. It is **clarity**.

I want a system I can inspect end-to-end:
- what I fetched
- what I chunked
- what I retrieved
- what the model cited
- what failed validation
- what got fixed

## What this is not

I'm intentionally not treating this as a production system.

It does **not** yet handle:
- large corpora
- serious document parsing
- high-quality reranking
- full grounding verification
- cost-aware caching
- access control
- observability
- evaluation pipelines

This repository is my attempt to keep RAG experimentation **small, inspectable, and honest**.

I am not treating this as a production system. The goal here is to explore the mechanics of retrieval-augmented generation in a way that is easy to understand, modify, and extend.

---

## Why I built this

A lot of RAG examples jump too quickly from "fetch some documents" to "here is a final answer". I wanted something a little more explicit.

I wanted a prototype where I could inspect the full path:

1. what I fetched
2. how I cleaned it
3. how I chunked it
4. how I retrieved evidence
5. what the model was allowed to cite
6. whether the answer passed validation
7. how the answer was repaired if it failed

This repo is the result of that exploration.

---

## What this project does

At a high level, the notebook implements a compact research-style RAG workflow:

- fetch source pages from a small set of URLs
- clean HTML into plain text
- split documents into overlapping chunks
- index the chunks using:
  - TF-IDF for keyword retrieval
  - embeddings for semantic retrieval
- combine retrieval results using reciprocal rank fusion
- generate multiple retrieval queries through a planner agent
- collect evidence across those queries
- generate a structured final answer
- validate that answer against source/citation rules
- repair the answer if it breaks those rules
- store lightweight episode summaries for later reuse

---

## Core ideas

### 1. Hybrid retrieval

I use two retrieval styles together:

- **Keyword retrieval** helps when the wording in the question matches the wording in the source.
- **Semantic retrieval** helps when the meaning matches even if the phrasing does not.

Instead of trying to directly compare their raw scores, I merge them using **Reciprocal Rank Fusion (RRF)**.

That gives me a simple and effective hybrid retriever without adding a full vector database to the prototype.

---

### 2. Episodic memory

I wanted the system to remember a little from earlier runs, but not in a heavyweight way.

So instead of storing full transcripts, I store **episode summaries** in SQLite:

- the question
- the URLs used
- the retrieval queries that were generated
- the sources that turned out useful

When I run a similar question later, the notebook can recall related past episodes and use them as hints.

This is intentionally lightweight. It is memory as "what worked before", not memory as "full conversation history".

---

### 3. Structured outputs

Both planning and answering are done through typed schemas.

That means the system does not just produce vague free-form text. It produces structured objects like:

- retrieval plans
- final answers with sections
- citation lists
- source lists

This makes the notebook easier to debug and easier to extend.

---

### 4. Validation and repair

I did not want answer generation to be "best effort and hope for the best".

So after the answer is generated, I run a small validation layer that checks things like:

- are the listed sources actually from the allowed set?
- are the citations actually from retrieved chunk IDs?
- are there enough citations overall?
- does the executive summary visibly include evidence?

If the answer fails validation, I run a repair step that tries to fix it while preserving as much of the original answer as possible.

That gives the notebook a small but useful **critique-and-repair loop**.

---

Depending on how I evolve the repo later, I may split the notebook into scripts or modules, but for now I am intentionally keeping the main implementation in notebook form.

---

## Notebook walkthrough

The notebook is organized around a few distinct stages.

### Setup and utilities

This part handles:

* dependencies
* API key loading
* helper functions for:

  * hashing
  * URL cleanup
  * HTML-to-text conversion
  * chunking
  * citation ID normalization

The implementation is intentionally simple because I want the notebook to stay readable.

---

### Source ingestion

The ingestion layer:

* fetches pages asynchronously
* removes noisy HTML elements like `script` and `style`
* extracts visible text
* truncates very large pages
* deduplicates near-identical content

This gives me a small source corpus to work with.

---

### Chunking

Each source page is split into overlapping character-based chunks.

This is not the most sophisticated approach, but it is a practical one for a prototype:

* easy to inspect
* easy to debug
* good enough for small experiments

Later, I could swap this for token-aware or section-aware chunking.

---

### Indexing and retrieval

The chunk corpus is indexed in two ways:

#### TF-IDF index

Used for lexical / keyword retrieval.

#### Embedding matrix

Used for semantic retrieval.

At query time, I run both paths and then fuse their rankings.

---

### Planning

Before retrieving evidence, I ask a planner agent to generate:

* an objective
* subtasks
* multiple retrieval queries
* quality checks for the final answer

The idea is to avoid relying on a single user question as the only retrieval query.

This tends to improve coverage.

---

### Evidence collection

For each retrieval query, I gather:

* top keyword hits
* top semantic hits
* fused top results

I preserve the results as **evidence groups**, each tied to a specific query.

This makes it easier to inspect what each search angle actually found.

---

### Answer generation

The writer agent receives:

* the user question
* the allowed URLs
* the allowed chunk IDs
* the collected evidence groups

It then produces a structured answer with sections like:

* title
* executive summary
* architecture
* retrieval strategy
* workflow
* implementation notes
* limitations
* citations
* sources

---

### Validation

The answer is then checked for a few provenance-oriented constraints.

Examples:

* sources must come from the allowed source set
* citations must come from the allowed chunk ID set
* the answer must include a minimum number of distinct citations
* the executive summary must visibly include citations

This does not prove the answer is perfectly true. It simply enforces a better standard than "the model wrote something plausible".

---

### Repair

If validation fails, the repair agent receives:

* the validation error
* the invalid answer
* the allowed URLs
* the allowed chunk IDs

The repair step attempts to return a corrected answer that now satisfies the rules.

---

### Memory persistence

After a successful run, I store a compact episode summary in SQLite.

That gives future runs a small amount of reusable experience.

---

## How to run

### 1. Clone the repository

```bash
git clone git@github.com:nishanthkottapalli/hybrid-rag.git
cd hybrid-rag
```

### 2. Create a virtual environment

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### 3. Install dependencies

If you want to install directly:

```bash
pip install openai openai-agents pydantic httpx beautifulsoup4 lxml scikit-learn numpy jupyter
```

### 4. Set your API key

You can export it in your shell:

```bash
export OPENAI_API_KEY="your_api_key_here"
```

Or let the notebook prompt you for it when you run it.

### 5. Launch Jupyter

```bash
jupyter notebook
```

Then open the notebook file and run the cells in order.

---

## Environment notes

This project expects an OpenAI API key for:

* embeddings
* agent execution

It also creates local SQLite files for memory during execution. Those are intentionally ignored through `.gitignore`.

---

## Example use case

This prototype works best for small research-style tasks where I already know the source set I want to use.

Example pattern:

* choose a small list of source URLs
* ask a question about them
* let the system plan, retrieve, synthesize, validate, and repair

This is especially useful when I want to experiment with:

* provenance-aware answers
* retrieval strategies
* compact agent workflows
* mini memory loops

---

## What this project is **not**

This is not a production RAG stack.

It does **not** currently aim to solve:

* large-scale document ingestion
* robust document parsing across many formats
* multi-user access control
* vector database infrastructure
* evaluation pipelines
* observability dashboards
* high-confidence grounding verification
* cost-aware caching and batching strategies
* production-grade security boundaries

That is intentional.

The point of this project is to make the moving pieces visible and easy to experiment with.

---

## Limitations

A few current limitations are worth stating clearly.

### HTML extraction is basic

The notebook uses a lightweight HTML-to-text process. It will not preserve every nuance of page structure.

### Chunking is naive

Chunks are character-based, not token-aware or section-aware.

### Memory is simple

Episode recall is lexical, not semantic.

### Validation is limited

The validator checks provenance rules, not deep factual correctness.

### Scale is intentionally small

The notebook is designed for a compact source set, not a large document platform.

---

## Why I still think this is useful

Even with those limitations, I find this prototype valuable because it makes the architecture concrete.

It is one thing to talk about:

* hybrid retrieval
* memory
* citations
* guardrails

It is another thing to actually wire them together and inspect where they succeed or fail.

This repo helps me do that.

---

## Design philosophy

A few choices shaped this repo:

### Keep it inspectable

I want to be able to understand the full path from question to answer.

### Keep it small

I do not want infrastructure complexity to hide the core ideas.

### Keep the answer accountable

If the answer is grounded in retrieved evidence, I want that to be visible.

### Keep the notebook honest

I want to be clear about what this prototype can and cannot do.

---

## Who this might be useful for

This repo may be useful if you are:

* learning how RAG systems are put together
* exploring provenance-aware retrieval workflows
* experimenting with lightweight memory in agent systems
* building a small research assistant prototype
* trying to understand answer validation and repair loops

---

## Personal note

I built this as a practical prototype, not as a polished framework.

I wanted something that felt close enough to a real system to be useful, but small enough that I could still understand every part of it.

That balance matters to me.

---

## License

I have added an MIT license file. If I decide to make this broadly reusable, I will likely add a proper license in a future revision.

---


## If you plan to use this in Production

For use on production, the first things I would improve are:

1. better chunking  
2. stronger reranking  
3. more useful episode recall  
4. saving final validated answers into memory  
5. a richer citation format for human readability  

For now, though, this is enough to prove the core idea.

---

## What Next?

Before making any further improvements though, I'm interested in making the planning layer to support
optimizing for 
- **Low Cost** - fewest web calls / fewer tokens
- **High Certainity** - more retrieval + verification


The point of this notebook is to make the core loop real enough that I can iterate on it.

---

## Acknowledgment

This notebook was inspired by my own experimentation around compact RAG workflows, retrieval structure, memory, and answer validation. The implementation in this repository is written  and shaped around how I prefer to reason about these systems.

---

## Contact

GitHub: [@nishanthkottapalli](https://github.com/nishanthkottapalli)

If this repo is useful to you, feel free to fork it, adapt it, and experiment with your own retrieval workflows.
