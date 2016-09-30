---
layout: post
title: "How To Build a RESTful JSON API in Go - Database Part II"
date: 2016-09-29 23:19:20 -0500
comments: false
categories: [Development, Go, Tips & Tricks]
---
In this portion of building a [RESTful JSON API with a Postgres database](https://github.com/elauqsap/echo-postgres-json-api), we explore the database powering the backend. I decided to use the builtin SQL library with a Postgres driver rather than using an ORM.

<!--more-->

### Database
Configuring the connection is important and we need to make sure it can used elsewhere. By exporting `Config` we can embed it in another configuration structure and populate them all at once using JSON tags. Plus by adding a method receiver we can easily return a connection after a configuration file has been parsed.

```go
type (
  // Config for postgres db
  Config struct {
    Auth
    Name           string `json:"name"`
    Host           string `json:"host"`
    Port           int    `json:"port"`
    SSL            string `json:"ssl_mode"`
    ConnectTimeout int    `json:"connect_timeout"`
  }
  // Store embeds an instance of a sql.DB so we can
  // inject new method receivers
  Store struct {
    *sql.DB
  }
)

// NewStore returns a configured *Store instance
func (c Config) NewStore() (*Store, error) {
	pg, err := Reverse(c.Auth)
	if err != nil {
		return nil, err
	}
	source := fmt.Sprintf("user=%s password=%s dbname=%s host=%s port=%d sslmode=%s connect_timeout=%d", pg.A, pg.B, c.Name, c.Host, c.Port, c.SSL, c.ConnectTimeout)
	db, err := sql.Open("postgres", source)
	if err != nil {
		return nil, err
	}
	if err = db.Ping(); err != nil {
		return nil, err
	}
	return &Store{db}, nil
}
```

Because Go allows embedding structures in each other I decided to embed a `*sql.DB` into a new structure so that I can expand upon it's functions. As seen above we create a new `Store{}` which contains an embedded pointer to a SQL database. From here we can create new pointer receiver methods that implement functionality that we need. In the example below I created two new methods to perform `Exec()` & `QueryRow()` with a SQL transaction.

```go
// EWT is execute with transaction
func (s *Store) EWT(st Statement) error {
	return s.Transact(func(tx *sql.Tx) error {
		res, err := tx.Exec(st.Query, st.Args...)
		if aff, _ := res.RowsAffected(); aff < 1 {
			return errors.New("no change during execution")
		}
		return err
	})
}

// QWT is query with transaction
func (s *Store) QWT(st Statement, v interface{}) error {
	return s.Transact(func(tx *sql.Tx) error {
		return tx.QueryRow(st.Query, st.Args...).Scan(v)
	})
}
```

Another important piece is the structure `PropertyMap` which is of type `map[string]interface{}`. This is necessary to read and write JSON values back to the database which we will later see in the Model implementation.

```go
// PropertyMap allows to map and store JSON data with Postgres
type PropertyMap map[string]interface{}

// Value map to a sql driver value
func (p PropertyMap) Value() (driver.Value, error) {
	j, err := json.Marshal(p)
	return j, err
}

// Scan map from sql return to a PropertyMap
func (p *PropertyMap) Scan(src interface{}) error {
	source, ok := src.([]byte)
	if !ok {
		return errors.New("Type assertion .([]byte) failed.")
	}

	var i interface{}
	err := json.Unmarshal(source, &i)
	if err != nil {
		return err
	}

	*p, ok = i.(map[string]interface{})
	if !ok {
		return errors.New("Type assertion .(map[string]interface{}) failed.")
	}

	return nil
}
```

### Migrations
The migration component allows the application to be portable while also serving a function in testing. It allows the test database to be easily wiped and the schema to be reconfigured. Which helps in making sure the test outcomes are always the same and no previous tests interfere with the returns.


Because the schema needs to be rebuilt in an order, I created a migration method which sorts each level of the map by their keys and then performs the same sorting on the inner map. This way all you need to do to expand upon the migration is to add your SQL statement with the given order of operation it needs.
```go
// migrations to perform
var migrations = map[int]map[int]string{
	// schema commands in the order to be performed
	mSCHEMA: map[int]string{
		0: `DROP SCHEMA IF EXISTS app CASCADE`,
		1: `CREATE SCHEMA IF NOT EXISTS app AUTHORIZATION appbot`,
	},
	// type commands in the order to be performed
	mTYPE: map[int]string{
		0: `CREATE TYPE app.roles AS ENUM ('user','manager','admin')`,
	},
	// table commands in the order to be performed
	mTABLE: map[int]string{
		0: `CREATE TABLE IF NOT EXISTS app.users (
					id SERIAL PRIMARY KEY,
					first varchar(100) NOT NULL CHECK (first <> ''),
					last varchar(100) NOT NULL CHECK (last <> ''),
					role app.roles NOT NULL DEFAULT 'user',
					api_key char(32) NOT NULL UNIQUE
			)`,
	},
	// index commands in the order to be performed
	mINDEX: map[int]string{
		0: `CREATE INDEX role_idx ON app.users (role)`,
	},
}
```

### Modeling
All that is left now is to build your models and make sure they implement the `CRUD` interface. We only have one model for this example but they should all be the same layout just different structures. As each structure should model the table it will be working with. The JSON tags on each element allow us to easily bind data so that it can be used by both the API and database.

Each `CRUD` pointer receiver method as defined by the interface should return a `Statement`. This is essentially the `query` & `args` parameters of the `*sql.DB`'s `Exec()` & `QueryRow()`.

```go
type (
	// User models the app.users table
	User struct {
		ID    int    `json:"id"`
		First string `json:"first"`
		Last  string `json:"last"`
		Role  string `json:"role"`
		Key   string `json:"api_key"`
	}
)

// Create builds the Statement to insert a user into the database
func (u *User) Create() Statement {
	u.GenerateKey(32)
	if len(u.Role) <= 0 {
		u.Role = USER
	}
	return Statement{
		"INSERT INTO app.users (first,last,role,api_key) VALUES ($1,$2,$3,$4)",
		[]interface{}{u.First, u.Last, u.Role, u.Key},
	}
}

// Read creates the Statement to read a user from the database
func (u *User) Read() Statement {
	return Statement{
		"SELECT ROW_TO_JSON(u) FROM (SELECT * FROM app.users WHERE id = $1) AS u",
		[]interface{}{u.ID},
	}
}

// Update creates the Statement to update a user in the database
func (u *User) Update(v interface{}) Statement {
	// no merging needed to ignore v
	return Statement{
		"UPDATE app.users SET first = $1,last = $2,role = $3 WHERE id = $4",
		[]interface{}{u.First, u.Last, u.Role, u.ID},
	}
}

// Delete creates the Statement to delete a user from the database
func (u *User) Delete() Statement {
	return Statement{
		"DELETE FROM app.users WHERE id = $1",
		[]interface{}{u.ID},
	}
}
```
