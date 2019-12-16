This project will allow you to create a local persisted SonarQube instance that acts as your own personal sandbox
for performing static analysis. You deploy this onto your local Kubernetes cluster (via Minikube) where you would
expose your service.

**Installed will be the latest LTS SonarQube 7.9.1-community version.**

To learn more about the 7.9 LTS features of SonarQube, please refer to the following documents
[7.9 LTS SonarQube Features](https://www.sonarqube.org/sonarqube-7-9-lts/)

To learn more about Minikube and/or Kubernetes, please refer to the following document
[Minikube](https://kubernetes.io/docs/setup/learning-environment/minikube/)

---
#### Explanation of each Kubernetes config file
In order to persist changes within SonarQube, a persistent volume such as Postgres is required.

##### Postgres related
- `sonar-pv-postgres.yml`

    This is the Postgres **p**ersistent **v**olume that abstracts away the persistent storage.

    They have a lifecycle that is independent of the **Pod** and/or **Deployment** resources and are thus not impacted by their health.

    More information can be found here [Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistent-volumes)


- `sonar-pvc-postgres.yml`

    **P**ersistent **V**olume **C**laim.

    This is the request for storage by a user. A PVC consumes PV resources.

    More information can be found here [Persistent Volume Claims](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims)


- `sonar-postgres-deployment.yml`

    This is where the container specification for the Postgres Pod resource lives.

    The config file also connects the Postgres claim to the Pod resource via volume mounts.

    More information can be found here [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)


- `sonar-postgres-service.yml`

    This is where the Postgres service gets exposed (as a network service) to other resources (like the SonarQube pod resource).

    More information can be found here [Services](https://kubernetes.io/docs/concepts/services-networking/service/)

##### SonarQube related
- `sonarqube-deployment.yml`

    This is where the container specification for the 7.9.1-community version of SonarQube lives.

    The config file states that 2GB of memory is required.

    More information can be found here [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)


- `sonarqube-service.yml`

    This is where the SonarQube service gets exposed (as a network service) to other resources.

    It is with this config file that we are allowed to hit the SonarQube service from a browser.

    More information can be found here [Services](https://kubernetes.io/docs/concepts/services-networking/service/)

---
#### Install and run SonarQube locally

#####  Install Sonar scanner and Minikube (Kubernetes cluster)
1. `brew install sonar-scanner`
2. `brew install minikube`

#####  Configure Minikube to support Sonar server and start cluster (takes a few minutes)
1. `minikube config set memory 4096`
2. `minikube config set cpus 2`
3. `minikube config set vm-driver hyperkit`
4. `minikube start`

#####  Deploy your instance of SonarQube (will take a few minutes for pods to fully warm up and load SonarQube)
1. `kubectl create secret generic postgres-pwd --from-literal=password={some made up password}`
2. `kubectl create -f sonar-pv-postgres.yml`
3. `kubectl create -f sonar-pvc-postgres.yml`
4. `kubectl create -f sonar-postgres-deployment.yml`
5. `kubectl create -f sonarqube-deployment.yml`
6. `kubectl create -f sonarqube-service.yml`
7. `kubectl create -f sonar-postgres-service.yml`

##### Once all the pods are up and running, view your SonarQube instance (this will open a new browser tab with SonarQube)
1. `minikube service sonar`

##### To delete the Kubernetes deployment is as follows
1. `kubectl delete deployment sonar-postgres`
2. `kubectl delete deployment sonarqube`

#####  To stop Minikube cluster
1. `minikube stop`

#####  To delete the Minikube cluster
1. `minikube delete`

---
#### Perform static analysis

##### Java
1. In the root of your Java project, add an empty `sonar-project.properties` file. The `sonar-scanner` service will be 
looking for this file when performing static analysis.

2. Paste the following into the newly created `sonar-project.properties` file:
```
sonar.projectKey={name of project}
sonar.host.url=http://192.168.##.##:##### (url from minikube service sonar command)
sonar.login=${env.SONAR_TOKEN}
sonar.java.binaries=build/classes
sonar.sources=src/main/java
``` 

An example config would look like the following
```
sonar.projectKey=notificationemailproc
sonar.host.url=http://192.168.64.9:31828
sonar.login=${env.SONAR_TOKEN}
sonar.java.binaries=build/classes
sonar.sources=src/main/java
```

3. Create and copy a new SonarQube token by going to the SonarQube instance in the browser and navigating to 
My Account -> Security tab -> Enter Token Name -> Generate -> Copy token generated

4. In your .bashrc or .zshrc file, add the following line
`export SONAR_TOKEN={SonarQube token that was just copied}`

5. Reload your rc file
`source ~/.bashrc` or `source ~/.zshrc`

6. Run `sonar-scanner` in the project root directory. Once static analysis is finished, you can view the results in your 
SonarQube instance in the browser.

---
#### Useful SonarQube plugins
- FindBugs
- Checkstyle
- Mutation Analysis