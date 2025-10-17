###  tips

+ این نسخه ای هست که اینجا استفاده میشه:

```
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@v1.2.0
go install google.golang.org/protobuf/cmd/protoc-gen-go@v1.28.1
```

+  ساخت

```
protoc \            
  --proto_path=src/presentation/grpc/proto \
  --go_out=src/presentation/grpc/pbs --go_opt=paths=source_relative \
  --go-grpc_out=src/presentation/grpc/pbs --go-grpc_opt=paths=source_relative \
  src/presentation/grpc/proto/plannig.proto
```

or

از روت پروژه


```
.
├── go.mod
├── go.sum
├── main.go
└── proto
    └── PublicLimitDepthsV3Api.proto


protoc --go_out=. --go_opt=paths=source_relative proto/PublicLimitDepthsV3Api.proto

```

# websocket

### Thread Safety for the Connection

Thread Safety for the Connection: The websocket.Conn from the Gorilla WebSocket library is not fully thread-safe