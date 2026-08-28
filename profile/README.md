## Replace your entire recommendation pipeline with one query.

RecQL is a database-agnostic query language for recommendations, personalization, and ranked retrieval. It gives you one way to express candidate retrieval, filtering, scoring, ranking, and reordering, without wiring your application logic to a specific database or search engine. It even allows you to mix-and-match data across systems.

```sql
SELECT score(expression='0.6 * click_rate + 0.4 * similarity') AS s,
       diversity(score=s, strength=0.3) AS ranked, *
FROM retrieve(
  similarity(embedding_ref='als', limit=100),
  text_search(query='$query', mode='lexical', limit=100)
)
WHERE in_stock = true
ORDER BY ranked
LIMIT 20
```

One query - hybrid retrieval, model scoring, and diversity reranking - that runs against your database of choice. No new pipeline to build. No new infrastructure. No rewrite when you switch backends.

## The Problem

Recommendation systems increasingly depend on data spread across operational databases, warehouses, and analytical systems.

Each one exposes search and ranking differently:

* SQL filters and ordering
* Full-text and lexical search
* Vector similarity and approximate nearest neighbor (ANN) search
* Database-specific scoring and ranking functions

So teams end up rebuilding the same retrieval pipeline by hand for every backend they touch - hundreds of lines of glue code and logic to express what is, at heart, a single query.

## How RecQL Solves It

RecQL defines a portable retrieval and ranking model that compiles to the native full-text and vector search capabilities of your database.

A single RecQL query expresses a full four-stage recommendation pipeline:

**Retrieve → Filter → Score → Reorder**

and combines multiple retrieval strategies, including:

* Structured queries
* Lexical search
* Vector similarity
* Popularity and trending
* Model-generated scores
* Hybrid and rank-fusion retrieval

Database adapters translate RecQL into the appropriate SQL, query API, or native search primitives - so the same query runs on Postgres today and Snowflake tomorrow, without a rewrite.

## What You Can Build

Every one of these is the same four-stage query - **retrieve → filter → score → reorder** - with different pieces swapped in. No new service per use case. No new database.

**A personalized "For You" feed:**

```sql
SELECT score(expression='click_rate', input_user_id='$user_id') AS s,
       diversity(score=s, strength=0.3) AS feed, *
FROM retrieve(similarity(embedding_ref='als', encoder='interaction_pooling',
                         input_user_id='$user_id', limit=200))
ORDER BY feed LIMIT 20
```

**Hybrid search (lexical + semantic, fused):**

```sql
SELECT score(expression='0.5 * retrieval.semantic + 0.5 * retrieval.lexical') AS s, *
FROM retrieve(
  text_search(query='$q', mode='vector',  limit=100, name='semantic'),
  text_search(query='$q', mode='lexical', limit=100, name='lexical')
)
ORDER BY s LIMIT 20
```

**Reranking an external candidate set with diversity:**

```sql
SELECT score(expression='click_rate', input_user_id='$user_id') AS s,
       diversity(score=s, strength=0.4) AS r, *
FROM retrieve(ids($candidate_ids))
ORDER BY r LIMIT 10
```

Every example below runs live in the [playground](https://github.com/recql/recql-playground/tree/main/examples) against MovieLens.

### Recommendations

| Use case | The pattern it powers |
| --- | --- |
| [Similar items](https://github.com/recql/recql-playground/tree/main/examples/similar_items) | "More like this" - Netflix-style item-to-item |
| [Similar users](https://github.com/recql/recql-playground/tree/main/examples/similar_users) | Lookalike audiences, user-to-user matching |
| [Complement items](https://github.com/recql/recql-playground/tree/main/examples/complement_items) | "Frequently bought together" |

### Search

| Use case | The pattern it powers |
| --- | --- |
| [Lexical](https://github.com/recql/recql-playground/tree/main/examples/search/lexical) | Classic keyword / BM25 search |
| [Semantic](https://github.com/recql/recql-playground/tree/main/examples/search/semantic) | Vector search over embeddings |
| [Hybrid](https://github.com/recql/recql-playground/tree/main/examples/search/hybrid) | Lexical + semantic, fused in one query |
| [Personalized](https://github.com/recql/recql-playground/tree/main/examples/search/personalized) | Search results ranked to the user |

### Feeds

| Use case | The pattern it powers |
| --- | --- |
| [For You](https://github.com/recql/recql-playground/tree/main/examples/feeds/for_you) | Personalized home feed |
| [Popular](https://github.com/recql/recql-playground/tree/main/examples/feeds/popular) | Most-engaged, right now |
| [New](https://github.com/recql/recql-playground/tree/main/examples/feeds/new) | Freshest items first |
| [Trending](https://github.com/recql/recql-playground/tree/main/examples/feeds/trending) | Popularity with time decay |

### Ranking & Control

| Use case | The pattern it powers |
| --- | --- |
| [Reranking](https://github.com/recql/recql-playground/tree/main/examples/reranking) | Reorder candidates from any source |
| [Boosted](https://github.com/recql/recql-playground/tree/main/examples/boosted) | Promote items by business rules |
| [Faceted filtering](https://github.com/recql/recql-playground/tree/main/examples/faceted_filtering) | Filter by attributes and facets |
| [Filter bubbles](https://github.com/recql/recql-playground/tree/main/examples/filter_bubbles) | Inject diversity, break narrow recs |
| [Pagination](https://github.com/recql/recql-playground/tree/main/examples/pagination) | Stable paging across requests |

## History

RecQL originated as ShapedQL, the recommendation query language behind the [Shaped real-time retrieval engine](https://www.ycombinator.com/companies/shaped), where it was developed to express declarative search, recommendation, and personalization independently of the underlying data system. As database internals and personalization system experts, we loved the concept, and later adopted it for our own [NEXTGRES](https://nextgres.ai) SIDE (Signal, Intelligence, Decision, Experience) engine.

Now, it is now being developed as a standalone specification and ecosystem for portable recommendation queries!

## Database Support

RecQL is designed to support both transactional and analytical databases.

| Backend                    | Structured | Lexical | Vector |
| -------------------------- | :--------: | :-----: | :----: |
| PostgreSQL                 |      ✓     |    ✓    |    ✓   |
| MariaDB                    |      ✓     |    ✓    |    ✓   |
| Oracle Database            |      ✓     |    ✓    |    ✓   |
| MySQL                      |      ✓     |    ✓    |    ◐   |
| SQL Server / Azure SQL     |      ✓     |    ✓    |    ✓   |
| MongoDB (Atlas/Community)  |      ✓     |    ✓    |    ✓   |
| ClickHouse                 |      ✓     |    ◐    |    ✓   |
| SingleStore                |      ✓     |    ✓    |    ✓   |
| Snowflake                  |      ✓     |    ✓    |    ✓   |
| Databricks                 |      ✓     |    ✓    |    ✓   |
| BigQuery                   |      ✓     |    ◐    |    ✓   |
| Elasticsearch / OpenSearch |      ✓     |    ✓    |    ✓   |
| Pinecone                   |      ✓     |    ✓    |    ✓   |
| Qdrant                     |      ✓     |    ✓    |    ✓   |
| Weaviate                   |      ✓     |    ✓    |    ✓   |
| Milvus / Zilliz            |      ✓     |    ✓    |    ✓   |

**✓** Native support · **◐** Partial or deployment-dependent

> Note: Capabilities and ranking semantics vary by database. RecQL exposes each backend's capabilities rather than requiring every system to implement identical search algorithms.

## Status

RecQL is under active development as an open specification. The language, adapters, and reference tooling are evolving in the open - contributions, adapter implementations, and real-world use cases are welcome.

## License

MIT

