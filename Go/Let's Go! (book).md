##### Chapter 2 - Foundations

Three essentials for a web application:
- Handler. If you worked with MVC pattern before you can think of the handlers like controllers. They're responsible for executing your application logic and writing HTTP response headers and bodies.
- Router (servemux in Go terms). It stores the mapping between URL routing patterns for the application and the corresponding handler. Usually one servemux containing all the applications routes.
- Web server. You can establish a web server in Go that listens for incoming requests as part of the application itself. No need for external third-party servers.

When creating your server and use http.ListenAndServe(addr, handle) you can just call it in a variable called err as it can't return any nil errors.

If you use a handle with the route "/" the servemux will treat it as a catch-all pattern. So if you don't have anything else routed every request to the URL regardless of path will return that handle. This is because the path ends with the "/". It's called a sub-tree path pattern, and these are matched whenever the start of a request matches the sub-tree part. It's like using a wildcard at the end "/\*\*"  and that is why it catches everything. If you want the path but without the catch all behaviour you can use {\$}, it makes it so the route is single slash followed by nothing else. It only allowed to use this special char on routes ending with slash, if you try to use it on route that doesn't match the criteria it will result in a runtime panic.

When using handlers you can set them up without using a servemux, they will then be stored in the default servemux which is a global variable in the http package. There are several reasons to avoid this. First and forth-most is the clarity as it's less explicit. Secondly, for security and maintainability as routes can be registered anywhere and even third-party packages can register routes if compromised. Always just use your own locally scoped servemux.

You can use wildcard in the route patterns. Wildcard segments in a route pattern a denoted by an identifier inside {} brackets e.g. {productID}. You can use several different wildcards in a route e.g. "category/{categoryID}/products/{productID}". You can only use one wildcard between each path segment (the part between slashes) and the wildcard needs to fill the full part of that path segment.

You can retrieve the wildcard segment in your handler as a string using its identifier and the r.PathValue() method, as example below.

```go
func example(w http.ResponseWriter, r *http.Request) {
  categoryID := r.PathValue("categoryID")
...
}
```

As the path value always is a string and untrusted user input remember to sanitise the data you get back.

When using route patterns with wildcard segments it's possible that patterns will overlap, for example "/post/edit" and "/post/{id}". If a request comes for /post/edit it's a valid match for both patterns but Go servemux will always go to the most specific route pattern and as {id} can be anything the first route wins out. So it doesn't matter what order the routes are defined and registered, it won't change the behaviour. In certain edge cases with overlapping routes, neither route is more specific than the other, the servemux will consider the patterns as conflicting and will panic at runtime. Regardless the servemux behaviour and that it can handle most overlaps you should avoid using routes that overlaps as it increases the risk of bugs and unintended behaviour. 

If a route pattern ends with a wildcard and the wildcard's identifier ends with `...` then the wildcard will match any and all remaining segments of a request path. `/post/{path...}` will match requests like /post/a, post/a/b, /post/a/b/c etc like a sub-tree but with the difference that you can access the value with r.PathValue()

According to HTTP good practice you should restrict your application so that it only responds to requests with an appropriate HTTP method. In Go handlers you just put the method in the route, like the example below.  HTTP methods are case sensitive and should always be upper case, and you can only include one method per route pattern (HEAD is included in GET so technically two in all GET).

```go
mux.HandleFunc("GET /{$}", home)
```

It's of course possible to add different methods to the same route path. Most specific route wins here too, if you add the HTTP method it will be more specific than just the route. Try to think about how you name the handlers, example postfix the name of the method productPost, prefix it postProduct or give it a name like newProduct etc just try to make it clear so it works for your brain on solo projects or make it easy for others to understand on projects with multiple people.

The book covers Go 1.22 and a lot of the functionality for the book is very new for the standard library, more things might be introduced in the future but at the time of writing there are third-party routers that allow you to use multiple HTTP methods in single routes and using regex for route patterns etc etc. Recommended third-party routers to look into if standard library doesn't cover everything you need is httprouter, chi, flow and gorilla/mux.


[HTTP responses](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)
By default every response the handlers send is a 200 OK.
You can change what it sends with the w.WriteHeader() and put the code in as a parameter. You can only call w.WriteHeader() once per response and after it's been written it can be changed. If you call it a second time Go will log a warning message. You also have to call the WriteHeader before the Write. To make it easier to read and not having to remember every code you should use the net/http constants for the codes e.g. `http.StatusCreated` for 201. You can find the constants [here](https://pkg.go.dev/net/http#pkg-constants)

The header is a map and you can customise the headers with adding additional to the map with w.Header().Add(). This needs to be done before w.Write and w.WriteHeader otherwise it will have no effect.

w.Write() is the most straight forward way to send specific HTTP response bodies, it's the simplest and most fundamental way of doing it but it's a far more common practice to pass the http.ResponseWriter value to another function. Because it got the Writer() method is satisfies the io.Writer interface, so any function that uses an io.Writer parameter you can pass the http.ResponseWriter into. Functions like io.WriteString() and fmt.Fprintf() can be used to write the response bodies too.

Go automatically sets the Content-Type header by detecting content in the response body. If it can guess it falls back on `Content-Type: application/octet-stream`. Go uses http.DetectContentType() function for it and it works quite well but a common issue is that it can't distinguish JSON from plain text so by default JSON responses will be sent as `Content-Type: text/plain; charset=utf-8` , you can prevent this by setting the header manually like code below.

```go
w.Header().Set("Content-Type", "application/json")

//JSON example
w.Write([]byte(`{"name":"Alex"}`))
```

You can manipulate and read the headers with Add(), Set(), Del(), Values()

```go
// Set a new cache-control header. If an existing "Cache-Control" header exists
// it will be overwritten.
w.Header().Set("Cache-Control", "public, max-age=31536000")

// In contrast, the Add() method appends a new "Cache-Control" header and can
// be called multiple times.
w.Header().Add("Cache-Control", "public")
w.Header().Add("Cache-Control", "max-age=31536000")

// Delete all values for the "Cache-Control" header.
w.Header().Del("Cache-Control")

// Retrieve the first value for the "Cache-Control" header.
w.Header().Get("Cache-Control")

// Retrieve a slice of all values for the "Cache-Control" header.
w.Header().Values("Cache-Control")
```

There isn't one Go project structure that everyone uses but the most used structure is having a `cmd` directory with the main.go and application-specific code. Then a `internal` which contain the code that shouldn't be accessed by external sources. Then you can have other ui, pkg and others. This give the project scalability and makes it cleaner. Internal is also only accessible from the root of your project and if you'd try to import the internals to another project Go doesn't allow it.

You can parse template files containing html markup with Go's html/template package. Creating template files and then you can call template.ParseFiles(vardiac...) if you then want to write it as the body response you use .ExecuteTemplate on the parsed data. If it's only a complete html markdown code you can use .Execute.

Templating composition works like you define the name of the template with `{{define "example"}}` and the places you want to inject template you use `{{template "text" .}}` (the dot represents the dynamic data you want to pass to the invoked template) and you finish the template `{{end}}`. Example from the book below.

```html
{{define "base"}}
<!doctype html>
<html lang='en'>
    <head>
        <meta charset='utf-8'>
        <title>{{template "title" .}} - Snippetbox</title>
    </head>
    <body>
        <header>
            <h1><a href='/'>Snippetbox</a></h1>
        </header>
        <main>
            {{template "main" .}}
        </main>
        <footer>Powered by <a href='https://golang.org/'>Go</a></footer>
    </body>
</html>
{{end}}
```

Then another template file can be like the book example below, and when using the html/template package it will be injected.

```html
{{define "title"}}Home{{end}}

{{define "main"}}
    <h2>Latest Snippets</h2>
    <p>There's nothing to see here yet!</p>
{{end}}
```

In some projects you might want to break out certain bits into partials that can be reused in different pages or layouts. For example a nav bar. You can just make a new template define it and add it to the files you parse.

You can also use block instead of a template, it acts like template except it allows you to specify some default content if the template being invoked doesn't exist in the current template. Used for default content like a sidebar.

The net/http package got a function called FileServer which returns a handler you can use to serve files over HTTP from a specific directory. Like the example below you just give the path to the directory

```go
fileServer := http.FileServer(http.Dir("./ui/static"))
```

When the handler receives a request for a file it removes the leading slash from the request URL path and then search the `./ui/static` for the file. This will result in a 404 not found as the path it will search would be `./ui/static/static`. To avoid this the http package got a function called http.StripPrefix() that you use for the router handle e.g.

```go
mux.Handle("GET /static/", http.StripPrefix("/static", fileServer))
```

The FileServer handler has some additional features, it automatically uses path.Clean() before searching for a file which removes `.` and `..` from the URL path which helps stopping traversal attacks. It also supports range requests, which can be useful if you want a large file to be served in parts, for example a resumable download. Headers Last-Modified and If-Modified-Since are transparently supported. which can help reduce latency.

FileServer won't be reading files from the hard drive when app is up and running as Windows and Unix OS cache recently used files in RAM (at least frequently used ones) so it's more likely grab the files there making it quicker than the slow trip to the hard drive.

If you want to serve a single file from a handler for a example a download you can use http.ServeFile(). It's important to note that file path isn't sanitised with this function so you should do it yourself to keep the app secure from attacks.

If you don't want the directory listing to show when using the FileServer there are a few different ways you can write your code, three different solutions can be found in [this blog post](https://www.alexedwards.net/blog/disable-http-fileserver-directory-listings).

Handlers are objects that satisfy the handler interface, which is it needs a method called ServeHTTP that takes a responsewriter and pointer to request.
`ServeHTTP(http.ResponseWriter, *http.Request)`. Instead of making your own objects with this method it's easier to just make a function that takes the responsewriter and the request and with the help of the HandlerFunc it adds a ServeHTTP to the function.

```go
// some func called Home

mux.Handle("/", http.HandlerFunc(home))

//with Go syntatic sugar it can then be written like this instead

mux.HandleFunc("/", home)
```

The servemux is also a handler, which instead of responding passes the request to another handler. Chaining handlers is something very common in Go, you can think of a Go web application as a chain of ServeHTTP() methods being called one after another.

All incoming HTTP requests are served in their own Go routines. On a busy server it means that handlers will be running concurrently which helps with the speed but you need to be aware and protect against race conditions when accessing shared resources from the handlers.

##### Chapter 3 - Configuration and error handling

It's common to use command-line flags to manage configuration settings at runtime. For example which address the server should listen to, can and should be different depending on dev, testing and prod environments. The easiest way of accept and parse flags in your application is to add something like the example below. In the example the flag `addr` is defined, the second value is the default address and the third a short help text what the flag controls.

```go
addr := flag.String("addr", ":4000", "HTTP network address")
// use Parse after all flags been defined and before they're called.
flag.Parse()
```

Running this in the terminal will run the application with ":80" instead of ":4000", `go run ./cmd/web -addr=":80"`. This means you don't need to hard code the address in the server code. When you use the flag variable in the code you need to remember that the value returned from flag is a pointer so \* needs to be used as prefix. If you use a port between 0-1023 you need root privileges to bind it. For example TLS often uses :80.

Flags can be different types, ints, bools, durations etc. If the value given can't be converted it will print an error and exit.

If I put in the following command with the help flag `go run ./cmd/web -help` I'll get the following sdout.

```terminal
 -addr string  
       HTTP network address (default ":4000")
```

When using boolean flags you can omit true and just have the flag as that is the default value. For false you need to specify it `-flag=false`.

Using environment variables is seen as good practice among several people but using environment variables over flags comes with some drawbacks, like you can't specify a default value, you don't get the -help functionality and an os.Getenv() is always a string. 

You can parse command-line flags into the memory addresses of pre-existing variables with functions like flag.StringVar(), flag.IntVar() etc. These functions are useful if you want to store all your configuration settings in a single struct. Example from the book below

```go
type config struct {
    addr      string
    staticDir string
}

...

var cfg config

flag.StringVar(&cfg.addr, "addr", ":4000", "HTTP network address")
flag.StringVar(&cfg.staticDir, "static-dir", "./ui/static", "Path to static assets")

flag.Parse()
```

When using log.Printf() and log.Fatal() functions you use the default logger package in Go that prefixes the message with date and time. For many applications that can be enough but if you need more complex logging you can use the log/slog package in the standard library. When you use the slog you get a more exact time (down to millisecond), severity level (debug, info, warn or error), the log message and optionally any number of key value pairs containing additional info.

Using slog you need to understand that all structured loggers have a structured logging handler associated with them (not the same as a HTTP handler) and it's the handler that controls how the log entries are formatted.

```go 
loggerHandler := slog.NewTextHandler(os.Stdout, &slog.HandlerOptions{...})
logger := slog.New(loggerHandler)

// or on one line

logger := slog.New(slog.NewTextHandler(os.Stdout, &slog.HandlerOptions{...}))
```

In the first line above we use the NewTextHandler to create the structured logging. The function accepts two arguments, the first it the write destination and the second is a pointer to HandlerOptions struct which you can use to customise the behaviour of the handler. If you're happy with defaults you can just past nil as the second argument. After that we actually create the structured logger by passing the handler to the slog.New() function.

Once you've got a structured logger you can use different severity levels by calling the Debug(), Info(), Warn() or Error(). All these are variadic methods which accept key value pairs. So you could code a log like `logger.Info("request received", "method", "GET", "path", "/")` were the first string is the message and the ones coming after KV.

There is no equivalent of log.Fatal() in a structured logger so instead you can log the error message and then exit the program with os.Exit(1).

If you forget to add a value to key for example `logger.Info("starting server", "addr")` you'll get a log but the attribute will have the key !BADKEY and "addr" as value. To avoid this you can use a safer syntax and use the slog.Any() function. `logger.Info("starting server", slog.Any("addr", ":4000")`. If you want to use a certain type you can use slog.String, slog.Bool etc. This makes things more verbose but reduce the risk of bugs so up to you if you want to use it or not.

The logger handler in the code example above was a text handler and the output was in plaintext but it's possible to write the logs as JSON instead. You just change the slog.NewTextHandler to slog.NewJSONHandler, it takes the same arguments.

By default the minimum log level is info and any entries with severity lower than info i.e. debug will be silently discarded. You can change that in the slog.HandlerOptions. You can also add source which shows the filename and line number of the calling source code in the entry.

```go
// Change minimum level to Debug

logger := slog.New(slog.NewTextHandler(os.Stdout, &slog.HandlerOptions{
    Level: slog.LevelDebug,
}))

// Add source

logger := slog.New(slog.NewTextHandler(os.Stdout, &slog.HandlerOptions{
    AddSource: true,
}))
```

All the logs in the book are using the os.Stdout and the big benefit of using that is that the application and logging are decoupled. The application itself isn't concerned with the routing or storage of the logs and that can make it easier to manage the logs differently depending on environment. When in dev it's easy to see message in the terminal and in staging or production you can redirect the stream elsewhere like on-disk files or logging service like Splunk. For example `go run ./cmd/web >>/tmp/web.log` , the double arrow appends to an existing file.

Custom loggers created in slog with the New function are concurrency safe. You can share a single logger and use it across multiple goroutines without needing to worry about race conditions.

When implementing the loggers to all functions it's a good idea to use dependency injection to make the dependency available for all handlers. You could use a global variable but dependency injections as it makes the code more explicit, less prone to errors and easier to unit test. A neat way you can do it by using a struct and making the functions to methods.

The pattern used in the book makes it so that you need all handlers in the same package. If you need to have them spread out you can use closure by creating a standalone config package which exports the struct have the handler functions close over this to form a closure.

To help separate concerns you can turn error handling to helper methods, which also helps DRY. Example from the book below

```go
func (app *application) serverError(w http.ResponseWriter, r *http.Request, err error) {
    var (
        method = r.Method
        uri    = r.URL.RequestURI()
    )

    app.logger.Error(err.Error(), "method", method, "uri", uri)
    http.Error(w, http.StatusText(http.StatusInternalServerError), http.StatusInternalServerError)
}
```

The http.StatusText turns a status code to a text representation. For example 404 will return "Bad Request".
If you want to add a stack trace you can use debug.Stack() and add the return converted to a string as KV in the http.Error.

When refactoring your code try to make the main file less crowded and things like routes can go to its own file for example.

##### Chapter 4 - Database-driven responses

Downloading an external package for your project is quite straight forward. use the go get command in the terminal followed by the address for the package (usually a github address). Example: `go get github.com/go-sql-driver/mysql` , if you don't specify a version you get the latest version. If you do want a specific you add the version to the link, like this example `go get github.com/go-sql-driver/mysql@v1.0.3`. You can also just put `@v1` and get the most recent version starting with 1.

After you used the get command the package and it's version will show in the go.mod file and if the package isn't imported in the code it will show the text "// indirect" as well. A go.sum file will also be added to your root, it will contain cryptographic checksums representing the content of the required package. You can verify so that downloaded packages on your machine matches the go.sum values to be confident nothing been altered. If someone else needs to download the dependencies for the project with `go mod download` they get an error if there are any mismatches between package and the checksum. Go will prevent any malicious changes to the packages in that way.

If you need to upgrade a package to another version you can use -u flag. To get the latest version you just put it like this `go get -u github.com/foo/bar` or if you want a specific version `go get -u github.com/foo/bar@v2.0.0`.

If you want to remove unused packages you can either do it with the go get command and set the version as none like this `go get github.com/foo/bar@none` or if you don't have any references to the package in the code the easier way `go mod tidy`.

When connecting to a database we use the sql.Open() function that takes the driver name and the DSN (data source name) which describes how to connect to the database. The DSN can be different depending on which database you use and you can set things like parseTime=true so convert sql time and date fields to time.Time objects. go-sql-driver is well documented on github. The sql.Open() returns a sql.DB object, which isn't a database connection but a pool of many connections. Go manages the connections in the pool as needed, opening and closing connections via the driver. The pool is safe for concurrent access so can be used via the web handlers safely. The pool is also intended to be long-lived so you normally initialise it in the main function then pass the pool to handlers. Calling sql.Open() in short-lived HTTP handlers is a waste of memory and network resources. The function doesn't create any connections it initialise the pool for future use, and actual connections are established lazily as and when they're needed. To check that everything is set up correctly when using sql.Open() you can call .Ping from the pool you created and check for errors. You should also defer the sql.Close() as good practice.

In Go it's normal to see the package with the driver for the database being imported with a blank import, which means that you put an underscore before the import. You usually don't use the package and only want the driver, so when using a blank import the package uses init() and the Go compiler won't raise any issue that the import isn't called. This is standard practice for most of Go's SQL drivers (you can put a comment before the import if you want to make it clear that you're using the import for a driver).

If you're used with MVC what's called models in Go structure can be compared to service layers. They are usually placed in the internal directory and structs can be matched with the DB fields and you can have model struct that wraps the DB connection pool.

```go
type Snippet struct {
    ID      int
    Title   string
    Content string
    Created time.Time
    Expires time.Time
}

type SnippetModel struct {
    DB *sql.DB
}
```

If using a structure like above you can then have the SnippetModel have methods communicating with the database and using SQL. Then you can inject them as a dependency to http handlers. The benefits of doing it like that is again cleaner separation of concerns, the database logic won't be tied to the handlers which means handlers are limited to dealing with HTTP, which makes everything easier to unit test. Using the model actions as methods on objects also makes an opportunity to create an interface and mock it for testing. 

When doing SQL injections in Go it's good practice to use placeholder parameters instead of interpolating data in the query. Placeholder's syntax looks different depending on what database you use. Placeholder notation is ? for MySQL, SQL Server and SQLite for example while PostgreSQL uses $N. Read the documentation for your database before using them. The reason for using placeholders is to help avoid SQL injection attacks, when using the methods below they make prepared statements in the background, and the parameters are transmitted after and the database treats them as pure data.

Go has three different methods to execute database queries.
- DB.Query() - Used for SELECT queries with multiple row returns.
- DB.QueryRow() - Used for SELECT queries with a single row return.
- DB.Exec() - Used for statements which don't return rows (like INSERT and DELETE).

When using the above methods you can add the statement as the first argument and then the values used for the placeholder in the order they are in the statement.

There are methods like LastInsertId() and RowsAffected() you can use on the result of a query but it's important to know which databases support what. Again, check documentation.

Using a single record query you usually use the id as the identifier which often is a BIGINT in databases. When mapping over the database values to a Go type you need to be aware of what types you're mapping.

- CHAR, VARCHAR and TEXT map to string.
- BOOLEAN maps to bool.
- INT maps to int and BIGINT maps to int64.
- DECIMAL and NUMERIC map to float.
- TIME, DATE and TIMESTAMP map to time.Time

MySQL needs you to specify in the DSN that TIME should be parsed to time.Time with the `parseTime=true` otherwise it will map to []byte. Quirks like that can be good to check when you map over data.

When you use Scan() on the result of your query you need to have the same number of arguments as columns you return.

If you want to check if an error matches a specific value you should use the errors.Is() instead of '\=\='. Even though the latter compiles it can cause issues as you can wrap errors and when you do that a new value is created, the equality operator doesn't check against the underlying original error. 

When returning multiple rows with the Query() remember to defer the close after the error check. Otherwise if the query returns an error you'll get a panic trying to close a nil result set. The data get iterated by .Next()  in a for loop and then you can Scan() every iteration. Example from the book

```go
func (m *SnippetModel) Latest() ([]Snippet, error) {
    stmt := `SELECT id, title, content, created, expires FROM snippets
    WHERE expires > UTC_TIMESTAMP() ORDER BY id DESC LIMIT 10`

    rows, err := m.DB.Query(stmt)
    if err != nil {
        return nil, err
    }

    defer rows.Close()

    var snippets []Snippet

    for rows.Next() {
        var s Snippet
        err = rows.Scan(&s.ID, &s.Title, &s.Content, &s.Created, &s.Expires)
        if err != nil {
            return nil, err
        }
        snippets = append(snippets, s)
    }

    if err = rows.Err(); err != nil {
        return nil, err
    }

    return snippets, nil
}
```

In the example above it checks errors after the loop to retrieve any errors that was encountered during the iteration. It's important to call this because we can't assume that a successful iteration was completed over the full result set. For example if you're working with a large data set and you lose the connection to the connection pool halfway through the rows.Next() might return false rather than returning an error and you think it's gone through without any problems.

One thing that Go doesn't do well when it comes to SQL is NULL values in the database records. Go can't convert the NULL into a string, there are workarounds but if you control the database it's easier to just work with not null constraints on the columns and sensible default values as necessary.

Another thing about the calls to Exec, Query, and QueryRow methods is that they can use any connection from the connection pool, even if you for example got two Exec calls in a row next to each other in the code it's not guaranteed that they will use the same connection. Sometimes you need it to be done on the same connection, i.e. locking a table in mysql you must unlock it with the same connection. To guarantee you use the same connection you can wrap all the statements in a transaction. The transaction starts with Begin() method of the db pool, were you defer Rollback() after you done all your statements and actions you use the Commit().

Prepared statements were used in the backgrounds for the SQL query methods etc, but sometimes you want to make that prepared statement yourself because it can be a waste of resources having a new prepared statement being made every time you call one of those methods. A smart solution if you need to have a prepared statement is to have it as field in your model struct. You need to be careful with prepared statements as well because they make a  new one for every connection that uses it and under heavy load a large amount of them might be created and depending on the database there's also a limit on how many you can have running.

##### Chapter 5 - Dynamic HTML templates

When using templates you can import the dynamic data from a struct. Any data you pass as the final parameter in ExecuteTemplate() is represented as a ".". You can then use the name of the field from the struct to get the dynamic data, i.e. `.ID`. The html/template package only allows you to pass one item of dynamic data but in many occasions you need more, if that's the case you can just wrap the dynamic data in a struct. So often it will become a struct within a struct, and then you call the data in the template with an extra field name, for example  `.exampleStruct.ID`.

The html/template package automatically escapes any data that is yielded between {{}} tags (like javascript). This behaviour helps avoiding XSS attacks and it's way you should use the template from html package over the text one.

If the type you're yielding between the {{}} tags has methods defined against it you can call them when templating. Calling a field with the type time.Time means that you can call on things like .Weekday and AddDate. When using a method like AddDate you don't put the parameters in parenthesis but just have a space between like this, `{{.Created.AddDate 0 6 0}}` this example adds 6 months to the time.

HTML comments always gets stripped out when using the html/template package so avoid using conditional comments when you write the HTML.

In templates you can use actions and functions to control the dynamic data. Book's list below and documentation on [this link](https://pkg.go.dev/text/template#hdr-Functions).

| Action                                  | Description                                                                                                                                                                                                                                                                                     |
| --------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `{{if .Foo}} C1 {{else}} C2 {{end}}`    | If .Foo is not empty then render the content C1,<br>otherwise render the content C2.                                                                                                                                                                                                            |
| `{{with .Foo}} C1 {{else}} C2 {{end}}`  | If .Foo is not empty, then set dot to the value of <br>.Foo and render the content C1, otherwise render the content C2.                                                                                                                                                                         |
| `{{range .Foo}} C1 {{else}} C2 {{end}}` | If the length of .Foo is greater than zero then loop over <br>each element, setting dot to the value of each<br>element and rendering the content C1. If the length<br>of .Foo is zero then render the content C2. The<br>underlying type of .Foo must be an array, slice,<br> map, or channel. |
It's important to know that the with and range actions change the value of the dot. Once you start using them, what dot represents can be different depending on where you are in the template and what you are doing.

| Function                       | Description                                                                                                                            |
| ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------- |
| `{{eq .Foo .Bar}}`             | Yields true if .Foo is equal to .Bar                                                                                                   |
| `{{ne .Foo .Bar}}`             | Yields true if .Foo is not equal to .Bar                                                                                               |
| `{{not .Foo}}`                 | Yields the boolean negation of .Foo                                                                                                    |
| `{{or .Foo .Bar}}`             | Yields .Foo if .Foo is not empty; otherwise yields .Bar                                                                                |
| `{{index .Foo i}}`             | Yields the value of .Foo at index i. The underlying type of<br>.Foo must be a map, slice or array, and i must be an integer <br>value. |
| `{{printf "%s-%s" .Foo .Bar}}` | Yields a formatted string containing the .Foo and .Bar <br>values. Works in the same way as fmt.Sprintf().                             |
| `{{len .Foo}}`                 | Yields the length of .Foo as an integer.                                                                                               |
| `{{$bar := len .Foo}}`         | Assign the length of .Foo to the template variable $bar                                                                                |
It's possible to combine functions using parentheses for example this example will render C1 if the length of Foo is greater than 99: `{{if (gt (len.Foo) 99)}} C1 {{end}}`.
Another example is this when the tag will render C1 if.Foo equals 1 and .Bar is less than or equal to 20: `{{if (and (eq .Foo 1) (le .Bar 20))}} C1 {{end}}`.

When ranging through values you can also control the loop with {{continue}} and {{break}}, continue skipping to next iteration and break stopping the loop.

To make rendering of the web page faster you should cache the pages that are rendered by the ParseFiles so the function doesn't have to run every time and the parsed templates will be stored in the in-memory cache.

When working with dynamic behaviours in HTML there's a risk of encountering runtime errors. Depending on how your code is structured you can sometimes get a 200 even though half the page doesn't work. In that case you can create a buffer that checks for errors first and if the buffer is ok you write the response as normal and if not send the status code for the error.

You can make custom template functions with the FuncMap that then needs to be registered with the Funcs() method before parsing the templates. In the book an example is used when the time.Time get formatted in a string that's more readable. A func map is created and then called when creating the template set, like this `ts, err := template.New(name).Funcs(functions).ParseFiles("./ui/html/base.tmpl")`. The functions mapped must only return one value (not counting errors, they are fine to add). In the template the function is used like this `{{humanDate .Created}}` but you can also use pipeline `{{.Created | humanDate}}`. A nice thing with pipe-lining is that you can make an arbitrarily long chain of functions which uses the output from one as the input for the next. Example used in the book `{{.Created | humanDate | printf "Created: %s"}}`.

##### Chapter 6 - Middleware

There is usually some shared functionality for many or all of the HTTP requests that you want when building a web app, for example logging or check cache etc. Common way to deal with this is to use middleware, this is some self-contained code which independently acts on a request before or after your normal application handlers. Currently in the app worked on in the book when it receives a HTTP request it calls on the servemux's ServeHTTP() method, that looks up relevant handler and calls the ServeHTTP() method of the next handler in the chain.

http.StripPrefix() that's been used in earlier chapters is a middleware is an example of middleware. 

The pattern for creating your own middleware looks like this.

```go
func myMiddleware(next http.Handler) http.Handler {
    fn := func(w http.ResponseWriter, r *http.Request) {
        // TODO: Execute our middleware logic here...
        next.ServeHTTP(w, r)
    }

    return http.HandlerFunc(fn)
}
```

The myMiddleware is like a wrapper for the next handler, which is passed as a parameter. When the fn is run it executes the middleware logic and then transfers control to the next handler. The final line the closure is converted to a http.Handler and returned using the HandlerFunc.

Usually this is simplified a little bit by using an anonymous function.

```go
func myMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        // TODO: Execute our middleware logic here...
        next.ServeHTTP(w, r)
    })
}
```

The position of the middleware is important to keep in mind when you're implementing it. If you position it before the servemux it will act on every request. If you place it between the servemux and the handler it will only act for that handler.

One way you can use the middleware positioned before the servemux is by setting the headers so you don't have to do that for each handler. Things like CSP and Referrer-Policy inline with OWASP guidelines can be useful, but if you set CSP remember to update the policy if you're adding something new (use console to debug when building to check so everything's working).

When using the middleware it's good to know that it flows in one direction and then back up.
`commonHeaders → servemux → application handler → servemux → commonHeaders`

```go
func myMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        // Any code here will execute on the way down the chain.
        next.ServeHTTP(w, r)
        // Any code here will execute on the way back up the chain.
    })
}
```

You could also use this flow to do early returns. They can be used for authentication for example when a handler should only serve something to an authorised user.

```go
func myMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        // If the user isn't authorized, send a 403 Forbidden status and
        // return to stop executing the chain.
        if !isAuthorized(r) {
            w.WriteHeader(http.StatusForbidden)
            return
        }

        // Otherwise, call the next handler in the chain.
        next.ServeHTTP(w, r)
    })
}
```

In a simple Go app a panic results in the application being terminated, but web apps can be a bit different and could be isolated to the goroutine serving the active HTTP request (every request is its own goroutine). So you don't want a panic in your handlers to bring down a server but rather just log it and give the user an error and close the HTTP connection. So there's another use for the middleware. By using `defer func()` in the middleware you can have an error handling that always will be run when Go unwinds the stack.

You should set a header "Connection" to "close" so the Go server automatically closes the connection in question when a panic happens.

If you're recovering panics through middleware be aware that if the handler spins up another goroutine you'd have to to make a panic recover for that in the handler as it won't be covered by the middleware.

There's a 3rd party package that helps making middleware chains easier to deal with called [alice](https://github.com/justinas/alice). Example of standard vs alice

`return myMiddleware1(myMiddleware2(myMiddleware3(myHandler)))`

`return alice.New(myMiddleware1, myMiddleware2, myMiddleware3).Then(myHandler)`

But you can also create chains that can be assigned to variable, appended to and reused.

```go
myChain := alice.New(myMiddlewareOne, myMiddlewareTwo)
myOtherChain := myChain.Append(myMiddleware3)
return myOtherChain.Then(myHandler)
```

##### Chapter 7 - Processing forms

When sending form data with a POST you can use r.ParseForm() that parses the request body. Then you can access the data with r.PostFrom by using its .Get() method. The field name is given as the parameter and if nothing can be found for that string the Get will return an empty string.

There is a method called r.PostFromValue() that's a shortcut version that calls the r.ParseForm() for you and then fetches the field value from r.PostForm. The author recommends against using it as it silently ignores any errors returned by r.ParseForm().

When using the r.PostForm.Get() method it only returns the first value from a specific form field. This is an issue if you got something like a group of checkboxes. If that's the case you have to work with the PostForm directly and range through it. The underlying type of the PostForm map is url.Values which in turn has the underlying type `map[string][]string`.

```html
<input type="checkbox" name="items" value="foo"> Foo
<input type="checkbox" name="items" value="bar"> Bar
<input type="checkbox" name="items" value="baz"> Baz
```
```go
for i, item := range r.PostForm["items"] {
    fmt.Fprintf(w, "%d: Item %s\n", i, item)
}
```

If you get the form data via a GET request as in query string parameters you can retrieve the values with the r.URL.Query().Get() method. This always returns a string value.

```go
func exampleHandler(w http.ResponseWriter, r *http.Request) {
    title := r.URL.Query().Get("title")
    content := r.URL.Query().Get("content")

    ...
}
```

It's important to validate the untrusted user input. It should be done to ensure that the form data is present, of the correct type and meets the business rules we got. The book is showing how it can be done server side for a handler with map\[string]string, and if  statements that populates the map if they encounter issues and then before writing the normal response a check is done if the map length is larger than 0. This can later be more gracefully done by keeping the field errors in a struct.

If you're using validation in more than one place or want to have validation in it's own place it's recommended to make validator package to abstract behaviour and reduce boilerplate code in the handlers.

When it comes to generics in Go, it's a pretty new language feature and best-practices are still getting established.  So when is it recommended to use generics? 
- If you're writing repeated boilerplate code for different data types.
- When you're writing code and find yourself reaching for the any (empty interface{}) type.
You probably shouldn't use generics in these cases
- If it makes the code harder to understand or less clear.
- If all the types that you need have a common set of methods, in which case it's better to define and use a normal interface instead.
- Just because you can. Prefer non-generic by default and switch to generic only if you need it.

There are external third party packages that can help you decode form data into structs. Author recommends go-playground/form or gorilla/schema.

```go
type snippetCreateForm struct {
    Title               string `form:"title"`
    Content             string `form:"content"`
    Expires             int    `form:"expires"`
    validator.Validator `form:"-"`
}
```

Here's an example of it being used with the go-playground one. Just like when marshalling JSON if the struct tag is \`form:"-"\` the decoder ignores that field during decoding. These are useful as it can convert types and the amount of code is less and arguably more readable.

##### Chapter 8 - Stateful HTTP

When it comes to sessions there are a lot of security consideration and proper implementation can be difficult. The author recommends using third party solutions like gorilla/sessions or his own alexedwards/scs. Sessions is the most known, but doesn't provide renew session IDs which is needed if you're using server-side session stores.

The session token is a random string which is sent to the application with every request the browser makes. The middleware using the scs for example checks if a session cookie is present and then retrieves the session data and checking so the session hasn't expired etc.

You can therefore use the token to check if something been done, by adding the request context and then delete it by using methods like PopString(). 

##### Chapter 9 - Server and security improvements

So far in the book http.ListenAndServer() been used which is useful for tutorials and short examples but it's more common to use a http.Server stuct in the real-world. Example in the book refactoring it.

```go
srv := &http.Server{
	Addr: *addr
	Handler: app.routes()
	
	// Error logger added as per text below
	ErrorLog: slog.NewLogLogger(logger.Handler(), slog.LevelError)
}

err = srv.ListenAndServe()
```

You can't configure the http.Server to use a structured logger directly, instead you have to convert the structured logger handler into a \*log.Logger which writes log entries at a specific fixed level.

You should have HTTPS rather than plain HTTP for your web app. HTTPS is essentially HTTP snet across a TLS (Transport Layer Security) connection.The traffic is then encrypted and signed which helps ensure privacy and integrity during transit.

For production the author recommends using Let's Encrypt for the TLS certificate but when in dev the simplest thing is to generate your own self-signed certificate. The crypto/tls package can be used for that. Can generate certs by using the lib like below. Path may be different depending on how you installed Go.

`go run /lib/go/src/crypto/tls/generate_cert.go --rsa-bits=2048 --host=localhost`

rsa-bits= 2048 generates a 2048-bit RSA key pair which cryptographically secure public key and private key. It stores the private key in a key.pem file and generates a self-signed TLS certificate for the host localhost containing the public key which stores in the cert.pem.

If you want to generate certs for testing that are trusted by web browser so you don't get security warnings when developing you can look into mkcert tool, which is a third party package.

When you update the ListenAndServe to ListenAndServeTLS you should also add to the session manage to have the cookies as secure, so nothing will be sent over unsecure http connection. You pass in the key and cert.pem to the ListenAndServeTLS. When you got TLS running any HTTP request will return a 400 Bad request error.
Remember to gitignore the certs and keys if you're keeping them in the project dir.

As of Go 1.22 the elliptic curves that have optimised assembly implementation are CurveP256 and X25519. You should just use them for your TLS handshakes as they but less strain on you CPU and make the server more performant. You add them in a field called CurvePreference, which is in a struct called tls.Config that you can have in your http.Server struct. `CurvePreference: []tls.CurveID{tls.X25519, tls.CurveP256}`

In the http.Server struct you should add idle, read and write timeouts. 
By default Go enables keep-alives on all accepted connections. This helps reduce latency because a client can reuse the same connection for multiple requests without having to repeat the TLS handshake. The keep-alive connection will auto close after a couple of minutes (exactly how long depends on OS). IdleTimeout sets the time how long keep-alives will be open. If you set Read timeout but not IdleTimeout the IdleTimeout will be the same as the read timeout which isn't optimal so you should set all these three.

Read timeout is the time allowed for request headers or body still being read after first accepted, after that time has passed Go will close the connection. Setting a shorter time will help mitigate risk of slow-client attacks like slowloris.

Write timeouts should be set to longer than read timeouts if you're using HTTPS. If data is written to the connection more than the set timeout time after the read of the request header finished, Go closes the connection. The idea of write timeout is not to prevent long running handlers but to prevent the data that the handler returns from taking too long to write.

##### Chapter 10 - User authentication

Start of the chapter is creating new routes and handlers for login and create users. Important note is when if the form doesn't clear validation it puts the user inputs as values except for the password as we don't want to risk the plain-text to be cached by the browser or other intermediaries.

The book uses a regex for email validation, the pattern is the one recommended by WC3 and Web Hypertext Application Technology Working Group.
```regex
"^[a-zA-Z0-9.!#$%&\'*+/=?^_`{|}~-]+@[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?(?:\\.[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?)*$"
```

It's essential that you don't have the password of users as plain text in your database for a myriad of reasons. Go got several ways of hash passwords with /x/crypto package. Author recommends using the bcrypt algorithm as it got helper functions designed for hashing and checking passwords.

When hashing the password you'll use bcrypt.GenerateFromPassword(). For example
`hash, err := bcrypt.GenerateFromPassword([]byte("my plain text password"), 12)`
The second parameter 12 is the cost. You can put it between 4 and 31, 12 means 4096 (2^12) bycrpt iterations will be used for the password hash. The higher the cost the harder for an attacker to crack but it also a cost on your end generating it so you should have that in mind. Author says 12 is a reasonable minimum to have. The result of a bcrypt is a hash like this `$2a$12$NuTjWXm3KKntReFwyBVHyuf/to.HEwTy.eS206TNfkGfr6HzGJSWG`, 60 chars long.
When you use CompareHashAndPassword you get a return of nil if it's a match.
`err := bcrypt.CompareHashAndPassword(hash, []byte("my plain text password"))`

There are some databases that got functionality to hash passwords themselves but you should avoid using that as it will be vulnerable to side-channel timing attacks and unless you're very careful sending the plain text password to the database risks it accidentally being recorded in a log.

It's good practice to generate a new session ID when the auth state or privilege levels changes for a user (e.g. login operation). It helps mitigate the risk of a session fixation attack. Same goes for when logging out.

You can add and remove value from the session key to check if a user is logged in or not. To restrict access to logged out users for example it's easiest to just do it via middleware.

The routes in the book looks like this after implementing the new middleware

```go
func (app *application) routes() http.Handler {
	mux := http.NewServeMux()

	fileServer := http.FileServer(http.Dir("./ui/static/"))
	mux.Handle("GET /static/", http.StripPrefix("/static", fileServer))

	dynamic := alice.New(app.sessionManager.LoadAndSave)

	mux.Handle("GET /{$}", dynamic.ThenFunc(app.home))
	mux.Handle("GET /snippet/view/{id}", dynamic.ThenFunc(app.snippetView))
	mux.Handle("GET /user/signup", dynamic.ThenFunc(app.userSignup))
	mux.Handle("POST /user/signup", dynamic.ThenFunc(app.userSignupPost))
	mux.Handle("GET /user/login", dynamic.ThenFunc(app.userLogin))
	mux.Handle("POST /user/login", dynamic.ThenFunc(app.userLoginPost))

	protected := dynamic.Append(app.requireAuthentication)

	mux.Handle("GET /snippet/create", protected.ThenFunc(app.snippetCreate))
	mux.Handle("POST /snippet/create", protected.ThenFunc(app.snippetCreatePost))
	mux.Handle("POST /user/logout", protected.ThenFunc(app.userLogoutPost))

	standard := alice.New(app.recoverPanic, app.logRequest, commonHeaders)

	return standard.Then(mux)
}
```

If not using a package like alice the wrapping would look something like this
`mux.Handle("POST /snippet/create", app.sessionManager.LoadAndSave(app.requireAuthentication(http.HandlerFunc(app.snippetCreate))))`

CSRF protection is something that you should protect yourself against if you got forms etc. One way to mitigate those kinds of attacks is by making sure SameSite attribute is set on the session cookies (Lax or Strict). The one problem with SameSite is that it's only fully supported by 96% of browser worldwide when the book was written.  But there are no websites that supports TLS 1.3 and don't support SameSite cookies so if you enforce TLS 1.3 as the minimum version in your config you're good to go. Otherwise you should make middleware that generates CSRF token that gets inbedded in the form fields to prevent attacks. Packages to use for that is justinas/nosurf or gorilla/csrf.

##### Chapter 11 - Using request context

Every http.Request that gets processed has a context.Context object embedded in it which can be used to store info during the lifetime of the request. A common use is to pass info between middleware and other handlers. The book's example is a check if a user is authenticated once and then passing that info the other middleware and handlers.

What you do is creating a new copy of the request with the added key and then pass that on. It can look like this:

```go
// Where r is a *http.Request...
ctx := r.Context()
ctx = context.WithValue(ctx, "isAuthenticated", true)
r = r.WithContext(ctx)

// it can of course be made less verbose like this

ctx = context.WithValue(r.Context(), "isAuthenticated", true)
r = r.WithContext(ctx)
```

Request context values are stored with the type any so you will have to assert them to their original type before you use them.

Example
```go
isAuthenticated, ok := r.Context().Value("isAuthenticated").(bool)
if !ok {
    return errors.New("could not convert value to bool")
}
```

Best practice when making these is to also make your own context key to avoid key colliding. isAuthenticated can be used by other third party packages used by the app which would cause a collision. The code above with its own type:

```go
// Declare a custom "contextKey" type for your context keys.
type contextKey string

// Create a constant with the type contextKey that we can use.
const isAuthenticatedContextKey = contextKey("isAuthenticated")

...

// Set the value in the request context, using our isAuthenticatedContextKey 
// constant as the key.
ctx := r.Context()
ctx = context.WithValue(ctx, isAuthenticatedContextKey, true)
r = r.WithContext(ctx)

...

// Retrieve the value from the request context using our constant as the key.
isAuthenticated, ok := r.Context().Value(isAuthenticatedContextKey).(bool)
if !ok {
    return errors.New("could not convert value to bool")
}
```

Go documentation says you should only use request context to store info relevant to the lifetime of the specific request. That means you shouldn't pass things like loggers, database connection pool to the middleware and handlers. It's better to make things like that available to your handlers explicitly by making methods against a application struct or passing them in a closure.

##### Chapter 12 - File embedding

You can embed external files into the Go program itself. It makes it so that the Go program can be self-contained and have everything it need to run as part of the compiled binary. Which makes it easier to deploy or distribute the application.

Embedding something looks like this

```go
import (
    "embed"
)

//go:embed "static"
var Files embed.FS
```

The comment is not a normal comment but a comment directive. The comment instructs Go to store the files in an embedded file system (in this case in the directory 'static') when the application is being compiled.

``go:embed "<path>"`` is the command. The path is relative to the file the comment is in. You can also only use it on global variables at package level and not within functions or methods. The path can not contain . or .. elements, nor may it start or end with /. The file system is always rooted in the directory which contains the `go:embed` directive.

You can embed multiple paths in one directive. Path separators need to be forward slash even on Windows.

```go
//go:embed "static/css" "static/img" "static/js" 
var Files embed.FS
```

You can also do it for individual files. For example if you don't want to include things like Sass files in the embed. You can also use the wildcard \* character. 

```go
//go:embed "static/css/*.css" "static/img" "static/js" 
var Files embed.FS

----

//go:embed "static/css/*.css" "static/img" "static/js" 
var Files embed.FS
```

If a path to a directory is in the directive all files within it will be embedded except for files starting with . or \_, if you want to include those files you add a all: prefix to the path.

```go 
//go:embed "all:static"
var Files embed.FS
```

##### Chapter 13 - Testing

In Go it's standard practice to write your test in a file with the same name but suffixed with \_test for example `helpers_test.go`. The test file is usually in the same directory as the file it is testing from.

Unit test are contained in a normal Go function with signature func(\*testing.T). To be a valid unit test the function needs to begin with the word Test, typically you follow that with the name of the function, method or type you're testing. You use the t.Errorf() function to mark a test as failed and log a message about the failure. Example from the book below.

```go
package main

import (
    "testing"
    "time"
)

func TestHumanDate(t *testing.T) {
    // Initialize a new time.Time object and pass it to the humanDate function.
    tm := time.Date(2024, 3, 17, 10, 15, 0, 0, time.UTC)
    hd := humanDate(tm)

    // Check that the output from the humanDate function is in the format we
    // expect. If it isn't what we expect, use the t.Errorf() function to
    // indicate that the test has failed and log the expected and actual
    // values.
    if hd != "17 Mar 2024 at 10:15" {
        t.Errorf("got %q; want %q", hd, "17 Mar 2024 at 10:15")
    }
}
```

When you want to run the test you write `go test (the dir)`, you can add -v (verbose) as flag to get more information. 

If you want to have several test cases you can use a table-driven test. Expanding on the first example above the author shows how it can be done with adding a struct and then loop through the cases using t.Run

```go
package main

import (
    "testing"
    "time"
)

func TestHumanDate(t *testing.T) {
    // Create a slice of anonymous structs containing the test case name,
    // input to our humanDate() function (the tm field), and expected output
    // (the want field).
    tests := []struct {
        name string
        tm   time.Time
        want string
    }{
        {
            name: "UTC",
            tm:   time.Date(2024, 3, 17, 10, 15, 0, 0, time.UTC),
            want: "17 Mar 2024 at 10:15",
        },
        {
            name: "Empty",
            tm:   time.Time{},
            want: "",
        },
        {
            name: "CET",
            tm:   time.Date(2024, 3, 17, 10, 15, 0, 0, time.FixedZone("CET", 1*60*60)),
            want: "17 Mar 2024 at 09:15",
        },
    }

    for _, tt := range tests {
        // Use the t.Run() function to run a sub-test for each test case. The
        // first parameter to this is the name of the test (which is used to
        // identify the sub-test in any log output) and the second parameter is
        // and anonymous function containing the actual test for each case.
        t.Run(tt.name, func(t *testing.T) {
            hd := humanDate(tt.tm)

            if hd != tt.want {
                t.Errorf("got %q; want %q", hd, tt.want)
            }
        })
    }
}
```

Instead of adding the comparable like if hd != tt.want for every test the author recommends making your own assert with a generic comparable that checks if actual and expected are equal.

One tool to test responses from http is httptest.ResponseRecorder which records the status code, header and body instead of writing them to a http connection. You can also test the middleware like this. The book has a good example of testing a middleware that adds headers to every handler.

```go
package main

import (
    "bytes"
    "io"
    "net/http"
    "net/http/httptest"
    "testing"

    "snippetbox.alexedwards.net/internal/assert"
)

func TestCommonHeaders(t *testing.T) {
    // Initialize a new httptest.ResponseRecorder and dummy http.Request.
    rr := httptest.NewRecorder()

    r, err := http.NewRequest(http.MethodGet, "/", nil)
    if err != nil {
        t.Fatal(err)
    }

    // Create a mock HTTP handler that we can pass to our commonHeaders
    // middleware, which writes a 200 status code and an "OK" response body.
    next := http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        w.Write([]byte("OK"))
    })

    // Pass the mock HTTP handler to our commonHeaders middleware. Because
    // commonHeaders *returns* a http.Handler we can call its ServeHTTP()
    // method, passing in the http.ResponseRecorder and dummy http.Request to
    // execute it.
    commonHeaders(next).ServeHTTP(rr, r)

    // Call the Result() method on the http.ResponseRecorder to get the results
    // of the test.
    rs := rr.Result()

    // Check that the middleware has correctly set the Content-Security-Policy
    // header on the response.
    expectedValue := "default-src 'self'; style-src 'self' fonts.googleapis.com; font-src fonts.gstatic.com"
    assert.Equal(t, rs.Header.Get("Content-Security-Policy"), expectedValue)

    // Check that the middleware has correctly set the Referrer-Policy
    // header on the response.
    expectedValue = "origin-when-cross-origin"
    assert.Equal(t, rs.Header.Get("Referrer-Policy"), expectedValue)

    // Check that the middleware has correctly set the X-Content-Type-Options
    // header on the response.
    expectedValue = "nosniff"
    assert.Equal(t, rs.Header.Get("X-Content-Type-Options"), expectedValue)

    // Check that the middleware has correctly set the X-Frame-Options header
    // on the response.
    expectedValue = "deny"
    assert.Equal(t, rs.Header.Get("X-Frame-Options"), expectedValue)

    // Check that the middleware has correctly set the X-XSS-Protection header
    // on the response
    expectedValue = "0"
    assert.Equal(t, rs.Header.Get("X-XSS-Protection"), expectedValue)

    // Check that the middleware has correctly set the Server header on the 
    // response.
    expectedValue = "Go"
    assert.Equal(t, rs.Header.Get("Server"), expectedValue)

    // Check that the middleware has correctly called the next handler in line
    // and the response status code and body are as expected.
    assert.Equal(t, rs.StatusCode, http.StatusOK)

    defer rs.Body.Close()
    body, err := io.ReadAll(rs.Body)
    if err != nil {
        t.Fatal(err)
    }
    body = bytes.TrimSpace(body)
    
    assert.Equal(t, string(body), "OK")
}
```

When you do end to end testing you can make a test server. It takes a http handler as an argument so if you structured the project with a struct that takes all the app data like all routes and middleware you can just pass that and isolate all app routing via that method. If you're using test helpers for more than one place it can be good to make a test utils file. 


Author adds functionality to test server for it to keep cookies and not to be redirected, so test can be made on specific requests.

```go
func newTestServer(t *testing.T, h http.Handler) *testServer {
    // Initialize the test server as normal.
    ts := httptest.NewTLSServer(h)

    // Initialize a new cookie jar.
    jar, err := cookiejar.New(nil)
    if err != nil {
        t.Fatal(err)
    }

    // Add the cookie jar to the test server client. Any response cookies will
    // now be stored and sent with subsequent requests when using this client.
    ts.Client().Jar = jar

    // Disable redirect-following for the test server client by setting a custom
    // CheckRedirect function. This function will be called whenever a 3xx
    // response is received by the client, and by always returning a
    // http.ErrUseLastResponse error it forces the client to immediately return
    // the received response.
    ts.Client().CheckRedirect = func(req *http.Request, via []*http.Request) error {
        return http.ErrUseLastResponse
    }

    return &testServer{ts}
}
```


If you want to run all test in a Go project you can use `./...` wildcard pattern. So command would look like this `go test ./...`, and every test file in the project would run.

If you just want to run tests with specific names you can use the -run flag which allows you to specify a regex pattern. For example `go test -v -run="^TestPing$" ./cmd/web/` would only run tests with that function name. You can even go further and only do sub-tests with the pattern `{test regex}/{sub-test regex}` e.g. `go test -v -run="^TestHumanDate$/^UTC$" ./cmd/web`

You can use the same syntax but with -skip flag if you want to skip a specific test.

Tests gets cached and the cached version runs if no changes has been done but if you want to force a none cached run of test even without any changes you can use the -count flag. It tells go test how many times you want to execute each test. `go test -count=1 ./cmd/web`

If you want to terminate the tests after encountering the first failure you can add flag -failfast, otherwise all test will run.

You can add t.Parallel() in the test function if you want the test to be able to run parallel to other tests. Only tests with t.Parallel can be run like that and not all test should be parallel, for example integration tests using a db table being in a known state. 

There's also a -race flag that enables Go's race detector when running tests. It helps flag race conditions that exists in the application. It's limited though, it can only detect problems if there are any at runtime testing. A clear run doesn't mean you don't have any problematic race conditions. It also increase the overall running time for the tests so shouldn't be used for each test during dev but more of a tool for pre-commit.

When doing end to end testing of handlers that got things like anti-CSRF due to forms etc it's getting a bit more complicated. You have to simulate a real-life users workflow.

When compiling Go ignores any directory called 'testdata', so you can place stuff like sql files for integration tests there without any issues.

Running test you might want Go to skip certain long and taxing tests unless it's test before a commit. A common way to do it is to add testing.Short to the test to check if the -short flag was present in your test command and then call t.Skip if it was.

To see your test coverage you can use the -cover flag for test. If you want a more detailed breakdown, on method and function level you can use the -coverprofile. This will write out a cover profile to a specific location. It can then be shown in the terminal with a func flag or in the browser (with red and green and the code).

`go test -coverprofile=/tmp/profile.out ./...`

`go tool cover -html=/tmp/profile.out`
