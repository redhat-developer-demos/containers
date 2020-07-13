# GO examples

## GO same from base UBI

This examples uses Red Hat Go-toolset UBI as a base to compile the GO binary and then a UBI-minimal to commit the container

```ruby
FROM registry.access.redhat.com/ubi8/go-toolset AS builder
WORKDIR /go/src/github.com/Developer/Go-Dummy
COPY . .

RUN GO111MODULE=auto GOOS=linux go build -o dummy /go/src/github.com/Developer/Go-Dummy/cmd/dummy

FROM registry.access.redhat.com/ubi8/ubi-minimal
COPY --from=builder /go/src/github.com/Developer/Go-Dummy/dummy /usr/bin/

ENTRYPOINT ["/usr/bin/dummy"]
LABEL io.k8s.display-name="a go miroservice" \
      io.k8s.description="a cool go microservice" 
```

Building the container

```bash
buildah bud -t imageName -f ContainerFile .  
```
