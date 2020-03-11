---
lang: en
title: 'Migrating built-in authentication and authorization'
keywords: LoopBack 4.0, LoopBack 4, LoopBack 3, Migration
sidebar: lb4_sidebar
permalink: /doc/en/lb4/migration-auth-built-in.html
---

We have an application completely migrated from [`loopback-example-access-control`](https://github.com/strongloop/loopback-example-access-control). It builds with a LoopBack 4 authentication system based on `@loopback/authentication`, and an authorization system based on `@loopback/authorization`. This example exports the migrated token based authentication module as a component under `/src/components/jwt-authentication`. Our migration guide will use it as the reference.

## LoopBack 3 Authentication

The authentication system in LoopBack is explained and documented on [this page](https://loopback.io/doc/en/lb3/Introduction-to-User-model-authentication.html).

LoopBack 3 provides a built-in User model, which can be easily migrated to LoopBack 4 by `lb4 import-lb3-models`. And an AccessToken model to store a logged in user's information. Its authentication system works as:

When user logs in, it
  - creates a new token
  - token includes user info
  - token stored in db
  - returns token's id

After setting the token in the explorer, subsequent requests include the token in their headers, the authentication module
  - finds token record by id
  - returns the user info in the record
  - passes it to other modules in the system

Here is the diagram that describes the system:

![LoopBack 3 authentication](../../imgs/tutorials/access-control-migration/auth_example_scenario_model.png)

## LoopBack 4 Authentication

We can migrate it with following steps:

 - Migrate model definitions from LB3 User/AccessToken to LB4
 - Create a login controller to perform authentication
 - Create a token service to generate access tokens
 - Create an authentication strategy to validate access tokens

### Migrating models

_Migrating all functions for User need more effort, this section only focuses on model definition, login function_

You can run `lb4 import-lb3-models` to create an equivalent LoopBack 4 model `User`.

Import model `User`, `Project`, `Team` by the migration CLI:

```sh
$ cd myLB4App
$ lb4 import-lb3-models <path_to_loopback-example-access-control>
```

Choose model `User` for the prompt. It will be created under `src/models`.
Create the corresponding repository:

```sh
cd myLB4App
$ lb4 repository
? Please select the datasource DbDatasource
? Select the model(s) you want to generate a repository User
? Please select the repository base class DefaultCrudRepository (Legacy juggler bridge)
```

Create the user controller:

```sh
$ lb4 controller
? Controller class name: User
Controller User will be created in src/controllers/user.controller.ts

? What kind of controller would you like to generate? REST Controller with CRUD functions
? What is the name of the model to use with this CRUD repository? User
? What is the name of your CRUD repository? UserRepository
? What is the name of ID property? id
? What is the type of your ID? number
? Is the id omitted when creating a new instance? No
? What is the base HTTP path name of the CRUD operations? /users
   create src/controllers/user.controller.ts
   update src/controllers/index.ts

Controller User was created in src/controllers/
```

### Creating Login Function

For token based authentication, a user logs in by providing
correct credentials (email and password) in the payload of 'User/login', then
gets a token back with its identity information encoded and includes it in the
header of next requests.

Then add a new controller function `login` decorated with the REST decorators
that describe the request and response (see
[the complete user controller file](https://github.com/strongloop/loopback-next/tree/master/examples/access-control-migration/src/controllers/user.controller.ts)).
The core logic of `login` does 3 things:

- call `userService.verifyCredentials()` to verify the credentials and find the
  right user.
- call `userService.convertToUserProfile()` to convert the user into a standard
  principal shared across the authentication and authorization modules.
- call `jwtService.generateToken()` to encode the principal that carries the
  user's identity information into a JSON web token.

We will create the token service and user service in next section

### Creating Authentication Component

Learn LoopBack 4 authentication system in [this page](https://loopback.io/doc/en/lb4/Loopback-component-authentication.html).

_This tutorial doesn't include token persistency. You can create an AccessToken model+repository, modify the logic in Token service's verify token and generate token to store and retrieve it from database._

This demo uses token based authentication, and it uses the jwt authentication
strategy to verify a user's identity. The authentication setup is borrowed from
[loopback4-example-shopping](https://github.com/strongloop/loopback4-example-shopping/tree/master/packages).
The authentication system aims to understand **who sends the request**. It
retrieves the token from a request, decodes the user's information in it as
`principal`, then passes the `principal` to the authorization system which will
decide the `principal`'s access later.

Difference: JWT token encode all attributes into the id
but AccessToken stores such attributes in a DB, and only expose id

#### JWT Strategy

Takes in a request, extracts token from header, verify it and decode user info from it, and return the user profile

#### Services

Token service:

- generate token
  - encode user info by lib function `jwt.sign()`
- verify token
  - verify token by lib function `jwt.verify()`, it returns the decoded user
  - convert the user to user profile

#### Component

###
Describe usage of LB4 datasource and repository for the user/access token data access
 Migrate model definitions from LB3 User/AccessToken to LB4
 Create a login controller to perform authentication
 Create a token service to generate access tokens
 Create an authentication strategy to validate access tokens




