# Go Web Application

This is a simple website written in Golang. It uses the `net/http` package to serve HTTP requests.

## Prerequisites

kubectl – A command line tool for working with Kubernetes clusters.  
  For more information, see *Installing or updating kubectl*.

eksctl – A command line tool for working with EKS clusters that automates many individual tasks.  
  For more information, see *Installing or updating eksctl*.

AWS CLI – A command line tool for working with AWS services, including Amazon EKS.  
  For more information, see *Installing, updating, and uninstalling the AWS CLI*.  
  After installing the AWS CLI, configure it using:

```bash
aws configure
```

## Running the server

To run the server locally , execute the following command:

```bash
go build -o main .
./main
```

The server will start on port 8080. You can access it by navigating to `http://localhost:8080/Home` in your web browser.

# DevOpsify the go web application

The main goal of this project is to implement DevOps practices in the Go web application. The project is a simple website written in Golang. It uses the net/http package to serve HTTP requests.

DevOps practices include the following:

1.Creating Dockerfile (Multi-stage build)
2.Containerization
3.Continuous Integration (CI)
4.Continuous Deployment (CD)

# Create a Docker file

Create,build and run it to check if it's working or not.

```bash
docker build -t pankswork/go-web-app-test-1:v1 .
docker run -p 8080:8080 -it pankswork/go-web-app-test-1:v1
```

Visit http://localhost:8080/home

# Create folder k8s/manifests

Under manifests folder Create deployement,service and ingress yaml files.

# Create a AWS-EKS cluster

Create AWS EKS Cluster
```bash
eksctl create cluster   --name gocluster   --region us-east-1   --zones us-east-1a,us-east-1b   --node-type t3.small   --nodes 2 
```

Apply yaml files:

```bash
kubectl apply -f k8s/manifests/deployment.yaml
kubectl apply -f k8s/manifests/service.yaml
kubectl apply -f k8s/manifests/ingress.yaml
```

# Access the Website

We cant simply access the website yet beacuse if you run:

```bash
kubectl get ing
```

you will notice that there is no address, so we need to install the ingress controller which will assign the address for the ingress resource.

To install nginx ingress controller run:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.11.1/deploy/static/provider/aws/deploy.yaml
```

To check if its installed or not run:
```bash
kubectl get pods -n ingress-nginx
```

Since in our ingress.yaml we said host:go-web-app.local so we have to visit go-web-app.local/home but 1 more thing since this host doest have DNS mapped so we have to do it, so run:

```bash
kubectl get ing
```

copy address(eg:-a7e9f462781314585a9b86a2b3d365ee-a7932cde154756d6.elb.us-east-1.amazonaws.com) from here then run:

```bash
nslookup a7e9f462781314585a9b86a2b3d365ee-a7932cde154756d6.elb.us-east-1.amazonaws.com
```

copy any address under Non-authoritative answer eg:-98.91.154.113, 

Now go to:
```bash
sudo vim /etc/hosts
```
Two ways:
1.General way:

Add your address and host in the file and save it:-
98.91.154.113 go-web-app.local

2.For people who uses WSL ubuntu they have to do DNS mapping in their windows system too:

Open Notepad as Administrator>Press Windows Key

Type: Notepad>Right-click → Run as administrator

Open the hosts file>In Notepad, go to:

File → Open → C:\Windows\System32\drivers\etc\hosts

Change file type from “Text Documents” to “All Files (.)” so you can see it.>Add your mapping at the bottom

98.91.154.113 go-web-app.local

Replace 3.226.116.223 with your actual Load Balancer IP or DNS name if it resolves to multiple IPs.>Save and close.

Now visit go-web-app.local/home.

[NOTE:If you wanna check your website which ingress change type in service.yaml to Nodeport from ClusterIP>Run kubectl get svc to check changes and copy the port number eg:- 80:32099/TCP(copy 32099 from here) then run kubectl get nodes -o wide copy any nodes EXTERNALIP ADDRESS eg:- 54.161.25.151 then in your browser type 54.161.25.151:32099/home you should be able to see your website]

# Implement Helm:

Why helm?
Helm is like a package manager for Kubernetes — similar to how apt is for Ubuntu or npm is for Node.js.
It simplifies deploying, upgrading, and managing complex Kubernetes applications.

Without Helm:
You would manually create and manage multiple YAML files:
deployment.yaml
service.yaml
ingress.yaml
configmap.yaml
secret.yaml
and so on...

When you want to:
change an image tag,
add an environment variable,
or update replicas,
you’d have to edit several YAMLs and reapply them one by one — very error-prone.

With Helm:
Helm bundles all those YAMLs into a single “chart” — a reusable, versioned package of Kubernetes resources.
For example, instead of running:
```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f ingress.yaml
```

You can simply do:
```bash
helm install go-web-app ./go-web-app-chart
```

And Helm will:
Create all resources
Inject values dynamically (from values.yaml)
Track versions (so you can roll back)

After Installing Create a Folder helm under that run:

```bash
helm create go-web-app-chart
```

Then go to template folder and delete all files and then copy paste your deployment,service and ingress.yaml here.

Go to values.yaml file under go-web-app-chat and update it for image tag fetching.

Go to deployment.yaml and update image: pankswork/go-web-app:v1 to image: pankswork/go-web-app:{{ .Values.image.tag }} so that we can variablize image tag.

Now delete all deployment,service and ingress:
```bash
kubectl delete deploy go-web-app
kubectl delete svc go-web-appkubectl delete svc go-web-app
cd to helm folder and run:
helm install go-web-app ./go-web-app-chart
```
Now check again for deployement,service and ingress

# Implementing CI using Github Actions:

create a .github folder under that create one more folder named workflows under that create a file ci.yaml.
Make sure to add :
DOCKERHUB_PASSWORD
DOCKERHUB_USERNAME
TOKEN   #github token

onece you add,commit and push it will run the CI pipeline also go and check you values.yaml in the repo it will updated with that docker tag. 

# Implement CD:

Install ArgoCD:

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Access ArgoCD UI:
```bash
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```

Get the Loadbalancer service IP:

```bash
kubectl get svc argocd-server -n argocd
```

Wait for some time and then Copy EXTERNAL IP and paste it on browser to visit ardocd login page:

Username:admin
Password:(For Password run :

```bash
kubectl get secrets -n argocd
kubectl edit secret argocd-initial-admin-secret -n argocd 

#copy password eg:-S3dsOXZGRDBTS0Z2cmRkYg== then run: 
echo S3dsOXZGRDBTS0Z2cmRkYg== | base64 --decode 
```

copy the password to login)

CLick on New App>Update only these values:
Application name:go-web-app,Project name:default,sync policy:auto,Tick Self Heal,Repo :Your Github Repo URL,path with updated there you have to select,cluster url :default,namespace:default,value files:values.yaml then click on create.

Once its synced and healthy you can with the app via argocd and local host.
