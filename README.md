# apollo-koa-constraint-directive

![GitHub actions test coverage](https://github.com/alexanderVu/apollo-koa-constraint-directive/workflows/Test%20Coverage/badge.svg)
[![codecov](https://codecov.io/gh/alexanderVu/apollo-koa-constraint-directive/branch/master/graph/badge.svg)](https://codecov.io/gh/alexanderVu/apollo-koa-constraint-directive)
[![Known Vulnerabilities](https://snyk.io/test/github/alexanderVu/apollo-koa-constraint-directive/badge.svg?targetFile=package.json)](https://snyk.io/test/github/alexanderVu/apollo-koa-constraint-directive?targetFile=package.json)

Allows using @constraint as a directive to validate input and output data. This module is for [*Apollo Graphql Koa middleware*](https://www.apollographql.com/docs/apollo-server/integrations/middleware/#gatsby-focus-wrapper), and support the latest [Apollo GraphQL](https://www.apollographql.com/) version 2.

It is mainly based on the module from [graphql-constraint-directive](https://github.com/confuser/graphql-constraint-directive), which is for Apollo version 1 only.
This module is an Inspired by [Constraints Directives RFC](https://github.com/APIs-guru/graphql-constraints-spec) and OpenAPI

## Install

```bash
npm install apollo-koa-constraint-directive
```

## Usage

```js
const Koa = require('koa');
const bodyParser = require('koa-bodyparser');
const { ApolloServer, makeExecutableSchema, gql } = require('apollo-server-koa')
const ConstraintDirective = require('apollo-koa-constraint-directive')

const app = new Koa()
const schemaDirectives = {
  constraint: ConstraintDirective,
};
const typeDefs = gql`
  scalar ValidateString
  scalar ValidateNumber
  
  directive @constraint(
    # String constraints
    minLength: Int
    maxLength: Int
    startsWith: String
    endsWith: String
    notContains: String
    pattern: String
    format: String

    # Number constraints
    min: Int
    max: Int
    exclusiveMin: Int
    exclusiveMax: Int
    multipleOf: Int
  ) on INPUT_FIELD_DEFINITION
  
  type Query {
    books: [Book]
  }
  type Book {
    title: String
  }
  type Mutation {
    createBook(input: BookInput): Book
  }
  input BookInput {
    title: String! @constraint(minLength: 5, format: "email")
  }`
const apollo = new ApolloServer({
  schema: makeExecutableSchema({ typeDefs, schemaDirectives })
});

app.use(bodyParser())
apollo.applyMiddleware({ app })
app.listen(8000)
```

## API

### String

#### minLength

```@constraint(minLength: 5)```
Restrict to a minimum length

#### maxLength

```@constraint(maxLength: 5)```
Restrict to a maximum length

#### startsWith

```@constraint(startsWith: "foo")```
Ensure value starts with foo

#### endsWith

```@constraint(endsWith: "foo")```
Ensure value ends with foo

#### contains

```@constraint(contains: "foo")```
Ensure value contains foo

#### notContains

```@constraint(notContains: "foo")```
Ensure value does not contain foo

#### pattern

```@constraint(pattern: "^[0-9a-zA-Z]*$")```
Ensure value matches regex, e.g. alphanumeric

#### format

```@constraint(format: "email")```
Ensure value is in a particular format

Supported formats:

- byte: Base64
- date-time: RFC 3339
- date: ISO 8601
- email
- ipv4
- ipv6
- uri
- uuid

#### password strength

```@constraint(passwordScore: 3)```
Ensure password value has estimated strength. [zxcvbn](https://github.com/dropbox/zxcvbn) is used under the hood. Possible strength values are between 1 and 5. Heigher is better

### Int/Float

#### min

```@constraint(min: 3)```
Ensure value is greater than or equal to

#### max

```@constraint(max: 3)```
Ensure value is less than or equal to

#### exclusiveMin

```@constraint(exclusiveMin: 3)```
Ensure value is greater than

#### exclusiveMax

```@constraint(exclusiveMax: 3)```
Ensure value is less than

#### multipleOf

```@constraint(multipleOf: 10)```
Ensure value is a multiple

### ConstraintDirectiveError

Each validation error throws a `ConstraintDirectiveError`. Combined with a formatError function, this can be used to customise error messages.

```js
{
  code: 'ERR_GRAPHQL_CONSTRAINT_VALIDATION',
  fieldName: 'theFieldName',
  context: [ { arg: 'argument name which failed', value: 'value of argument' } ]
}
```

```js
const formatError = function (error) {
  if (error.originalError && error.originalError.code === 'ERR_GRAPHQL_CONSTRAINT_VALIDATION') {
    // return a custom object
  }
  return error
}

const apollo = new ApolloServer({
  schema: makeExecutableSchema({ typeDefs, schemaDirectives }),
  formatError
});

```
