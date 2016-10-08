---
layout: post
title: "How To Build a RESTful JSON API in Go - API Part III"
date: 2016-10-06 18:03:28 -0500
comments: false
categories: [Development, Go, Tips & Tricks]
---
Concluding our build of a [RESTful JSON API with a Postgres database](https://github.com/elauqsap/echo-postgres-json-api), we will explore the code that binds the backend queries to the HTTP methods. Instead of using the builtin networking tools for HTTP, I decided to use [echo](https://echo.labstack.com/).

<!--more-->

### Disclaimer
A true RESTful service uses HTTP methods coupled with Unified Resource Identifiers to traverse an application. In this example, the input data from the client is also JSON.

### Server
As we discussed in the previous part of the tutorial, by adding JSON tags we can embed the database configuration. That way we can load both configs from the same file.

```go
type (
	// Config for the server and database
	Config struct {
		Server struct {
			Bind    string `json:"bind"`
			Cert    string `json:"cert"`
			Key     string `json:"key"`
			LogPath string `json:"log,omitempty"`
		} `json:"server"`
		Database database.Config `json:"database"`
	}
)
```
With our configuration file modeled, all we need to do is read it into memory and `Unmarshal()` the data into the structure. From there we can call the method receivers on both `Database{}` and `Server{}` for setup. `NewServer()` creates an echo instance, binds middleware, and groups routes for the User API. It also calls `database.Config.NewStore()` which creates and passes back a connection to the database.

```go
// NewConfig loads the configurations into a structure
// to be used as a global operator at runtime
func NewConfig(path string) (conf *Config) {
	data, _ := ioutil.ReadFile(path)
	if err := json.Unmarshal(data, &conf); err != nil {
		return nil
	}
	return conf
}

// NewServer returns a configured api server instance
func (c *Config) NewServer() (*echo.Echo, error) {
	e := echo.New()
	if logFile, err := os.OpenFile(c.Server.LogPath, os.O_WRONLY|os.O_APPEND|os.O_CREATE, 0660); err != nil {
		e.Use(middleware.Logger())
	} else {
		e.Use(middleware.LoggerWithConfig(middleware.LoggerConfig{Output: logFile}))
	}
	e.Use(middleware.Recover())
	api := e.Group("/api/v1")
	if store, err := c.Database.NewStore(); err == nil {
		data := &Data{store}
		handlers := &Handlers{User: data}
		api.POST("/user", handlers.User.CreateUser)
		api.GET("/user", handlers.User.ReadUser)
		api.PUT("/user", handlers.User.UpdateUser)
		api.DELETE("/user", handlers.User.DeleteUser)
	} else {
		return nil, err
	}
	return e, nil
}
```

### User API
In `NewServer()` there is a variable called `handlers` which is where all of the HTTP handlers for the echo routes are defined. `Handlers{}` are essentially a definition of the CRUD methods and some defined type. They can be used because `*database.Store{}` is embedded in `Data{}` and the interface `UserCRUD{}` is implemented as pointer receiver methods for `Data{}`. Therefore by initializing `Handlers.User` with `*Data{}`, the pointer receiver methods are available to `Handlers.User` so they can be passed as a handler for the route.

```go
type (
	// Handlers are the HTTP handlers to be used by the API router
	Handlers struct {
		User UserCRUD
	}
)
```
All that is left to do from here is implement each CRUD method of the `UserCRUD` interface. The example below shows how I implemented `CreateUser()`. Since `echo.Context` is passed to this pointer method I use an anonymous structure to bind to the `POST` data. That way I can control which data the user actually gets to define. If binding fails the server sends an `400` error and returns. If it passes then a `database.User` is created from the request. Since this is a pointer method with the database embedded in it, all that is left is to pass the `database.Query` that represents the newly defined `database.User`. Depending on the transaction response, a success or error message is sent back.

```go
func (d *Data) CreateUser(c echo.Context) error {
	bind := struct {
		First string `json:"first"`
		Last  string `json:"last"`
		Role  string `json:"role"`
	}{}
	if err := c.Bind(&bind); err != nil {
		return c.JSON(http.StatusBadRequest, map[string]interface{}{
			"code":    1010,
			"message": "invalid user or json format in request body",
		})
	}
	user := &database.User{First: bind.First, Last: bind.Last, Role: bind.Role}
	if err := d.EWT(user.Create()); err != nil {
		return c.JSON(http.StatusInternalServerError, map[string]interface{}{
			"code":    1011,
			"message": "user could not be created",
		})
	}
	return c.JSON(http.StatusOK, map[string]interface{}{
		"code":    1000,
		"message": "user was successfully created",
	})
}
```
There are multiple ways to create a RESTful API in golang even though it is an opinionated language. I came from a Ruby on Rails background and I found this structure to be similar.
