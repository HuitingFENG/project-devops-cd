# project-devops-cd
  - M2 SE1 promo 2024
  - GrpLog(x)

## Contributors: 
  - Van Alenn PHAM
  - Senhua LIU
  - Jonathan EL BAZ
  - Camille FOUR
  - Huiting FENG

## Tools
  - Docker
  - Kubernetes
  - Jenkins
  - Buildpack utility
  - Prometheus stack (Helm Chart)
  - Grafana (Helm Chart)
  - grafana/loki-stack (Grafana Offical Helm Chart)


## Steps
### Part One: Build and deploy an application using Docker / Kubernetes and Jenkins pipeline. 
1. Diagram of solution, target architecture and tool chain suggested

- Docker: A platform for developing, shipping, and running applications inside containers.
- Kubernetes (Minikube): Minikube runs a single-node Kubernetes cluster on our computer for users looking to try out Kubernetes or develop with it day-to-day.
- Helm: a package manager for Kubernetes, simplifying deployment of apps and services.
- Prometheus and Grafana: Prometheus is an open-source monitoring system with a dimensional data model, and Grafana is an open-source, feature-rich metrics dashboard and graph editor Graphite, Elasticsearch, OpenTSDB, Prometheus, and InfluxDB.

2. Customize and deploy the application on local docker engine
- go to the /webapi/main.go file, do some modifications so that the /whoami endpoint displays our team's name
![](/images/0.png)
- go to the /webapi folder, write a 'Dockerfile' for this application
- build the docker image: 
```
docker build -t project-devops-cd .
```
- run the container locally: 
```
docker run -p 8080:8080 project-devops-cd
```
- list the running containers:
```
docker ps
```
and we know that there are two running containers:
![](/images/1.png)
- go to check the application in a web browser or use command lines:
```
curl http://localhost:8080/
```
```
curl http://localhost:8080/whoami/
```
![](/images/2.png)
- 
- use Jenkins on Docker to set up Jenkins (MacOS)
```
docker run -d -p 9090:8080 -p 50000:50000 --name jenkins -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts
```
![](/images/3.png)
the result shows that our jenkins instance is accessible on port 9090 for the web interface and port 50000 for agent connections. We can then access the Jenkins web interface (http://localhost:9090) by navigating to in our web browser.
- find the initial admin password for our jenkins instance running in a docker container:
```
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```
99dbc65928a3418eb9261cef89fd0b72
- choose to install the recommended plugins, skip to continue as admin and choose to not configure the url of jenkins
- create a new job (type is pipeline) and configure source control using this url of github project on the configuration page
- start a Jenkins agent via Java Web Start (JNLP) and make the agent machine can reach the Jenkins master server at 'localhost:9090', but we need to ensure that the agent is running on the same machine as the Jenkins server.
```
curl -sO http://localhost:9090/jnlpJars/agent.jar
java -jar agent.jar -url http://localhost:9090/ -secret bb18ff68dc081ffa2943a8f8a2e28164345d1ad47d9214a59ef1f363d2d51e1b -name "jenkins-slave" -workDir "/tmp"
```
```
docker run -p 9090:8080 -p 50000:50000 jenkins/jenkins:lts
```
63997f3750a64fc5aa8bb3c928fe5709
- go to Jenkins web interface and get Jenkins API token : 11143b75437776f2eaa7ffb4cdbaffdb8d
- add build step directly using the pipeline script on Jenkins or in our case use a Jenkinsfile on local machine(cf. /webapi/Jenkinsfile)
- we will combine the "curl" command and a "Jenkinsfile" to provide a automated, and version-controlled way to manage our CI/CD pipeline.
- after create and write pipeline script on the "Jenkinsfile"
- use "curl" to trigger builds
```
curl -X POST http://localhost:9090/job/project-devops-cd/build --user admin:11143b75437776f2eaa7ffb4cdbaffdb8d  
```
- wait a moment, manually check the build status in the Jenkins UI to see if completed and then display the output of build on our local machine's terminal
```
curl http://localhost:9090/job/project-devops-cd/lastBuild/consoleText --user admin:11143b75437776f2eaa7ffb4cdbaffdb8d 
``` 
- scroll down to the end of result and verify the output on terminal
![](/images/4.png)
- go to the web browser (http://localhost:9090/job/project-devops-cd/) and check the output of build to compare results from terminal
![](/images/5.png)



3. Deploy the application on Kubernetes (minikube) cluster
  
  - 3.1. In the development environment

  - 3.2. In the production environment

4. Build the docker image with buildpack utility and compare it with dockerfile option


### Part Two: Monitoring and incident management for containerized application.
1. Access to Granafa web UI and configure a data source with the deployed Prometheus service URL
  - username: admin
  - password: 8tpAxdyYFIjiS8YbbV2RuLDKLnyPFAoDHzq4t3Gk


2. Configure Alert Manager component and setup Alerts


3. Configure another alert and send it by email to teacher


### Part Three: Logs Management
1. Install the grafana/loki-stack chart from Grafana Official Helm Chart

2. Configure the Loki datasource and create a query on the Grafana application



## References:

  https://github.com/efrei2023/ST2DCE-PRJ