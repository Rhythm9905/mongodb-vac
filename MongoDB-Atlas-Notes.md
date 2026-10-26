# MongoDB Atlas & Mongosh — Practical Notes

> Clean notes prepared from a MongoDB Atlas terminal session. The original connection hostname and database username are intentionally not included. Never commit database passwords or private connection strings to Git.

## 1. Connect to MongoDB Atlas

Open Command Prompt and navigate to the folder containing `mongosh.exe` if `mongosh` is not available in PATH.

```bat
cd "C:\path\to\mongosh\bin"
```

Connect using the connection string and database username from MongoDB Atlas:

```bash
mongosh "<YOUR_ATLAS_CONNECTION_STRING>" --apiVersion 1 --username "<DATABASE_USERNAME>"
```

Enter the password when prompted. Do not write the password directly in a command that you plan to save or push to GitHub.

The transcript showed MongoDB Server **8.0.34** and Mongosh **2.10.0** at the time of that session.

## 2. Basic database commands

| Command | Purpose |
|---|---|
| `show dbs` | Display databases available to the connected user. |
| `use Poornima` | Switch to the `Poornima` database. MongoDB creates it on disk when data is first stored if it does not already exist. |
| `show collections` | List collections in the current database. |
| `db.Students.find()` | Display documents in the `Students` collection. |
| `db.Company.find()` | Display documents in the `Company` collection. |
| `db.getCollection("Students").find()` | Access a collection by passing its name to `getCollection()`. |

Example workflow:

```javascript
show dbs
use Poornima
show collections
```

Collections shown in the session included `Company`, `Students`, `Product`, and `products`.

## 3. Read documents with `find()`

### Display all documents

```javascript
db.Students.find()
db.Company.find()
db.products.find()
```

Add parentheses to call `find()`. Writing `db.Company.find` without `()` displays the function itself rather than querying the collection.

### Example student document

The `Students` collection contained documents with student information and an embedded `address` object:

```javascript
{
  name: "Riddhima",
  age: 21,
  cgpa: 9,
  address: {
    city: "Jaipur",
    pincode: 302020
  }
}
```

- `name`, `age`, and `cgpa` are regular fields.
- `address` is an embedded document, with `city` and `pincode` as its fields.
- MongoDB automatically assigns an `_id` field when one is not supplied.

## 4. Create a collection

```javascript
db.createCollection("Product")
```

This creates a collection named `Product`.

**Important lesson from the session:** MongoDB collection names are case-sensitive. The transcript created `Product` but inserted records into `products`:

```javascript
db.createCollection("Product")
db.products.insertMany([...])
```

These are different collection names. As a result, `db.Product.find()` did not show the documents inserted into `db.products`. Choose one name and use it consistently.

## 5. Insert multiple documents with `insertMany()`

The practical used a `products` collection with fields for product name, production year, processor information, camera specifications, and availability. Nested details were stored as embedded documents.

Example document structure:

```javascript
{
  Name: "Iphone 11",
  ProductionYr: 2024,
  ProcessorDetails: {
    Processor: "Gen7",
    CPU: 6,
    Threads: 12
  },
  CameraSpec: {
    Camera: "50px",
    Sensor: "CMOS",
    MotionCapture: true
  },
  Available: true
}
```

Example insertion:

```javascript
db.products.insertMany([
  {
    Name: "Iphone 11",
    ProductionYr: 2024,
    ProcessorDetails: { Processor: "Gen7", CPU: 6, Threads: 12 },
    CameraSpec: { Camera: "50px", Sensor: "CMOS", MotionCapture: true },
    Available: true
  },
  {
    Name: "Iphone 12",
    ProductionYr: 2023,
    ProcessorDetails: { Processor: "Gen6", CPU: 4, Threads: 8 },
    CameraSpec: { Camera: "48px", Sensor: "IR Grade", MotionCapture: false },
    Available: true
  }
])
```

The actual session inserted an initial batch of **10** product documents and then another batch of **20**, for **30** documents in `products`.

## 6. Count documents

```javascript
db.products.countDocuments()
db.Students.countDocuments()
```

`countDocuments()` returns the number of documents matching the filter; with no filter, it counts all documents in that collection.

In the session, `db.products.countDocuments()` returned `10` after the first insertion and the product collection later contained `30` documents. The command `db.student.countDocuments()` returned `0` because the collection in the database was named `Students`, not `student`.

## 7. Filter documents

### Match a Boolean field

Find products that are available:

```javascript
db.products.find({ Available: true })
```

### Use the `$gt` comparison operator

Find products with a production year greater than 2022:

```javascript
db.products.find({ ProductionYr: { $gt: 2022 } })
```

`$gt` means **greater than**. The condition is applied to the `ProductionYr` field.

### Query a nested field

Find products where the nested `Processor` field is `Gen11`:

```javascript
db.products.find({ "ProcessorDetails.Processor": "Gen11" })
```

Use dot notation to query a field inside an embedded document.

## 8. Projection — select fields to display

Display only the product name and production year:

```javascript
db.products.find({}, { Name: 1, ProductionYr: 1 })
```

- The first `{}` is the filter, so it matches all documents.
- The second object is the projection.
- `1` includes the specified field in the output.
- MongoDB includes `_id` by default unless it is explicitly excluded.

## 9. Aggregation — group and count

Count the number of product documents for each production year:

```javascript
db.products.aggregate([
  {
    $group: {
      _id: "$ProductionYr",
      total: { $sum: 1 }
    }
  }
])
```

Explanation:

- `aggregate([...])` runs an aggregation pipeline.
- `$group` groups documents that have the same production year.
- `_id: "$ProductionYr"` makes the production year the group key.
- `$sum: 1` adds one for every document in a group, producing a count named `total`.

The session's output showed counts grouped by years from 2016 through 2025.

## 10. Sort documents

Sort products by production year in descending order (newest year first):

```javascript
db.products.find().sort({ ProductionYr: -1 })
```

- `1` sorts in ascending order.
- `-1` sorts in descending order.

## 11. Common errors and fixes

| Mistake in the session | Correct approach | Why |
|---|---|---|
| `show Collection` or `show Collections` | `show collections` | Use the supported shell command exactly as shown. |
| `db.getCollection()` | `db.getCollection("Students")` | Supply the collection name as an argument. |
| `db.Company.find` | `db.Company.find()` | Parentheses call the function. |
| `db.Product.find()` after inserting into `db.products` | Use `db.products.find()` or standardize the name | `Product` and `products` are distinct names. |
| `db.product.find()` | Use the exact collection name, e.g. `db.products.find()` | Collection-name capitalization and spelling matter. |
| `db.student.countDocuments()` returns `0` | Check `show collections`, then use `db.Students.countDocuments()` | The collection in the session was `Students`. |

## 12. Quick revision sheet

| Concept | Command / syntax |
|---|---|
| List databases | `show dbs` |
| Select a database | `use Poornima` |
| List collections | `show collections` |
| Read all documents | `db.products.find()` |
| Insert many documents | `db.products.insertMany([...])` |
| Count documents | `db.products.countDocuments()` |
| Filter by Boolean value | `db.products.find({ Available: true })` |
| Greater-than comparison | `db.products.find({ ProductionYr: { $gt: 2022 } })` |
| Query nested field | `db.products.find({ "ProcessorDetails.Processor": "Gen11" })` |
| Project selected fields | `db.products.find({}, { Name: 1, ProductionYr: 1 })` |
| Group and count | `db.products.aggregate([{ $group: { _id: "$ProductionYr", total: { $sum: 1 } } }])` |
| Sort descending | `db.products.find().sort({ ProductionYr: -1 })` |

## 13. Push these notes to GitHub

Save this file as `MongoDB-Atlas-Notes.md` inside your repository, then run these commands from the repository folder:

```bash
git add MongoDB-Atlas-Notes.md
git commit -m "Add MongoDB Atlas practical notes"
git push origin main
```

If your default branch is not named `main`, replace `main` with your branch name.

## Security reminder

Do not commit Atlas passwords, private connection strings, API keys, or other credentials. Use placeholders in notes and keep real credentials outside the repository.
