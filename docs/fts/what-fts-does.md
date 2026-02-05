# Query Processing Overview

When MySQL receives a full-text search query, it goes through several stages:

## Process

**1. Query Parsing**

MySQL parses the search string to identify search terms and operators. 

It recognizes special syntax like boolean operators (+, -, >, <, ~, *, ""), stopwords, and phrase searches. 

The parser tokenizes the query string similar to how documents were indexed.

**2. Stopword Filtering**

MySQL filters out stopwords (common words like "the", "is", "at") based on the configured stopword list. 

These are ignored because they appear too frequently to be useful for relevance ranking.

**3. Query Expansion (Natural Language Mode)**

In natural language mode, MySQL may expand the query internally. 

It identifies the search terms that will be used to probe the full-text index.

**4. Index Lookup**

MySQL uses the FULLTEXT index to quickly locate documents containing the search terms. 

The index is typically organized as an inverted index where each word maps to a list of documents (and positions within those documents) where it appears.

**5. Relevance Calculation**

For each matching document, MySQL calculates a relevance score based on:

- Term frequency (how often the word appears in the document)
- Inverse document frequency (how rare the word is across all documents)
- Document length normalization
- Word proximity (for phrase searches)

The formula is similar to TF-IDF (Term Frequency-Inverse Document Frequency).

**6. Boolean Evaluation (Boolean Mode)**

In boolean mode, MySQL applies the boolean operators to determine which documents satisfy the query conditions. 

Required terms (+), excluded terms (-), and optional terms are processed according to boolean logic.

**7. Result Filtering and Sorting**

MySQL filters results based on the WHERE clause conditions (including the MATCH() condition itself) and sorts results by relevance score (or other specified ORDER BY criteria).

**8. Result Return**

Finally, MySQL returns the matching rows with their relevance scores, applying any LIMIT clauses to restrict the result set size.


## Index Maintenance

**Insert/Update/Delete Operations:**

InnoDB maintains FULLTEXT indexes using auxiliary tables and a document ID system. New content is tokenized and added to in-memory buffers before being flushed to the index. Updates mark old documents as deleted and insert new versions, while deletes flag documents for later cleanup rather than immediate removal.

**Performance Impact:**

High-volume inserts create write overhead as indexes are maintained in real-time. Index fragmentation accumulates from updates and deletes, requiring periodic optimization with `OPTIMIZE TABLE`.

**Best Practices:**

- Batch inserts in transactions to reduce overhead
- Monitor index size and fragmentation regularly
- Tune configuration parameters like `innodb_ft_cache_size`
- For extreme volumes, consider delayed indexing patterns or external search engines like Elasticsearch

*The key tradeoff: better search performance costs write speed.*

<br>