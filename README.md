Terminal

cd into folder to put synbiohub-docker in

git clone -b snapshot <https://github.com/SynBioHub/synbiohub-docker.git>

docker-compose -f ./synbiohub-docker/docker-compose.yml up

Remember to remove the old image and container before using a new one. Otherwise, Docker will just continue to use the old one. 

