---
title: "Testing PostgreSQL Databases With Go"
date: 2020-11-29T19:51:00-05:00
type: post
author: James
categories:
  - Software Development
tags:
  - unit tests
  - PostgreSQL
  - Go
comments: true
---

While setting up a Go project for database access using the PostgreSQL database I was looking for an efficient way to quickly unit test different database access functions without a lot of extra setup or time for each test. Also to make the tests more useful it helps if the actual Postgres drivers and database is used.

For this purpose I found an interesting [Go package called txdb](https://github.com/DATA-DOG/go-txdb) that's described as a **Single transaction based sql.Driver for Go**. It allows to run a series of SQL commands in a test that get automatically rolled back after the test is complete. This allows the next test to start with pristine database conditions to keep the test independent. In practice I found this very useful and it helped me more quickly develop a correct database schema and database access code.

#### How to Use txdb

When setting up the tests the database must first be registered as with txdb via the Register function, and that is best done in an init() function, e.g.

{{< highlight "go" >}}
func init() {
  txdb.Register("txdb", "postgres", postgresDSN)
}
{{< / highlight >}}

Where "txdb" is the identifier that will be passed to the Open for each test, "postgres" is the driver, and postgresDSN contains the connection string for the Postgres DB.

Then in each test the "txdb" will be passed to the sql.Open function and the second argument that is normally the DSN is an identifier that is passed to the connection pool.

{{< highlight "go" >}}
func TestCreateUserAndCategories(t *testing.T) {
  db, err := sql.Open("txdb", "identifier")
  defer db.Close()
  // Normal db commands and testing code follows...
}
{{< / highlight >}}

The defer command ensures that the db will close and txdb will clear the transactions at the end of the test. Repeat the same pattern for all other database tests.
