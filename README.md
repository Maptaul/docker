# Docker Communication Demo

Three endpoints demonstrate container networking.

## Build and run

```bash
docker build -t docker-demo .
docker run -p 3000:3000 docker-demo
```

## Container -> Internet

```bash
curl http://localhost:3000/internet/1
```

## Container -> Host / Atlas

For a local MongoDB instance:

```bash
docker run -d --name demo-app -p 3000:3000 \
  -e MONGO_ATLAS_URI="mongodb://host.docker.internal:27017/demo" \
  docker-demo
```

```bash
curl http://localhost:3000/host-db
```

For MongoDB Atlas, provide a complete URI:

```bash
docker run -d --name demo-app -p 3000:3000 \
  -e MONGO_ATLAS_URI="mongodb+srv://<user>:<password>@<cluster>.mongodb.net/demo" \
  docker-demo
```

## Container -> Container (MongoDB)

```bash
docker network create demo-net

docker run -d --name mongo-container --network demo-net \
  -e MONGO_INITDB_ROOT_USERNAME=root \
  -e MONGO_INITDB_ROOT_PASSWORD=secret \
  mongo:7

docker run -d --name demo-app --network demo-net -p 3000:3000 \
  -e MONGO_CONTAINER_URI="mongodb://root:secret@mongo-container:27017/demo?authSource=admin" \
  docker-demo

curl http://localhost:3000/container-db
```

Docker resolves `mongo-container` as a hostname inside `demo-net`.

## Cleanup

```bash
docker stop demo-app mongo-container
docker rm demo-app mongo-container
docker network rm demo-net
```
