# Solution

## Technical architecture of the system

The typical **Customer** workflow for generating an itinerary: he can optionally apply localization filters such as coordinates, a radius-based area lookup, or specific store or brand names to narrow down the search. Next, the user selects a store and chooses the desired products. The system then displays the available offers, with the option to filter and view only the products that are available in the selected store. After making the necessary selections, the user presses the “**Generate Route**”\*\* \*\*button, and the system produces a final image-based graph representing the route.

# APIs

To support a "one-click" onboarding experience for Owners, we provide composite operations. When a store registers a new Stand, the system automatically upserts the underlying Product and Article records if they do not already exist.

To satisfy complex intents - such as de-listing an item from a shelf without deleting it from the brand catalog; we expose Lower-Level APIs. Articles and Products exist independently of Stands. Removing a Stand entry does not destroy the Article record, allowing it to be easily re-placed or utilized by other stores of the same brand.

## Services

### 1. Route Service

- Actors: Customers
- Resources: Routes
- **/routes** - POST, GET, DELETE
- Store and stand selection interface

### 2. Map Service

- Actors: Employee
- Resources: Stands, Floors, Nodes, Edges
- **/stores/{storeId}/stands** - POST, GET, LIST, PUT, DELETE
- **/stores/{storeId}/floors** - POST, GET, PUT, DELETE
- Floor modelling interface

### 3. Business Service

- Actors: Brand/Store Owners
- Resources: Brands, Stores, Offers
- **/stores** = POST, GET, LIST, PATCH, DELETE
- **/brands** = POST, GET, PATCH, DELETE
- **/brands/{brandId}/articles** - POST, GET, PATCH, DELETE
- **/products** - POST, GET, PATCH, DELETE
- **/offers** - POST, GET, LIST, PATCH, DELETE

## Repositories

1. InAndOut-API-modelling
2. InAndOut-Route-Service
3. InAndOut-Mapping-Service
4. InAndOut-Business-Service
5. InAndOut-Customer-Interface
6. InAndOut-Mapping-Interface

# Authorization

The roles with higher ownership should inherit the permissions of those with less. For example a Store Owner should be able to edit the structure of the store.

## Employee/Owner Authentication

## Resource Deletion

## Rate limiting

# TSP Caching

This operation is computationally intensive and may generate significant costs. CDN caches avoid expensive recomputations and will be used. The CDN machine could also host a Redis instance.

Challenges:

- CPU Expense: High CPU overhead is required to recompute the same solution repeatedly.
- Solution Invalidation: Employees might update the store map, invalidating the cached solutions.

Redis Data Structure:

- Key Structure - `tsp:{storeId}:{mappingVersion}:{standsHash}`
- Value Structure - the actual output of the operation

Hash stand ids algoritms:

```
1.  Sort: Take the list of Stand ids and sort them numerically: [10, 2, 5] -> [2, 5, 10].
2.  Stringify: Join the sorted IDs with a delimiter: "2,5,10".
3.  Hash: Apply a fast hashing algorithm (like MD5 or SHA-1) to the string to produce a fixed-length suffix: a1b2c3d4.
```

Example:

`tsp:101:v4:a1b2c3d4` =
```json
[
  [
    { "id": 1, "name": "Entrance" },
    { "id": 12, "name": "Aisle 1" }
  ],
  [
    { "id": 12, "name": "Aisle 1" },
    { "id": 45, "name": "Milk Stand" }
  ]
]
```

# References

- [https://petstore3.swagger.io/](https://petstore3.swagger.io/)
- [https://dbdiagram.io/d/InAndOut-693154e7d6676488ba8f6f4d](https://dbdiagram.io/d/InAndOut-693154e7d6676488ba8f6f4d)
- [https://github.com/apache/commons-math/blob/master/commons-math-examples/examples-sofm/tsp/src/main/java/org/apache/commons/math4/examples/sofm/tsp/TravellingSalesmanSolver.java](https://github.com/apache/commons-math/blob/master/commons-math-examples/examples-sofm/tsp/src/main/java/org/apache/commons/math4/examples/sofm/tsp/TravellingSalesmanSolver.java)
- [https://github.com/TheAlgorithms/Java/blob/master/src/main/java/com/thealgorithms/graph/TravelingSalesman.java](https://github.com/TheAlgorithms/Java/blob/master/src/main/java/com/thealgorithms/graph/TravelingSalesman.java)
- [https://github.com/swagger-api/swagger-petstore/blob/master/src/main/resources/openapi.yaml](https://github.com/swagger-api/swagger-petstore/blob/master/src/main/resources/openapi.yaml)
- [https://github.com/aws/aws-parallelcluster/blob/e6e636ac3550d7a2aabf10ee0245f0ddc9bdc5af/api/spec/smithy/model/parallelcluster.smithy#L8](https://github.com/aws/aws-parallelcluster/blob/e6e636ac3550d7a2aabf10ee0245f0ddc9bdc5af/api/spec/smithy/model/parallelcluster.smithy#L8)
- [https://dev.routific.com/use-cases/travelling-salesman-problem](https://dev.routific.com/use-cases/travelling-salesman-problem)
