# GraphQL con Apollo

## Setup

```kotlin
dependencies {
    implementation("com.apollographql.apollo3:apollo-runtime:3.8.2")
}
```

## Query

```kotlin
query GetUsers {
  users {
    id
    name
    email
  }
}

// Uso
val response = apolloClient.query(GetUsersQuery()).execute()
val users = response.data?.users
```

## Mutation

```kotlin
mutation CreateUser($name: String!, $email: String!) {
  createUser(name: $name, email: $email) {
    id
    name
  }
}

// Uso
apolloClient.mutation(CreateUserMutation(name, email)).execute()
```

## Recursos
- [Apollo Android](https://www.apollographql.com/docs/kotlin/)
