# .NET Core Examples

## Simple Multi-Stage example based on Red Hat's UBI images

```bash
FROM registry.access.redhat.com/ubi8/dotnet-31 AS build
COPY . ./
WORKDIR /opt/app-root/src/app
RUN dotnet build -c Release -o compile && \
    cp -a wwwroot ./compile/

FROM registry.access.redhat.com/ubi8/dotnet-31-runtime AS runtime
COPY --from=build /opt/app-root/src/app/compile .

LABEL io.k8s.display-name="a dotnet miroservice" \
      io.k8s.description="a dotnet microservice" 

EXPOSE 8080
ENTRYPOINT ["dotnet"]
CMD ["app.dll"]
```

## Building the container

```bash
buildah bud -t imagaName -f ContainerFile .  
```

## Running the image
```bash
podman run --rm -dt -p 8080:8080 localhost/imagaName
```