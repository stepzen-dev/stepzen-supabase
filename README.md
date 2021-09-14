# Stepzen + Supabase

An example Supabase application connected to StepZen's `@rest` directive through PostgREST.

## Outline

* [Setup StepZen Project](#setup-stepzen-project)
  * `stepzen.config.json`
  * `index.graphql`
  * `config.yaml`
* [Setup Supabase Project](#setup-supabase-project)
  * Create a New Table
  * Create an Authors Table
  * Create an Author
* [Types and Queries](#types-and-queries)
  * `author.graphql`
  * Deploy StepZen Endpoint
  * Run a test query on StepZen
* [Mutations](#mutations)
  * Create a Books Table
  * `POST` book
  * `GET` books
  * `book.graphql`
  * Run a test mutation on StepZen

## Setup StepZen Project

```bash
touch stepzen.config.json index.graphql config.yaml
mkdir schema
touch schema/author.graphql schema/book.graphql
```

### `stepzen.config.json`

```json
{
  "endpoint": "api/stepzen-supabase"
}
```

### `index.graphql`

```graphql
schema
  @sdl(
    files: [
      "schema/author.graphql",
      "schema/book.graphql",
    ]
  ) {
  query: Query
}
```

### `config.yaml`

```yaml
configurationset:
  - configuration:
      name: supabase_config
      apikey: xxxx
```

## Setup Supabase Project

Select new project.

<img width="729" alt="01-supabase-dashboard-new-project" src="https://user-images.githubusercontent.com/12433465/133342734-8ac8e7fd-683d-4aec-9645-f4cc7fc21067.png">

After the project has been created, scroll down to see instructions for connecting to your database.

<img width="1024" alt="02-connecting-to-your-project" src="https://user-images.githubusercontent.com/12433465/133342736-52366fb1-9d24-4409-b851-caa10a6d8d41.png">

Copy the URL in the Config section and add `/rest/v1/` to get your PostgREST endpoint. Copy the anon public key and replace it with `xxxx` in the following curl command.

```bash
curl --request GET \
  --url 'https://xxxx.supabase.co/rest/v1/' \
  --header "apikey: xxxx"
```

This will introspect your endpoint and tell you about the specification.

```json
{
   "swagger":"2.0",
   "info":{
      "title":"PostgREST API",
      "version":"8.0.0",
      "description":"standard public schema"
   },
   "host":"0.0.0.0:3000",
   "basePath":"/",
   "schemes":[
      "http"
   ],
   "consumes":[
      "application/json",
      "application/vnd.pgrst.object+json",
      "text/csv"
   ],
   "produces":[
      "application/json",
      "application/vnd.pgrst.object+json",
      "text/csv"
   ],
   "paths":{
      "/":{
         "get":{
            "tags":[
               "Introspection"
            ],
            "summary":"OpenAPI description (this document)",
            "produces":[
               "application/openapi+json",
               "application/json"
            ],
            "responses":{
               "200":{
                  "description":"OK"
               }
            }
         }
      }
   },
   "parameters":{
      "preferParams":{...},
      "preferReturn":{...},
      "preferCount":{...},
      "select":{...},
      "on_conflict":{...},
      "order":{...},
      "range":{...},
      "rangeUnit":{...},
      "offset":{...},
      "limit":{...}
   },
   "externalDocs":{
      "description":"PostgREST Documentation",
      "url":"https://postgrest.org/en/v8.0/api.html"
   }
}
```

### Create a New Table

Click "Create a new table" in the Table Editor.

<img width="818" alt="03-create-a-new-table" src="https://user-images.githubusercontent.com/12433465/133343066-4f48efc6-2944-49ec-b13b-8dab95677c9f.png">

### Create an Authors Table

Add a `name` column with a `text` type.

<img width="666" alt="04-create-authors-table" src="https://user-images.githubusercontent.com/12433465/133343068-1864a3d9-8f6c-447b-ba1e-8ed21570c495.png">

```bash
curl --request GET \
  --url 'https://xxxx.supabase.co/rest/v1/authors' \
  --header "apikey: xxxx"
```

You will receive an empty array:

```json
[]
```

### Create an Author

Click "Insert row" and create an author.

<img width="665" alt="05-create-an-author" src="https://user-images.githubusercontent.com/12433465/133344630-94174314-63a5-4494-b7fc-41665afdfe9a.png">

Run the previous curl command again.

```bash
curl --request GET \
  --url 'https://xxxx.supabase.co/rest/v1/authors' \
  --header "apikey: xxxx"
```

This time you will receive an author object.

```json
[
  {
    "id":1,
    "created_at":"2021-09-14T22:59:33+00:00",
    "updated_at":"2021-09-14T22:59:33+00:00",
    "name":"person1"
  }
]
```

## Types and Queries

Add the ability to query for authors with StepZen queries.

### `author.graphql`

Create an `Author` type with types for `id`, `name`, `created_at`, and `updated_at`.

```graphql
# schema/author.graphql

type Author {
  id: ID!
  name: String!
  created_at: DateTime!
  updated_at: DateTime!
}
```

The `authors` query returns an array of `Author` objects.

```graphql
# schema/author.graphql

type Query {
  authors: [Author]
    @rest(
      endpoint: "https://xxxx.supabase.co/rest/v1/authors",
      configuration: "supabase_config",
      headers: [{ name: "apikey" value: "$apikey" }]
    )
}
```

### Deploy StepZen Endpoint

```bash
stepzen start
```

### Run a test query on StepZen

Open [localhost:5000/api/stepzen-supabase](https://localhost:5000/api/stepzen-supabase) and run the following query.

```graphql
query AUTHORS_QUERY {
  authors {
    id
    name
    created_at
    updated_at
  }
}
```

<img width="742" alt="06-authors-query" src="https://user-images.githubusercontent.com/12433465/133345765-79d17139-30af-4618-bdef-0da73821254b.png">

## Mutations

Add the ability to create books with StepZen mutations.

### Create a Books Table

Create `books` table on Supabase with the same schema as the `authors` table.

### `POST` book

Create a `book` with a curl POST request.

```bash
curl --request POST \
  --url "https://xxxx.supabase.co/rest/v1/books" \
  --header "apikey: xxxx" \
  --header 'content-type: application/json' \
  --data '{ "name": "curl-book-test" }'
```

### `GET` books

Return the `curl-book-test` book.

```bash
curl --request GET \
  --url 'https://xxxx.supabase.co/rest/v1/books' \
  --header "apikey: xxxx"
```

```json
[
  {
    "id":1,
    "created_at":"2021-09-14T23:10:39.831747+00:00",
    "updated_at":"2021-09-14T23:10:39.831747+00:00",
    "name":"curl-book-test"
  }
]
```

### `book.graphql`

Create a `Book` type with types for `id`, `name`, `created_at`, and `updated_at`.

```graphql
# schema/book.graphql

type Book {
  id: ID!
  name: String!
  created_at: DateTime!
  updated_at: DateTime!
}
```

The `books` query returns an array of `Book` objects.

```graphql
# schema/book.graphql

type Query {
  books: [Book]
    @rest(
      endpoint: "https://xxxx.supabase.co/rest/v1/books",
      configuration: "supabase_config",
      headers: [{ name: "apikey" value: "$apikey" }]
    )
}
```

The `createBook` mutation takes the `name` of a book as an argument.

```graphql
# schema/book.graphql

type Mutation {
	createBook(name: String!): Book
		@rest(
      endpoint: "https://xxxx.supabase.co/rest/v1/books"
		  configuration: "supabase_config"
      headers: [{ name: "apikey" value: "$apikey" }]
		  method: POST
		  postbody: "{\"name\": \"{{.Get \"name\"}}\"}"
		)
}
```

### Run a test mutation on StepZen

Create a book with a GraphQL mutation.

```graphql
mutation CREATE_BOOK {
  createBook(name: "stepzen-book-test") {
    id
    name
    created_at
    updated_at
  }
}
```

Return all the books.

```graphql
query BOOKS_QUERY {
  books {
    id
    name
    created_at
    updated_at
  }
}
```

<img width="691" alt="07-books-query-after-mutation" src="https://user-images.githubusercontent.com/12433465/133346348-c515307d-1997-44d7-b78a-48e44daa4a55.png">