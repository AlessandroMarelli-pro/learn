# MongoDB & Database Design - Intermediate Level Answers

---

## 1. Explain embedding vs referencing in MongoDB schema design

**Embedding (Denormalization):**
- Store related data in single document
- Nested objects or arrays
- One query to get all data

**When to embed:**
- One-to-few relationships
- Data accessed together
- No frequent updates
- Need atomic operations

**Referencing (Normalization):**
- Store relationships using document IDs
- Separate collections
- Multiple queries or $lookup

**When to reference:**
- One-to-many or many-to-many
- Large subdocuments
- Frequently updated data
- Need to query independently

**Hybrid:** Embed frequently accessed fields, reference full data.

---

## 2. How would you design a schema for a multi-tenant SaaS application?

**Approach 1: Shared collection with tenant_id**
- Add tenant_id to every document
- Single database, single collection
- Filter all queries by tenant_id
- Pros: Simple, cost-effective
- Cons: Data leakage risk, harder per-tenant scaling

**Approach 2: Database per tenant**
- Separate database for each tenant
- Strong isolation
- Pros: Security, easy backup/restore per tenant
- Cons: More complex management, resource overhead

**Approach 3: Collection per tenant**
- Dynamic collection names
- Moderate isolation
- Pros: Better than shared, some isolation
- Cons: Management complexity

**Best practice:** Shared collection with strict tenant_id filtering for most SaaS, database per tenant for enterprise/high-value customers.

---

## 3. What are compound indexes and when should you use them?

**Compound index:** Index on multiple fields together.

**Order matters:**
- Can use for queries on prefix fields
- Cannot use for non-prefix fields alone

**ESR Rule (Equality, Sort, Range):**
- Equality filters first
- Sort fields second
- Range filters last
- Optimizes query performance

**Use cases:**
- Frequently queried field combinations
- Queries with sort
- Covering indexes (all query fields in index)

**Trade-off:** More specific, less flexible than separate indexes.

---

## 4. Explain the aggregation pipeline and its common stages

**Aggregation pipeline:** Sequence of stages processing documents.

**Common stages:**

**$match:** Filter documents (like WHERE)
**$group:** Group by field and aggregate (like GROUP BY)
**$project:** Select/compute fields (like SELECT)
**$sort:** Order results (like ORDER BY)
**$limit:** Limit results
**$skip:** Skip documents (pagination)
**$lookup:** Join collections (like JOIN)
**$unwind:** Deconstruct arrays
**$addFields:** Add computed fields

**Pipeline flow:** Documents pass through stages sequentially, each transforming data.

**Performance:** Put $match early to reduce documents processed.

---

## 5. How do you implement full-text search in MongoDB?

**Text indexes:**
- Create with type "text"
- Index string fields
- Supports multiple fields

**Search operator:**
- `$text` query operator
- Tokenizes and searches
- Case-insensitive

**Features:**
- Phrase search with quotes
- Exclude terms with minus
- Score results by relevance
- Language-specific stemming

**Limitations:**
- One text index per collection
- No partial word matching
- Limited customization

**Better alternative:** Elasticsearch for complex search, keep MongoDB for data storage, sync via change streams.

---

## 6. What are the best practices for indexing strategies?

**Best practices:**

**1. Index frequently queried fields**
**2. Use compound indexes for multi-field queries**
**3. Follow ESR rule (Equality, Sort, Range)**
**4. Create covering indexes when possible**
**5. Don't over-index (slows writes)**
**6. Monitor with explain()**
**7. Remove unused indexes**
**8. Consider index selectivity**

**Monitoring:**
- Use `explain("executionStats")`
- Check index usage
- Monitor query performance
- Identify slow queries

---

## 7. Explain the difference between `update()`, `updateOne()`, and `updateMany()`

**updateOne(filter, update):**
- Updates first matching document
- Returns match/modified count
- Recommended for single updates

**updateMany(filter, update):**
- Updates all matching documents
- Returns match/modified count
- Use when bulk updating

**update() [deprecated]:**
- Legacy method
- Behavior depends on options
- Use updateOne/updateMany instead

**findOneAndUpdate():**
- Updates and returns document
- Can return before or after update
- Atomic operation

---

## 8. How do you implement pagination efficiently in MongoDB?

**Method 1: Skip/Limit (traditional)**
- Simple but slow for large offsets
- Skip scans all skipped documents
- Use only for small datasets

**Method 2: Range queries (cursor-based)**
- Query by _id or indexed field
- More efficient for large datasets
- No skipped documents
- Recommended approach

**Method 3: Bucket pattern**
- Pre-aggregate data into pages
- Very fast but less flexible

**Best practice:** Use range queries with proper indexing for scalability.

---

## 9. What is the purpose of `$lookup` in aggregation?

**$lookup:** Performs left outer join between collections.

**Purpose:**
- Combine data from multiple collections
- Simulate SQL JOIN
- Denormalize on read

**When to use:**
- Referenced data needed together
- Reporting/analytics
- Ad-hoc queries

**When NOT to use:**
- High-frequency queries (prefer embedding)
- Performance-critical paths
- Large result sets

**Alternative:** Embed frequently accessed data, use $lookup for occasional needs.

---

## 10. How would you handle database migrations in MongoDB?

**Strategies:**

**1. Application-level:**
- Handle multiple schema versions in code
- Gradual migration during reads/writes
- No downtime

**2. Migration scripts:**
- Tools like migrate-mongo
- Version tracking
- Batch updates

**3. Schema versioning:**
- Add schema_version field
- Migrate on read
- Background migration job

**Best practices:**
- Test migrations on copy
- Backup before migration
- Monitor migration progress
- Handle rollback scenario

---

## 11. Explain the concept of write concerns and read preferences

**Write Concern:** Acknowledgment level for write operations.

**Levels:**
- **w: 0:** No acknowledgment (fire and forget)
- **w: 1:** Acknowledged by primary (default)
- **w: "majority":** Acknowledged by majority (safest)

**Trade-off:** Safety vs performance.

**Read Preference:** Which replica set member to read from.

**Options:**
- **primary:** Always primary (default, consistent)
- **primaryPreferred:** Primary if available
- **secondary:** Read from secondary (may be stale)
- **secondaryPreferred:** Secondary if available
- **nearest:** Lowest latency

---

## 12. How do you implement transactions in MongoDB?

**Requirements:**
- Replica set or sharded cluster
- MongoDB 4.0+

**Purpose:**
- Multi-document ACID operations
- Ensure data consistency
- All-or-nothing execution

**Usage:**
- Start session
- Start transaction
- Perform operations
- Commit or abort

**Best practices:**
- Keep transactions short
- Use only when necessary
- Handle transaction errors
- Consider retry logic

**Alternative:** Design schema to avoid transactions (embed related data).

---

## 13. What are MongoDB validators and how do they work?

**Validators:** Schema validation rules enforced by MongoDB.

**Types:**
- **JSON Schema:** Standard validation
- **Query operators:** $exists, $type, etc.

**Validation levels:**
- **strict:** Reject invalid documents (default)
- **moderate:** Apply to new/updated documents only

**Validation actions:**
- **error:** Reject writes (default)
- **warn:** Log warning but allow

**Use cases:**
- Enforce data quality
- Required fields
- Data type constraints
- Value ranges

**Note:** Mongoose provides schema validation at application level.

---

## 14. How would you optimize slow queries in MongoDB?

**Steps:**

**1. Identify slow queries:**
- Enable profiling
- Check logs
- Monitor query times

**2. Use explain():**
- Analyze execution plan
- Check if indexes used
- Look for COLLSCAN (bad)

**3. Create appropriate indexes:**
- Index queried fields
- Use compound indexes
- Follow ESR rule

**4. Optimize query:**
- Use projection (select only needed fields)
- Limit results
- Avoid $where operator

**5. Schema optimization:**
- Consider embedding
- Denormalize frequently joined data

**6. Monitor:**
- Track improvements
- Check index usage
- Remove unused indexes

---

## 15. Explain the difference between Mongoose schemas and models

**Schema:** Defines structure, validation, defaults, virtuals, methods.
- Blueprint for documents
- Not directly used for queries
- Can have plugins

**Model:** Compiled version of schema, used for querying.
- Constructor for documents
- Provides query methods
- Interfaces with MongoDB

**Relationship:**
- Schema defines structure
- Model = mongoose.model('Name', schema)
- Model uses schema for operations

**Usage:**
- Define schema once
- Create model from schema
- Use model for CRUD operations
