# MongoDB & Database Design - Advanced Level Answers

---

## 1. How would you design a MongoDB schema for optimal query performance in a B2B SaaS with multiple products?

**Design approach:**

**Tenant isolation:**
- tenant_id in every document
- Compound indexes starting with tenant_id
- Ensures data segregation

**Product separation:**
- Separate collections per product OR
- Product-specific databases OR
- Type field with product identifier

**Embedded vs referenced:**
- Embed frequently accessed metadata
- Reference large or shared data
- Denormalize for read performance

**Indexing strategy:**
- tenant_id + query fields compound indexes
- Product-specific indexes
- TTL indexes for temporary data

**Sharding consideration:**
- Shard key: tenant_id (hashed) for distribution
- Or compound: tenant_id + product_id

**Best practice:** Balance isolation, performance, and operational complexity based on tenant size and query patterns.

---

## 2. Explain the trade-offs between different indexing strategies (B-tree, text, geospatial)

**B-tree (default):**
- **Pros:** General-purpose, efficient range queries, sorted results
- **Cons:** Memory overhead, slower writes
- **Use:** Most queries, equality and range

**Text indexes:**
- **Pros:** Full-text search, relevance scoring
- **Cons:** One per collection, large size, limited features
- **Use:** Simple search, small datasets

**Geospatial (2d, 2dsphere):**
- **Pros:** Location queries, proximity search
- **Cons:** Specific use case only
- **Use:** Location-based features

**Hashed indexes:**
- **Pros:** Even distribution for sharding
- **Cons:** No range queries, only equality
- **Use:** Shard keys

**Trade-offs:** Storage space, write performance, query capabilities, memory usage.

---

## 3. How would you implement a scalable audit log system in MongoDB?

**Design considerations:**

**1. Separate collection:**
- Dedicated audit_logs collection
- Don't mix with business data
- Easy to archive/purge

**2. TTL index:**
- Auto-delete old logs (e.g., 90 days)
- Comply with retention policies
- Save storage

**3. Capped collection (alternative):**
- Fixed size, circular buffer
- High-throughput writes
- Automatic old data removal

**4. Minimal indexes:**
- Only essential queries indexed
- Write performance critical

**5. Batch writes:**
- Buffer and write in batches
- Reduce database load

**6. Schema:**
- timestamp, user_id, action, resource_type, resource_id, changes, metadata

**Scaling:** Archive to data warehouse for long-term analysis, keep recent in MongoDB.

---

## 4. What are the considerations for implementing multi-document transactions?

**When to use:**
- Financial operations
- Inventory management
- Critical consistency requirements

**Performance impact:**
- Slower than single operations
- Locks resources
- Can cause contention

**Best practices:**
- Keep transactions short
- Limit operations per transaction
- Use only when necessary
- Implement retry logic

**Alternatives:**
- Redesign schema to avoid (embed related data)
- Two-phase commit at application level
- Eventual consistency patterns

**Requirements:**
- Replica set or sharded cluster
- Proper error handling
- Transaction timeout handling

---

## 5. How would you optimize MongoDB performance for high write throughput?

**Strategies:**

**1. Minimize indexes:**
- Each index slows writes
- Keep only essential indexes

**2. Bulk operations:**
- Use insertMany, bulkWrite
- Reduce round trips

**3. Unacknowledged writes:**
- w: 0 (if acceptable)
- Fire and forget

**4. Sharding:**
- Distribute writes across shards
- Good shard key selection

**5. Hardware:**
- SSD storage
- More RAM
- Dedicated storage

**6. WiredTiger configuration:**
- Increase cache size
- Tune checkpoint intervals

**7. Avoid:**
- Complex validation
- Pre/post hooks
- Large documents

---

## 6. Explain the internals of MongoDB's WiredTiger storage engine

**WiredTiger features:**

**1. Document-level locking:**
- Fine-grained concurrency
- Better than collection-level
- Higher throughput

**2. Compression:**
- Snappy (default), zlib, zstd
- Reduces storage
- Trade-off: CPU vs space

**3. Checkpoints:**
- Periodic snapshots to disk
- Recovery point
- Default: 60 seconds

**4. Journaling:**
- Write-ahead log
- Durability
- Recovery from crashes

**5. Cache:**
- In-memory cache for frequently accessed data
- Default: 50% of RAM - 1GB
- Working set should fit in cache

**Performance:** Designed for SSD, concurrent workloads.

---

## 7. How would you implement time-series data storage in MongoDB?

**MongoDB 5.0+ Time-series collections:**
- Optimized for time-stamped data
- Better compression
- Efficient queries

**Bucket pattern (pre-5.0):**
- Group measurements into documents
- One document per time bucket (hour, day)
- Array of measurements
- Reduces document count

**Schema example:**
- timestamp, sensor_id, bucket_start, measurements array

**Indexing:**
- Compound index on (sensor_id, timestamp)
- TTL index for auto-expiration

**Querying:**
- Aggregation for time-range queries
- Efficient for recent data

**Archival:** Move old data to data warehouse.

---

## 8. What strategies would you use for handling very large documents (near the 16MB limit)?

**Approaches:**

**1. GridFS:**
- Store files > 16MB
- Chunks into smaller pieces
- Built-in MongoDB feature

**2. Split into multiple documents:**
- One main document
- Related documents for large data
- Reference by ID

**3. External storage:**
- Store large blobs (S3, Azure Blob)
- Keep reference in MongoDB

**4. Compression:**
- Compress data before storing
- Store as Binary type

**5. Redesign:**
- Question if you need all data together
- Separate frequently vs rarely accessed

**Prevention:** Monitor document sizes, set limits, paginate large arrays.

---

## 9. How would you design a MongoDB sharding key for a multi-tenant application?

**Shard key requirements:**
- High cardinality
- Even distribution
- Query locality

**Options:**

**1. Hashed tenant_id:**
- Even distribution
- Good for many small tenants
- Cons: Scatter-gather for tenant queries

**2. Tenant_id (not hashed):**
- Query locality (single shard)
- Cons: Uneven if tenant sizes vary
- Risk of hotspots

**3. Compound key (tenant_id + date/id):**
- Balance distribution and locality
- Best for time-based data

**Considerations:**
- Tenant size distribution
- Growth patterns
- Query patterns
- Hot tenants

**Best practice:** Hashed tenant_id for many similar-sized tenants, compound key for mixed sizes.

---

## 10. Explain how to implement complex analytics queries efficiently

**Strategies:**

**1. Aggregation framework:**
- Use for complex transformations
- Optimize with $match early
- Index support

**2. Materialized views:**
- Pre-compute aggregations
- Store results in collection
- Refresh periodically or on-demand

**3. Change streams:**
- Real-time aggregation updates
- React to data changes

**4. Read-only replicas:**
- Run analytics on secondary
- Don't impact primary

**5. Export to data warehouse:**
- For very complex analytics
- Periodic sync
- Use specialized tools (BigQuery, Redshift)

**Optimization:**
- Proper indexes
- Limit result sizes
- Use $project to reduce data
- Shard for parallel processing

---

## 11. How would you handle MongoDB backup and disaster recovery strategies?

**Backup strategies:**

**1. mongodump:**
- Logical backup
- Flexible restore
- Slower for large datasets

**2. Filesystem snapshots:**
- Fast, efficient
- Requires consistent state
- LVM, cloud snapshots

**3. Replica set backup:**
- Backup from secondary
- No primary impact

**4. Continuous backup:**
- MongoDB Atlas Ops Manager
- Point-in-time recovery
- Enterprise feature

**Disaster recovery:**
- **RTO (Recovery Time Objective):** How long to restore
- **RPO (Recovery Point Objective):** How much data loss acceptable

**Best practices:**
- Automate backups
- Test restores regularly
- Off-site storage
- Multiple backup methods
- Monitor backup success

---

## 12. What are the performance implications of different schema design patterns?

**Patterns:**

**1. Embedded documents:**
- **Pros:** Single query, atomic updates, good locality
- **Cons:** Document size growth, duplication
- **Use:** One-to-few, data accessed together

**2. Reference/linking:**
- **Pros:** No duplication, flexibility
- **Cons:** Multiple queries, no atomicity
- **Use:** One-to-many, many-to-many

**3. Subset pattern:**
- **Pros:** Frequently accessed data embedded, full data referenced
- **Cons:** Some duplication
- **Use:** Large documents, access patterns vary

**4. Computed pattern:**
- **Pros:** Pre-calculated results, fast reads
- **Cons:** Update overhead, potential staleness
- **Use:** Expensive calculations, analytics

**5. Bucket pattern:**
- **Pros:** Reduced document count, efficient updates
- **Cons:** Complexity, potential growth
- **Use:** Time-series, high-frequency updates

---

## 13. How would you implement a search engine feature with MongoDB and potentially Elasticsearch?

**Hybrid approach:**

**MongoDB:**
- Primary data store
- Source of truth
- Simple searches

**Elasticsearch:**
- Advanced search
- Full-text with features
- Faceted search, suggestions

**Synchronization:**

**1. Change streams (real-time):**
- MongoDB change streams
- Stream to Elasticsearch
- Near real-time sync

**2. Batch sync:**
- Periodic full sync
- Simple but stale data

**3. Application-level:**
- Write to both on update
- Risk of inconsistency

**Best practice:**
- Use change streams for sync
- MongoDB for data integrity
- Elasticsearch for search UX
- Fallback to MongoDB if ES down

---

## 14. Explain how to monitor and optimize MongoDB memory usage

**Memory components:**

**1. WiredTiger cache:**
- Default: 50% RAM - 1GB
- Most important for performance
- Monitor cache hit ratio

**2. Operating system cache:**
- File system cache
- Caches compressed data
- Complements WiredTiger

**3. Indexes:**
- Should fit in RAM
- Monitor index sizes

**Monitoring:**
- `serverStatus` command
- Cache statistics
- Page faults
- Working set size

**Optimization:**
- Increase RAM if needed
- Optimize indexes
- Reduce document size
- Use projections
- Archive old data

**Target:** Working set fits in RAM for best performance.

---

## 15. How would you design a schema versioning strategy for a rapidly evolving SaaS product?

**Strategies:**

**1. Version field:**
- Add schema_version to documents
- Handle versions in application code
- Migrate on read (lazy migration)

**2. Feature flags:**
- Boolean flags for features
- Gradual rollout
- Easy rollback

**3. Backward compatibility:**
- New fields optional
- Default values
- Don't remove old fields immediately

**4. Dual-write period:**
- Write to both old and new fields
- Transition period
- Remove old after migration

**5. Background migration:**
- Batch update old documents
- Low priority job
- Monitor progress

**Best practices:**
- Never breaking changes without migration
- Test with production data copy
- Monitor migration errors
- Have rollback plan
- Document schema changes
