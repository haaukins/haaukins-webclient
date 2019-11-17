# gRPC - Vue.js - Go - Envoy
It is a basic gRPC Hello Word example. It is implemented in go and Vue.js, respectively server and client side.
To enable the communication between them there is a proxy (Envoy), implemented in the Dockerfile.

## Prerequisites
- Go v1.11 and Golang
- npm
- Docker
- install grpc and protocol Buffer3 (protobuf)
```bash
# Install Go gRPC
$ go get -u google.golang.org/grpc

# Make sure you grab the latest version
$ curl -OL https://github.com/google/protobuf/releases/download/v3.6.1/protoc-3.6.1-linux-x86_64.zip

# Unzip
$ unzip protoc-3.6.1-linux-x86_64.zip -d protoc3

# Move protoc to /usr/local/bin/
$ sudo mv protoc3/bin/* /usr/local/bin/

# Move protoc3/include to /usr/local/include/
$ sudo mv protoc3/include/* /usr/local/include/

# Install protobuf for Go
$ go get -u github.com/golang/protobuf/protoc-gen-go

# Install protobuf for Web
$ git clone https://github.com/grpc/grpc-web /tmp/grpc-web
$ cd /tmp/grpc-web && sudo make install-plugin
```

## How to run it
Open three terminal, one for the server, one for the client and the other one for the Proxy

### Client
```bash
$ cd client
$ npm install
$ cp proto_grpc_web_pb.js proto_pb.js node_modules 
```
### Server
```bash
$ cd server
$ go run server.go
```

### Proxy
```bash
$ docker build -t envoy . 
$ docker run -d -p 8080:8080 --network=host envoy:latest
```

Run the client `npm run serve` and browse `http://localhost:8081/`

#### Developer Notes
Below the list of useful command i have used:
- ``vue create hello-world`` Create a Vue.js base project (It create a folder containing the project)
- ``protoc -I proto/ proto/proto.proto --go_out=plugins=grpc:proto`` Create the proto compiled code for Go (Run in the main folder)
- ``protoc proto.proto --js_out=import_style=commonjs,binary:../client --grpc-web_out=import_style=commonjs,mode=grpcwebtext:../client`` (Run in the proto folder)

The following links were useful to create this project:
- `https://medium.com/@aravindhanjay/a-todo-app-using-grpc-web-and-vue-js-4e0c18461a3e`
- `https://medium.com/swlh/ditching-rest-with-grpc-web-and-envoy-bfaa89a39b32`
