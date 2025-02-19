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

The book is using postgresql for its database. Creating a database is the same as in most SQL based databases. 
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

The connection pool got four methods that you can configure.

- SetMaxOpenConns(), allows you set an upper limit for open connections. Default value for it is unlimited. An open connection counts as all idle ones plus the in-use ones. The higher the value the more queries can be performed concurrently and lower risk of the pool becoming a bottleneck for the application. By default postgres got a hard limit of 100 open connections (it can be changed in postgresql.conf). To avoid HTTP requests to hang if no connection is available it's important to set a timeout with context.
- SetMaxIdleConns(), upper limit on number of idle connections. The default value is 2. Allowing idle connections improve performance because it's less likely that a new connection needs to be established. But keeping idle connections takes up memory and if a connection idles too long it may become unusable. Only keep connections idle if they're likely to get used again. Max idle connections should always be less or equal to max open connections.
- SetConnMaxLifetime(), limit for the time that a connection can be reused. Default value is no limit and connections will be reused forever. Once every second Go runs a cleanup op that removes expired connections from the pool. 
- SetConnMaxIdleTime(), limit for how long a connection can be idle before getting marked as expired.

In general higher open conns and max idle conns values lead to better performance but returns will be diminishing, so avoid having too large of a idle connection pool as it could lead to reduced performance. Therefore always set a ConnMaxIdleTime.

##### Chapter 6 - SQL Migrations

The concept of SQL migrations works like this:

For every change that you make to your database schema you create a pair of migration files. One file is the 'up' one which contains the SQL statements to implement the change, and the other is the 'down' one which contains the SQL statements to roll back the change.

The pair of migration files are usually numbered sequentially or with a Unix timestamp to indicate in which order they should be applied.

A tool or script is usually used to apply changes or rollbacks so only the necessary SQL statements are executed.

Benefits of using migrations is that the database schema is described by the SQL migration files and because they are just regular files they can be tracked along with the rest of the code in version control. It's also easy to replicate current database on other machines. And of course the possibility to roll back changes.

The book uses golang-migrate as a tool for migration.
`migrate create -seq -ext=.sql -dir=./migrations create_movies_table`
- -seg flag gives the files numbering instead of default unix timestamps
- -ext flag indicates that the files should have .sql extension
- -dir flag indicates where to store the files
- lastly is the name of the file.
Two files will be created, one up and one down.

Bigserial is used for type for primary key and Go's equivalent is int64.

Null values can be a bit awkward to work with in Go so try to set NOT NULL restrictions when possible in your database.

The datatype to use for strings in Postgres should preferably be text over varchar and varchar(x). [Blog comparing them](https://www.depesz.com/2010/03/02/charx-vs-varcharx-vs-varchar-vs-text/)

To execute the migration files against the database
`migrate -path=./migrations -database=$GREENLIGHT_DB_DSN up`

you can use down to roll back everything.

To check which version your db is running
`migrate -path=./migrations -database=$GREENLIGHT_DB_DSN version`

If you want to migrate to a specific version you type out the version.
`migrate -path=./migrations -database=$GREENLIGHT_DB_DSN goto 1`

If you make a syntax error in the SQL migration file it can be a bit confusing to fix. The migration tool will work up to the syntax error which means that a half of the statements can have been applied. The dirty field in schema_migrations table will be set to true. If this happens you can't just use a down migration but rather have to investigate where the error happened and what partial was applied and manually roll back those. Then you must force the version number in schema_migrations to the correct value.

`migrate -path=./migrations -database=$EXAMPLE_DSN force 1`

After the force the database is considered clean again.

You can also use migration files from remote sources
Here are a few examples

```bash
migrate -source="s3://<bucket>/<path>" -database=$EXAMPLE_DSN up
migrate -source="github://owner/repo/path#ref" -database=$EXAMPLE_DSN up
migrate -source="github://user:personal-access-token@owner/repo/path#ref" -database=$EXAMPLE_DSN up
```

##### Chapter 7 - CRUD Operations

**Create** 
Inserting into a postgres database you use $N notations to represent the placeholder parameters. It's important to use these for untrusted data to avoid SQL injection attacks. 

If you want to make more than one statement in the same call pq supports that but you can't use the $N notations. If you need those you have to make separate calls in some way.

```go
INSERT INTO movies (title, year, runtime, genres) 
VALUES ($1, $2, $3, $4)
RETURNING id, created_at, version
```

Returning is a postgres clause that is not part of SQL standard that makes it so you can use the return values from any record that is being manipulated. Remember to use QueryRow() if using returning as it will return a row of data.

If you want to store a slice of strings or ints for example, like is done with genres in the book example, you have to pass it through pq.Array() before executing the query.

```go
func (m MovieModel) Insert(movie *Movie) error {
    query := `
        INSERT INTO movies (title, year, runtime, genres)        VALUES ($1, $2, $3, $4)        RETURNING id, created_at, version`

    args := []any{movie.Title, movie.Year, movie.Runtime, pq.Array(movie.Genres)}

    return m.DB.QueryRow(query, args...).Scan(&movie.ID, &movie.CreatedAt, &movie.Version)
}
```

**Read**
If you're using an auto-incrementing id it should never be a value below 1 by default so to avoid unnecessary calls to the database you can set an if check for your get method to see that the value isn't below 1. 

So why not just use a uint instead of int64 as the ID in the Go code? The answer is that is best practice to try to align the Go code types with postgres' to avoid overflows or other compatibility issues. Postgres doesn't have uint.

| PostgreSQL type           | Go type                                               |
| ------------------------- | ----------------------------------------------------- |
| `smallint`, `smallserial` | `int16` (-32768 to 32767)                             |
| `integer`, `serial`       | `int32` (-2147483648 to 2147483647)                   |
| `bigint`, `bigserial`     | `int64` (-9223372036854775808 to 9223372036854775807) |
Another reason is that the database/sql package doesn't support integer values greater than the biggest int64 number and biggest uint64 would be bigger than that and generate runtime errors if it was used.

**Update**
When updating you can use the helpers you already using from the create and read. Update the values, preferably by taking a pointer and mutate it in-place.

All in all the method will be very similar to a create one, especially if the functionality is such as that you need to give all the parameters.

**Delete**
Just like when reading we need to check if the record exists and we can do the 1 check again to avoid unnecessary calls.

As we're not looking for any query return for the delete we should use the Exec(). Exec actually return a sql.Result object which contains info on how many rows the query affected. If the rows affected is 1, the delete has been succesful but if it's 0 it means the value didn't exist. Use that to give info to the user, a zero can result in a 404 for example.

For the status code to send back with a successful request the author recommends 200 if the users are mainly humans, with a message of "successfully deleted" or similar. If on the other hand it's mainly machines you can go with a 204 - No content and an empty response body.

##### Chapter 8 - Advanced CRUD Operations

Last chapter the update required all fields to update the value but it's more user friendly to allow partial updates if not all values are getting changed. If not all values are given how are we supposed to tell if the value has a zero-value or it's not just provided. 

To solve this we can use pointers for the input. A pointer's zero value is nil. So instead of figuring out if a zero for an int is the intent from the user or a zero-value we can make the value a pointer. When grabbing the values and assigning them to the struct we're working with we can do a simple if check to see that they're not nil and assign them. When parsing the request only new values that are not nil will go through.

Partial updates should technically be of the PATCH value.

Unless you write your own JSON parser you will have one type of request that won't work as intended. If the user adds null as a value the parser will go through without changing the values. Author says in most cases it suffice to explain the behaviour in the documentation.

You have to account for race conditions. Using http.Server each http request gets handled in their own goroutine. So if two update requests happens at the exact same time it can create issues.
The author chooses an optimistic locking as a solution by using the version number in the record. So the WHERE clause gets updated so it only find something if version number matches.

Status code should be 409 - Conflict with the error generated.

As mentioned before, use context to timeout things. For CRUD operations you should add a context to timeout queries if they haven't been completed within a certain time.

##### Chapter 9 - Filtering, Sorting, and Pagination

When using the api you want to be able to set parameters for how the output will be by setting query sting parameters.
`/v1/movies?title=godfather&genres=crime,drama&page=1&page_size=5&sort=-year` example from the book.

Create helper functions to read the query string for the different values. Parameters not specific to the section can be its own Filter struct. 

Example of a helper function to read string value from query string.

```go
func (app *application) readString(qs url.Values, key string, defaultValue string) string {
    s := qs.Get(key)
    if s == "" {
        return defaultValue
    }
    return s
}
```

Validation is important to set as well to have some sanity checks for the values the user put in.

To set optional values with title and genres in this case the book uses this query

```sql
SELECT id, created_at, title, year, runtime, genres, version
FROM movies
WHERE (LOWER(title) = LOWER($1) OR $1 = '') 
AND (genres @> $2 OR $2 = '{}') 
ORDER BY id
```

@> is the contains operator for postgres, it returns true if each value in the placeholder appears in the database.

The problem with the above sql is that the search in title needs to match the exact title. There are different ways around this. The author recommends using to_tsvector that splits the string into lexemes. `The Breakfast Club` -> `'breakfast' 'club' 'the'`

```sql
SELECT id, created_at, title, year, runtime, genres, version
FROM movies
WHERE (to_tsvector('simple', title) @@ plainto_tsquery('simple', $1) OR $1 = '') 
AND (genres @> $2 OR $2 = '{}')     
ORDER BY id
```

@@ is a matches operator. plainto_tsquery() takes the search value and turns into a formatted query, it strips special characters and inserts & between the words. So with the @@ operator to_tsvector and plainto_tsquery have to match all the words. 

You might want to run the lexemexes in another language you can add it in the query.
`WHERE (to_tsvector('english', title) @@ plainto_tsquery('english', $1) OR $1 = '')`

The pagination can be done with limit and offset in the query.

LIMIT = page_size
OFFSET = (page - 1) * page_size

##### Chapter 10 - Rate Limiting

The book uses a package x/time/rate to help with the rate limiting. It uses a token bucket rate limiter.

> A Limiter controls how frequently events are allowed to happen. It implements a “token bucket” of size `b`, initially full and refilled at rate `r` tokens per second.

The bucket starts with b tokens in it, and every request will remove one token from the bucket. Every second the bucket gets refilled with r tokens up to the size of b. If the bucket gets empty the api should return a 429 - Too many requests.

You build a middleware that takes the NewLimiter to make a global limiter like below.

```go
func (app *application) rateLimit(next http.Handler) http.Handler {
    limiter := rate.NewLimiter(2, 4)
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        if !limiter.Allow() {
            app.rateLimitExceededResponse(w, r)
            return
        }
        next.ServeHTTP(w, r)
    })
}
```

Normally though you set a rate limiter on the ip address rather than a global one. A straight forward way of doing that is to create a map of rate limiters using the ip as key. Maps are not safe for concurrent use so you will have to use a mutex in that case as every http request will be its own goroutine.

The middleware might look something like this instead then.

```go
func (app *application) rateLimit(next http.Handler) http.Handler {
    var (
        mu      sync.Mutex
        clients = make(map[string]*rate.Limiter)
    )

    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        ip, _, err := net.SplitHostPort(r.RemoteAddr)
        if err != nil {
            app.serverErrorResponse(w, r, err)
            return
        }
        mu.Lock()
        if _, found := clients[ip]; !found {
            clients[ip] = rate.NewLimiter(2, 4)
        }
        if !clients[ip].Allow() {
            mu.Unlock()
            app.rateLimitExceededResponse(w, r)
            return
        }
        mu.Unlock()
        next.ServeHTTP(w, r)
    })
}
```

You may also want to add some kind of clearing of the limiter to avoid the map taking up too much resources. 

##### Chapter 11 - Graceful Shutdown

