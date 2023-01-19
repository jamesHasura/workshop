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

Click on the Play button on the GraphiQL interface to execute the query.

Now let's go ahead and query the data that we just inserted.

`
query {
  users {
    id
    name
    created_at
  }
}

`

Note that some columns like created_at have default values, even though you did not insert them during the mutation.


#### Subscription

Let's run a subscription query over users table to watch for changes to the table.

`
subscription {
  users {
    id
    name
    created_at
  }
}
`

Initially, the subscription query will return the existing results in the response.

Now let's insert new data into the users table and see the changes appearing in the response.

In a new browser tab, Head over to Console -> DATA tab -> default -> public -> users -> Insert Row and insert another row.

And switch to the previous GRAPHIQL tab and see the subscription response returning 2 results.

An active subscription query will keep returning the latest set of results depending on the query.

### Create todos table

Now let's move on to creating the other model: todos

The todos table will have the following columns:

` id (type Integer (auto-increment))`

` title (type Text) `

` is_completed (type Boolean and default false) `

` is_public (type Boolean and default false) `

` created_at (type Timestamp and default now()) `

` user_id (type Text) `

The columns are associated with properties of todo items.

Remember to set the id column to the primary key.

In the Hasura Console, head over to the DATA tab section and click on Create Table. Enter the values for creating the table as mentioned above.

### Explore todos on GraphQL API

Similar to the users table, the todos table created in the previous step would have an auto-generated GraphQL API for us to explore.

Let's go ahead and start exploring the GraphQL API for todos table.

#### Mutation

Head over to Console -> API -> GraphiQL tab and insert a todo using GraphQL Mutations.

