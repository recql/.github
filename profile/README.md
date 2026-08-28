# RecQL

**Write recommendation logic once. Run it where the data already lives.**

RecQL is a database-agnostic query language for recommendations, personalization, and ranked retrieval. It gives you one way to express candidate retrieval, filtering, scoring, ranking, and reordering, without wiring your application logic to a specific database or search engine.

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

One query - hybrid retrieval, model scoring, and diversity reranking - that runs against your database of choice.

## The Problem

Recommendation systems increasingly depend on data spread across operational databases, warehouses, and analytical systems.

Each one exposes search and ranking differently:

* SQL filters and ordering
* Full-text and lexical search
* Vector similarity and approximate nearest neighbor (ANN) search
* Database-specific scoring and ranking functions

So teams end up rebuilding the same retrieval pipeline by hand for every backend they touch - hundreds of lines of glue code to express what is, at heart, a single query.

## How RecQL Solves It

RecQL defines a portable retrieval and ranking model that compiles to the native full-text and vector search capabilities of each database.

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

