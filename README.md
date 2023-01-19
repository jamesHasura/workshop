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

### Mutation

Let's add a user using a GraphQL Mutation. Copy the following code into the GraphiQL interface.

`mutation {
  insert_users(objects:[{id: "1", name:"Praveen"}]) {
    affected_rows
  }
}`

Click on the Play button on the GraphiQL interface to execute the query.

Now let's go ahead and query the data that we just inserted.


`query {
  users {
    id
    name
    created_at
  }
}`

Note that some columns like created_at have default values, even though you did not insert them during the mutation.


### Subscription

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

`id (type Integer (auto-increment))`

`title (type Text)`

`is_completed (type Boolean and default false)`

`is_public (type Boolean and default false)`

`created_at (type Timestamp and default now())`

`user_id (type Text)`

The columns are associated with properties of todo items.

Remember to set the id column to the primary key.

In the Hasura Console, head over to the DATA tab section and click on Create Table. Enter the values for creating the table as mentioned above.

### Explore todos on GraphQL API

Similar to the users table, the todos table created in the previous step would have an auto-generated GraphQL API for us to explore.

Let's go ahead and start exploring the GraphQL API for todos table.

### Mutation

Head over to Console -> API -> GraphiQL tab and insert a todo using GraphQL Mutations.

`mutation {
  insert_todos(objects:[{title: "My First Todo", user_id: "1"}]) {
    affected_rows
  }
}`

### Query

Now let's go ahead and query the data that we just inserted.

`query {
  todos {
    id
    title
    is_public
    is_completed
    user_id
  }
}`

### Relationships

Relationships enable you to make nested object queries if the tables/views in your database are connected.

GraphQL schema relationships can be either of

#### object relationships (one-to-one)

#### array relationships (one-to-many)


### Object Relationships

Let's say you want to query todos and more information about the user who created it. This is achievable using nested queries if a relationship exists between the two. This is a one-to-one query and hence called an object relationship.

An example of such a nested query looks like this:

`query {
  todos {
    id
    title
    user {
      id
      name
    }
  }
}`

In a single query, you are able to fetch todos and its related user information. This can be very powerful because you can nest to any level.

### Array Relationships

Let's look at an example query for array relationships.

`query {
  users {
    id
    name
    todos {
      id
      title
    }
  }
}`

In this query, you are able to fetch users and for each user, you are fetching the todos (multiple) written by that user. Since a user can have multiple todos, this would be an array relationship.

Relationships can be captured by foreign key constraints. Foreign key constraints ensure that there are no dangling data. Hasura Console automatically suggests relationships based on these constraints.

Though the constraints are optional, it is recommended to enforce these constraints for data consistency.

The above queries won't work yet because we haven't defined the relationships yet. But this gives an idea of how nested queries work.


### Create Foreign Key

In the todos table, the value of user_id column must be ideally present in the id column of users table. Otherwise, it would result in inconsistent data.

Postgres allows you to define foreign key constraint to enforce this condition.

Let's define one for the user_id column in todos table.

Head over to Console -> DATA -> todos -> Modify page.

Scroll down to Foreign Keys section at the bottom and click on Add.

![image](https://user-images.githubusercontent.com/104997905/213422410-1dda7807-6ee1-4c49-86f9-7dc14cbbdb36.png)

Select the Reference table as users

`Choose the From column as user_id and To column as id`

We are enforcing that the user_id column of todos table must be one of the values of id in users table.

Click on Save to create the foreign key.

Great! Now you have ensured data consistency.

### Create Relationship

Now that the foreign key constraint is created, Hasura Console automatically suggests relationships based on that.

Head over to Relationships tab under todos table and you should see a suggested relationship like below:

![image](https://user-images.githubusercontent.com/104997905/213423073-06e4fef1-d52a-4c44-a894-286425ad705b.png)

Click on Add in the suggested object relationship.

Enter the relationship name as user (already pre-filled) and click on Save.

![image](https://user-images.githubusercontent.com/104997905/213423197-566c7cbe-32a2-429e-98d9-98da3a0f0904.png)

A relationship has now been established between todos and users table.

### Try out Relationship Queries

Let's explore the GraphQL APIs for the relationship created.

`query {
  todos {
    id
    title
    user {
      id
      name
    }
  }
}`

As you can see, in the same response, you are getting the results for the user's information, exactly like you queried. This is an example of a one-to-one query/object relationship.

### Data Transformations

One of the realtime features of the todo app is to display the list of online users. We need a way to fetch this information based on the value of last_seen which tells when the user was last online.

So far we were building tables and relationships. Postgres allows you to perform data transformations using:

- Views
- SQL Functions

In this example, we are going to make use of Views. This view is required by the app to find the users who have logged in and are online in the last 30 seconds.

### Create View

The SQL statement for creating this view looks like this:

`CREATE OR REPLACE VIEW "public"."online_users" AS
 SELECT users.id,
    users.last_seen
   FROM users
  WHERE (users.last_seen >= (now() - '00:00:30'::interval));`
  
Let's add this view and track the view with Hasura to be able to query it.

Head to Console -> DATA -> SQL page.

![image](https://user-images.githubusercontent.com/104997905/213424283-c1fb28aa-0f41-49e9-8a40-c727f7c7619a.png)

### Subscription to Online Users

Now let's test by making a subscription query to the online_users view.

`subscription {
  online_users {
    id
    last_seen
  }
}`

In another tab, update an existing user's last_seen value to see the subscription response getting updated.

![image](https://user-images.githubusercontent.com/104997905/213424505-47784bfb-6683-4bdf-bd87-046bb686cb6a.png)

Enter the value as now() for the last_seen column and click on Save.

Now switch back to the tab where your subscription query is running to see the updated response.

### Create relationship to user

Now that the view has been created, we need a way to be able to fetch user information based on the id column of the view. Let's create a manual relationship from the view online_users to the table users using the id column of the view.

Head to Console -> Data -> online_users -> Relationships page.

Add a new relationship manually by choosing the relationship type to be Object Relationship. Enter the relationship name as user. Select the configuration for the current column as id and the remote table would be users and the remote column would be id again.

We are mapping the current view's id column to users table's id column to create the relationship.

![image](https://user-images.githubusercontent.com/104997905/213424851-32bb6184-7727-43d6-b000-de1290e32004.png)

Let's explore the GraphQL APIs for the relationship created.

`query {
  online_users {
    id
    last_seen
    user {
      id
      name
    }
  }
}`

Great! We are completely done with data modeling for the app.

### Authorization

In this part of the tutorial, we are going to define role-based access control rules for each of the models that we created.

Access control rules help in restricting querying on a table based on certain conditions.

In this realtime todo app use-case, we need to restrict all querying only for logged in users. Also, certain columns in tables do not need to be exposed to the user.

The aim of the app is to allow users to manage their own todos only but should be able to view all the public todos.

Setup todos table permissions
Head over to the Permissions tab under todos table to add relevant permissions.

### Insert permission

We will allow logged-in users creating a new todo entry to only specify the is_public and title columns.

In the enter new role textbox, type in “user”

Click on edit (pencil) icon for “insert” permissions. This would open up a section below, which lets you configure custom checks and allow columns.

In the custom check, choose the following condition

`{"user_id":{"_eq":"X-Hasura-User-Id"}}`

![image](https://user-images.githubusercontent.com/104997905/213425585-fc87e572-2072-43e9-83b6-ba6dc24c29a9.png)

Now under "Column insert permissions", select the title and is_public columns.

![image](https://user-images.githubusercontent.com/104997905/213425835-c0ef251a-d39e-467a-b653-31376ab5663e.png)

Finally under "Column presets", select user_id from from session variable mapping to X-HASURA-USER-ID.

Note: Session variables are key-value pairs returned from the authentication service for each request. When a user makes a request, the session token maps to a USER-ID. This USER-ID can be used in permission to show that inserts into a table are only allowed if the user_id column has a value equal to that of USER-ID, the session variable.

Click on Save Permissions.

### Select permission

We will allow users to view a todo entry if it is public or if they are logged-in users.

Now click on edit icon for "select" permissions. In the custom check, choose the following condition

`{"_or":[{"is_public":{"_eq":true}},{"user_id":{"_eq":"X-Hasura-User-Id"}}]}`

![image](https://user-images.githubusercontent.com/104997905/213426288-a41f0c46-43a9-4bd3-a4d8-baaa946626b1.png)

Under "Column select permissions", select all the columns.

![image](https://user-images.githubusercontent.com/104997905/213426352-9c5ba8e8-a5db-4d75-8b13-2af388ae0b9a.png)

Click on Save Permissions

### Update permission

We will only allow the is_completed column to be updated by a user.

Now click on edit icon for "update" permissions. In the pre-update custom check, choose With same custom checks as insert.

And under "Column update permissions", select the is_completed column.

![image](https://user-images.githubusercontent.com/104997905/213426672-43b9c27e-97a9-4fa2-87da-f19152d57513.png)

Click on Save Permissions once done.

### Delete permission

Only logged-in users are allowed to delete a todo entry.

Finally for delete permission, under custom check, choose With same custom checks as insert, pre update.


![image](https://user-images.githubusercontent.com/104997905/213426766-c68296bd-10ac-4e87-a75a-2763c0de52f6.png)

Click on Save Permissions and you are done with access control for todos table.

### Setup users table permissions

We also need to allow select and update operations into users table. On the left sidebar, click on the users table to navigate to the users table page and switch to the Permissions tab.

### Select permission

Click on the Edit icon (pencil icon) to modify the select permission for role user. This would open up a section below which lets you configure its permissions.

Here the users should be able to access every other user's id and name data.

![image](https://user-images.githubusercontent.com/104997905/213427174-9cfecb63-e18e-4e65-9d43-2c1c98ed5cf5.png)

Click on Save Permissions

### Update permission

The user who is logged in should be able to modify only their own record. So let’s set that permission now.

Now click on edit icon for "update" permissions. In the pre-update custom check, choose With custom check with following condition.

`{"id":{"_eq":"X-Hasura-User-Id"}}`

Under column update permissions, select last_seen column, as this will be updated from the frontend app.

![image](https://user-images.githubusercontent.com/104997905/213428108-be20767a-b3dd-4c7e-bc41-4159967a9fc0.png)

Click on Save Permissions and you are done with access control rules for users table.



