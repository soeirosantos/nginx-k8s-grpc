FROM iron/go:dev
RUN go get -u google.golang.org/grpc
RUN go get -u github.com/golang/protobuf/protoc-gen-go
ADD . /go/src/google.golang.org/grpc/examples/helloworld/server
RUN go install google.golang.org/grpc/examples/helloworld/server
ENTRYPOINT ["/go/bin/server"]
EXPOSE 50051