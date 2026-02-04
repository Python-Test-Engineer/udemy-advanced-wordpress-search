# Advanced WordPress Search

In my work building chatbots and voice/video agents, getting the right answer is paramount.

I built a number of plugins for WP Search that used Full Text Search and Vector/Semantic Search for my site [https://ancestors-ai.com/](https://ancestors-ai.com/) and I decided to share my learnings with others.

## Full text Search

Full Text Search, (FTS), using BM25, is an industry standard and the course looks at what the components of BM25 are, both from theory and with a number of eductational plugins that enable experimentation.

## FTS modes

We will cover the three types - Natural Language, Boolean Expression and Query Expansion.


## Semantic Search

We build our own custom Vector Database and Vector Search. We do this because our hosting service for MySQL may not have the available Vector Field, as in the case of mine, (Siteground).

This enables Hybrid Search - getting results based on what the user enters, (lexical) and what the user means, (semantic).


## Hybrid Search

FTS is what the users says, Semantic search is what the user means.

We combine the results of both in a normalised way and have the ability to weight one more than the other.


## RAG

This then gives us the tradionaly search results but it can also be used to combine the restults to give comprehensive context for Retrieval Augmented Generation, (RAG).

To do this, we create a context of information based on the result set and ask an LLM to generate a response to the user query based on the context retrieved along with an explanation of why it gave its final answer.

This can lead to Agentic RAG, explained more fully here [https://advanced-wordpress-search.netlify.app/rag/agentic-rag/](https://advanced-wordpress-search.netlify.app/rag/agentic-rag/).

<br>