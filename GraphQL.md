# Common end-point of GraphQL Api
```
/graphql
/api
/api/graphql
/graphql/api
/graphql/graphql
```

# Analyzing Graphql Api Schema via Introspection

Introspection is a built-in GraphQL function that enables us to query a server for information about the schema. It can represent a serious information disclosure risk, as it can be used to access potentially sensitive information `such as field descriptions` and help an attacker to learn how they can interact with the API. 

## Analyzing via Burp

We can use `Burp InQL` for introspection, `InQL > Provide URL of the GraphQL Endpoint > Analyze`. Now we can get the information of GraphQL structure.

## Manuall analyzing

In some cases, developers disable this Introspection feature. However we can bypass this by manually analyzing the

# Exploitation

## Accessing private GraphQL posts

Step:
  1. Copy the url that have Graphql endpoint.
  2. InQL >> Paste url >> Analyze
  3. In the box below we can get the analyzed details of the Graphql api.
  4. Send the query to the repeater and modify the details if needed. 
  
