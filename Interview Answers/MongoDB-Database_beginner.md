# MongoDB & Database Design - Beginner Level Answers

---

## 1. What is MongoDB and how does it differ from SQL databases?

**MongoDB:** NoSQL document database storing data in flexible JSON-like documents (BSON).

**Key differences:**
- **Structure:** Documents vs tables/rows
- **Schema:** Flexible vs fixed
- **Relationships:** Embedded documents vs joins
- **Scaling:** Horizontal (sharding) vs vertical primarily
- **Queries:** MongoDB query language vs SQL
- **Transactions:** Limited (improving) vs full ACID

**When to use MongoDB:**
- Flexible/evolving schema
- Rapid development
- Horizontal scaling needs
- Document-oriented data

---

## 2. Explain the structure of a MongoDB document

**Document:** BSON (Binary JSON) object with key-value pairs.

**Structure:**
- **_id:** Unique identifier (auto-generated ObjectId)
- **Fields:** Key-value pairs
- **Values:** Strings, numbers, booleans, dates, arrays, embedded documents, null

**Characteristics:**
- Maximum size: 16MB
- Schema-less (documents in same collection can differ)
- Rich data types
- Nested structures supported

---

## 3. What is a collection in MongoDB?

**Collection:** Group of MongoDB documents (similar to SQL table).

**Characteristics:**
- No fixed schema
- Documents can have different fields
- Named with strings
- Created implicitly on first insert
- Can have indexes

**Namespace:** `database.collection`

---

## 4. What are the basic CRUD operations in MongoDB?

**CREATE:**
- `insertOne()`: Single document
- `insertMany()`: Multiple documents

**READ:**
- `find()`: Query documents (returns cursor)
- `findOne()`: Single document

**UPDATE:**
- `updateOne()`: First match
- `updateMany()`: All matches
- `replaceOne()`: Replace entire document

**DELETE:**
- `deleteOne()`: First match
- `deleteMany()`: All matches

---

## 5. What is the `_id` field in MongoDB?

**_id:** Primary key, unique identifier for each document.

**Characteristics:**
- Automatically created if not provided
- Immutable
- Must be unique within collection
- Automatically indexed
- Can be any BSON type (usually ObjectId)

**ObjectId:**
- 12-byte identifier
- Contains timestamp (creation time)
- Globally unique
- Sortable by creation time

---

## 6. Explain the difference between `find()` and `findOne()`

**find():**
- Returns cursor to all matching documents
- Can iterate through results
- Returns empty cursor if no matches
- Supports skip, limit, sort

**findOne():**
- Returns single document
- Returns null if no match
- Faster for single result
- No cursor overhead

**Use find() when:** Need multiple documents or pagination
**Use findOne() when:** Need single document by unique field

---

## 7. What is an index in MongoDB and why is it important?

**Index:** Data structure that improves query performance.

**How it works:**
- Creates pointers to documents
- Speeds up lookups
- Similar to book index

**Benefits:**
- Faster queries
- Efficient sorting
- Unique constraints

**Trade-offs:**
- Slower writes
- Storage overhead
- Memory usage

**Default:** _id field automatically indexed

---

## 8. How do you update a document in MongoDB?

**Update operators:**
- **$set:** Set field value
- **$unset:** Remove field
- **$inc:** Increment number
- **$push:** Add to array
- **$pull:** Remove from array
- **$rename:** Rename field

**Methods:**
- `updateOne()`: First match
- `updateMany()`: All matches
- `replaceOne()`: Replace entire document
- `findOneAndUpdate()`: Return updated document

---

## 9. What is the aggregation framework?

**Aggregation:** Pipeline for data processing and transformation.

**Purpose:**
- Complex queries
- Data analysis
- Reporting
- Transformations

**Pipeline stages:**
- Process documents sequentially
- Each stage transforms data
- Output of one stage feeds next

**Common operations:**
- Filtering
- Grouping
- Sorting
- Computing values
- Joining collections

---

## 10. Explain the purpose of MongoDB drivers (like Mongoose)

**MongoDB driver:** Library connecting Node.js to MongoDB.

**Native driver:**
- Official MongoDB driver
- Low-level API
- Direct MongoDB commands
- Lightweight

**Mongoose (ODM):**
- Schema definition
- Validation
- Middleware (hooks)
- Relationships
- Type casting
- Higher-level abstraction

**Choose native for:** Performance, flexibility
**Choose Mongoose for:** Structure, validation, complex apps

---

## 11. What are the advantages of using MongoDB for a SaaS application?

**Advantages:**

**1. Flexible schema:**
- Evolve features without migrations
- Different customer configurations

**2. Horizontal scaling:**
- Sharding for growth
- Handle increasing data/traffic

**3. Fast development:**
- No schema changes
- Rapid iteration

**4. Document model:**
- Matches JSON APIs
- Natural for JavaScript/Node.js

**5. Multi-tenancy:**
- Flexible tenant isolation strategies
- Per-tenant customization

---

## 12. How do you delete documents in MongoDB?

**Methods:**

**deleteOne(filter):**
- Deletes first matching document
- Returns deletion count

**deleteMany(filter):**
- Deletes all matching documents
- Returns deletion count

**findOneAndDelete(filter):**
- Deletes and returns document
- Useful when you need the deleted data

**Warning:** No `delete()` without filter (must use `deleteMany({})` to delete all).

---

## 13. What is a replica set?

**Replica set:** Group of MongoDB servers maintaining identical data.

**Components:**
- **Primary:** Receives all writes
- **Secondaries:** Replicate primary data
- **Arbiter:** Voting only (no data)

**Benefits:**
- **High availability:** Automatic failover
- **Data redundancy:** Multiple copies
- **Read scalability:** Read from secondaries
- **Zero downtime:** Maintenance without outage

**Failover:** If primary fails, secondary is elected.

---

## 14. Explain what sharding is in MongoDB

**Sharding:** Horizontal partitioning of data across multiple servers.

**Purpose:**
- Scale beyond single server
- Distribute data and load
- Handle large datasets

**Components:**
- **Shards:** Store data subsets
- **Mongos:** Query router
- **Config servers:** Store metadata

**Shard key:** Determines how data is distributed.

**Use when:** Data exceeds single server capacity.

---

## 15. What are the data types supported by MongoDB?

**Common types:**
- **String:** Text data
- **Number:** Integer (32/64-bit), Double
- **Boolean:** true/false
- **Date:** ISODate
- **ObjectId:** Unique identifier
- **Array:** List of values
- **Object:** Embedded document
- **Null:** Absence of value

**Special types:**
- **Binary:** Binary data
- **RegExp:** Regular expressions
- **JavaScript:** Code
- **Timestamp:** Internal MongoDB timestamp
- **Decimal128:** High-precision decimal
- **MinKey/MaxKey:** Comparison values
