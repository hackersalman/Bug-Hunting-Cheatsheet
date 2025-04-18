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
