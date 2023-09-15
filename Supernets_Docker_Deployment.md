# Deploy Supernet with Docker on Ubuntu 22.04

## Prerequisites:
- Docker & docker-compose [[Installation instructions](https://github.com/integrations-Polygon/Supernets-Edge-Tutorials/blob/master/Install_Docker.md)]
- [Hardware Requirements](https://wiki.polygon.technology/docs/supernets/operate/supernets-requirements)

## 1. Clone Repository
```
git clone https://github.com/0xPolygon/polygon-edge.git
cd polygon-edge
```
## 2. Install core-contracts
```
make download-submodules
```

## 3. Compile Contracts 
```
cd core-contracts && npm install && npm run compile
cd ../docker/local
```

## 4. Build and Run Containers
```
docker-compose build
docker-compose up -d
```

## 5. Stop / Destroy Containers
Stop Containers
```
docker-compose stop
```
Destroy Environments
```
docker-compose down
```