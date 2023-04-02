# HW: NoSQL & MongoDB

**All answers are in this README.md**

## Question 1

> You're creating a database to contain a set of sensor measurements from a two-dimensional grid.
> Each measurement is a time-sequence of readings, and each reading contains ten labeled values.
> Should you use the relational model or MongoDB? Please justify your answer.

Since our requirements is to collect data from sensor measurements, we are ensuring that data collection is rapid and continuous.
We do not need to care for accuracy since raw measurements can be used to incorporate for any other calculations anyways.

**Since we expect speed to store data, we should be using MongoDB instead.** (or even better, a real time-series database.)

## Question 2

> For each of the following applications, IoT, E-commerce, Gaming and Finance.
>
> Propose an appropriate Relational Model or MongoDB database schema. For each application, clearly justify your choice of database.

* **IoT**
  Internet-of-Things are mostly consisted of sensors and real-time measurements. We always expect speed in data updates. We should be using MongoDB. The following is an example of application schemas:

  Sensor schema
  ```ts
  {
    _id: uuid,
    name: string,
  }
  ```

  Measurement schema
  ```ts
  {
    _id: uuid,
    sensor_id: uuid,
    parameters: {
      param1: object,
      param2: object,
      ...
    }
  }
  ```

* **E-commerce**
  E-commerce website depends on reliabilty between sellers, customers and system. Almost any purchase should be recorded to not be lost.
  We don't expect data to be lost as they are sensitive to the service. So we are using relational database.

  User schema
  ```sql
  (
    UserID int NOT NULL PRIMARY KEY,
    Username varchar(255) NOT NULL,
    Address varchar(255),
    City varchar(255)
  )
  ```

  Store schema
  ```sql
  (
    StoreID int NOT NULL PRIMARY KEY,
    StoreName varchar(255) NOT NULL,
    Seller int FOREIGN KEY REFERENCES User(UserID)
  )
  ```

  Product schema
  ```sql
  (
    ProductID int NOT NULL PRIMARY KEY,
    ProductName varchar(255) NOT NULL,
    ProductDescription varchar(1024) NOT NULL,
    SoldBy int FOREIGN KEY REFERENCES Store(StoreID)
  )
  ```

  Transaction schema
  ```sql
  (
    TransactionID int NOT NULL PRIMARY KEY,
    Buyer int FOREIGN KEY REFERENCES User(UserID),
    ProductBought int FOREIGN KEY REFERENCES Product(ProductID),
    TimestampOccured datetime NOT NULL PRIMARY KEY,
    Amount int NOT NULL
  )

* **Gaming**
  In online games, there are many things happening inside. Sometimes, servers might want to capture events occured by players or changes of game world.
  For practical use, it could have been a hybrid between relational database and NoSQL; but for ease, only MongoDB can also suffice.

  Player schema
  ```ts
  {
    _id: uuid,
    name: string,
    registeredAt: Date
  }
  ```

  Event schema
  ```ts
  {
    _id: uuid,
    occuredAt: Date,
    event: string,
    playerAssociated: uuid
  }
  ```

* **Finance**
  With finance website we require the record transactions, amount transfers and bookkeeping. Reliability to store data is import here.
  We'll be using relational database.

  Account schema
  ```sql
  (
    AccountID int NOT NULL PRIMARY KEY,
    Holder varchar(255) NOT NULL,
    PhoneNumber varchar(16) NOT NULL,
    Amount int
  )
  ```

  Transaction schema
  ```sql
  (
    TransactionID int NOT NULL PRIMARY KEY,
    AccountAssociated int FOREIGN KEY REFERENCES Account(AccountID),
    AmountChanged int NOT NULL
    OccuredAt datetime NOT NULL PRIMARY KEY
  )
  ```

## Question 3

> Create MongoDB database with following information.

Let `marks` be the name for collection.

Create database with the following function,
```sh
db.marks.insertMany([
    {"name":"Ramesh","subject":"maths","marks":87},
    {"name":"Ramesh","subject":"english","marks":59},
    {"name":"Ramesh","subject":"science","marks":77},
    {"name":"Rav","subject":"maths","marks":62},
    {"name":"Rav","subject":"english","marks":83},
    {"name":"Rav","subject":"science","marks":71},
    {"name":"Alison","subject":"maths","marks":84},
    {"name":"Alison","subject":"english","marks":82},
    {"name":"Alison","subject":"science","marks":86},
    {"name":"Steve","subject":"maths","marks":81},
    {"name":"Steve","subject":"english","marks":89},
    {"name":"Steve","subject":"science","marks":77},
    {"name":"Jan","subject":"english","marks":0,"reason":"absent"}
])
```

### Give MongoDB statements (with results) for the following queries

* Find the total marks for each student across all subjects.
  ```sh
  db.marks.aggregate([
    {
      $group: { _id: "$name", totalMarks: { $sum: "$marks" } }
    }
  ])
  ```

* Find the maximum marks scored in each subject.
  ```sh
  db.marks.aggregate([
    {
      $group: { _id: "$subject", maxMarks: { $max: "$marks" } }
    }
  ])
  ```

* Find the minimum marks scored by each student.
  ```sh
  db.marks.aggregate([
    {
      $group: { _id: "$name", minMarks: { $min: "$marks" } }
    }
  ])
  ```

* Find the top two subjects based on average marks.
  ```sh
  db.marks.aggregate([
    {
      $group: {
        _id: "$subject",
        avgMarks: { $avg: "$marks" }
      }
    },
    {
      $sort: { avgMarks: -1 }
    },
    {
      $limit: 2
    }
  ])
  ```