In this example, I will show you how to push the docker image and its cache separately using buildctl. As an image registry, we are going to use https://hub.docker.com/_/registry, running as a container on our host.

As a docker image for this example, I will use https://github.com/vv173/buildkit-frontend

To be able to push images on a local registry using buildctl we need to add a configuration file to the buildkit daemon, that will allow "push" to the insecure registry using HTTP. We also need to enable host network sharing. The total command will look like this:

```
docker run --rm --privileged -d -v $(pwd):/etc/buildkit --network=host --name buildkit moby/buildkit --config /etc/buildkit/buildkitd.toml
```

Now we can start our local registry:

```
docker run -d -p 5000:5000 --restart always --name registry registry:2
```

Let's build our container image, and push it with cache separately to our local registry.

```
buildctl build \
    --frontend=dockerfile.v0 \
    --local context=. \
    --local dockerfile=. \
    --output type=image,name=localhost:5000/node-app:image,push=true \
    --export-cache type=registry,ref=localhost:5000/node-app:buildcache \
    --import-cache type=registry,ref=localhost:5000/node-app:buildcache \
    --opt build-arg:DOCKER_USERNAME=$DOCKER_USERNAME \
    --opt build-arg:DOCKER_PASSWORD=$DOCKER_PASSWORD \
    --ssh default
```

The error "importing cache manifest from localhost:5000/node-app:buildcache" can be ignored, because we don't have a cache in the registry at the moment.

After that command, we can remove all local buildkit cache and try to run this build again.

As you can see buildkit successfully imported the cache from the local registry and lasted about 40 seconds less.
