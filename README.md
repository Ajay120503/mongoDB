# MongoDB:

## Contents

1. [Database foundations](#1-database-foundations)
2. [MongoDB architecture and terminology](#2-mongodb-architecture-and-terminology)
3. [Installation and connection](#3-installation-and-connection)
4. [BSON and data types](#4-bson-and-data-types)
5. [Practice dataset](#5-practice-dataset)
6. [Create operations](#6-create-operations)
7. [Read operations and projection](#7-read-operations-and-projection)
8. [Query operators](#8-query-operators)
9. [Arrays and nested documents](#9-arrays-and-nested-documents)
10. [Update operations](#10-update-operations)
11. [Delete replace and bulk operations](#11-delete-replace-and-bulk-operations)
12. [Sorting counting and pagination](#12-sorting-counting-and-pagination)
13. [Aggregation pipelines](#13-aggregation-pipelines)
14. [Relationships and schema design](#14-relationships-and-schema-design)
15. [Indexes and query performance](#15-indexes-and-query-performance)
16. [Database schema validation](#16-database-schema-validation)
17. [Atomicity and transactions](#17-atomicity-and-transactions)
18. [Replication and consistency](#18-replication-and-consistency)
19. [Sharding](#19-sharding)
20. [How MongoDB works in a backend](#20-how-mongodb-works-in-a-backend)
21. [Mongoose concepts](#21-mongoose-concepts)
22. [Complete Express and Mongoose backend](#22-complete-express-and-mongoose-backend)
23. [Run and test the API](#23-run-and-test-the-api)
24. [References populate and lookup](#24-references-populate-and-lookup)
25. [Mongoose middleware and advanced features](#25-mongoose-middleware-and-advanced-features)
26. [Native MongoDB driver](#26-native-mongodb-driver)
27. [Authentication authorization and security](#27-authentication-authorization-and-security)
28. [Production deployment and operations](#28-production-deployment-and-operations)
29. [Advanced MongoDB features](#29-advanced-mongodb-features)
30. [Common errors and fixes](#30-common-errors-and-fixes)
31. [SQL to MongoDB comparison](#31-sql-to-mongodb-comparison)
32. [Interview questions and answers](#32-interview-questions-and-answers)
33. [Practice exercises with solutions](#33-practice-exercises-with-solutions)
34. [Revision sheet and learning plan](#34-revision-sheet-and-learning-plan)
35. [Official references and verification scope](#35-official-references-and-verification-scope)

## 1. Database foundations

A **database** stores organized information so an application can retrieve and change it reliably. A **DBMS** provides the software for storage, queries, permissions, concurrency, and recovery.

MongoDB is a document-oriented database. It stores records as BSON documents, which resemble JavaScript objects. MongoDB is often called NoSQL; this does not mean it cannot represent relationships or support transactions.

| Relational concept | MongoDB equivalent | Example |
|---|---|---|
| Database | Database | `college_notes` |
| Table | Collection | `students` |
| Row | Document | One student |
| Column | Field | `name` |
| Primary key | `_id` | An ObjectId |
| JOIN | `$lookup` or application queries | Students with courses |
| Schema | Flexible document structure; optional validators | Required name and email |

**Choose by access patterns.** MongoDB fits applications where related data naturally forms documents: catalogs, profiles, content, activity records, and many web backends. A relational database can be a better fit when normalized relationships and extensive relational constraints dominate. Neither database type is automatically faster.

**Key point:** Flexible schema means you can evolve document structure. It does not remove the need for a deliberate data model.

## 2. MongoDB architecture and terminology

```mermaid
flowchart TD
    A[MongoDB deployment] --> B[College database]
    A --> C[Shop database]
    B --> D[Students collection]
    B --> E[Courses collection]
    D --> F[Student document]
    F --> G[Scalar fields]
    F --> H[Arrays and embedded objects]
```

| Component | Purpose |
|---|---|
| MongoDB server / `mongod` | Stores data and executes database operations |
| `mongosh` | Interactive database shell |
| Compass | Graphical client for browsing and querying data |
| Atlas | Managed MongoDB service |
| Node.js driver | Connects Node.js to MongoDB |
| Mongoose | ODM library built on the Node.js driver |
| `mongos` | Router used with a sharded cluster |

A server receives commands, selects a query plan, accesses indexes and documents, and returns results. Storage engines handle physical persistence and caching; application code should not depend on database file layout.

**MongoDB and Mongoose are different:** MongoDB is the database. Mongoose is an optional Node.js modeling library.

## 3. Installation and connection

### 3.1 Option A: MongoDB Atlas

1. Sign in to Atlas and create a project and deployment.
2. Create a database user with access to the learning database.
3. Allow the IP address from which your backend connects.
4. Open the deployment connection workflow and choose the appropriate client.
5. Copy its connection string and insert your own credentials and database name.

Example URI shape:

```text
mongodb+srv://APP_USER:ENCODED_PASSWORD@YOUR_CLUSTER/college_api?retryWrites=true&w=majority
```

Use the exact hostname Atlas provides. Percent-encode special characters in username/password components; do not encode the entire URI. Your Atlas website login is separate from a database user. See [MongoDB getting started](https://www.mongodb.com/docs/get-started/).

### 3.2 Option B: Local database with Docker

With Docker already installed:

```bash
docker run -d --name mongodb-notes \
  -p 127.0.0.1:27017:27017 \
  -v mongodb_notes_data:/data/db \
  mongo:8.0
```

This is a **local learning deployment**, bound to loopback, without database authentication. The named volume keeps data across container recreation. Do not expose this configuration publicly.

Open the shell inside the container:

```bash
docker exec -it mongodb-notes mongosh
```

Stop or restart it:

```bash
docker stop mongodb-notes
docker start mongodb-notes
```

Local URI:

```text
mongodb://127.0.0.1:27017/college_api
```

For a native installation, follow the [official OS-specific installation instructions](https://www.mongodb.com/docs/manual/installation/). Repository commands vary by OS release. Compass and mongosh are clients; installing only a client does not install a running database server.

### 3.3 Essential shell commands

Run these inside **mongosh**, not your operating-system terminal:

```javascript
show dbs
use college_notes
db
show collections
db.createCollection("courses")
db.stats()
```

`use` selects a database; an empty selected database is not persisted merely by selecting it. A write creates it if needed.

**Destructive examples — practice data only:**

```javascript
db.courses.drop()       // Removes the collection and its indexes
// db.dropDatabase()    // Removes the selected database
```

## 4. BSON and data types

BSON is the binary document format MongoDB uses. It supports types beyond JSON, such as dates, ObjectIds, binary data, and decimal values.

```javascript
{
  _id: ObjectId("66a000000000000000000001"),
  name: "Ajay",
  age: 23,
  active: true,
  skills: ["JavaScript", "MongoDB"],
  address: { city: "Pune", pin: "411001" },
  joinedAt: ISODate("2026-01-10T00:00:00Z"),
  middleName: null
}
```

| Type | Example | Application use |
|---|---|---|
| String | `"Pune"` | Names and text |
| Int32 / Int64 / Double | `NumberInt(23)` | Counts and measurements |
| Decimal128 | `NumberDecimal("199.95")` | Exact decimal representation |
| Boolean | `true` | Flags |
| Date | `new Date()` | An instant in time |
| ObjectId | `ObjectId()` | Common identifier type |
| Object | `{ city: "Pune" }` | Embedded related data |
| Array | `["JS", "Java"]` | Multiple values |
| Null | `null` | Explicitly no value |
| Binary | BSON binary data | Binary payloads |

**Important distinctions:**

- A missing field differs from a field explicitly set to `null`.
- BSON Date stores an instant with millisecond precision; local formatting belongs in the presentation layer.
- BSON Timestamp is a separate type used mainly for internal database purposes.
- ObjectId is a 12-byte identifier commonly displayed as 24 hexadecimal characters. It is not an authorization token.
- Use integer minor units, such as paise, or a deliberate Decimal128 strategy for money. JavaScript floating-point arithmetic is not exact decimal arithmetic.
- A BSON document has a 16 MiB size limit. Avoid indefinitely growing arrays.

## 5. Practice dataset

Run this **once in a fresh `college_notes` database**. Later snippets are independent exercises; mutations change subsequent results. Reset intentionally if you need the original outputs.

```javascript
use college_notes

db.students.insertMany([
  {
    _id: ObjectId("66a000000000000000000001"),
    name: "Ajay", email: "ajay@example.com", age: 23,
    course: "MSc CA", marks: 85, active: true,
    skills: ["JavaScript", "MongoDB"],
    address: { city: "Pune", pin: "411001" },
    subjects: [{ name: "DBMS", score: 90 }, { name: "Java", score: 80 }],
    createdAt: ISODate("2026-01-10T00:00:00Z")
  },
  {
    _id: ObjectId("66a000000000000000000002"),
    name: "Harshada", email: "harshada@example.com", age: 22,
    course: "MSc CA", marks: 92, active: true,
    skills: ["Java", "MongoDB"],
    address: { city: "Pune", pin: "411004" },
    subjects: [{ name: "DBMS", score: 95 }, { name: "Java", score: 89 }],
    createdAt: ISODate("2026-01-11T00:00:00Z")
  },
  {
    _id: ObjectId("66a000000000000000000003"),
    name: "Akash", email: "akash@example.com", age: 24,
    course: "MSc CS", marks: 76, active: false,
    skills: ["Python", "SQL"],
    address: { city: "Mumbai", pin: "400001" },
    subjects: [{ name: "DBMS", score: 72 }, { name: "Java", score: 80 }],
    createdAt: ISODate("2026-01-12T00:00:00Z")
  },
  {
    _id: ObjectId("66a000000000000000000004"),
    name: "Neha", email: "neha@example.com", age: 21,
    course: "MSc CS", marks: 88, active: true,
    skills: ["Python", "MongoDB"],
    address: { city: "Nashik", pin: "422001" },
    subjects: [{ name: "DBMS", score: 88 }, { name: "Java", score: 88 }],
    createdAt: ISODate("2026-01-13T00:00:00Z")
  }
])
```

Expected starting count: `4`.

## 6. Create operations

**Theory:** Insert operations add documents. MongoDB generates `_id` when you omit it. An existing unique value can cause a duplicate-key error. CRUD methods and single-document atomicity are described in the [official CRUD guide](https://www.mongodb.com/docs/manual/crud/).

```javascript
// One document
db.courses.insertOne({ code: "DBMS", title: "Database Management", credits: 4 })

// Several documents
db.courses.insertMany([
  { code: "JAVA", title: "Java Programming", credits: 4 },
  { code: "NET", title: "Computer Networks", credits: 3 }
])
```

`insertOne()` returns write metadata including `insertedId`. `insertMany()` returns inserted IDs. Ordered insertion stops processing at the first error; with `{ ordered: false }`, MongoDB attempts other documents. Neither mode makes the batch an all-or-nothing transaction.

## 7. Read operations and projection

```javascript
db.students.find()                              // Cursor over matching documents
db.students.findOne({ email: "ajay@example.com" }) // One document or null
db.students.find({ course: "MSc CA" })
db.students.findOne({ _id: ObjectId("66a000000000000000000001") })
```

**Projection** selects returned fields:

```javascript
db.students.find({}, { name: 1, marks: 1, _id: 0 })
db.students.find({}, { subjects: 0, address: 0 })
```

In ordinary inclusion/exclusion projection, do not mix `1` and `0` except for `_id`. A projection reduces returned data; it does not decide which documents match.

`find()` returns a cursor in mongosh/native driver. Mongoose `find()` returns a query that resolves to an array when awaited. Avoid loading a huge collection into memory.

## 8. Query operators

### 8.1 Comparisons

| Operator | Meaning | Example filter |
|---|---|---|
| `$eq` | Equal | `{ marks: { $eq: 85 } }` |
| `$ne` | Not equal | `{ course: { $ne: "MSc CA" } }` |
| `$gt` | Greater than | `{ marks: { $gt: 80 } }` |
| `$gte` | At least | `{ marks: { $gte: 80 } }` |
| `$lt` | Less than | `{ age: { $lt: 23 } }` |
| `$lte` | At most | `{ age: { $lte: 23 } }` |
| `$in` | Any listed value | `{ course: { $in: ["MSc CA", "MSc CS"] } }` |
| `$nin` | None of listed values | `{ course: { $nin: ["BCA"] } }` |

```javascript
db.students.find({ marks: { $gte: 80, $lte: 90 } })
// Original dataset: Ajay and Neha
```

### 8.2 Logical conditions

```javascript
// Implicit AND
db.students.find({ active: true, marks: { $gte: 85 } })

// OR
db.students.find({ $or: [{ marks: { $gte: 90 } }, { age: { $lt: 22 } }] })

// Explicit AND
db.students.find({ $and: [{ age: { $gte: 22 } }, { age: { $lte: 23 } }] })

// NOR: neither condition matches
db.students.find({ $nor: [{ active: false }, { marks: { $lt: 80 } }] })

// NOT applies to a field predicate
db.students.find({ marks: { $not: { $gte: 80 } } })
```

### 8.3 Existence, type, null, regex, expressions

```javascript
db.students.find({ phone: { $exists: false } })
db.students.find({ name: { $type: "string" } })
db.students.find({ middleName: null }) // Explicit null OR missing
db.students.find({ middleName: { $type: "null" } }) // Explicit BSON null

db.students.find({ name: { $regex: "^A", $options: "i" } })
db.students.find({ $expr: { $gt: ["$marks", { $multiply: ["$age", 3] }] } })
```

`$ne`, `$nin`, and negation can also match missing fields depending on the predicate. Add explicit existence requirements when that distinction matters. Regex search is not equivalent to a full-text search engine. See [query predicates](https://www.mongodb.com/docs/manual/reference/mql/query-predicates/).

## 9. Arrays and nested documents

Use dot notation for nested fields:

```javascript
db.students.find({ "address.city": "Pune" })
db.students.find({ skills: "MongoDB" })
db.students.find({ skills: { $all: ["JavaScript", "MongoDB"] } })
db.students.find({ skills: { $size: 2 } })
```

**Why `$elemMatch` matters:** Multiple conditions on an array of documents may otherwise be satisfied by different elements.

```javascript
// A single subject must be DBMS AND have score >= 90
db.students.find({
  subjects: { $elemMatch: { name: "DBMS", score: { $gte: 90 } } }
})
```

Compare with `{ "subjects.name": "DBMS", "subjects.score": { $gte: 90 } }`, where the name and score can come from different subjects.

Exact embedded-object equality is more restrictive than matching selected nested fields and can depend on field order. Prefer dot notation for field-level conditions.

## 10. Update operations

**Theory:** An update filter identifies documents; update operators describe changes. Each affected document is updated atomically, but `updateMany()` is not one atomic operation across all documents.

```javascript
db.students.updateOne(
  { email: "ajay@example.com" },
  { $set: { marks: 89, "address.city": "Pune" } }
)

db.students.updateMany({ course: "MSc CS" }, { $set: { active: true } })
```

| Operator | Purpose |
|---|---|
| `$set` | Set selected fields |
| `$unset` | Remove fields |
| `$inc` | Increment/decrement a numeric field |
| `$mul` | Multiply a numeric field |
| `$min` / `$max` | Change only if supplied value is smaller/larger |
| `$rename` | Rename a field |
| `$currentDate` | Set current date |
| `$push` | Append to an array |
| `$addToSet` | Add a value if absent |
| `$pull` | Remove matching array elements |
| `$pop` | Remove first (`-1`) or last (`1`) array element |
| `$setOnInsert` | Set fields only when an upsert inserts |

```javascript
db.students.updateOne({ name: "Ajay" }, { $inc: { marks: 1 } })
db.students.updateOne({ name: "Ajay" }, { $unset: { middleName: "" } })
db.students.updateOne({ name: "Ajay" }, { $addToSet: { skills: "Node.js" } })
db.students.updateOne({ name: "Ajay" }, { $push: { skills: { $each: ["Git", "Docker"] } } })
db.students.updateOne({ name: "Ajay" }, { $pull: { skills: "Docker" } })
```

`$addToSet` prevents adding another equal value; it does not clean duplicates already present.

### Update array elements

```javascript
// First matching array element
db.students.updateOne(
  { name: "Ajay", "subjects.name": "DBMS" },
  { $set: { "subjects.$.score": 96 } }
)

// All matching elements selected by arrayFilters
db.students.updateMany(
  {},
  { $inc: { "subjects.$[subject].score": 1 } },
  { arrayFilters: [{ "subject.score": { $lt: 80 } }] }
)
```

### Upsert

```javascript
db.students.updateOne(
  { email: "new@example.com" },
  {
    $set: { name: "New Student", marks: 70 },
    $setOnInsert: { createdAt: new Date() }
  },
  { upsert: true }
)
```

An upsert updates a match or inserts if none matches. Create a unique index on the business identity, such as email, to prevent duplicates under concurrent requests. Inspect `matchedCount`, `modifiedCount`, and `upsertedId` as applicable. A match with unchanged values can have `modifiedCount: 0`.

Reference: [update operators](https://www.mongodb.com/docs/manual/reference/mql/update/).

## 11. Delete replace and bulk operations

```javascript
db.students.deleteOne({ email: "new@example.com" })
// Removes ALL inactive matches; use only when intended:
db.students.deleteMany({ active: false })
```

An empty filter in `deleteMany({})` removes all documents. It preserves the collection and indexes.

`replaceOne()` replaces the document body, retaining the existing `_id` when omitted. Other omitted fields disappear. Use `$set` when you intend a partial change.

```javascript
db.courses.replaceOne(
  { code: "NET" },
  { code: "NET", title: "Networking Fundamentals", credits: 4 }
)

db.courses.bulkWrite([
  { insertOne: { document: { code: "OS", title: "Operating Systems", credits: 4 } } },
  { updateOne: { filter: { code: "JAVA" }, update: { $set: { credits: 5 } } } }
])
```

Bulk operations reduce command overhead. They do not provide transaction semantics automatically. A **soft delete**, such as setting `deletedAt`, enables recovery but requires every applicable query and index policy to account for deleted records.

## 12. Sorting counting and pagination

```javascript
db.students.find().sort({ marks: -1, _id: 1 })
db.students.find().sort({ _id: 1 }).skip(2).limit(2)
db.students.countDocuments({ active: true })
db.students.estimatedDocumentCount()
db.students.distinct("course")
```

`1` sorts ascending, `-1` descending. Add a unique tie-breaker for repeatable ordering. `countDocuments(filter)` counts matches; `estimatedDocumentCount()` uses collection metadata and accepts no query filter.

**Offset pagination:** `skip = (page - 1) × limit`. Easy for small datasets, but large offsets require scanning past earlier entries.

**Cursor pagination:** Use the last seen sort key:

```javascript
db.students.find({ _id: { $gt: ObjectId("66a000000000000000000002") } })
  .sort({ _id: 1 })
  .limit(2)
```

This paginates by `_id`, not guaranteed event chronology. For `createdAt` sorting, use both `createdAt` and `_id` in the cursor predicate and matching compound index. Pagination is not a frozen snapshot when concurrent writes occur.

## 13. Aggregation pipelines

**Theory:** An aggregation pipeline transforms a stream of documents through ordered stages. Use it for summaries, computed fields, grouped reports, and joining collection data. Ordinary pipelines return results without modifying source documents; `$merge` and `$out` are write stages. See [aggregation operations](https://www.mongodb.com/docs/manual/aggregation/).

```mermaid
flowchart TD
    A[Student documents] --> B[Filter active students]
    B --> C{Report branch}
    C --> D[Group by course]
    C --> E[Sort highest marks]
    D --> F[Course summary]
    E --> G[Top students]
```

| Stage | Purpose |
|---|---|
| `$match` | Filter documents |
| `$project` | Select or compute fields |
| `$set` / `$addFields` | Add computed fields |
| `$group` | Aggregate by a key |
| `$sort` | Order results |
| `$skip` / `$limit` | Page or cap results |
| `$unwind` | Produce documents from array elements |
| `$lookup` | Join related collection data |
| `$count` | Count incoming documents |
| `$facet` | Run several subpipelines |
| `$bucket` | Group into numeric ranges |
| `$setWindowFields` | Window calculations such as ranks |
| `$merge` / `$out` | Write pipeline results |

### Course report

```javascript
db.students.aggregate([
  { $match: { active: true } },
  { $group: {
      _id: "$course",
      totalStudents: { $sum: 1 },
      averageMarks: { $avg: "$marks" },
      highestMarks: { $max: "$marks" }
  } },
  { $sort: { averageMarks: -1, _id: 1 } }
])
```

Original dataset: MSc CA → count `2`, average `88.5`, highest `92`; MSc CS → count `1`, average `88`, highest `88`.

`"$marks"` refers to a field. `$sum: 1` counts documents in each group. `_id: null` groups all incoming documents together.

### Grade calculation

```javascript
db.students.aggregate([
  { $project: {
    _id: 0, name: 1, marks: 1,
    grade: { $switch: {
      branches: [
        { case: { $gte: ["$marks", 90] }, then: "A+" },
        { case: { $gte: ["$marks", 80] }, then: "A" }
      ],
      default: "B"
    } }
  } }
])
```

### Array report

```javascript
db.students.aggregate([
  { $unwind: "$skills" },
  { $group: { _id: "$skills", students: { $sum: 1 } } },
  { $sort: { students: -1, _id: 1 } }
])
```

### Results and count together

```javascript
db.students.aggregate([
  { $match: { active: true } },
  { $sort: { marks: -1, _id: 1 } },
  { $facet: {
    data: [{ $skip: 0 }, { $limit: 2 }],
    total: [{ $count: "count" }]
  } }
])
```

Place selective filters early where semantics permit. Index usefulness depends on stages and ordering. Large sorts, groups, and facets need memory planning; `allowDiskUse` does not remove every stage or document-size limit.

## 14. Relationships and schema design

MongoDB supports **embedding** and **referencing**. Design around what is read and updated together. See [data modeling](https://www.mongodb.com/docs/manual/data-modeling/).

### Embedding

```javascript
{ name: "Ajay", address: { city: "Pune", pin: "411001" } }
```

Useful for bounded, owned data normally read together. One document update can atomically change parent and embedded fields.

### Referencing

```javascript
// enrollment document
const enrollment = { studentId: ObjectId("66a000000000000000000001"), courseCode: "DBMS" };
```

Useful when entities have independent lifecycles or related records grow without a useful bound. References are not automatically enforced foreign keys.

```mermaid
erDiagram
    STUDENT ||--o{ ENROLLMENT : has
    COURSE ||--o{ ENROLLMENT : contains
    STUDENT {
        ObjectId _id
        string email
        string name
    }
    COURSE {
        ObjectId _id
        string code
        string title
    }
    ENROLLMENT {
        ObjectId _id
        ObjectId studentId
        ObjectId courseId
        date enrolledAt
    }
```

| Situation | Usually consider |
|---|---|
| One address per student | Embed |
| Millions of comments on a post | Separate collection with parent ID |
| Students taking multiple courses | Enrollment collection |
| Historical purchase price | Embed an order-item price snapshot |
| Shared editable course details | Reference |

Avoid unbounded arrays, excessive duplication without synchronization rules, and modeling every relationship as a join by habit. Define deletion behavior: restrict, cascade in application code, or keep historical references.

## 15. Indexes and query performance

An index is an additional structure that helps MongoDB locate and order data. It consumes space and adds write work. Create indexes for measured query patterns. See [indexes](https://www.mongodb.com/docs/manual/indexes/).

```javascript
db.students.createIndex({ email: 1 }, { unique: true })
db.students.createIndex({ course: 1, marks: -1, _id: 1 })
db.students.createIndex({ skills: 1 })
db.students.getIndexes()
```

| Index type | Use |
|---|---|
| Single field | Lookup by one field |
| Compound | Multiple filter/sort fields |
| Unique | Prevent duplicate indexed values |
| Multikey | Index array values |
| TTL | Expire documents based on a date field |
| Partial | Index only documents matching a condition |
| Text | Basic text search |
| `2dsphere` | Geographic queries |
| Hashed | Hashed-key access and some sharding designs |

Compound field order matters. Equality–Sort–Range is a useful starting guideline, then verify with actual query plans. Prefixes of compound indexes can support some simpler queries.

```javascript
db.students.find({ course: "MSc CA" })
  .sort({ marks: -1, _id: 1 })
  .explain("executionStats")
```

Inspect `nReturned`, `totalDocsExamined`, `totalKeysExamined`, and plan stages such as `COLLSCAN` or `IXSCAN`. An index scan alone does not prove a query is efficient.

```javascript
// Separate practice collection: expiration happens asynchronously
db.sessions.createIndex({ expiresAt: 1 }, { expireAfterSeconds: 0 })
db.sessions.insertOne({ tokenHash: "example-hash", expiresAt: new Date(Date.now() + 60000) })
```

Do not rely on TTL cleanup to enforce exact authentication expiry; check the expiration timestamp on requests.

A unique index fails to build if existing data violates it. Optional unique fields need deliberate partial-index/null handling. A compound multikey index has restrictions on indexing multiple array fields in the same document.

## 16. Database schema validation

Mongoose validation runs in application code. MongoDB collection validation can also protect writes from other clients.

```javascript
db.createCollection("validatedStudents", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["name", "marks"],
      properties: {
        name: { bsonType: "string", minLength: 2 },
        marks: { bsonType: ["int", "long", "double", "decimal"], minimum: 0, maximum: 100 }
      }
    }
  },
  validationLevel: "strict",
  validationAction: "error"
})
```

```javascript
db.validatedStudents.insertOne({ name: "Ajay", marks: 85 }) // Accepted
// db.validatedStudents.insertOne({ name: "A", marks: 150 }) // Rejected
```

Use `collMod` to change an existing collection validator. Plan existing-data cleanup before tightening rules. Database validation does not implement all business rules or reference integrity automatically.

## 17. Atomicity and transactions

**Atomicity:** Either an operation completes or its changes are not applied. Single-document writes are atomic. Use a transaction when multiple documents must change together. Transactions require a replica set or sharded cluster; the standalone Docker setup above does not support them. See [transactions](https://www.mongodb.com/docs/manual/core/transactions/).

### Atomic stock decrement

Example fragment assuming a `Product` Mongoose model and validated integer quantity:

```javascript
const product = await Product.findOneAndUpdate(
  { _id: productId, stock: { $gte: quantity } },
  { $inc: { stock: -quantity } },
  { returnDocument: "after" }
);
if (!product) throw new Error("Product missing or insufficient stock");
```

The stock condition and decrement occur together. Reading stock and later writing a calculated value can lose updates.

### Multi-document transaction

Conceptual fragment; `Product` and `Order` are application models, not part of the student API below:

```javascript
await mongoose.connection.transaction(async (session) => {
  const product = await Product.findOneAndUpdate(
    { _id: productId, stock: { $gte: quantity } },
    { $inc: { stock: -quantity } },
    { session, returnDocument: "after" }
  );
  if (!product) throw new Error("Insufficient stock");

  await Order.create([
    { userId, items: [{ productId, quantity, pricePaise: product.pricePaise }] }
  ], { session });
});
```

```mermaid
flowchart TD
    A[Start transaction] --> B[Conditionally reduce stock]
    B --> C{Stock available?}
    C -->|No| D[Abort transaction]
    C -->|Yes| E[Insert order]
    E --> F{Insert succeeds?}
    F -->|No| D
    F -->|Yes| G[Commit changes]
```

Use the session on every operation. Keep transaction operations sequential, not `Promise.all`. Retryable callbacks can run again, so do not charge a card or send an email inside them. Use an idempotent external workflow or transactional outbox for side effects.

ACID: atomicity, consistency, isolation, durability. Transactions do not replace good data modeling.

## 18. Replication and consistency

A **replica set** maintains copies of the dataset. A primary accepts normal writes, secondaries replicate, and eligible members can elect a new primary after failure. See [replication](https://www.mongodb.com/docs/manual/replication/).

```mermaid
flowchart TD
    A[Backend driver] --> P[Primary]
    P -->|Replication| S1[Secondary A]
    P -->|Replication| S2[Secondary B]
    S1 -.->|Election participation| S2
    S2 -.->|Election participation| P
```

| Setting | Question it answers |
|---|---|
| Read preference | Which eligible member should receive a read? |
| Read concern | What visibility/consistency guarantees should a read request? |
| Write concern | What acknowledgement is required for a write? |

Secondary reads may lag. `w: "majority"` requests majority acknowledgement; it does not make every possible read linearizable. Select guarantees based on the operation. Replication improves availability but is not a backup: accidental deletions can replicate too.

## 19. Sharding

**Sharding** distributes a collection across multiple shards. A shard key determines data distribution and affects query routing. See [sharding](https://www.mongodb.com/docs/manual/sharding/).

```mermaid
flowchart TD
    A[Backend] --> R[mongos router]
    R --> S1[Shard A replica set]
    R --> S2[Shard B replica set]
    R --> C[Config server replica set]
    C -.->|Routing metadata| R
```

A query containing an appropriate shard-key condition may target fewer shards. Other queries can scatter across shards. Poor keys create uneven load or hot spots. Consider cardinality, frequency distribution, write patterns, and common filters.

**Replication copies data; sharding partitions data.** Many applications should improve their schema, queries, and indexes before adding sharding complexity.

## 20. How MongoDB works in a backend

The frontend sends HTTP requests. Express validates and routes them. Backend code checks permissions, executes database operations, and returns JSON. The frontend must not receive database credentials.

```mermaid
sequenceDiagram
    participant U as Browser
    participant R as Express route
    participant C as Controller
    participant M as Mongoose
    participant D as MongoDB
    U->>R: POST /api/students
    R->>C: Parsed request
    C->>C: Validate allowed fields
    C->>M: Student.create(data)
    M->>D: Insert command
    D-->>M: Stored document
    M-->>C: Student document
    C-->>U: 201 JSON response
```

| Backend layer | Responsibility |
|---|---|
| Route | URL and HTTP method mapping |
| Middleware | Shared parsing, authentication, validation, errors |
| Controller | Translate HTTP request into an operation |
| Service | Business workflow, useful as complexity grows |
| Model | Data structure and database operations |
| Database | Persistence, indexes, concurrency, constraints |

A small API can use controllers and models directly. Add a service layer when workflows span models or must be reused.

## 21. Mongoose concepts

A **schema** declares application-level fields and behavior. A **model** is compiled from a schema and exposes database methods. A **document** is an instance returned or constructed through a model. See [Mongoose schemas](https://mongoosejs.com/docs/guide.html).

```javascript
const studentSchema = new mongoose.Schema({
  name: { type: String, required: true },
  marks: { type: Number, min: 0, max: 100 }
}, { timestamps: true });

const Student = mongoose.model("Student", studentSchema);
const student = new Student({ name: "Ajay", marks: 85 });
await student.save();
```

Mongoose normally maps `Student` to the `students` collection. Set a collection name explicitly if naming must be fixed. `timestamps: true` manages `createdAt` and `updatedAt`.

| Method | Typical resolved value |
|---|---|
| `create()` | Saved document, or documents for array input |
| `find()` | Array |
| `findOne()` / `findById()` | Document or `null` |
| `updateOne()` | Write result metadata |
| `findByIdAndUpdate()` | Document or `null` |
| `deleteOne()` | Metadata with `deletedCount` |
| `findByIdAndDelete()` | Deleted document or `null` |
| `aggregate()` | Array of plain result objects |

Use `returnDocument: "after"` to receive the updated document. Mongoose's `unique: true` declares a unique index; it is **not** a validator. Query updates need `runValidators: true`, and their validators have limitations. See [Mongoose validation](https://mongoosejs.com/docs/validation.html).

## 22. Complete Express and Mongoose backend

This is a complete **local learning API** with CRUD, filtering, pagination, validation, aggregation, and centralized error handling. Authentication is deliberately explained separately in Chapter 27; these endpoints are not ready for public exposure without access controls.

### 22.1 Create the project

Use a currently supported Node.js LTS compatible with your selected dependencies. The example uses ES modules, Express 5, Mongoose 9, and Zod 4.

```bash
mkdir mongodb-student-api
cd mongodb-student-api
npm init -y
npm install express@5 mongoose@9 dotenv cors helmet zod@4
npm install --save-dev nodemon
npm pkg set type=module
npm pkg set scripts.start="node src/server.js"
npm pkg set scripts.dev="nodemon src/server.js"
mkdir -p src/config src/models src/controllers src/routes src/middleware src/validation
```

Commit `package-lock.json` so installs can be reproduced with `npm ci`.

| File | Purpose |
|---|---|
| `.env` | Local configuration |
| `.gitignore` | Exclude secrets and dependencies |
| `src/config/db.js` | Database connection |
| `src/models/Student.js` | Schema and indexes |
| `src/validation/student.js` | HTTP input validation |
| `src/middleware/errors.js` | Shared errors |
| `src/controllers/studentController.js` | CRUD and report operations |
| `src/routes/studentRoutes.js` | Endpoint mappings |
| `src/app.js` | Express application |
| `src/server.js` | Startup and shutdown |

### 22.2 `.env`

```dotenv
PORT=5000
NODE_ENV=development
MONGODB_URI=mongodb://127.0.0.1:27017/college_api
CLIENT_ORIGIN=http://localhost:5173
```

For Atlas, replace only the URI with your deployment connection string. Never use a frontend environment variable such as `VITE_MONGODB_URI` for database credentials.

### 22.3 `.gitignore`

```gitignore
node_modules/
.env
.env.*
!.env.example
*.log
```

You may create `.env.example` using the local, credential-free values above.

### 22.4 `src/config/db.js`

```javascript
import mongoose from "mongoose";

export async function connectDB() {
  if (!process.env.MONGODB_URI) {
    throw new Error("MONGODB_URI is required");
  }

  await mongoose.connect(process.env.MONGODB_URI, {
    serverSelectionTimeoutMS: 10000,
    maxPoolSize: 10,
    autoIndex: process.env.NODE_ENV !== "production"
  });

  console.log("MongoDB connected");
}
```

Open one reusable connection per application process. Do not connect and disconnect for each request. The pool supports concurrent work; tune it according to workload and deployment connection limits. Modern Mongoose does not need old options such as `useNewUrlParser` or `useUnifiedTopology`. See [connections](https://mongoosejs.com/docs/connections.html).

### 22.5 `src/models/Student.js`

```javascript
import mongoose from "mongoose";

const studentSchema = new mongoose.Schema({
  name: {
    type: String, required: true, trim: true,
    minlength: 2, maxlength: 80
  },
  email: {
    type: String, required: true, trim: true, lowercase: true,
    maxlength: 254
  },
  age: {
    type: Number, required: true, min: 16, max: 100,
    validate: { validator: Number.isInteger, message: "Age must be an integer" }
  },
  course: {
    type: String, required: true, enum: ["MSc CA", "MSc CS", "MCA", "BCA"]
  },
  marks: { type: Number, required: true, min: 0, max: 100 },
  active: { type: Boolean, default: true },
  skills: {
    type: [{ type: String, trim: true, maxlength: 40 }],
    default: [],
    validate: { validator: value => value.length <= 20, message: "At most 20 skills" }
  }
}, {
  timestamps: true,
  strict: "throw"
});

studentSchema.index({ email: 1 }, { unique: true });
studentSchema.index({ course: 1, createdAt: -1, _id: -1 });
studentSchema.index({ createdAt: -1, _id: -1 });

export default mongoose.model("Student", studentSchema);
```

There is one email index declaration, avoiding duplicate declarations. Application validation handles email syntax; the database index enforces uniqueness. Existing duplicates must be fixed before that index can be built.

### 22.6 `src/validation/student.js`

```javascript
import { z } from "zod";

const fields = {
  name: z.string().trim().min(2).max(80),
  email: z.string().trim().email().max(254).transform(value => value.toLowerCase()),
  age: z.number().int().min(16).max(100),
  course: z.enum(["MSc CA", "MSc CS", "MCA", "BCA"]),
  marks: z.number().min(0).max(100),
  active: z.boolean(),
  skills: z.array(z.string().trim().min(1).max(40)).max(20)
};

export const createStudentSchema = z.object({
  ...fields,
  active: fields.active.optional(),
  skills: fields.skills.optional()
}).strict();

export const updateStudentSchema = z.object(fields)
  .partial()
  .strict()
  .refine(value => Object.keys(value).length > 0, {
    message: "Provide at least one field to update"
  });

export const listStudentsSchema = z.object({
  page: z.coerce.number().int().min(1).max(10000).default(1),
  limit: z.coerce.number().int().min(1).max(100).default(10),
  course: fields.course.optional(),
  minMarks: z.coerce.number().min(0).max(100).optional(),
  active: z.enum(["true", "false"]).optional()
}).strict();
```

Body numbers must be JSON numbers, not numeric strings. Query strings are converted intentionally. Unknown fields, arbitrary query operators, and empty updates are rejected. The course list is a teaching rule, not a universal list of courses.

### 22.7 `src/middleware/errors.js`

```javascript
import { ZodError } from "zod";

export class AppError extends Error {
  constructor(status, message) {
    super(message);
    this.status = status;
  }
}

export function errorHandler(err, req, res, next) {
  if (res.headersSent) return next(err);

  if (err instanceof ZodError) {
    return res.status(400).json({
      success: false,
      message: "Invalid input",
      errors: err.issues.map(issue => ({
        field: issue.path.join("."), message: issue.message
      }))
    });
  }

  if (err.code === 11000) {
    return res.status(409).json({ success: false, message: "Email already exists" });
  }

  if (["ValidationError", "CastError", "StrictModeError"].includes(err.name)) {
    return res.status(400).json({ success: false, message: "Invalid student data" });
  }

  if (err.type === "entity.parse.failed") {
    return res.status(400).json({ success: false, message: "Invalid JSON body" });
  }

  if (err.type === "entity.too.large") {
    return res.status(413).json({ success: false, message: "Request body is too large" });
  }

  if (err instanceof AppError) {
    return res.status(err.status).json({ success: false, message: err.message });
  }

  // Avoid logging full request bodies or credentials.
  console.error("Request failed", { name: err.name });
  return res.status(500).json({ success: false, message: "Internal server error" });
}
```

`400` means malformed/invalid input; `404` means resource not found; `409` means conflict; `500` means an unexpected server failure. Do not expose raw database errors to API clients.

### 22.8 `src/controllers/studentController.js`

```javascript
import Student from "../models/Student.js";
import { AppError } from "../middleware/errors.js";
import {
  createStudentSchema, updateStudentSchema, listStudentsSchema
} from "../validation/student.js";

export async function createStudent(req, res) {
  const input = createStudentSchema.parse(req.body);
  const student = await Student.create(input);
  res.status(201).json({ success: true, data: student });
}

export async function listStudents(req, res) {
  const query = listStudentsSchema.parse(req.query);
  const filter = {};

  if (query.course !== undefined) filter.course = query.course;
  if (query.minMarks !== undefined) filter.marks = { $gte: query.minMarks };
  if (query.active !== undefined) filter.active = query.active === "true";

  const [students, total] = await Promise.all([
    Student.find(filter)
      .select("name email age course marks active skills createdAt updatedAt")
      .sort({ createdAt: -1, _id: -1 })
      .skip((query.page - 1) * query.limit)
      .limit(query.limit)
      .lean(),
    Student.countDocuments(filter)
  ]);

  res.json({
    success: true,
    data: students,
    pagination: {
      page: query.page, limit: query.limit,
      total, pages: Math.ceil(total / query.limit)
    }
  });
}

export async function getStudent(req, res) {
  const student = await Student.findById(req.params.id).lean();
  if (!student) throw new AppError(404, "Student not found");
  res.json({ success: true, data: student });
}

export async function updateStudent(req, res) {
  const input = updateStudentSchema.parse(req.body);
  const student = await Student.findByIdAndUpdate(
    req.params.id,
    { $set: input },
    { returnDocument: "after", runValidators: true }
  );
  if (!student) throw new AppError(404, "Student not found");
  res.json({ success: true, data: student });
}

export async function deleteStudent(req, res) {
  const student = await Student.findByIdAndDelete(req.params.id);
  if (!student) throw new AppError(404, "Student not found");
  res.status(204).end();
}

export async function studentStats(req, res) {
  const data = await Student.aggregate([
    { $group: {
      _id: "$course",
      students: { $sum: 1 },
      averageMarks: { $avg: "$marks" },
      highestMarks: { $max: "$marks" }
    } },
    { $sort: { students: -1, _id: 1 } },
    { $project: {
      _id: 0, course: "$_id", students: 1,
      averageMarks: { $round: ["$averageMarks", 2] },
      highestMarks: 1
    } }
  ]);
  res.json({ success: true, data });
}
```

The list and count are independent reads; concurrent changes can make them momentarily differ. This is acceptable for this learning API. `lean()` returns plain objects without document methods such as `save()` and skips document hydration. It is useful for read-only output; it does not remove query execution cost. See [lean queries](https://mongoosejs.com/docs/tutorials/lean.html).

### 22.9 `src/routes/studentRoutes.js`

```javascript
import { Router } from "express";
import { AppError } from "../middleware/errors.js";
import {
  createStudent, listStudents, getStudent,
  updateStudent, deleteStudent, studentStats
} from "../controllers/studentController.js";

const router = Router();

router.param("id", (req, res, next, id) => {
  if (!/^[a-fA-F0-9]{24}$/.test(id)) {
    return next(new AppError(400, "Invalid student ID"));
  }
  next();
});

router.get("/stats", studentStats);
router.route("/").get(listStudents).post(createStudent);
router.route("/:id")
  .get(getStudent)
  .patch(updateStudent)
  .delete(deleteStudent);

export default router;
```

Put `/stats` before `/:id`. Express 5 forwards rejected promises from async handlers to error middleware; Express 4 requires an async wrapper or explicit `next(err)` handling.

### 22.10 `src/app.js`

```javascript
import express from "express";
import cors from "cors";
import helmet from "helmet";
import mongoose from "mongoose";
import studentRoutes from "./routes/studentRoutes.js";
import { AppError, errorHandler } from "./middleware/errors.js";

const app = express();

app.use(helmet());
app.use(cors({ origin: process.env.CLIENT_ORIGIN || "http://localhost:5173" }));
app.use(express.json({ limit: "32kb" }));

app.get("/health", (req, res) => {
  const ready = mongoose.connection.readyState === 1;
  res.status(ready ? 200 : 503).json({ status: ready ? "ok" : "unavailable" });
});

app.use("/api/students", studentRoutes);
app.use((req, res, next) => next(new AppError(404, "Route not found")));
app.use(errorHandler);

export default app;
```

CORS controls browser access to responses; it does not authenticate callers or protect the API from non-browser requests. The health endpoint is a basic readiness indicator, not a full dependency audit.

### 22.11 `src/server.js`

```javascript
import "dotenv/config";
import mongoose from "mongoose";
import app from "./app.js";
import { connectDB } from "./config/db.js";
import Student from "./models/Student.js";

async function start() {
  await connectDB();

  // Wait for development indexes, including unique email, before accepting writes.
  if (process.env.NODE_ENV !== "production") await Student.init();

  const port = Number(process.env.PORT || 5000);
  if (!Number.isInteger(port) || port < 1 || port > 65535) {
    throw new Error("Invalid PORT");
  }

  const server = app.listen(port, "127.0.0.1", () => {
    console.log(`API running at http://127.0.0.1:${port}`);
  });

  server.on("error", () => {
    console.error("HTTP server failed to start");
    process.exit(1);
  });

  let stopping = false;
  function shutdown() {
    if (stopping) return;
    stopping = true;
    const timeout = setTimeout(() => process.exit(1), 10000);
    timeout.unref();
    server.close(async () => {
      try {
        await mongoose.disconnect();
        clearTimeout(timeout);
        process.exit(0);
      } catch {
        process.exit(1);
      }
    });
  }

  process.on("SIGINT", shutdown);
  process.on("SIGTERM", shutdown);
}

start().catch(() => {
  console.error("Startup failed: check database access and configuration");
  process.exit(1);
});
```

Startup waits for the database; shutdown stops accepting HTTP connections before disconnecting. For hosted/container deployments, bind to the host required by the provider, commonly `0.0.0.0`, and add the production protections in Chapters 27–28.

**Production index prerequisite:** Because automatic index creation is disabled in production, create and verify the required indexes through a controlled migration before serving traffic. Setting `NODE_ENV=production` alone does not make this demo production-ready.

## 23. Run and test the API

Start the local MongoDB container from Chapter 3, then run:

```bash
npm run dev
```

### Endpoint map

| Method | Path | Purpose | Success |
|---|---|---|---|
| POST | `/api/students` | Create | 201 |
| GET | `/api/students` | List/filter/page | 200 |
| GET | `/api/students/stats` | Course report | 200 |
| GET | `/api/students/:id` | Fetch one | 200 |
| PATCH | `/api/students/:id` | Update selected fields | 200 |
| DELETE | `/api/students/:id` | Delete | 204 |

### Create a student

```bash
curl -i -X POST http://127.0.0.1:5000/api/students \
  -H 'Content-Type: application/json' \
  -d '{"name":"Ajay","email":"ajay@example.com","age":23,"course":"MSc CA","marks":85,"skills":["MongoDB","Node.js"]}'
```

Expected response shape, with actual ID and timestamps generated at runtime:

```json
{
  "success": true,
  "data": {
    "name": "Ajay",
    "email": "ajay@example.com",
    "age": 23,
    "course": "MSc CA",
    "marks": 85,
    "active": true,
    "skills": ["MongoDB", "Node.js"],
    "_id": "GENERATED_24_HEX_ID",
    "createdAt": "GENERATED_TIMESTAMP",
    "updatedAt": "GENERATED_TIMESTAMP",
    "__v": 0
  }
}
```

### Read and filter

```bash
curl 'http://127.0.0.1:5000/api/students?page=1&limit=5&minMarks=80&active=true'
curl http://127.0.0.1:5000/api/students/stats
```

Copy the `_id` from your create response and replace `STUDENT_ID` below:

```bash
curl http://127.0.0.1:5000/api/students/STUDENT_ID

curl -i -X PATCH http://127.0.0.1:5000/api/students/STUDENT_ID \
  -H 'Content-Type: application/json' \
  -d '{"marks":94}'

curl -i -X DELETE http://127.0.0.1:5000/api/students/STUDENT_ID
```

Do not attempt to parse JSON from a successful `204` response; it has no body.

### Meaningful API checks

| Test | Expected |
|---|---|
| Valid create | 201 and persisted document |
| Same email again | 409 |
| Marks 150 | 400 |
| Unknown field `role` | 400 |
| Missing name on create | 400 |
| PATCH with `{}` | 400 |
| Invalid ID text | 400 |
| Validly formatted but nonexistent ID | 404 |
| `limit=1000` | 400 |
| Malformed JSON | 400 |
| Delete existing student | 204 |
| Read deleted student | 404 |

### Frontend usage

Browser-side example for a frontend running at the configured origin:

```javascript
async function loadStudents() {
  const response = await fetch("http://127.0.0.1:5000/api/students?limit=10");
  const body = await response.json();
  if (!response.ok) throw new Error(body.message || "Request failed");
  return body.data;
}
```

The browser calls Express; Express calls MongoDB. Never connect a regular web frontend using a database username and password.

## 24. References populate and lookup

### 24.1 Mongoose references

Illustrative extension to the backend; create this model before using the query:

```javascript
import mongoose from "mongoose";

const courseSchema = new mongoose.Schema({
  title: { type: String, required: true },
  coordinator: { type: mongoose.Schema.Types.ObjectId, ref: "Student" }
});
const Course = mongoose.model("Course", courseSchema);

const courses = await Course.find().populate("coordinator", "name email");
```

`populate()` replaces a reference in returned results by fetching related documents. It does not permanently embed them or enforce referential integrity. A missing single referenced document generally becomes `null`. Avoid excessive population depth and large fan-outs. See [population](https://mongoosejs.com/docs/populate.html).

### 24.2 MongoDB `$lookup`

Run in the practice database, whose students retain their original IDs:

```javascript
db.enrollments.insertOne({
  studentId: ObjectId("66a000000000000000000001"),
  courseCode: "DBMS"
})

db.enrollments.aggregate([
  { $lookup: {
    from: "students",
    localField: "studentId",
    foreignField: "_id",
    as: "student"
  } },
  { $unwind: "$student" },
  { $project: { _id: 0, courseCode: 1, studentName: "$student.name" } }
])
```

`$lookup` runs within MongoDB's pipeline. `populate()` is a Mongoose feature generally using additional queries. They are not identical implementations. To preserve records with no match when unwinding, use `{ $unwind: { path: "$student", preserveNullAndEmptyArrays: true } }`.

## 25. Mongoose middleware and advanced features

### Middleware

Register middleware **before compiling the model**:

```javascript
const articleSchema = new mongoose.Schema({ title: String, titleLower: String });

articleSchema.pre("save", function () {
  if (this.isModified("title") && this.title) {
    this.titleLower = this.title.toLowerCase();
  }
});

const Article = mongoose.model("Article", articleSchema);
```

Document hooks such as `save` do not automatically run for query updates like `findOneAndUpdate`. Query middleware has a different `this` context. Ensure every write path preserves your invariant; do not assume one hook covers everything.

### Virtuals, methods, statics

```javascript
const personSchema = new mongoose.Schema({ firstName: String, lastName: String });
personSchema.virtual("fullName").get(function () {
  return `${this.firstName} ${this.lastName}`;
});
personSchema.methods.greet = function () {
  return `Hello ${this.firstName}`;
};
personSchema.statics.findByFirstName = function (name) {
  return this.find({ firstName: name });
};
personSchema.set("toJSON", { virtuals: true });
const Person = mongoose.model("Person", personSchema);
```

Virtuals are computed properties, not stored fields. Methods run on documents; statics run on models. Use ordinary functions when Mongoose supplies `this`.

### Other concepts

| Feature | Meaning and practical caution |
|---|---|
| `select: false` | Exclude a field by default; still explicitly sanitize sensitive responses |
| Defaults | Applied to `undefined`, not every falsy value |
| Discriminators | Related schema variants in one collection |
| Subdocuments | Embedded objects with their own schema behavior |
| `lean()` | Plain objects without hydrated document behavior |
| Optimistic concurrency | Detect conflicting `save()` edits using version checks |
| `__v` | Version key, not a timestamp or automatic universal lock |

For conflicting edit workflows, consider `optimisticConcurrency: true` and handle version conflicts. For query updates, implement a deliberate version predicate/increment if needed. Mongoose generally does not cast values inside aggregation pipelines; construct ObjectIds explicitly when matching them there.

## 26. Native MongoDB driver

Mongoose is optional. The native driver gives direct collection access without Mongoose schemas or document middleware.

```bash
npm install mongodb
```

Standalone script `native-demo.js`, run from the project root:

```javascript
import "dotenv/config";
import { MongoClient } from "mongodb";

if (!process.env.MONGODB_URI) throw new Error("MONGODB_URI is required");
const client = new MongoClient(process.env.MONGODB_URI);

try {
  await client.connect();
  const db = client.db("native_demo");
  const collection = db.collection("students");
  const result = await collection.insertOne({ name: "Ajay", marks: 85 });
  const student = await collection.findOne({ _id: result.insertedId });
  console.log(student);
  const students = await collection.find({ marks: { $gte: 80 } }).limit(10).toArray();
  console.log(students);
} finally {
  await client.close();
}
```

```bash
node native-demo.js
```

Closing the client at the end is appropriate for a one-off script. A long-running backend should reuse its client and close it during shutdown. Use `for await...of` cursor iteration for large result streams rather than unbounded `toArray()`.

## 27. Authentication authorization and security

**Authentication** establishes identity. **Authorization** decides which data and operations that identity may access.

```mermaid
flowchart TD
    A[Incoming request] --> B{Valid identity?}
    B -->|No| C[401 response]
    B -->|Yes| D{Allowed action and ownership?}
    D -->|No| E[403 or concealed 404]
    D -->|Yes| F[Validate permitted fields]
    F --> G[Scoped database operation]
```

A typical registration flow validates input, hashes the password with a dedicated password hashing library, creates the user, and returns sanitized data. Login verifies the hash before issuing a session or token. Never store plaintext passwords or implement a password hash using a fast general-purpose hash alone.

### Ownership must be part of the query

Illustrative fragment assuming authentication middleware established `req.user.id`, an ownership-bearing `Post` model, and a validated `title`:

```javascript
const post = await Post.findOneAndUpdate(
  { _id: req.params.id, author: req.user.id },
  { $set: { title } },
  { returnDocument: "after", runValidators: true }
);
```

The frontend hiding an edit button does not enforce ownership. Tenant applications must include the trusted tenant scope in queries, reports, and reference lookups.

### Avoid operator injection and mass assignment

Do not use unvalidated input directly:

```javascript
// Anti-patterns: do not copy into a backend
// User.findOne(req.body)
// Student.find(req.query)
// User.findByIdAndUpdate(id, req.body)
```

A caller might supply an object such as `{ "$ne": null }` where your code expects a string, or attempt to update a privileged field. Validate types, construct allowed filters, and explicitly allow editable fields. Mongoose filter sanitization can be defense in depth, not a replacement for validation.

### Backend security checklist

- Keep connection credentials in server environment configuration.
- Use least-privilege database users and restricted network access.
- Use TLS for remote connections.
- Add authentication, ownership checks, and rate limiting before public use.
- For cookie sessions, choose appropriate `HttpOnly`, `Secure`, `SameSite`, and CSRF protections.
- Limit request bodies, pagination sizes, and expensive user-controlled operations.
- Escape or constrain search input; do not accept arbitrary regular expressions from strangers.
- Return only permitted fields; never serialize password hashes or reset secrets.
- Log request IDs and useful diagnostics without tokens or full sensitive payloads.
- Enforce token expiry during reads even when TTL indexes perform cleanup.

## 28. Production deployment and operations

### Deployment sequence

1. Provision the database and narrowly scoped application user.
2. Configure backend secret environment variables and network access.
3. Apply schema/data migrations and indexes with a controlled process.
4. Start the application after successful database connection.
5. Configure the provider's port and host binding requirements.
6. Add HTTPS, authentication, request limits, and readiness checks.
7. Verify read/write flows, duplicate protection, failure handling, and backups.
8. Monitor latency, errors, query plans, and connection usage.

### Backups and restoration

With MongoDB Database Tools installed, these local learning examples back up and restore an **isolated practice database**:

```bash
mongodump --uri='mongodb://127.0.0.1:27017' \
  --db=college_api --archive=college_api.archive --gzip

mongorestore --uri='mongodb://127.0.0.1:27017' \
  --archive=college_api.archive --gzip \
  --nsFrom='college_api.*' --nsTo='college_api_restore.*'
```

The restore targets a separate namespace so you can inspect it. Do not add `--drop` unless replacement is explicitly intended. A simple dump of an actively changing multi-collection dataset is not automatically a transactionally consistent backup. Choose a supported consistent backup strategy, encrypt backups, and practice restoration. Managed backup availability depends on deployment configuration.

### Migrations

A flexible schema still needs migration discipline. Example: add a default only where a field is missing:

```javascript
db.students.updateMany(
  { active: { $exists: false } },
  { $set: { active: true } }
)
```

For large datasets, batch changes and measure load. Make migrations restartable, track applied versions, and deploy code compatible with old and new shapes during rollout.

### Monitoring and performance

Watch query latency, documents examined, CPU, storage, working-set pressure, replication lag, pool saturation, and error rate. Use projections, bounded results, suitable indexes, and realistic load measurements before changing architecture. Avoid logging every query with sensitive parameters in production.

A retry after a network failure may occur after the server already committed a write. Use unique request/idempotency keys for operations that must not happen twice. Driver retry behavior does not replace application-level idempotency.

## 29. Advanced MongoDB features

### Change streams

Watch database changes on supported replica-set or sharded deployments. Example fragment with a Mongoose `Student` model:

```javascript
const stream = Student.watch();
stream.on("change", change => {
  console.log(change.operationType, change.documentKey);
});
stream.on("error", () => {
  console.error("Change stream requires recovery");
});
// During shutdown: await stream.close();
```

Use durable resume-token handling and idempotent consumers for reliable processing. Recovery depends on retained history. A change stream is not itself a complete durable job queue. See [change streams](https://www.mongodb.com/docs/manual/changeStreams/).

### Time-series collections

For sensor or measurement data:

```javascript
db.createCollection("temperatureReadings", {
  timeseries: {
    timeField: "recordedAt",
    metaField: "sensor",
    granularity: "minutes"
  }
})

db.temperatureReadings.insertOne({
  recordedAt: new Date(),
  sensor: { deviceId: "esp32-01", room: "Lab" },
  temperature: 26.4
})
```

Choose stable metadata and granularity matching the measurement interval. Time-series collections have feature-specific update/index restrictions; check your server version before designing write workflows.

### Geospatial queries

```javascript
db.places.createIndex({ location: "2dsphere" })
db.places.insertOne({
  name: "Pune Point",
  location: { type: "Point", coordinates: [73.8567, 18.5204] }
})
db.places.find({ location: { $near: {
  $geometry: { type: "Point", coordinates: [73.8567, 18.5204] },
  $maxDistance: 5000
} } })
```

GeoJSON coordinate order is **longitude, latitude**. The distance here is in meters.

### Text search and search indexes

```javascript
db.articles.createIndex({ title: "text", body: "text" })
db.articles.insertOne({ title: "Learning MongoDB", body: "Database indexing basics" })
db.articles.find({ $text: { $search: "MongoDB" } })
```

A regular text index differs from MongoDB Search indexes. Advanced search and vector retrieval require their own index definitions and deployment support. Vector search compares embeddings for similarity; it does not guarantee factual correctness or replace authorization filters.

### Other features to recognize

| Feature | What to learn next |
|---|---|
| GridFS | Chunked file storage for files beyond document limits; object storage is also common |
| Views | Saved read-only aggregation definitions, not automatically materialized copies |
| Capped collections | Bounded collections with specialized insertion/retention behavior |
| Collation | Locale-aware string comparison; coordinate query and index collation |
| Window functions | Ranking, running totals, and partitioned calculations |
| `$merge` | Store aggregation output for reporting workflows |
| Client-side encryption | Encryption and key management beyond transport security |
| Schema versioning | Evolve document formats while keeping old data readable |
| Partial indexes | Enforce/index only eligible subsets, such as active records |

## 30. Common errors and fixes

| Error or symptom | Likely cause | Practical check |
|---|---|---|
| `ECONNREFUSED 127.0.0.1:27017` | Local MongoDB stopped/wrong port | Start container/service and verify mapped port |
| Server selection timeout | Network, DNS, firewall, unavailable deployment | Check URI, reachability, Atlas access rules |
| Authentication failed | Wrong database credentials or auth database | Use database user, check `authSource` when applicable |
| SRV DNS lookup failure | DNS issue or wrong hostname | Check exact Atlas URI and DNS resolution |
| `E11000 duplicate key` | Unique index conflict | Return 409; inspect intended uniqueness |
| Cast to ObjectId failed | Invalid ID text | Validate ID format before querying |
| ValidationError | Schema rule rejected data | Inspect field types, ranges, enum spelling |
| Query returns empty array | Wrong DB/collection/filter/type | Confirm namespace and ObjectId vs string |
| Buffering timeout | Queries issued before connection is ready | Await startup connection |
| Duplicate index warning | Same index declared twice | Use one declaration per intended index |
| Transaction number error | Standalone deployment | Use replica set/Atlas for transactions |
| Browser CORS error | Origin differs from allowed origin | Compare protocol, hostname, and port |
| Slow list API | Missing index, huge offset, unbounded work | Use explain, limits, and cursor pagination |
| Updated data missing from response | Returned pre-update document | Use `returnDocument: "after"` |
| `save()` absent | Result came from `lean()`/aggregation | Use model updates or fetch hydrated document |
| Schema changed but old data unchanged | Schema change is not a data migration | Write a deliberate migration |

`localhost` inside one container refers to that container, not another container or your host. For a composed backend/database setup, use the database service hostname and the internal port.

## 31. SQL to MongoDB comparison

| SQL intent | MongoDB example |
|---|---|
| `SELECT * FROM students` | `db.students.find()` |
| `SELECT name FROM students` | `db.students.find({}, {name:1, _id:0})` |
| `WHERE marks >= 80` | `{marks: {$gte:80}}` |
| `WHERE course IN (...)` | `{course: {$in:["MSc CA","MSc CS"]}}` |
| `ORDER BY marks DESC` | `.sort({marks:-1})` |
| `LIMIT 5` | `.limit(5)` |
| `COUNT(*)` with filter | `countDocuments(filter)` |
| `GROUP BY course` | `$group: {_id:"$course", count:{$sum:1}}` |
| `UPDATE ... SET marks=90` | `updateOne(filter, {$set:{marks:90}})` |
| `DELETE ... WHERE ...` | `deleteMany(filter)` |
| Related-table join | `$lookup` with matching fields |

The mappings express similar intent, not identical semantics. Missing fields, nulls, arrays, types, and relationship constraints require special attention.

## 32. Interview questions and answers

### 1. What is MongoDB?

A document-oriented database that stores BSON documents and supports querying, indexes, aggregation, replication, sharding, and transactions.

### 2. What is the difference between a collection and a document?

A collection groups records. A document is one record containing fields, arrays, and possibly embedded documents.

### 3. Is MongoDB schema-less?

It has a flexible schema by default. Applications can enforce Mongoose schemas and database-level collection validators.

### 4. What is BSON?

A binary document representation that supports JSON-like structures plus types such as Date, ObjectId, and Decimal128.

### 5. Why use `_id`?

It identifies a document uniquely within a collection. MongoDB commonly generates an ObjectId when `_id` is absent. Sharded designs have additional uniqueness considerations when `_id` is not part of the shard key.

### 6. What is the difference between `find()` and `findOne()`?

`find()` represents multiple results; `findOne()` returns one matching document or null. The exact client-side return wrapper depends on shell, driver, or ODM.

### 7. What is projection?

Selecting the fields returned by a query. It reduces response size and can participate in covered-query optimization.

### 8. What is an index?

A structure that helps locate/order matching data at the cost of additional storage and write maintenance.

### 9. Is `unique: true` a Mongoose validator?

No. It declares a database unique index. Handle duplicate-key failures, including concurrent requests.

### 10. What is aggregation?

A stage-based data-processing framework for transformations, grouping, calculations, and related-data lookup.

### 11. When should data be embedded?

When it belongs to the parent, is bounded, and is usually accessed together. Consider document growth and update patterns.

### 12. When should references be used?

For independently managed entities, large or growing relationships, or shared records that should not be copied repeatedly.

### 13. Does MongoDB support joins?

Yes, aggregation supports `$lookup`. Mongoose also offers `populate()`, which operates differently.

### 14. What is an upsert?

An update that inserts when no document matches. A unique business-key index is useful to prevent concurrent duplicate inserts.

### 15. What does atomic mean?

An operation's changes happen together or not at all. MongoDB provides single-document atomic writes and multi-document transactions on supported deployments.

### 16. What is a replica set?

A group of database members maintaining replicated data and supporting primary election for availability.

### 17. How does sharding differ from replication?

Sharding partitions data across shards; replication maintains copies. A sharded cluster usually uses replica sets for shards.

### 18. What does `lean()` do?

It skips Mongoose document hydration, returning plain objects with lower application-side overhead and without document methods.

### 19. Why not connect MongoDB in every route?

Each process should reuse a connection pool. Repeated connection creation adds overhead and can exhaust limits.

### 20. How do you prevent NoSQL injection?

Validate types, allowlist fields, build server-controlled filters, and never pass arbitrary request objects into query/update APIs.

### 21. How do you prevent lost inventory updates?

Use an atomic conditional update such as a stock predicate with `$inc`; use transactions if related documents must change together.

### 22. Why can offset pagination become slow?

MongoDB must traverse entries before the requested offset. Cursor pagination avoids increasingly large skips for sequential browsing.

### 23. Does `updateMany()` make every change atomic together?

No. Individual document updates are atomic; the whole operation is not a multi-document transaction.

### 24. Is replication a backup?

No. Bad writes and deletions can replicate. Backups must preserve recoverable earlier states.

### 25. How would you diagnose a slow API?

Measure request and database timings; inspect filters, indexes, explain statistics, result size, population fan-out, and connection pressure before optimizing.

## 33. Practice exercises with solutions

Use the original Chapter 5 dataset for stated answers. Mutation exercises change it.

### Exercise 1: Active students with marks at least 85

```javascript
db.students.find({ active: true, marks: { $gte: 85 } }, { name: 1, _id: 0 })
```

Answer: Ajay, Harshada, Neha.

### Exercise 2: Pune students who know MongoDB

```javascript
db.students.find({ "address.city": "Pune", skills: "MongoDB" })
```

Answer: Ajay and Harshada.

### Exercise 3: Top two students

```javascript
db.students.find({}, { name: 1, marks: 1, _id: 0 }).sort({ marks: -1, _id: 1 }).limit(2)
```

Answer: Harshada 92, Neha 88.

### Exercise 4: Average marks for each course, including inactive students

```javascript
db.students.aggregate([
  { $group: { _id: "$course", average: { $avg: "$marks" } } },
  { $sort: { _id: 1 } }
])
```

Answer: MSc CA 88.5; MSc CS 82.

### Exercise 5: Students whose DBMS subject score is at least 90

```javascript
db.students.find({ subjects: { $elemMatch: { name: "DBMS", score: { $gte: 90 } } } })
```

Answer: Ajay and Harshada.

### Exercise 6: Add Express skill without adding an equal duplicate

```javascript
db.students.updateOne({ email: "ajay@example.com" }, { $addToSet: { skills: "Express" } })
```

Running again does not append a second equal `"Express"` string.

### Exercise 7: Identify duplicate emails in another imported collection

```javascript
db.importedStudents.aggregate([
  { $group: { _id: "$email", count: { $sum: 1 } } },
  { $match: { count: { $gt: 1 } } }
])
```

Decide a canonical record and merge policy before deleting duplicates. Normalize casing deliberately before enforcing case-insensitive business uniqueness.

### Exercise 8: Add a protected update endpoint

Extend the learning API with verified authentication and an owner ID. Acceptance criteria: unauthenticated requests fail; another user's record cannot be changed; permitted edits succeed; role fields cannot be mass-assigned.

### Exercise 9: Enrollment model

Create Student, Course, and Enrollment models. Add a unique compound index on `{ studentId: 1, courseId: 1 }`. Verify both references before enrollment and define how course/student deletion is handled. Test duplicate enrollment under simultaneous requests.

### Exercise 10: Performance comparison

Load synthetic practice records, run a course-filtered sorted query, collect explain statistics, add a suitable index, and compare examined keys/documents. Do not report an index as beneficial based only on its existence.

## 34. Revision sheet and learning plan

### Core command sheet

```javascript
// Create
db.collection.insertOne(document)
db.collection.insertMany(documents)

// Read
db.collection.find(filter, projection)
db.collection.findOne(filter)

// Update
db.collection.updateOne(filter, { $set: changes })
db.collection.updateMany(filter, { $inc: increments })

// Delete
db.collection.deleteOne(filter)
db.collection.deleteMany(filter)

// Analyze
db.collection.aggregate(pipeline)
db.collection.countDocuments(filter)
db.collection.find(filter).explain("executionStats")

// Index
db.collection.createIndex(keys, options)
db.collection.getIndexes()
```

The identifiers above are placeholders describing method signatures, not a script to run as-is.

### Twelve-session learning plan

| Session | Study | Practical outcome |
|---|---|---|
| 1 | Chapters 1–4 | Connect and understand document types |
| 2 | Chapters 5–7 | Insert and retrieve practice data |
| 3 | Chapters 8–9 | Write comparison and array queries |
| 4 | Chapters 10–12 | Update safely and paginate |
| 5 | Chapter 13 | Build course and skills reports |
| 6 | Chapters 14–16 | Choose a model and add constraints |
| 7 | Chapters 17–19 | Explain concurrency and scaling |
| 8 | Chapters 20–22 | Assemble the complete backend |
| 9 | Chapter 23 | Run success and failure API cases |
| 10 | Chapters 24–27 | Add relationships and ownership |
| 11 | Chapters 28–30 | Practice backup and troubleshooting |
| 12 | Chapters 31–33 | Revise and complete exercises |

### Concepts to remember

- Store data according to access patterns.
- Validate request data and enforce database uniqueness.
- ObjectId format validity does not prove a document exists or belongs to the caller.
- Use atomic conditional updates for shared counters/inventory.
- Use transactions only when a multi-document invariant requires them.
- Reuse database connections and bound query result sizes.
- Indexes improve selected reads while adding write and storage cost.
- Authentication, authorization, CORS, and database access rules solve different problems.
- Test failure paths as deliberately as successful operations.

## 35. Official references and verification scope

The examples are authored for this guide. Core syntax and practices were checked against official documentation. The complete backend is designed to be assembled from Chapter 22; specialized fragments identify their assumed models. Shell exercise results refer to the original dataset, not a database changed by earlier mutation examples.

**Validation scope:** Local structural and JavaScript syntax checks are recorded during preparation. A live MongoDB/Atlas deployment is not provisioned by this document, so database integration and deployment-specific behavior must be verified using Chapter 23.

| Topic | Official reference |
|---|---|
| Getting started | [MongoDB setup](https://www.mongodb.com/docs/get-started/) |
| Installation | [MongoDB installation](https://www.mongodb.com/docs/manual/installation/) |
| CRUD | [CRUD operations](https://www.mongodb.com/docs/manual/crud/) |
| Query predicates | [Query operator reference](https://www.mongodb.com/docs/manual/reference/mql/query-predicates/) |
| Updates | [Update operator reference](https://www.mongodb.com/docs/manual/reference/mql/update/) |
| Aggregation | [Aggregation operations](https://www.mongodb.com/docs/manual/aggregation/) |
| Data modeling | [Data modeling guide](https://www.mongodb.com/docs/manual/data-modeling/) |
| Indexing | [Indexes](https://www.mongodb.com/docs/manual/indexes/) |
| Transactions | [Transaction guide](https://www.mongodb.com/docs/manual/core/transactions/) |
| Replication | [Replica sets](https://www.mongodb.com/docs/manual/replication/) |
| Sharding | [Sharding guide](https://www.mongodb.com/docs/manual/sharding/) |
| Change streams | [Change stream guide](https://www.mongodb.com/docs/manual/changeStreams/) |
| Mongoose schemas | [Schema guide](https://mongoosejs.com/docs/guide.html) |
| Mongoose validation | [Validation guide](https://mongoosejs.com/docs/validation.html) |
| Mongoose connections | [Connection guide](https://mongoosejs.com/docs/connections.html) |
| Populate | [Population guide](https://mongoosejs.com/docs/populate.html) |
| Lean | [Lean query guide](https://mongoosejs.com/docs/tutorials/lean.html) |
| Express errors | [Express error handling](https://expressjs.com/en/guide/error-handling.html) |
| Native driver | [MongoDB Node.js driver](https://www.mongodb.com/docs/drivers/node/current/) |
| Zod | [Zod API](https://zod.dev/api) |

**Viewing diagrams:** Open this file in a Markdown viewer with Mermaid support, such as GitHub's rendered Markdown. In VS Code, use a preview extension that supports Mermaid if your current preview displays diagram source text. The file keeps diagrams in standard fenced `mermaid` blocks without HTML labels or custom rendering directives.
