# minikube-prometheus-demo
A sample application deployment on minikube and setting up Prometheus and Grafana into Minikube cluster.

# Required Steps to Deploy Service/Application using Minikube + Kubernetes
## Requirements:
- Install Docker (**https://docs.docker.com/get-docker/**)
- Install Minikube (**https://minikube.sigs.k8s.io/docs/start/**)
- Any application

## Create a Docker Image for an application
### Steps:
1. Clone the git repository that contains the application you wish to deploy to Minikube
2. **`cd`** to project directory
3. Create an empty docker file and named as **`Dockerfile`** in the root project directory
4. Paste following content in **`Dockerfile`** (for any application using npm dependency manager)
```
FROM node:alpine

WORKDIR /app

COPY package.json ./

COPY package-lock.json ./

COPY ./ ./

RUN npm install --save

CMD ["npm", "run", "start"]
```
5. Run the following command to build a new container image:
```
$ docker build -t <docker-username>/<project-build-name> .
```
6. Allow docker to build an image
7. Upload image to Docker hub (Can be done through docker UI)

## Deploying docker image to Minikube cluster using Kubernetes
### Steps:
1. Run the following command to start Minikube cluster:
```
$ minikube start
```
2. Run the following command once Minikube is started to create a service pod:
```
$ kubectl create deployment <pod-name> --image=docker.io/<image-name>
```
3. Run the following command to view pods to ensure service is running:
```
$ kubectl get po -A
```
4. Run the following command to expose the service to port:
```
$ kubectl expose deployment <pod-name> --type=NodePort --port=80
```
5. Run the following command to forward service to view it on localhost:
```
$ kubectl port-forward <pod-name> 3000
```



# Required Steps to Deploy Prometheus and Grafana into Minikube Cluster Using Helm Chart 
Prometheus is used to monitor Kubernetes Cluster and other resources running on it.  
Grafana helps to visualize metrics recorded by Prometheus and display them in dashboards. 

## Start Minikube 
Run following command to start minikube 
```
minikube start 
```

## Install Helm 
1. Download Helm from **https://github.com/helm/helm/releases** 
2. Unzip the downloaded folder and set that “path” at environment variables. 
3. Test command **`helm`** in PowerShell. 

## Install Prometheus 

1. Add Prometheus repository 
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts 
```
2. Install provided Helm chart for Prometheus 
```
helm install prometheus prometheus-community/prometheus 
```
3. Expose the prometheus-server service via NodePort 
```
kubectl expose service prometheus-server --type=NodePort --target-port=9090 --name=prometheus-server-np 
```
4. Check services 
```
kubectl get svc 
```
5. Access Prometheus UI by exposing service URL 
```
minikube service prometheus-server-np –url 
``` 

**Prometheus UI**
####
![Prometheus UI](https://github.com/cirruslabs-io/minikube-prometheus-demo/blob/main/Images/1.%20Prometheus%20UI.png?raw=true)

## Install Grafana 

1. Add Grafana helm repo 
```
helm repo add grafana https://grafana.github.io/helm-charts 
```
2. Install Grafana chart 
```
helm install grafana grafana/grafana 
```
3. Expose Grafana service via NodePort to access Grafana UI 
```
kubectl expose service grafana --type=NodePort --target-port=3000 --name=grafana-np 
```
4. Check exposed service 
```
kubectl get services 
```
## Access Grafana UI
1. Get Grafana admin credentials (Note: **`base64 --decode`** is an unrecognizeable command on Windows natively. To use base64 commands either use a linux kernel or to use with Windows install base64 through a package manager such as Chocolatey or use an online base64 decoder tool)
```
kubectl get secret --namespace default grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo 
```
2. Access Grafana UI by exposing Grafana service in minikube 
```
minikube service grafana-np –url 
```
3. Access UI and enter admin credentials to login
####
![Grafana Login UI](https://github.com/cirruslabs-io/minikube-prometheus-demo/blob/main/Images/2.%20Grafana%20UI%20-%20login.png?raw=true)
&nbsp;<br>

4. Use the **`prometheus-server:80`** URL to configure the Prometheus data source.
####
![Grafana Data Source Config UI](https://github.com/cirruslabs-io/minikube-prometheus-demo/blob/main/Images/3.%20Grafana%20-%20Add%20Prometheus%20Datasource.png?raw=true)
&nbsp;<br>

5. Import default Prometheus dashboards 
####
![Grafana Default Dashboard](https://github.com/cirruslabs-io/minikube-prometheus-demo/blob/main/Images/4.%20Grafana%20-%20Import%20Default%20Prometheus%20Dashboard.png?raw=true)
&nbsp;<br>

6. If desired, a community based Grafana dashboard can be imported using id 6417. In the dashboard configuration select the Prometheus Data source that was created in step 4 to link the new dashboard to the data source. 
####
![Grafana Import Dashboard Config UI](https://github.com/cirruslabs-io/minikube-prometheus-demo/blob/main/Images/5.%20Grafana%20-%20Import%20Community%20Dashboard.png?raw=true)
&nbsp;<br>
&nbsp;<br>

7. Confirm the import dialog to be redirected to the new Dashboard where cluster information/metrics can be viewed.
####
![Grafana Community Dashboard](https://github.com/cirruslabs-io/minikube-prometheus-demo/blob/main/Images/6.%20Grafana%20-%20Cluster%20Dashboard.png?raw=true)
