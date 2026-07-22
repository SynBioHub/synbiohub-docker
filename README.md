Terminal
cd into folder to put synbiohub-docker in
git clone -b snapshot <https://github.com/SynBioHub/synbiohub-docker.git>
docker-compose -f ./synbiohub-docker/docker-compose.yml up
Remember to remove the old image and container before using a new one. Otherwise, Docker will just continue to use the old one. 
For spoofing purposes: on setup, change URI prefix to https://synbiohub.org/

To run on [sbol-db](https://github.com/marpaia/sbol-db) instead of Virtuoso, use `docker-compose.sboldb.yml` in place of `docker-compose.yml`:
docker-compose -f ./synbiohub-docker/docker-compose.sboldb.yml up
sbol-db answers on the `virtuoso` network alias at port 8890 and exposes the same HTTP SPARQL endpoints, so SynBioHub's triplestore configuration is unchanged — the triplestore is selected purely by which compose file you use.
