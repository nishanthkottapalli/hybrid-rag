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

## If you plan to use this in Production

For use on production, the first things I would improve are:

1. better chunking  
2. stronger reranking  
3. more useful episode recall  
4. saving final validated answers into memory  
5. a richer citation format for human readability  

For now, though, this is enough to prove the core idea.

## What Next?

Before making any further improvements though, I'm interested in making the planning layer to support
optimizing for 
- **Low Cost** - fewest web calls / fewer tokens
- **High Certainity** - more retrieval + verification

That is okay.

The point of this notebook is to make the core loop real enough that I can iterate on it.
