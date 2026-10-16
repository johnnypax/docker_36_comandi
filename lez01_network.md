docker network create network_cyber_uno
docker network create network_cyber_due

docker run -it --name ubu_net_uno --network network_cyber_uno ubuntu:24.04
docker run -it --name ubu_net_due --network network_cyber_uno --network network_cyber_due ubuntu:24.04
docker run -it --name ubu_net_tre --network network_cyber_due ubuntu:24.04

apt-get update
apt-get install net-tools
apt-get install iputils-ping