Cheatsheet can be found here : https://www.geeksforgeeks.org/mongodb/mongodb-cheat-sheet/

# MongoDB Core Concepts - Interview Preparation

## Document-Oriented Database

### Simple Explanation

A document-oriented database stores data in documents (like JSON objects) instead of rows and columns like traditional databases. Think of it like storing complete filing folders instead of spreadsheet rows.

**Example:** Instead of spreading a user's information across multiple tables (users table, addresses table, orders table), you store everything about a user in one document:

```json
{
  "name": "John Doe",
  "email": "john@example.com",
  "address": {
    "street": "123 Main St",
    "city": "Paris"
  },
  "orders": [
    { "item": "laptop", "price": 1000 },
    { "item": "mouse", "price": 25 }
  ]
}
```

### Technical Explanation

MongoDB is a NoSQL database that stores data in BSON (Binary JSON) format. BSON extends JSON with additional data types like Date, ObjectId, Binary data, and others.

**Key characteristics:**

- **Self-contained documents:** Each document contains all related data, reducing the need for joins
- **BSON format:** Binary encoding provides efficient storage and traversal
- **Rich data types:** Supports nested documents, arrays, and complex data structures
- **Horizontal scalability:** Designed for distributed systems from the ground up

**BSON advantages over JSON:**

- Faster to traverse (includes length prefixes)
- More data types (Int32, Int64, Decimal128, Date, Binary)
- More space-efficient for certain data types
- Supports ordered keys

### Why It's Useful

- **Faster development:** Data structure matches your application objects
- **Flexibility:** Easy to add new fields without altering entire database
- **Performance:** Related data is stored together, reducing joins
- **Natural fit for modern applications:** APIs and web apps work with JSON natively

---

## Collections and Documents

### Simple Explanation

**Documents** are individual records (like a single JSON object). **Collections** are groups of documents (like a folder containing multiple files).

Think of it like:

- **SQL:** Table (collection) with rows (documents)
- **File system:** Folder (collection) with files (documents)

```
Database: MyApp
  ├── Collection: users
  │     ├── Document: {_id: 1, name: "Alice"}
  │     └── Document: {_id: 2, name: "Bob"}
  └── Collection: products
        ├── Document: {_id: 1, name: "Laptop"}
        └── Document: {_id: 2, name: "Phone"}
```

### Technical Explanation

#### Documents

- **Structure:** BSON objects with key-value pairs
- **Size limit:** Maximum 16MB per document
- **\_id field:** Every document must have a unique `_id` field (auto-generated ObjectId if not provided)
- **ObjectId:** 12-byte identifier: 4-byte timestamp + 5-byte random value + 3-byte counter

```javascript
{
  _id: ObjectId("507f1f77bcf86cd799439011"),
  name: "John Doe",
  age: 30,
  tags: ["developer", "mongodb"],
  address: {
    street: "123 Main St",
    city: "Paris"
  },
  createdAt: ISODate("2024-01-15T10:30:00Z")
}
```

#### Collections

- **Dynamic schema:** Documents in the same collection can have different fields
- **No enforced structure:** Unlike SQL tables, no predefined columns
- **Naming conventions:** Cannot contain `$`, null character, or start with `system.`
- **Namespaces:** Collections exist within databases (database.collection)

**Capped Collections:** Fixed-size collections that maintain insertion order and automatically remove oldest documents when full (useful for logs).

```javascript
db.createCollection('logs', { capped: true, size: 100000, max: 5000 });
```

---

## Schema-less / Flexible Schema

### Simple Explanation

Unlike SQL databases where you must define the structure (columns) before inserting data, MongoDB lets you insert documents with any structure. Each document in a collection can have completely different fields.

**Example:**

```javascript
// Document 1
{ name: "Alice", age: 25 }

// Document 2 - completely different structure
{ name: "Bob", email: "bob@example.com", skills: ["JS", "Python"] }

// Both can exist in the same collection!
```

It's like having a filing cabinet where each folder can contain different types of forms, rather than requiring all folders to have identical forms.

### Technical Explanation

#### Flexible Schema Benefits

1. **Rapid iteration:** Add fields without migrations
2. **Polymorphic data:** Store related but different entities in one collection
3. **Sparse data:** Only store fields that exist (no NULL columns)
4. **Evolutionary design:** Schema evolves with application needs

#### Schema Validation (Optional)

MongoDB 3.6+ supports optional schema validation using JSON Schema:

```javascript
db.createCollection('users', {
  validator: {
    $jsonSchema: {
      bsonType: 'object',
      required: ['name', 'email'],
      properties: {
        name: {
          bsonType: 'string',
          description: 'must be a string and is required',
        },
        email: {
          bsonType: 'string',
          pattern: '^.+@.+$',
          description: 'must be a valid email',
        },
        age: {
          bsonType: 'int',
          minimum: 0,
          maximum: 150,
        },
      },
    },
  },
  validationLevel: 'moderate', // or "strict"
  validationAction: 'error', // or "warn"
});
```

#### Trade-offs

**Advantages:**

- Agile development
- No downtime for schema changes
- Natural data modeling

**Disadvantages:**

- Potential data inconsistency without discipline
- Application-level schema management required
- Can lead to technical debt if not managed

**Best practice:** Use schema validation in production to enforce data quality while maintaining flexibility.

---

## Indexing

### Simple Explanation

An index is like a book's index - instead of reading every page to find a topic, you look at the index which points you directly to the right page.

Without an index, MongoDB must scan every document (collection scan) to find matches. With an index, it jumps directly to the relevant documents.

**Example:** Finding a user by email without an index requires checking every document. With an index on email, MongoDB goes directly to the matching document.

### Technical Explanation

#### Index Types

##### 1. Single Field Index

```javascript
db.users.createIndex({ email: 1 }); // 1 = ascending, -1 = descending
```

##### 2. Compound Index

```javascript
db.users.createIndex({ lastName: 1, firstName: 1 });
// Order matters! Good for queries on lastName or both, not just firstName
```

##### 3. Multikey Index

Automatically created when indexing array fields:

```javascript
db.products.createIndex({ tags: 1 });
// Works on: { tags: ["electronics", "sale"] }
```

##### 4. Text Index

For full-text search:

```javascript
db.articles.createIndex({ content: 'text', title: 'text' });
db.articles.find({ $text: { $search: 'mongodb tutorial' } });
```

##### 5. Geospatial Index

For location queries:

```javascript
db.places.createIndex({ location: '2dsphere' });
db.places.find({
  location: {
    $near: {
      $geometry: { type: 'Point', coordinates: [2.3522, 48.8566] },
      $maxDistance: 5000,
    },
  },
});
```

##### 6. Hashed Index

For hash-based sharding:

```javascript
db.users.createIndex({ userId: 'hashed' });
```

##### 7. Unique Index

Enforces uniqueness:

```javascript
db.users.createIndex({ email: 1 }, { unique: true });
```

##### 8. Partial Index

Index only documents that match filter:

```javascript
db.orders.createIndex(
  { status: 1, orderDate: 1 },
  { partialFilterExpression: { status: 'active' } },
);
```

##### 9. TTL Index

Automatically delete documents after time:

```javascript
db.sessions.createIndex(
  { createdAt: 1 },
  { expireAfterSeconds: 3600 }, // Delete after 1 hour
);
```

#### Index Internals

MongoDB uses **B-tree** data structures for indexes:

- O(log n) search complexity
- Balanced tree maintains performance
- Indexes stored separately from collection data

#### Index Performance Considerations

**Query Plans:**

```javascript
db.users.find({ email: 'john@example.com' }).explain('executionStats');
```

**Key metrics:**

- `IXSCAN`: Index scan (good)
- `COLLSCAN`: Collection scan (bad for large collections)
- `nReturned`: Documents returned
- `totalDocsExamined`: Documents examined (should be close to nReturned)
- `executionTimeMillis`: Query execution time

**Index Overhead:**

- **Write performance:** Every insert/update/delete must update indexes
- **Storage:** Each index consumes disk space
- **Memory:** Working set includes frequently used indexes

**Best practices:**

- Create indexes for frequent queries
- Use compound indexes to cover multiple query patterns
- Limit to 5-10 indexes per collection
- Monitor index usage: `db.collection.aggregate([{ $indexStats: {} }])`
- Drop unused indexes

**ESR Rule (Equality, Sort, Range):**
For compound indexes, order fields as:

1. Equality conditions
2. Sort conditions
3. Range conditions

```javascript
// Query: { status: "active", createdAt: { $gte: date } } .sort({ priority: -1 })
// Optimal index:
db.tasks.createIndex({ status: 1, priority: -1, createdAt: 1 });
```

---

## Replication

### Simple Explanation

Replication means keeping copies of your data on multiple servers. It's like having backup copies of important documents in different locations - if one burns down, you still have the others.

In MongoDB, you have one main server (primary) that handles all writes, and backup servers (secondaries) that copy everything the primary does. If the primary fails, one of the backups automatically becomes the new primary.

### Technical Explanation

#### Replica Set Architecture

A replica set is a group of MongoDB instances that maintain the same data set:

**Components:**

- **Primary:** Receives all write operations
- **Secondaries:** Replicate the primary's oplog and maintain copies of data
- **Arbiter (optional):** Participates in elections but doesn't hold data

```
         ┌─────────────┐
         │   Primary   │ ◄─── All writes go here
         └──────┬──────┘
                │
      ┌─────────┴─────────┐
      │                   │
      ▼                   ▼
┌───────────┐       ┌───────────┐
│Secondary 1│       │Secondary 2│ ◄─── Async replication
└───────────┘       └───────────┘
```

#### How Replication Works

##### Oplog (Operations Log)

- Special capped collection: `local.oplog.rs`
- Records all operations that modify data
- Idempotent operations (can be applied multiple times safely)
- Secondaries tail the primary's oplog and apply operations

```javascript
// Oplog entry example
{
  "ts": Timestamp(1699564800, 1),
  "t": NumberLong(1),
  "h": NumberLong("1234567890"),
  "v": 2,
  "op": "i", // insert
  "ns": "mydb.users",
  "o": { "_id": ObjectId("..."), "name": "Alice" }
}
```

**Operation types:**

- `i`: insert
- `u`: update
- `d`: delete
- `c`: command (e.g., create collection)
- `n`: no-op

#### Automatic Failover

**Election Process:**

1. Secondaries detect primary is unreachable (heartbeat timeout: 10s default)
2. Eligible secondaries call for election
3. Members vote based on:
   - Priority settings
   - Data freshness (oplog position)
   - Network connectivity
4. Candidate with majority votes becomes new primary
5. Typical failover time: 10-30 seconds

**Priority and Voting:**

```javascript
rs.initiate({
  _id: 'myReplSet',
  members: [
    { _id: 0, host: 'mongo1:27017', priority: 2 }, // Preferred primary
    { _id: 1, host: 'mongo2:27017', priority: 1 },
    { _id: 2, host: 'mongo3:27017', priority: 0 }, // Never becomes primary
    { _id: 3, host: 'mongo4:27017', arbiterOnly: true }, // Voting only
  ],
});
```

#### Read Preferences

**Read preference modes:**

1. **primary (default):** All reads from primary
2. **primaryPreferred:** Primary if available, else secondary
3. **secondary:** All reads from secondaries
4. **secondaryPreferred:** Secondary if available, else primary
5. **nearest:** Read from member with lowest network latency

```javascript
db.collection.find().readPref('secondary');
```

**Read Concern Levels:**

- **local:** Returns most recent data (might be rolled back)
- **available:** Same as local for replica sets
- **majority:** Returns data acknowledged by majority (cannot be rolled back)
- **linearizable:** Read your own writes guarantee
- **snapshot:** Read from consistent snapshot (transactions)

#### Write Concerns

Controls acknowledgment level for writes:

```javascript
db.collection.insertOne(
  { name: 'Alice' },
  { writeConcern: { w: 'majority', j: true, wtimeout: 5000 } },
);
```

**Parameters:**

- `w`: Number of acknowledgments required
  - `1`: Primary only (fast, less durable)
  - `"majority"`: Majority of nodes (durable)
  - `<number>`: Specific number of nodes
- `j`: Wait for journal commit (true = more durable)
- `wtimeout`: Max time to wait for write concern

#### Replication Lag

Time difference between primary and secondary oplog positions:

```javascript
rs.printSecondaryReplicationInfo();
```

**Causes:**

- Network latency
- Secondary under heavy read load
- Hardware differences
- Large/complex operations

**Monitoring:**

```javascript
db.serverStatus().repl;
```

---

## Sharding

### Simple Explanation

Sharding is splitting your data across multiple servers (shards) to handle more data than a single server can hold. It's like having multiple filing cabinets instead of one - when one gets full, you add another.

**Example:** You have 10 million users. Instead of storing all on one server:

- Server 1: Users A-H (4 million)
- Server 2: Users I-P (3 million)
- Server 3: Users Q-Z (3 million)

Each server only manages a portion of the data, making operations faster and allowing you to store more total data.

### Technical Explanation

#### Sharded Cluster Components

```
                    ┌──────────────┐
                    │    mongos    │ ◄─── Query router
                    │ (Query Router)│
                    └───────┬──────┘
                            │
        ┏━━━━━━━━━━━━━━━━━━━┻━━━━━━━━━━━━━━━━━━━┓
        ▼                   ▼                   ▼
  ┌──────────┐        ┌──────────┐        ┌──────────┐
  │ Shard 1  │        │ Shard 2  │        │ Shard 3  │
  │(Replica  │        │(Replica  │        │(Replica  │
  │  Set)    │        │  Set)    │        │  Set)    │
  └──────────┘        └──────────┘        └──────────┘
       ▲                    ▲                    ▲
       └────────────────────┴────────────────────┘
                            │
                  ┌─────────┴─────────┐
                  │   Config Servers  │ ◄─── Metadata storage
                  │  (Replica Set)    │
                  └───────────────────┘
```

**Components:**

1. **Shards:** Store subset of data (each shard is a replica set)
2. **Config servers:** Store metadata and cluster configuration (replica set of 3+)
3. **mongos:** Query routers that direct operations to appropriate shards

#### Shard Keys

The shard key determines how data is distributed:

```javascript
// Enable sharding on database
sh.enableSharding('mydb');

// Shard collection
sh.shardCollection('mydb.users', { userId: 1 });
```

**Shard key properties:**

- **Immutable:** Cannot change after document creation
- **Indexed:** Must have an index on shard key
- **Present in all documents:** Required field
- **Used for routing:** Determines which shard contains data

#### Shard Key Strategies

##### 1. Hashed Sharding

```javascript
sh.shardCollection('mydb.users', { userId: 'hashed' });
```

**Advantages:**

- Even distribution of data
- Good for random access patterns

**Disadvantages:**

- Range queries scatter across all shards
- Cannot use hashed prefix for compound shard keys

##### 2. Range-based Sharding

```javascript
sh.shardCollection('mydb.users', { country: 1, userId: 1 });
```

**Advantages:**

- Efficient range queries
- Related data co-located

**Disadvantages:**

- Risk of hotspots if data not evenly distributed
- Requires careful key selection

##### 3. Zoned Sharding

Assign specific ranges to specific shards:

```javascript
sh.addShardTag('shard0000', 'EU');
sh.addShardTag('shard0001', 'US');

sh.addTagRange(
  'mydb.users',
  { country: 'FR', userId: MinKey },
  { country: 'FR', userId: MaxKey },
  'EU',
);
```

#### Chunk Management

**Chunks:** Contiguous range of shard key values (default 128MB)

```
Shard 1: [MinKey, 1000], [1000, 2000]
Shard 2: [2000, 3000], [3000, 4000]
Shard 3: [4000, 5000], [5000, MaxKey]
```

**Balancer:** Background process that migrates chunks between shards to maintain balance

```javascript
// Check balancer status
sh.getBalancerState();

// Start/stop balancer
sh.startBalancer();
sh.stopBalancer();

// Set balancer window
db.settings.update(
  { _id: 'balancer' },
  { $set: { activeWindow: { start: '23:00', stop: '06:00' } } },
  { upsert: true },
);
```

#### Query Routing

**Targeted query (uses shard key):**

```javascript
db.users.find({ userId: 12345 });
// mongos routes to single shard
```

**Scatter-gather query (doesn't use shard key):**

```javascript
db.users.find({ email: 'john@example.com' });
// mongos queries all shards and merges results
```

**Query analysis:**

```javascript
db.users.find({ userId: 12345 }).explain();
```

#### Choosing a Good Shard Key

**Characteristics of ideal shard key:**

1. **High cardinality:** Many distinct values
2. **Even distribution:** No hotspots
3. **Query isolation:** Queries hit single shard when possible
4. **Write distribution:** Writes spread across shards

**Bad shard keys:**

- **Monotonically increasing:** `{ _id: 1 }` - all new writes go to one shard
- **Low cardinality:** `{ country: 1 }` with few countries - uneven distribution
- **Frequently updated:** `{ status: 1 }` - causes excessive chunk migrations

**Good shard keys:**

- **Hashed \_id:** `{ _id: "hashed" }` - even distribution, random writes
- **Compound with high cardinality:** `{ tenantId: 1, timestamp: 1 }`
- **Application-specific:** Depends on access patterns

#### Limitations and Considerations

**Limitations:**

- Cannot unshard a collection (must create new collection)
- Shard key cannot exceed 512 bytes
- Unique indexes must include shard key
- Some operations require scatter-gather (slower)

**Operational overhead:**

- More complex deployment and monitoring
- Network latency between components
- Chunk migration can impact performance
- Requires more servers (minimum 3 config, 2+ shards, 1+ mongos)

---

## ACID Properties

### Simple Explanation

ACID stands for four properties that guarantee database transactions are processed reliably:

- **Atomicity:** All-or-nothing - either the entire transaction succeeds or none of it does. Like transferring money: either both debit and credit happen, or neither does.
- **Consistency:** Data follows all rules before and after transaction. Like maintaining account balances can't go negative.
- **Isolation:** Transactions don't interfere with each other. Two people withdrawing from the same account at the same time don't cause problems.
- **Durability:** Once committed, data stays saved even if the server crashes immediately after.

### Technical Explanation

#### MongoDB ACID Support Evolution

**Before 4.0:** Single document operations were atomic
**4.0+:** Multi-document transactions within single replica set
**4.2+:** Multi-document transactions across shards (distributed transactions)

#### Atomicity

**Single document operations (always atomic):**

```javascript
// Entire update is atomic
db.accounts.updateOne(
  { _id: 1 },
  {
    $inc: { balance: -100 },
    $push: { transactions: { type: 'debit', amount: 100 } },
  },
);
```

**Multi-document transactions:**

```javascript
const session = db.getMongo().startSession();
session.startTransaction();

try {
  const accounts = session.getDatabase('bank').accounts;

  // Debit from account 1
  accounts.updateOne({ _id: 1 }, { $inc: { balance: -100 } }, { session });

  // Credit to account 2
  accounts.updateOne({ _id: 2 }, { $inc: { balance: 100 } }, { session });

  // Both succeed or both fail
  session.commitTransaction();
} catch (error) {
  session.abortTransaction();
  throw error;
} finally {
  session.endSession();
}
```

#### Consistency

MongoDB enforces consistency through:

**1. Schema validation:**

```javascript
db.createCollection('accounts', {
  validator: {
    $jsonSchema: {
      properties: {
        balance: { bsonType: 'double', minimum: 0 },
      },
    },
  },
});
```

**2. Unique indexes:**

```javascript
db.users.createIndex({ email: 1 }, { unique: true });
```

**3. Read/Write concerns:**

```javascript
// Ensure read reflects majority-committed data
db.collection.find().readConcern('majority');

// Ensure write acknowledged by majority
db.collection.insertOne(doc, { writeConcern: { w: 'majority' } });
```

#### Isolation

**Isolation levels (via Read Concern):**

##### 1. Local (default)

```javascript
db.collection.find().readConcern('local');
```

- Returns most recent data
- May return data not yet replicated
- May read data later rolled back

##### 2. Majority

```javascript
db.collection.find().readConcern('majority');
```

- Returns data acknowledged by majority
- Cannot be rolled back
- May not reflect most recent writes

##### 3. Snapshot (transactions only)

```javascript
session.startTransaction({
  readConcern: { level: 'snapshot' },
  writeConcern: { w: 'majority' },
});
```

- Reads from consistent snapshot
- All reads in transaction see same data version
- Prevents read skew and phantom reads

##### 4. Linearizable

```javascript
db.collection.find({ _id: 1 }).readConcern('linearizable');
```

- Strongest isolation
- Read-your-own-writes guarantee
- Only for single-document reads
- Performance cost

**Transaction isolation:**

```javascript
// Transaction 1
session1.startTransaction({
  readConcern: { level: 'snapshot' },
  writeConcern: { w: 'majority' },
});

// Transaction 2 won't see Transaction 1's changes until committed
session2.startTransaction({
  readConcern: { level: 'snapshot' },
  writeConcern: { w: 'majority' },
});
```

#### Durability

**Write concern for durability:**

```javascript
db.collection.insertOne(
  { data: 'important' },
  {
    writeConcern: {
      w: 'majority', // Replicated to majority
      j: true, // Written to journal
      wtimeout: 5000, // Timeout after 5s
    },
  },
);
```

**Journal:**

- Write-ahead log on disk
- Ensures data survives crashes
- Group commits every 100ms (default)
- All operations journaled before acknowledging

**Durability guarantees:**

| Write Concern   | Journal    | Durability Level                         |
| --------------- | ---------- | ---------------------------------------- |
| `w: 1`          | `j: false` | Lost if crash before journal flush       |
| `w: 1`          | `j: true`  | Survives single node crash               |
| `w: "majority"` | `j: false` | Survives node failures (maybe not crash) |
| `w: "majority"` | `j: true`  | Survives failures and crashes            |

#### Transaction Performance Considerations

**Overhead:**

- Transactions use resources (locks, memory)
- Hold locks until commit/abort
- Can cause contention under high concurrency

**Best practices:**

- Keep transactions short
- Limit operations per transaction (< 1000 documents)
- Use appropriate timeout: `session.startTransaction({ maxCommitTimeMS: 30000 })`
- Handle `TransientTransactionError` with retry logic
- Avoid long-running transactions

**Retry logic:**

```javascript
async function runTransactionWithRetry(txnFunc, session) {
  while (true) {
    try {
      await txnFunc(session);
      await session.commitTransaction();
      break;
    } catch (error) {
      if (error.hasErrorLabel('TransientTransactionError')) {
        console.log('Transient error, retrying...');
        continue;
      }
      throw error;
    }
  }
}
```

#### Causal Consistency

Guarantees read-after-write and monotonic reads:

```javascript
const session = db.getMongo().startSession({ causalConsistency: true });

// Write
session.getDatabase('db').collection.insertOne({ x: 1 });

// Read will see the write (even from secondary)
const doc = session.getDatabase('db').collection.findOne({ x: 1 });
```

---

## Aggregation Pipeline

### Simple Explanation

The aggregation pipeline is like an assembly line for data processing. Your data enters at one end and goes through multiple stages, with each stage transforming the data in some way.

**Example:** Find average order value by customer

1. **Stage 1:** Filter orders from last year
2. **Stage 2:** Group by customer
3. **Stage 3:** Calculate average per customer
4. **Stage 4:** Sort by highest average

It's like making a smoothie: select fruits (filter) → blend them (transform) → pour into glass (output).

### Technical Explanation

#### Pipeline Structure

Aggregation pipeline consists of ordered stages:

```javascript
db.collection.aggregate([
  { $stage1: { ... } },
  { $stage2: { ... } },
  { $stage3: { ... } }
])
```

Each stage passes its output to the next stage.

#### Common Pipeline Stages

##### 1. $match (Filter Documents)

```javascript
db.orders.aggregate([
  {
    $match: {
      status: 'completed',
      orderDate: { $gte: ISODate('2024-01-01') },
    },
  },
]);
```

- Like `find()` query
- Should be early in pipeline for performance
- Can use indexes

##### 2. $project (Select/Transform Fields)

```javascript
db.users.aggregate([
  {
    $project: {
      fullName: { $concat: ['$firstName', ' ', '$lastName'] },
      email: 1,
      _id: 0,
    },
  },
]);
```

- Include/exclude fields
- Create computed fields
- Reshape documents

##### 3. $group (Aggregate Data)

```javascript
db.orders.aggregate([
  {
    $group: {
      _id: '$customerId',
      totalSpent: { $sum: '$amount' },
      orderCount: { $sum: 1 },
      avgOrderValue: { $avg: '$amount' },
      orders: { $push: '$orderId' },
    },
  },
]);
```

**Accumulator operators:**

- `$sum`: Sum values
- `$avg`: Average values
- `$min` / `$max`: Min/max values
- `$first` / `$last`: First/last value
- `$push`: Create array of values
- `$addToSet`: Create array of unique values

##### 4. $sort (Order Results)

```javascript
db.products.aggregate([{ $sort: { price: -1, name: 1 } }]);
```

- 1 = ascending, -1 = descending
- Can use indexes if early in pipeline
- Memory limit: 100MB (use `allowDiskUse: true` for larger)

##### 5. $limit and $skip (Pagination)

```javascript
db.products.aggregate([
  { $sort: { createdAt: -1 } },
  { $skip: 20 },
  { $limit: 10 },
]);
```

##### 6. $lookup (Join Collections)

```javascript
db.orders.aggregate([
  {
    $lookup: {
      from: 'customers',
      localField: 'customerId',
      foreignField: '_id',
      as: 'customerInfo',
    },
  },
]);
```

**Advanced lookup with pipeline:**

```javascript
db.orders.aggregate([
  {
    $lookup: {
      from: 'products',
      let: { orderItems: '$items' },
      pipeline: [
        { $match: { $expr: { $in: ['$_id', '$$orderItems'] } } },
        { $project: { name: 1, price: 1 } },
      ],
      as: 'productDetails',
    },
  },
]);
```

##### 7. $unwind (Deconstruct Arrays)

```javascript
// Input: { _id: 1, items: ["A", "B", "C"] }
db.orders.aggregate([{ $unwind: '$items' }]);
// Output:
// { _id: 1, items: "A" }
// { _id: 1, items: "B" }
// { _id: 1, items: "C" }
```

**Preserve null/empty arrays:**

```javascript
{ $unwind: { path: "$items", preserveNullAndEmptyArrays: true } }
```

##### 8. $addFields (Add New Fields)

```javascript
db.products.aggregate([
  {
    $addFields: {
      discountedPrice: { $multiply: ['$price', 0.9] },
      inStock: { $gt: ['$quantity', 0] },
    },
  },
]);
```

##### 9. $bucket (Group into Ranges)

```javascript
db.products.aggregate([
  {
    $bucket: {
      groupBy: '$price',
      boundaries: [0, 100, 500, 1000, 5000],
      default: 'Other',
      output: {
        count: { $sum: 1 },
        products: { $push: '$name' },
      },
    },
  },
]);
```

##### 10. $facet (Multiple Pipelines)

```javascript
db.products.aggregate([
  {
    $facet: {
      priceStats: [
        {
          $group: {
            _id: null,
            avg: { $avg: '$price' },
            max: { $max: '$price' },
          },
        },
      ],
      categoryCounts: [{ $group: { _id: '$category', count: { $sum: 1 } } }],
      topProducts: [{ $sort: { sales: -1 } }, { $limit: 5 }],
    },
  },
]);
```

##### 11. $out and $merge (Write Results)

```javascript
// Replace entire collection
db.orders.aggregate([
  { $match: { status: 'completed' } },
  { $out: 'completed_orders' },
]);

// Merge into existing collection
db.orders.aggregate([
  { $group: { _id: '$customerId', total: { $sum: '$amount' } } },
  {
    $merge: {
      into: 'customer_stats',
      whenMatched: 'merge',
      whenNotMatched: 'insert',
    },
  },
]);
```

#### Complex Example: E-commerce Analytics

```javascript
db.orders.aggregate([
  // Stage 1: Filter recent orders
  {
    $match: {
      orderDate: { $gte: ISODate('2024-01-01') },
      status: 'completed',
    },
  },

  // Stage 2: Unwind items array
  { $unwind: '$items' },

  // Stage 3: Join with products
  {
    $lookup: {
      from: 'products',
      localField: 'items.productId',
      foreignField: '_id',
      as: 'productInfo',
    },
  },

  // Stage 4: Unwind product info
  { $unwind: '$productInfo' },

  // Stage 5: Calculate revenue per category
  {
    $group: {
      _id: '$productInfo.category',
      totalRevenue: {
        $sum: { $multiply: ['$items.quantity', '$items.price'] },
      },
      orderCount: { $sum: 1 },
      uniqueCustomers: { $addToSet: '$customerId' },
    },
  },

  // Stage 6: Add computed fields
  {
    $addFields: {
      avgOrderValue: { $divide: ['$totalRevenue', '$orderCount'] },
      customerCount: { $size: '$uniqueCustomers' },
    },
  },

  // Stage 7: Sort by revenue
  { $sort: { totalRevenue: -1 } },

  // Stage 8: Format output
  {
    $project: {
      category: '$_id',
      totalRevenue: { $round: ['$totalRevenue', 2] },
      avgOrderValue: { $round: ['$avgOrderValue', 2] },
      orderCount: 1,
      customerCount: 1,
      _id: 0,
    },
  },
]);
```

#### Performance Optimization

##### 1. Pipeline Optimization

MongoDB automatically optimizes pipelines:

- Moves `$match` and `$sort` early
- Combines adjacent stages when possible
- Uses indexes when available

##### 2. Index Usage

```javascript
// $match can use indexes
db.orders.aggregate([
  { $match: { customerId: 123 } }, // Uses index on customerId
  { $sort: { orderDate: -1 } }, // Uses index if exists
]);
```

##### 3. Projection Early

```javascript
// Good: Project early to reduce data
db.orders.aggregate([
  { $match: { status: 'completed' } },
  { $project: { customerId: 1, amount: 1, _id: 0 } }, // Reduce data
  { $group: { _id: '$customerId', total: { $sum: '$amount' } } },
]);
```

##### 4. Memory Limits

- Default: 100MB per stage
- Use `allowDiskUse: true` for larger operations:

```javascript
db.collection.aggregate(pipeline, { allowDiskUse: true });
```

##### 5. Explain Plans

```javascript
db.orders.aggregate(pipeline, { explain: true });
```

#### Expression Operators

**Comparison:**

- `$eq`, `$ne`, `$gt`, `$gte`, `$lt`, `$lte`

**Arithmetic:**

- `$add`, `$subtract`, `$multiply`, `$divide`, `$mod`

**String:**

- `$concat`, `$substr`, `$toUpper`, `$toLower`, `$split`

**Array:**

- `$size`, `$arrayElemAt`, `$slice`, `$filter`, `$map`, `$reduce`

**Date:**

- `$year`, `$month`, `$dayOfMonth`, `$hour`, `$dateToString`

**Conditional:**

- `$cond`, `$ifNull`, `$switch`

**Example with expressions:**

```javascript
db.users.aggregate([
  {
    $project: {
      fullName: { $concat: ['$firstName', ' ', '$lastName'] },
      age: {
        $dateDiff: {
          startDate: '$birthDate',
          endDate: '$$NOW',
          unit: 'year',
        },
      },
      isAdult: {
        $gte: [
          { $subtract: [new Date(), '$birthDate'] },
          18 * 365 * 24 * 60 * 60 * 1000,
        ],
      },
      status: {
        $switch: {
          branches: [
            { case: { $lt: ['$loginCount', 5] }, then: 'new' },
            { case: { $lt: ['$loginCount', 50] }, then: 'regular' },
            { case: { $gte: ['$loginCount', 50] }, then: 'power' },
          ],
          default: 'unknown',
        },
      },
    },
  },
]);
```

---

## Query Performance

### Simple Explanation

Query performance is about making your database queries run fast. It's like organizing your closet - if everything is thrown in randomly, it takes forever to find a shirt. But if you organize by type and color, you can grab what you need instantly.

**Key factors:**

- **Use indexes:** Like a book's index to find information quickly
- **Limit results:** Don't fetch more data than you need
- **Query efficiently:** Ask for exactly what you need, not everything
- **Monitor slow queries:** Find and fix problematic queries

### Technical Explanation

#### Query Execution Plans

**Analyze queries with explain():**

```javascript
db.users.find({ email: 'john@example.com' }).explain('executionStats');
```

**Execution plan levels:**

1. **queryPlanner:** Shows selected plan
2. **executionStats:** Shows execution statistics
3. **allPlansExecution:** Shows all considered plans

**Key metrics to examine:**

```javascript
{
  executionStats: {
    executionSuccess: true,
    nReturned: 1,              // Documents returned
    executionTimeMillis: 2,     // Total execution time
    totalDocsExamined: 1,       // Documents scanned
    totalKeysExamined: 1,       // Index entries scanned
    executionStages: {
      stage: "FETCH",
      inputStage: {
        stage: "IXSCAN",        // Index scan (good!)
        keyPattern: { email: 1 },
        indexName: "email_1"
      }
    }
  }
}
```

**Execution stages:**

- **COLLSCAN:** Collection scan (scans all documents - bad for large collections)
- **IXSCAN:** Index scan (uses index - good!)
- **FETCH:** Fetches full documents
- **SORT:** In-memory sort (expensive without index)
- **SORT_KEY_GENERATOR:** Generates sort keys
- **PROJECTION:** Projects fields

**Ideal scenario:**

- `totalKeysExamined` ≈ `nReturned` (minimal unnecessary scanning)
- Stage: `IXSCAN` (using index)
- Low `executionTimeMillis`

#### Index Optimization

##### Index Selection Strategy

**MongoDB query optimizer:**

1. Identifies candidate indexes for query
2. Creates execution plans for each candidate
3. Runs trial executions with sample data
4. Caches winning plan for query shape
5. Reevaluates after:
   - 1000 writes to collection
   - Index rebuild
   - Significant data changes

**Query plan cache:**

```javascript
// View cached plans
db.collection.getPlanCache().list();

// Clear plan cache
db.collection.getPlanCache().clear();
```

##### Covering Queries

Queries served entirely from index (no document fetch):

```javascript
// Create index with all queried fields
db.users.createIndex({ email: 1, name: 1, age: 1 });

// Covering query (super fast!)
db.users
  .find({ email: 'john@example.com' }, { email: 1, name: 1, age: 1, _id: 0 })
  .explain('executionStats');

// Look for: totalDocsExamined: 0
// Stage: "IXSCAN" without "FETCH"
```

##### Index Intersection

MongoDB can combine multiple indexes:

```javascript
// Two separate indexes
db.users.createIndex({ firstName: 1 });
db.users.createIndex({ lastName: 1 });

// Query uses both (AND_SORTED or AND_HASH)
db.users.find({ firstName: 'John', lastName: 'Doe' });
```

**Note:** Usually better to create compound index than rely on intersection.

##### Index Prefix

Compound indexes support queries on prefixes:

```javascript
// Index: { a: 1, b: 1, c: 1 }

// Supported queries:
{ a: 1 }                    // Uses index
{ a: 1, b: 1 }             // Uses index
{ a: 1, b: 1, c: 1 }       // Uses index (full)

// NOT supported:
{ b: 1 }                    // Cannot use index
{ c: 1 }                    // Cannot use index
{ b: 1, c: 1 }             // Cannot use index
```

#### Query Optimization Techniques

##### 1. Projection (Select Specific Fields)

```javascript
// Bad: Fetches all fields
db.users.find({ status: 'active' });

// Good: Only needed fields
db.users.find({ status: 'active' }, { name: 1, email: 1, _id: 0 });
```

##### 2. Limit Results

```javascript
// Bad: Returns all matches
db.logs.find({ level: 'error' });

// Good: Limit to needed amount
db.logs.find({ level: 'error' }).limit(100);
```

##### 3. Use Appropriate Operators

**Efficient:**

```javascript
// Equality (index-friendly)
db.users.find({ status: 'active' });

// Range (index-friendly)
db.orders.find({ amount: { $gte: 100, $lte: 500 } });

// IN with small array (index-friendly)
db.users.find({ status: { $in: ['active', 'pending'] } });
```

**Less efficient:**

```javascript
// $ne (scans entire collection)
db.users.find({ status: { $ne: 'deleted' } });

// $nin (scans entire collection)
db.users.find({ status: { $nin: ['deleted', 'banned'] } });

// $not (cannot use index efficiently)
db.users.find({ age: { $not: { $gt: 25 } } });

// Regular expressions without anchor (full scan)
db.users.find({ email: { $regex: /example/ } });

// Better: Anchored regex (can use index)
db.users.find({ email: { $regex: /^john/ } });
```

##### 4. Avoid Large Skip Values

```javascript
// Very slow for large skip values
db.products.find().skip(100000).limit(10);

// Better: Range-based pagination
db.products.find({ _id: { $gt: lastSeenId } }).limit(10);
```

##### 5. Batch Operations

```javascript
// Bad: Multiple single operations
for (let doc of documents) {
  db.collection.insertOne(doc);
}

// Good: Single batch operation
db.collection.insertMany(documents);
```

#### Monitoring and Profiling

##### 1. Database Profiler

```javascript
// Enable profiler (level 2 = all operations)
db.setProfilingLevel(2, { slowms: 100 });

// Level 0: Off
// Level 1: Log slow operations only
// Level 2: Log all operations

// View slow queries
db.system.profile.find().sort({ ts: -1 }).limit(10);

// Find slowest queries
db.system.profile.find({ millis: { $gt: 100 } }).sort({ millis: -1 });
```

##### 2. Current Operations

```javascript
// View currently running operations
db.currentOp()

// Filter long-running queries
db.currentOp({ "secs_running": { $gt: 5 } })

// Kill slow operation
db.killOp(<opid>)
```

##### 3. Server Status

```javascript
// Overall server statistics
db.serverStatus();

// Connection statistics
db.serverStatus().connections;

// Operation counters
db.serverStatus().opcounters;
```

##### 4. Collection Statistics

```javascript
// Collection stats
db.collection.stats();

// Index statistics
db.collection.aggregate([{ $indexStats: {} }]);
```

#### Common Performance Anti-Patterns

##### 1. Missing Indexes

```javascript
// Problem: No index on frequently queried field
db.users.find({ email: 'john@example.com' }); // COLLSCAN

// Solution:
db.users.createIndex({ email: 1 });
```

##### 2. Unbounded Queries

```javascript
// Problem: Returns entire collection
db.logs.find({});

// Solution: Add filters and limits
db.logs.find({ date: { $gte: recentDate } }).limit(1000);
```

##### 3. Large Document Updates

```javascript
// Problem: Updating entire large document
db.articles.updateOne({ _id: id }, { $set: largeDocument });

// Solution: Update only changed fields
db.articles.updateOne(
  { _id: id },
  { $set: { content: newContent, updatedAt: new Date() } },
);
```

##### 4. Inefficient Array Updates

```javascript
// Problem: Updating array element (requires fetching document)
const doc = db.posts.findOne({ _id: id });
doc.comments[0].text = 'updated';
db.posts.replaceOne({ _id: id }, doc);

// Solution: Use positional operator
db.posts.updateOne(
  { _id: id, 'comments._id': commentId },
  { $set: { 'comments.$.text': 'updated' } },
);
```

##### 5. Document Growth

```javascript
// Problem: Documents growing (causes relocation)
db.users.updateOne({ _id: id }, { $push: { activities: newActivity } });

// Solution: Cap array size or use separate collection
db.users.updateOne(
  { _id: id },
  {
    $push: {
      activities: {
        $each: [newActivity],
        $slice: -100, // Keep only last 100
      },
    },
  },
);
```

#### Performance Tuning Checklist

1. **Indexes:**

   - Create indexes on frequently queried fields
   - Use compound indexes for multiple-field queries
   - Use covering indexes when possible
   - Remove unused indexes

2. **Query patterns:**

   - Use projection to limit returned fields
   - Limit result sets
   - Avoid $ne, $nin, $not when possible
   - Use anchored regex for text search

3. **Schema design:**

   - Embed related data to avoid joins
   - Avoid unbounded arrays
   - Consider document size (16MB limit)
   - Denormalize when read-heavy

4. **Monitoring:**

   - Enable slow query logging
   - Monitor with explain()
   - Track index usage
   - Analyze query patterns

5. **Hardware/Configuration:**
   - Sufficient RAM for working set
   - SSD for better I/O
   - Appropriate connection pool size
   - WiredTiger cache configuration

#### Working Set Concept

**Working set:** Frequently accessed data and indexes

```javascript
// Check if working set fits in RAM
const stats = db.serverStatus();
const workingSetMB =
  stats.wiredTiger.cache['bytes currently in the cache'] / (1024 * 1024);
const totalRAM = stats.mem.resident;

// Ideal: workingSetMB < totalRAM
```

**If working set > RAM:**

- More disk I/O (slower queries)
- Page faults increase
- Consider:
  - Adding RAM
  - Optimizing indexes
  - Archiving old data
  - Sharding

```

This covers all the major MongoDB concepts with both simple and technical explanations. Good luck with your interview!
```
