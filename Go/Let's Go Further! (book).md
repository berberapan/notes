##### Chapter 2 - Getting Started

```
├── bin
├── cmd
│   └── api
│       └── main.go
├── internal
├── migrations
├── remote
├── go.mod
└── Makefile
```

This projects skeleton looks like this. 
- `bin` is where the compiled binaries for the app will be.
- `cmd` is where application specific code should be placed. I.e. running the server, reading and writing HTTP requests etc
- `internal` will house the supportive packages needed by the api. Things like interacting with the database, doing validation, sending emails etc. Only code from this project can use packages in the internal directory.
- `migrations` will contain SQL migration files for the db.
- `remote` will contain config files and setup scripts for prod server.

As in the previous book the author recommends having the handlers as methods of the app struct you set in the main function. "This is an effective and idiomatic way to make dependencies available to our handlers without resorting to global variables or closures".

When it comes to versioning for APIs there are two common ways. Either you prefix the URL with the API version, e.g /v1/, /v2/ or you use custom Accept and Content-Type headers on request and responses to convey the version, e.g `Accept: application/vnd.app-v1`. The latter is the purer when looking at HTTP semantics but the former is way clearer from a user point of view, and is very common to see being used.

httprouter that is used in the book doesn't allow conflicting routes, if you need conflicting routes use chi, gorilla mux or flow instead. Why third party router is used instead of standard library router is that the standard one sends plaintext 404 and 405 responses when matching routes can't be found even if the routes usually returns JSON and it doesn't automatically handle OPTIONS requests. These are things that may change in the future but hasn't yet as of Go 1.23.

##### Chapter 3 - Sending JSON Responses

JSON in essence is just text. It's got control characters that gives the text structure and meaning but it's just text. That means that anything that uses the io.Writer interface can write a JSON respone in a handler. The only difference from sending plain text is that you need to set the header `Content-Type` to `application/json` so the client can interpret it correctly.

When formatting the text use back-ticks as the JSON format needs the quote signs. It's also useful to use `%q` for values as it adds the quote signs with the variable value.

If you want to encode native Go objects you can use two different options from the standard library's encoding/json. Either json.Marshal() function or you can declare and use a json.Encoder type. json.Marshal is generally the better choice even though it's very slightly slower than using Encoder.

`func Marshal(v any) ([]byte, error)`, the function takes any so you can pass any Go type in this function.

Table below shows how different Go types are mapped to JSON types when encoded. Any pointer values will be encoded as the value pointed to.

| Go type                                            | ⇒   | JSON type                  |
| -------------------------------------------------- | --- | -------------------------- |
| bool                                               | ⇒   | JSON boolean               |
| string                                             | ⇒   | JSON string                |
| int*, uint\*, float\*, rune                        | ⇒   | JSON number                |
| array, slice                                       | ⇒   | JSON array                 |
| struct, map                                        | ⇒   | JSON object                |
| nil pointers, interface values, slices, maps, etc. | ⇒   | JSON null                  |
| chan, func, complex\*                              | ⇒   | Not supported              |
| time.Time                                          | ⇒   | RFC3339-format JSON string |
| []byte                                             | ⇒   | Base64-encoded JSON string |
When encoding a map it will sort it in alphabetical order. A []byte slice will be encoded as a base64 JSON string rather than an array.

When exporting fields in a struct you have to have the names start with a capital letter, otherwise they won't be encoded. If you don't specify a value for a struct field the JSON encoding will give it a zero value. If no name is set on JSON level the fields name will be assigned as JSON objects. To set the JSON name you add it to the end of the field like in the example below.

```go
type Example struct {
  ID int64 `json:"id"`
  Name string `json:"name,omitempty"`
  Password string `-`
}
```

If you want a field to be hidden if it got a zero value or is empty you can use `omitempty` in the json tag, just put `,omitempty` within the quote marks. Sometimes you want to hide the field regardless, this could be down to several things, like info you don't want to show the end user. In those cases you use `-` and no name for the field.

To make JSON easier to read for terminal use you can use json.MarshalIndent() which adds whitespace to the output, putting each element on a separate line, etc. It makes it a better user experience for people making the request through for example curl but it affects the performance as it takes more memory and takes more time to run.

There is no right or wrong on how to structure a JSON response but the author recommends enveloping responses to make it clearer what's going on. Whatever is decided to do be consistent and clear across the API endpoints.

When marshaling a type in Go it checks if it fulfils the Marshaler interface which is MarshalJSON() ([]byte, error), we can use this to customise how we want our fields to be represented. Example used in the book is a field called runtime in the response for a movie. The field is just a int32 type so it will just return a JSON number but you can create its own type so that it returns a string like "102 mins" instead of 102.

##### Chapter 4 - Parsing JSON Requests

When you're decoding JSON it usually better to work with the json.Decoder over json.Unmarshal, so other way around from the encoding.

When decoding you have to pass a non-nil pointer otherwise you'll get an error at runtime. If the target is a struct the fields must be exported just like when you encode. Any JSON key/value pairs that can't be mapped will be silently ignored.

If you are making a public facing API consider using your own errors as errors returned by decode might be too specific and show underlying structures or it could return with not enough info. The book uses a helper function to triage potential errors.

It can be a good idea to limit what a user can send in the api calls. Setting up you api you have to think about what the maximum size of a request body should be for example. If you leave it unlimited you're very vulnerable to denial-of-service attacks. Max size for the request body is set with `http.MaxBytesReader()`

Just like in the previous book the author recommends to use a validator package in the internals to handle validations of inputs.

##### Chapter 5 - Database Setup and Configuration

The book is using postgresql for its database. Creating a databse is the same as in most SQL based databases. 
`CREATE DATABASE {name};` 
and to connect to it you use the `\c` command. The backslash is used for commands in postgres. Below are a few of the many commands.

```
\c - connect to database
\l - list all databases
\dt - list tables
\du - list users
\? - full list of available commands
```

Postgres also got extensions you can add to add additional features. Some ship with postgres [found here](https://www.postgresql.org/docs/current/contrib.html) and other you need to download [found here](https://www.postgresql.org/download/products/6-postgresql-extensions/)

According to the author the default settings are quite conservative so there's often a lot of room to improve the performance of the database by tweaking values in the `postgresql.conf` file. You can read more about the settings [here](https://www.enterprisedb.com/postgres-tutorials/how-tune-postgresql-memory)

Just like in the previous book a database driver is used and then a function to establish connection with the database and ping with a context timeout of 5 seconds.

```go
func openDB(cfg config) (*sql.DB, error) {
	db, err := sql.Open("postgres", cfg.db.dsn)
	if err != nil {
		return nil, err
	}
	ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
	defer cancel()
	err = db.PingContext(ctx)
	if err != nil {
		db.Close()
		return nil, err
	}
	return db, nil
}
```

The db connection pool contains two types of connections, idle and in-use ones. Connections gets marked as in-use when they are performing a task, e.g. sending a query. When a task is completed it goes back to idle status.

