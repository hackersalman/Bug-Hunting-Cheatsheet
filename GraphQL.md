## Common end-point of GraphQL Api

In some cases GraphQL endpoint is not visible, so we can try to bruteforce the endpoint.
```
/graphql
/graphql/
/graphql/v1
/graphql/v2
/graphql/api
/graphql/endpoint
/graphql/query
/graphql/playground
/graphql-console
/graphql-explorer
/api
/api/graphql
/api/v1/graphql
/api/v2/graphql
/api/gql
/api/query
/api/data/graphql
/gql
/gql/
/gql/api
/gql/v1
/gql/v2
/gql/query
/backend/graphql
/internal/graphql
/dev/graphql
/v1/graphql
/v2/graphql
/server/graphql
/graphql-server
/graphql/engine
/graphql-service
/graphql-gateway
/graphql_dev
/graphql_beta
/graphql_test
/graphql_test_api
/query
/query/graphql
/graphql/queryEngine
/graph
/graph/query
/graphqljson
/json/graphql
/graphql.json
/graphqlapi
/graphqlinterface
/graphqlapi/v1
/graphql/graph
/graphql-service/v1
```

## Analyzing Graphql Api With Burp

We can analyze Graphql api in various way using Burpsuite. Here are some methodologies:

**Analyzing via Introspection :**

`Repeater > Graphql > Set Introspection Query > Send > GraphQL > Save GraphQL queries to site map > Target > Site map` 

**Analyzing via InQL :**

`InQL > Provide URL of the GraphQL Endpoint > Analyze`

## Bypassing GraphQL introspection defenses

In some cases, developers disable this Introspection feature. However we can bypass this by manually analyzing the endpoint.

1. **Bypass with special characters**:
   Developers may use weak regex to block `__schema`. You can insert characters like **spaces, newlines, or commas**, which are ignored by GraphQL but may bypass the regex.

   * Example:

     ```graphql
     query {
       __schema
       {
         queryType { name }
       }
     }
     ```

2. **Change request method**:
   Introspection may be disabled only for certain request types.

   * Try **GET** instead of **POST**.
   * Use `Content-Type: application/x-www-form-urlencoded` instead of JSON.

3. **Example GET request with encoded query**:

   ```
   GET /graphql?query=query%7B__schema%0A%7BqueryType%7Bname%7D%7D%7D
   GET /graphql?query=query%7B__schema%7Btypes%7Bname%7D%7D%7D
   GET /graphql?query=query%7B__type%28name%3A%22User%22%29%7Bfields%7Bname%7D%7D%7D
   GET /graphql?query=query%7B__schema%7Bdirectives%7Bname%20locations%7D%7D%7D
   GET /graphql?query=query%7B__schema%7BmutationType%7Bfields%7Bname%7D%7D%7D%7D
   GET /graphql?query=query%7B__schema%7BsubscriptionType%7Bfields%7Bname%7D%7D%7D%7D
   GET /graphql?query=query%7B__type%28name%3A%22User%22%29%7Bfields%7Bname%20args%7Bname%20type%7Bname%20kind%7D%7D%7D%7D%7D
   GET /graphql?query=query%7B__schema%0A%7BqueryType%0A%7Bname%7D%7D%7D
   GET /graphql?query=query%7B__typename%7D
   ```
For more: [Portswigger](https://portswigger.net/web-security/graphql#bypassing-graphql-introspection-defenses)

Portswigger Lab: [Finding a hidden GraphQL endpoint](https://portswigger.net/web-security/graphql/lab-graphql-find-the-endpoint)

PoC: [YouTube](https://youtu.be/RBKU4s0-Gd0?si=a92s8NIBi2z0DmUj)

## Accessing private GraphQL posts

Step:
  1. Copy the url that have Graphql endpoint.
  2. InQL >> Paste url >> Analyze
  3. In the box below we can get the analyzed details of the Graphql api.
  4. Send the query to the repeater and modify the details if needed. 
  
## Bypassing rate limiting using aliases

Details: [Portswigger](https://portswigger.net/web-security/graphql#bypassing-rate-limiting-using-aliases)

Portswigger Lab: [Bypassing GraphQL brute force protections](https://portswigger.net/web-security/graphql/lab-graphql-brute-force-protection-bypass)

PoC: [YouTube](https://www.youtube.com/watch?v=sKWah9eqWR4)

## GraphQL CSRF

Details: [Portswigger](https://portswigger.net/web-security/graphql#bypassing-rate-limiting-using-aliases)

Portswigger Lab: [Performing CSRF exploits over GraphQL](https://portswigger.net/web-security/graphql/lab-graphql-csrf-via-graphql-api)

PoC: [YouTube](https://www.youtube.com/watch?v=FVW2ZRNAyw8)
