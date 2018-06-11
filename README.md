# jenkins_with_slave_CI-CD
This docker compose file will launch a jenkins master and slave for building a CI-CD Deployment pipeline

This project contains a jenkins-master directory with the required files for:
* Building  the docker image
* Autoconfigure Jenkins with a basic level of security
* Install all plugins required for creating a Ci/CD pipeline
* JNLP connection to the slave

This project also contains a jenkins-slave directory with the required files for:
* Building  the docker image
* Autoconfigure Jenkins slave to discover the master
* JNLP connection to the master

In order to run the project, you just run the following command on the root directory:

```docker-compose up -d```
