# Deploy Supernet with Docker on Ubuntu 22.04

This tutorial assumes PolyBFT consensus

## Prerequisites:
- Docker & docker-compose [[Installation instructions](https://github.com/integrations-Polygon/Supernets-Edge-Tutorials/blob/master/Install_Docker.md)]
- Hardware Requirements

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
cd core-contracts && npm install && npm run compile && cd -
go run ./consensus/polybft/contractsapi/artifacts-gen/main.go
```

## 4. Build and Run Containers
```
export EDGE_CONSENSUS=polybft
docker-compose -f ./docker/local/docker-compose.yml up -d --build
```

## 5. Stop / Destroy Containers
Stop Containers
```
docker-compose -f ./docker/local/docker-compose.yml stop
```
Destroy Environments
```
docker-compose -f ./docker/local/docker-compose.yml down -v
```