## Common end-point of GraphQL Api
```
/graphql
/graphql/v1
/api
/api/graphql
/graphql/api
/graphql/graphql
```

## Analyzing Graphql Api With Burp

We can analyze Graphql api in various way using Burpsuite. Here are some methodologies:

**Analyzing via Introspection :**

`Repeater > Graphql > Set Introspection Query > Send > GraphQL > Save GraphQL queries to site map > Target > Site map` 

**Analyzing via InQL :**

`InQL > Provide URL of the GraphQL Endpoint > Analyze`

<!-- ## Manuall analyzing

In some cases, developers disable this Introspection feature. However we can bypass this by manually analyzing the -->

## Accessing private GraphQL posts

Step:
  1. Copy the url that have Graphql endpoint.
  2. InQL >> Paste url >> Analyze
  3. In the box below we can get the analyzed details of the Graphql api.
  4. Send the query to the repeater and modify the details if needed. 
  
