# gRPC

## Setup

```kotlin
dependencies {
    implementation("io.grpc:grpc-okhttp:1.60.0")
    implementation("io.grpc:grpc-protobuf-lite:1.60.0")
    implementation("io.grpc:grpc-kotlin-stub:1.4.0")
}
```

## Proto

```protobuf
syntax = "proto3";

service UserService {
  rpc GetUser (UserRequest) returns (User);
  rpc ListUsers (Empty) returns (stream User);
}

message UserRequest {
  int32 id = 1;
}

message User {
  int32 id = 1;
  string name = 2;
}
```

## Cliente

```kotlin
val channel = ManagedChannelBuilder
    .forAddress("localhost", 50051)
    .usePlaintext()
    .build()

val stub = UserServiceGrpc.newBlockingStub(channel)
val user = stub.getUser(UserRequest.newBuilder().setId(1).build())
```

## Recursos
- [gRPC Kotlin](https://grpc.io/docs/languages/kotlin/)
