
buildctl build --frontend=dockerfile.v0 \
    --local context=. \
    --local dockerfile=. \
    --output type=image,name=localhost:5000/node-app:image,push=true \
    --export-cache type=registry,ref=localhost:5000/node-app:buildcache \
    --import-cache type=registry,ref=localhost:5000/node-app:buildcache