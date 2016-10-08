---
layout: post
title: "How To Build a RESTful JSON API in Go - Testing Part I"
date: 2016-09-27 22:11:53 -0500
comments: false
categories: [Development, Go, Tips & Tricks]
---
In my recent endeavor with Go I needed to create a [RESTful JSON API with a Postgres database](https://github.com/elauqsap/echo-postgres-json-api). This blog series outlines what I learned and the methods chosen to implement the API. First up in this blog series demonstrates the implementation of Behavioral Driven Development utilizing [GoConvey](https://github.com/smartystreets/goconvey).
<!--more-->

### Disclaimer
A true RESTful service uses HTTP methods coupled with Unified Resource Identifiers to traverse an application. In this example, the input data from the client is also JSON.

### GoConvey Setup
There are two primary parts to test within this example and I decided to break them up into "sub-packages". These "sub-packages" include api for handling the RESTful aspects and database for communicating with Postgres instance. Go makes testing very easy by sourcing any file `*_test.go` and running any test functions that start with `Test*`. For both "sub-packages" I created a single Test function and created a general structure to model each test structure.

Take the `database` sub-package for example. There is only one test function and that is in `database_test.go`. The other test file only contains test cases to be conveyed.
```bash
database
├── database.go
├── database_test.go
├── migrate.go
├── user.go
└── user_test.go
```
Diving deeper into `TestDatabase()`, we test out database connection items and migrate our schema for the test environment. The final portion of the function takes `[]ModelTest` and runs each test case.

```go
type (
	ModelTest struct {
		Title string
		Func  func(*Store) func()
	}
	TestConfig struct {
		Config `json:"database"`
	}
)

var Conf TestConfig
var Data *Store

func TestDatabase(t *testing.T) {
	Convey("The Database Should", t, func() {
		Convey("Be Configurable From A JSON File", func() {
			data, err := ioutil.ReadFile("../configs/example.config.json")
			So(err, ShouldBeNil)
			So(json.Unmarshal(data, &Conf), ShouldBeNil)
			So(Conf, ShouldNotBeEmpty)
			Data, err = Conf.NewStore()
			So(err, ShouldBeNil)
			So(Data, ShouldNotBeNil)
		})
		Convey("Have Migrations For The Schema", func() {
			So(Data.Migrate(false), ShouldBeNil)
		})
	})
	var modelTests = []ModelTest{UserTest}
	for _, model := range modelTests {
		Convey("The Database "+model.Title, t, model.Func(Data))
	}
}
```
As you can see the `ModelTest` structure is in essence the parameters necessary to run `Convey()`. Since `*testing.T` was passed in the outermost `Convey()` it is not necessary for these nested tests. There for we can pass it a `string` conveying what it will test and some arbitrary test `func()`. Below is the test of the `POST` portion of the RESTful interface for the user model. The other conveyed tests in `user_test.go` check `GET`, `PUT`, & `DELETE` implementation.

```go
var UserTest = ModelTest{
	Title: "User Model Should",
	Func: func(store *Store) func() {
		return func() {
			Convey("Implement The CRUD Interface", func() {
				So(&User{}, ShouldImplement, (*CRUD)(nil))
				Convey("A User Can Be Created", func() {
					So(store.EWT(user.Create()), ShouldBeNil)
					read := new(User)
					pm := new(PropertyMap)
					So(store.QWT(user.Read(), pm), ShouldBeNil)
					data, err := json.Marshal(pm)
					So(err, ShouldBeNil)
					So(json.Unmarshal(data, read), ShouldBeNil)
					So(read, ShouldResemble, user)
				})
      // Other Model tests go here ...
      })
    }
  },
}
```
### Running Tests
After writing all of our test cases for the model, we can implement the model using red to green testing. I will go into implementing a model at a later date but below is the output you would get after running `go test -v` with completed code.
```bash
=== RUN   TestDatabase

  The Database Should
    Be Configurable From A JSON File ✔✔✔✔✔
    Have Migrations For The Schema ✔


6 total assertions


  The Database User Model Should
    Implement The CRUD Interface ✔
      A User Can Be Created ✔✔✔✔✔✔
      A User Can Be Read ✔✔✔✔✔
      A User Can Be Updated ✔✔✔✔✔✔
      A User Can Be Deleted ✔✔


26 total assertions

--- PASS: TestDatabase (0.08s)
PASS
ok      github.com/elauqsap/echo-postgres-json-api/database     0.083
```
