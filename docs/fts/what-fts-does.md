# Query Processing Overview

When MySQL receives a full-text search query, it goes through several stages:

## Process

### Query Parsing

MySQL parses the search string to identify search terms and operators. 

It recognizes special syntax like boolean operators (+, -, >, <, ~, *, ""), stopwords, and phrase searches. 

The parser tokenizes the query string similar to how documents were indexed.

### Stemming

This is where 'run', 'running', 'runs' are stripped down into the core term 'run'.

### Stopword Filtering

MySQL filters out stopwords (common words like "the", "is", "at") based on the configured stopword list. 

These are ignored because they appear too frequently to be useful for relevance ranking.

### Index Lookup

MySQL uses the FULLTEXT index to quickly locate documents containing the search terms. 

The index is typically organized as an inverted index, (IDF), where each word maps to a list of documents (and positions within those documents) where it appears.

### Relevance Calculation

For each matching document, MySQL calculates a relevance score based on:

- Term frequency (how often the word appears in the document)
- Inverse document frequency (how rare the word is across all documents)
- Document length normalization
- Word proximity (for phrase searches - covered later...)

The formula is similar to TF-IDF (Term Frequency-Inverse Document Frequency).

### Result Filtering and Sorting

MySQL filters results based on the WHERE clause conditions (including the MATCH() condition itself) and sorts results by relevance score (or other specified ORDER BY criteria).

This can enable pre-filtering to avoid searching for example all categories if we have specified categories in our search.

### Result Return

Finally, MySQL returns the matching rows with their relevance scores, applying any LIMIT clauses to restrict the result set size.

PROMPT: 

> "What WordPress plugins help me with SEO and other similar search techniques"

RESPONSE:

## Processing the Query: 

"What WordPress plugins help me with SEO and other similar search techniques"

### Step 1: Break Into Words (Tokenization)

MySQL splits your query at spaces:

```
what | wordpress | plugins | help | me | with | seo | and | other | similar | search | techniques
```

### Step 2: Remove Stopwords

MySQL's default stopword list contains 36 common words including: what, with, the, is, at, and,for, from, how, in, of, on, or, that, to, was, when, where, who, will.

These words appear so frequently they provide no search value:

```
what ✗ | wordpress ✓ | plugins ✓ | help ✓ | me ✓ | with ✗ | seo ✓ | other ✓ | similar ✓ | search ✓ | techniques ✓
```

**Removed:** "what", "with". "and"

### Step 3: Check Word Length

MySQL requires words to be at least 3 characters (by default):

```
wordpress ✓ | plugins ✓ | help ✓ | me ✗ | seo ✓ | other ✓ | similar ✓ | search ✓ | techniques ✓
```

**Removed:** "me" (only 2 characters)

### Final Search Terms (9 words)

```
wordpress | plugins | help | seo | other | similar | search | techniques
```

## How Ranking Works: Single Example

Let's say there are **10,000 WordPress plugin descriptions** in the database.

We'll rank this plugin description:

**"Yoast SEO is the best WordPress SEO plugin for optimizing search rankings and implementing advanced SEO techniques"**

### The Ranking Formula

For each search word that appears in the document:

```
Score for that word = (Times it appears) × IDF × IDF

Where IDF = log10(Total documents / Documents containing this word)
```

**Rare words get higher IDF scores. Common words get lower IDF scores.**

---

### Calculating the Score

| Search Term | How Many Documents Contain It? | IDF Score | Times in This Document | Contribution to Total |
|-------------|-------------------------------|-----------|----------------------|---------------------|
| **wordpress** | 8,500 (very common) | 0.07 | 1 | 0.07 × 0.07 × 1 = **0.005** |
| **plugins** | 3,000 | 0.52 | 0 | **0.000** |
| **help** | 4,500 | 0.35 | 0 | **0.000** |
| **seo** | 1,500 (moderately rare) | 0.82 | 3 | 0.82 × 0.82 × 3 = **2.036** |
| **other** | 7,000 | 0.15 | 0 | **0.000** |
| **similar** | 1,800 | 0.74 | 0 | **0.000** |
| **search** | 5,000 | 0.30 | 1 | 0.30 × 0.30 × 1 = **0.091** |
| **techniques** | 300 (very rare) | 1.52 | 1 | 1.52 × 1.52 × 1 = **2.319** |

### Final Score: 4.45 points

```
0.005 + 0.000 + 0.000 + 2.036 + 0.000 + 0.000 + 0.091 + 2.319 = 4.45
```

## Why This Score Makes Sense

**What matters most:**

1. **"techniques"** contributes **2.319 points** (52% of total score)

   - Only appears in 300 out of 10,000 documents (rare = valuable)
   
2. **"seo"** contributes **2.036 points** (45% of total score)

   - Appears 3 times in the document
   - Only in 1,500 documents (moderately rare)

3. **"search"** contributes **0.091 points** (2% of total score)

   - Appears in half the documents (common)

**What barely matters:**

4. **"wordpress"** and **"and"** together contribute only **0.006 points** (0.1% of total score)

   - They're in almost every document, so they don't help distinguish good matches

**Missing words don't hurt much:**

- The document doesn't contain "plugins", "help", "other", or "similar"
- But it still ranks highly because it has the important semantic terms

## The Key Insight

**Rare, specific words dominate the ranking.** 

A document mentioning "SEO techniques" will rank far higher than one mentioning "WordPress plugins help" because:

- "techniques" is rare (high value)
- "wordpress" is everywhere (low value)
- "help" is common (low value)

This is why MySQL Full-Text Search automatically finds the most relevant results without you needing to specify which words are important.

<br>