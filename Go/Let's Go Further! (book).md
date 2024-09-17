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

