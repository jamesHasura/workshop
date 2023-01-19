# Workshop

## Create Users Table

Let's get started by creating the users table.

The users table will have the following columns:

`id (type Text and Unique)`

`name (type Text)`

`created_at (type Timestamp and default now())`

`last_seen (type Timestamp and nullable)`


The columns are associated with properties of users. The last_seen column is used to store the latest timestamp of when the user was online.

In the Hasura Console, head over to the DATA tab section and click on the database (from the left side navigation) that we connected earlier. The database name would be default and the schema name would be public. Once you land on the public schema, click on Create Table. Enter the values for creating the table as mentioned above. For the table property of Primary Key, use id.


## Explore users on GraphQL API

Hasura gives you Instant GraphQL APIs over Postgres, among other databases. Therefore, it can be tested on the table that we just created.

Let's go ahead and start exploring the GraphQL API for users table. We are going to use GraphiQL to explore the API. GraphiQL is the GraphQL integrated development environment (IDE). It's a powerful tool we'll use to interact with the API.

You can access GraphiQL by heading over to Console -> API -> GraphiQL tab.

#### Mutation

Let's add a user using a GraphQL Mutation. Copy the following code into the GraphiQL interface.

`
mutation {
  insert_users(objects:[{id: "1", name:"Praveen"}]) {
    affected_rows
  }
}
`



