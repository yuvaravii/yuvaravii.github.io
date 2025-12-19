---
title: "Addressing Pagination after retrieval from Vector DB"
date: 2025-12-19  00:00:00 +0800
categories: [Technology]
tags: [Machine-Learning]
---


# Pagination in Vector Databases 📚

## What is Pagination?

**Simple Definition:** Pagination is the process of dividing a large list of items into smaller chunks called "pages" so users can browse through them easily.

### Real-World Example 🌍

Think of Google search results:
- You search for "best laptops 2024"
- Google shows you 10 results on page 1
- You click "Next" to see results 11-20 on page 2
- You can jump to page 5 to see results 41-50

This is **pagination** - breaking down thousands of results into manageable pages!

---

## Why is Pagination Challenging with Vector Databases? 🤔

### Understanding the Problem

Let's break this down step by step:

#### 1. **How Vector Databases Work**

```
Text → Tokens → Embeddings → Vector Database
```

**Example:**
- Query: "wireless headphones"
- Gets converted to a vector: `[0.23, 0.45, 0.12, ...]` (hundreds of numbers)
- Vector DB finds similar vectors using **semantic similarity**

#### 2. **The Core Problem: Inconsistent Results**

Unlike traditional databases (SQL), vector databases use **semantic search**, which means:

| Traditional Database (SQL)                        | Vector Database                         |
| ------------------------------------------------- | --------------------------------------- |
| Results are **deterministic** (always same order) | Results can **vary slightly** each time |
| Uses exact matching                               | Uses similarity scores                  |
| Easy to paginate with OFFSET                      | Pagination is tricky!                   |

**Why Results Vary:**
- Vector databases are optimized for finding the "top N most similar" results
- Small changes in the database or algorithm can shift rankings
- Scores like `0.8523` vs `0.8521` are very close - order might change

#### 3. **The Pagination Dilemma**

**Scenario:** User searches for "wireless headphones"

```
First query:  [Result A, Result B, Result C, Result D, ...]
Second query: [Result A, Result C, Result B, Result D, ...]  ← Notice B and C swapped!
```

**Problem:** If a user is on page 2 and refreshes, they might:
- See duplicate results (Result B appears on both page 1 and page 2)
- Miss results (Result C skipped entirely)
- Get confused by inconsistent ordering

---

## Three Strategies to Solve This Problem 🛠️

### Strategy 1: Offset-Based Pagination ❌ (Not Recommended)

#### How It Works

```python
# Page 1: Get results 1-20
results = vector_db.search(query, limit=20, offset=0)

# Page 2: Get results 1-40, then throw away first 20
results = vector_db.search(query, limit=40, offset=0)
page_2 = results[20:40]

# Page 3: Get results 1-60, then throw away first 40
results = vector_db.search(query, limit=60, offset=0)
page_3 = results[40:60]
```

#### Real-World Analogy 🏪

Imagine asking a shopkeeper:
- **Page 1:** "Show me your top 20 products"
- **Page 2:** "Show me your top 40 products" (then you ignore the first 20)
- **Page 10:** "Show me your top 200 products" (then you ignore the first 180!)

**This is wasteful!** You're asking for 200 items but only using 20.

#### Problems

| Issue         | Impact                                                    |
| ------------- | --------------------------------------------------------- |
| **Wasteful**  | Fetches 420 results to show 100 (76% waste!)              |
| **Slow**      | Each page takes longer (page 10 is 3x slower than page 1) |
| **Expensive** | Vector computations are costly                            |

#### Performance Data

```
Page 1:  Fetch  20 → Use  20 → Waste   0 (0.045s)
Page 2:  Fetch  40 → Use  20 → Waste  20 (0.052s)
Page 3:  Fetch  60 → Use  20 → Waste  40 (0.061s)
Page 10: Fetch 200 → Use  20 → Waste 180 (0.156s) ⚠️ Getting slower!
```

---

### Strategy 2: Cursor-Based Pagination ⚠️ (Risky)

#### How It Works

```python
# Page 1: Get 20 results, remember the last score
results, cursor = vector_db.search(query, limit=20)
# cursor = {"last_score": 0.85, "last_id": "item_20"}

# Page 2: Get next 20 results AFTER the cursor
results, cursor = vector_db.search(query, limit=20, after=cursor)
# cursor = {"last_score": 0.78, "last_id": "item_40"}
```

#### Real-World Analogy 📖

Like reading a book with a bookmark:
- **Page 1:** Read pages 1-20, put bookmark at page 20
- **Page 2:** Start from bookmark, read pages 21-40
- **Page 3:** Start from new bookmark position

#### The Problem: Shifting Results 🎯

**Scenario:** While you're browsing, the database updates or re-ranks results

```
Initial state:
Page 1: [A(0.90), B(0.85), C(0.80), D(0.75), E(0.70)]
        Cursor: score=0.70 ↑

Database re-ranks (maybe new data added):
Page 2: [F(0.68), C(0.65), G(0.60), ...] 
        ↑ Wait! C was on page 1 before!
```

**Result:** You might see item C twice or miss item B entirely!

#### When It's Risky

- ❌ Search results that change frequently
- ❌ Multiple users searching simultaneously
- ❌ When consistency is critical (e-commerce, medical records)

#### When It's OK

- ✅ Social media feeds (duplicates are acceptable)
- ✅ Real-time data streams
- ✅ "Infinite scroll" interfaces

---

### Strategy 3: Two-Stage Pagination ✅ (Production-Ready)

#### How It Works

**Think of it like a shopping cart system:**

```
Stage 1 (Fetch Once):
🔍 Librarian fetches 1000 books and puts them on a cart
   → This happens ONCE per search

Stage 2 (Slice Many Times):
📖 You take books 1-20 from the cart (Page 1)
📖 You take books 21-40 from the cart (Page 2)
📖 You take books 81-100 from the cart (Page 5)
   → This is instant! Just slicing an array
```

#### Visual Example 🎨

```
User searches "wireless headphones"

┌─────────────────────────────────────────┐
│  STAGE 1: Vector Search (Expensive)     │
│  🔍 Query Qdrant → Get 1000 results     │
│  ⏱️  Takes: 0.234 seconds                │
│  💾 Store in cache for 10 minutes       │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│  CACHE (1000 results stored)            │
│  [Result1, Result2, ... Result1000]     │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│  STAGE 2: In-Memory Slicing (Fast)      │
│  Page 1:  results[0:20]   ⚡ 0.001s     │
│  Page 2:  results[20:40]  ⚡ 0.001s     │
│  Page 5:  results[80:100] ⚡ 0.001s     │
└─────────────────────────────────────────┘
```

#### Code Example with Explanation

```python
class VectorSearchPagination:
    """
    Two-Stage Pagination System
    Stage 1: Fetch large set once → Cache it
    Stage 2: Slice from cache → Super fast!
    """
    
    def __init__(self, client, cache_ttl_seconds=600):
        self.client = client
        self.cache = {}  # In production: use Redis
        self.ttl = cache_ttl_seconds  # Cache expires after 10 minutes
    
    def get_candidates(self, user_id: str, query: str, k: int = 1000):
        """
        STAGE 1: Fetch the big list (expensive, happens once)
        
        Example:
        - User searches "wireless headphones"
        - We fetch top 1000 results from vector DB
        - Store in cache for 10 minutes
        """
        cache_key = f"search:{user_id}:{hash(query)}"
        
        # Check if we already have this search cached
        if cache_key in self.cache:
            if self._is_cache_valid(self.cache[cache_key]):
                print("✅ Using cached results (instant!)")
                return self.cache[cache_key]["data"]
        
        # Cache miss - need to query the vector database
        print(f"🔍 Querying vector DB for {k} results...")
        results = self.client.search(
            collection_name="products",
            query_vector=embed(query),  # Convert text to vector
            limit=k,  # Get top 1000
            with_payload=True
        )
        
        # Store in cache
        self.cache[cache_key] = {
            "data": results,
            "expires_at": datetime.now() + timedelta(seconds=self.ttl)
        }
        
        return results
    
    def paginate(self, user_id: str, query: str, page: int, page_size: int = 20):
        """
        STAGE 2: Slice the cached results (fast, happens many times)
        
        Example:
        - User wants page 2 (results 21-40)
        - We just slice the cached 1000 results
        - No vector computation needed!
        """
        # Get the full candidate list (from cache if available)
        candidates = self.get_candidates(user_id, query)
        
        # Calculate which slice we need
        start = (page - 1) * page_size  # Page 2: (2-1) * 20 = 20
        end = start + page_size          # 20 + 20 = 40
        
        # Slice the list (super fast - just array indexing!)
        page_results = candidates[start:end]
        
        return {
            "results": page_results,
            "page": page,
            "total": len(candidates),
            "has_next": end < len(candidates)
        }
```

#### Step-by-Step Example 📝

```python
# Initialize the paginator
paginator = VectorSearchPagination(client, cache_ttl_seconds=600)

# User "alice" searches for "wireless headphones"
# ─────────────────────────────────────────────────

# Request Page 1
page1 = paginator.paginate("alice", "wireless headphones", page=1)
# Output: 🔍 Querying vector DB for 1000 results... (0.234s)
# Returns: Results 1-20

# Request Page 2 (same user, same query)
page2 = paginator.paginate("alice", "wireless headphones", page=2)
# Output: ✅ Using cached results (instant!) (0.001s)
# Returns: Results 21-40

# Request Page 5
page5 = paginator.paginate("alice", "wireless headphones", page=5)
# Output: ✅ Using cached results (instant!) (0.001s)
# Returns: Results 81-100

# Different user, same query
page1_bob = paginator.paginate("bob", "wireless headphones", page=1)
# Output: 🔍 Querying vector DB for 1000 results... (0.234s)
# Note: Separate cache per user!
```

#### Why This Works Best ✨

| Advantage         | Explanation                                 |
| ----------------- | ------------------------------------------- |
| **Consistent**    | Results don't change between pages (cached) |
| **Fast**          | Pages 2+ are instant (0.001s vs 0.045s)     |
| **Efficient**     | Only 1 vector search instead of multiple    |
| **User-Friendly** | Each user gets their own consistent view    |

---

## Performance Comparison 📊

### Scenario: User browses pages 1, 2, 3, 5, and 10

#### Strategy 1: Offset-Based ❌

```
Page 1:  Fetch  20 → Use  20 → Waste   0 → 0.045s
Page 2:  Fetch  40 → Use  20 → Waste  20 → 0.052s
Page 3:  Fetch  60 → Use  20 → Waste  40 → 0.061s
Page 5:  Fetch 100 → Use  20 → Waste  80 → 0.089s
Page 10: Fetch 200 → Use  20 → Waste 180 → 0.156s
─────────────────────────────────────────────────
TOTAL:   Fetch 420 → Use 100 → Waste 320 (76% waste!)
```

#### Strategy 2: Cursor-Based ⚠️

```
Page 1:  Fetch 20 → Use 20 → 0.043s
Page 2:  Fetch 20 → Use 20 → 0.041s ⚠️ Might skip/duplicate
Page 3:  Fetch 20 → Use 20 → 0.044s ⚠️ Might skip/duplicate
Page 5:  Fetch 20 → Use 20 → 0.042s ⚠️ Might skip/duplicate
Page 10: Fetch 20 → Use 20 → 0.045s ⚠️ Might skip/duplicate
─────────────────────────────────────────────────
TOTAL:   Fetch 100 → Use 100 (Efficient but risky!)
```

#### Strategy 3: Two-Stage ✅

```
Page 1:  🔍 Vector Search → Fetch 1000 → Cache → 0.234s
Page 2:  ✅ Cache Hit     → Slice array → 0.001s
Page 3:  ✅ Cache Hit     → Slice array → 0.001s
Page 5:  ✅ Cache Hit     → Slice array → 0.001s
Page 10: ✅ Cache Hit     → Slice array → 0.001s
─────────────────────────────────────────────────
TOTAL:   Fetch 1000 once, then instant slicing!
Total time: 0.238s (vs 0.403s for offset-based)
```

### Summary Table

| Strategy         | Vector Computations | Waste | Speed       | Consistency | Production-Ready |
| ---------------- | ------------------- | ----- | ----------- | ----------- | ---------------- |
| **Offset-Based** | 420                 | 76% ❌ | Slow ❌      | Good ✅      | No ❌             |
| **Cursor-Based** | 100                 | 0% ✅  | Fast ✅      | Poor ❌      | Risky ⚠️          |
| **Two-Stage**    | 1000 (once)         | 0% ✅  | Very Fast ✅ | Excellent ✅ | Yes ✅            |

---

## When to Use Each Strategy 🎯

### Use Offset-Based When:
- ❌ **Never recommended** for vector databases
- Only acceptable for tiny datasets (< 100 items)

### Use Cursor-Based When:
- ✅ Building social media feeds (Twitter, Instagram)
- ✅ Real-time data streams
- ✅ Infinite scroll interfaces
- ✅ Duplicates/skips are acceptable

### Use Two-Stage When:
- ✅ E-commerce search results
- ✅ Document search systems
- ✅ Any application requiring consistency
- ✅ **Production systems** (recommended!)

---

## Key Takeaways for Students 🎓

1. **Vector databases are different** from SQL databases - results can vary slightly
2. **Pagination is tricky** because we need consistent results across pages
3. **Offset-based pagination wastes resources** - fetches too much, uses too little
4. **Cursor-based pagination is fast but risky** - results might shift
5. **Two-stage pagination is the best** - fetch once, cache, then slice
6. **Caching is your friend** - makes subsequent pages instant
7. **Always consider the user experience** - consistency matters!

---

## Practice Exercise 💪

**Question:** A user searches for "gaming laptops" and you have 5000 results.

1. Using **offset-based pagination** (page_size=20), how many results would you fetch to show page 10?
   - **Answer:** 200 results (but only use 20!)

2. Using **two-stage pagination**, how many vector searches would you do to show pages 1, 5, and 10?
   - **Answer:** Just 1 vector search (fetch 1000 once, then slice)

3. Which strategy would you choose for an e-commerce website? Why?
   - **Answer:** Two-stage pagination - users expect consistent results when clicking "Next"

---

## Complete Production Code 💻

```python
import hashlib
from datetime import datetime, timedelta

class VectorSearchPagination:
    """
    Production-ready two-stage pagination for vector databases
    
    How it works:
    1. First request: Fetch large candidate set (e.g., 1000 results)
    2. Cache these results with a TTL (e.g., 10 minutes)
    3. Subsequent requests: Slice from cache (instant!)
    
    Benefits:
    - Consistent results across pages
    - Fast performance (cache hits are instant)
    - Efficient (only 1 vector search per query)
    """
    
    def __init__(self, client, cache_ttl_seconds=600):
        """
        Initialize the paginator
        
        Args:
            client: Vector database client (e.g., Qdrant)
            cache_ttl_seconds: How long to cache results (default: 10 minutes)
        """
        self.client = client
        self.cache = {}  # In production: use Redis or Memcached
        self.ttl = cache_ttl_seconds
    
    def _cache_key(self, user_id: str, query: str) -> str:
        """
        Generate a unique cache key for this search
        
        Format: search:{user_id}:{query_hash}
        Example: search:alice:a3f2b8c1
        """
        query_hash = hashlib.md5(query.encode()).hexdigest()[:8]
        return f"search:{user_id}:{query_hash}"
    
    def _is_cache_valid(self, cache_entry: dict) -> bool:
        """Check if cached data is still fresh (not expired)"""
        expires_at = cache_entry["expires_at"]
        return datetime.now() < expires_at
    
    def get_candidates(self, user_id: str, query: str, k: int = 1000):
        """
        STAGE 1: Fetch large candidate set from vector DB
        
        This is the expensive operation that queries the vector database.
        Results are cached so we only do this once per query.
        
        Args:
            user_id: Unique user identifier
            query: Search query text
            k: Number of candidates to fetch (default: 1000)
        
        Returns:
            List of search results with scores
        """
        cache_key = self._cache_key(user_id, query)
        
        # Try to get from cache first
        if cache_key in self.cache:
            entry = self.cache[cache_key]
            if self._is_cache_valid(entry):
                print(f"✅ Cache HIT: Serving {len(entry['data'])} results from cache")
                return entry["data"]
            else:
                print(f"⏰ Cache EXPIRED: Fetching fresh results")
        
        # Cache miss - query the vector database
        print(f"🔍 Cache MISS: Querying vector DB for {k} results...")
        results = self.client.search(
            collection_name="products",
            query_vector=embed(query),  # Convert text to vector embedding
            limit=k,
            with_payload=True
        )
        
        # Extract only the data we need (save memory)
        candidates = [
            {
                "id": r.id,
                "score": r.score,
                "title": r.payload.get("title"),
                "price": r.payload.get("price"),
                "image": r.payload.get("image_url")
            }
            for r in results
        ]
        
        # Store in cache with expiration time
        self.cache[cache_key] = {
            "data": candidates,
            "expires_at": datetime.now() + timedelta(seconds=self.ttl),
            "created_at": datetime.now()
        }
        
        print(f"💾 Cached {len(candidates)} results for {self.ttl} seconds")
        return candidates
    
    def paginate(self, user_id: str, query: str, page: int, page_size: int = 20):
        """
        STAGE 2: Slice cached results to get a specific page
        
        This is fast because we're just slicing an in-memory list.
        No vector computations needed!
        
        Args:
            user_id: Unique user identifier
            query: Search query text
            page: Page number (1-indexed)
            page_size: Number of results per page
        
        Returns:
            Dictionary with paginated results and metadata
        """
        # Get full candidate list (from cache if available)
        candidates = self.get_candidates(user_id, query)
        
        # Calculate array slice indices
        start = (page - 1) * page_size
        end = start + page_size
        
        # Slice the list (instant operation!)
        page_results = candidates[start:end]
        
        # Calculate pagination metadata
        total_results = len(candidates)
        total_pages = (total_results + page_size - 1) // page_size  # Ceiling division
        
        return {
            "results": page_results,
            "page": page,
            "page_size": page_size,
            "total": total_results,
            "total_pages": total_pages,
            "has_next": end < total_results,
            "has_prev": page > 1
        }

# ─────────────────────────────────────────────────────────────
# Example Usage
# ─────────────────────────────────────────────────────────────

# Initialize the paginator
paginator = VectorSearchPagination(client, cache_ttl_seconds=600)

# User "alice" searches for "wireless headphones"
print("=" * 60)
print("User: alice | Query: 'wireless headphones'")
print("=" * 60)

# Page 1
page1 = paginator.paginate("alice", "wireless headphones", page=1)
# Output: 🔍 Cache MISS: Querying vector DB for 1000 results...
#         💾 Cached 1000 results for 600 seconds
print(f"Page 1: {len(page1['results'])} results | Total: {page1['total']}")

# Page 2 (instant - from cache!)
page2 = paginator.paginate("alice", "wireless headphones", page=2)
# Output: ✅ Cache HIT: Serving 1000 results from cache
print(f"Page 2: {len(page2['results'])} results")

# Page 3 (instant - from cache!)
page3 = paginator.paginate("alice", "wireless headphones", page=3)
# Output: ✅ Cache HIT: Serving 1000 results from cache
print(f"Page 3: {len(page3['results'])} results")

# Different user, same query (separate cache)
print("\n" + "=" * 60)
print("User: bob | Query: 'wireless headphones'")
print("=" * 60)

page1_bob = paginator.paginate("bob", "wireless headphones", page=1)
# Output: 🔍 Cache MISS: Querying vector DB for 1000 results...
#         💾 Cached 1000 results for 600 seconds
print(f"Page 1: {len(page1_bob['results'])} results")
```

---

## Additional Resources 📚

- **Vector Databases:** Qdrant, Pinecone, Weaviate
- **Caching:** Redis, Memcached
- **Embeddings:** OpenAI, Sentence Transformers
- **Related Topics:** Semantic search, RAG (Retrieval-Augmented Generation)
