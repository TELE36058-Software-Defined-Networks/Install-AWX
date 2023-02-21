# Install AWX-operator on Ubuntu 20 using Minikube

VM - 4 CPUs and 8GIGS of RAM and 40Gig Harddrive

## Install Kubectl 

```
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl

chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
kubectl version --client
```

## Install Docker 

Update 

```
sudo apt-get update
sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

Add Ring

```
sudo mkdir -m 0755 -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

Add Repo

```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Perform update and install docker

```
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```


## Setup minikub
Install minikub

```
curl -Lo minikube https://github.com/kubernetes/minikube/releases/download/v1.21.0/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
```

Add docker users

```
sudo usermod -aG docker $USER
```

You need to log out of the terminal and log back in to add user to groups

```
groups $USER
```

## Start minikube 

Start minikube, this takes a awhile.

```
minikube start --addons=ingress --cpus=2 --install-addons=true --kubernetes-version=stable --memory=6g
```

Validate it's working:

```
kubectl get nodes
kubectl get pods
kubectl get pods -A
```

Install awx operator, this takes about 2-3 min to spin up

```
kubectl apply -f https://raw.githubusercontent.com/ansible/awx-operator/0.10.0/deploy/awx-operator.yaml
```

Create a deployment file call it vi awx-demo.yml

```
---
apiVersion: awx.ansible.com/v1beta1
kind: AWX
metadata:
  name: awx-demo
spec:
  service_type: nodeport
  ingress_type: none
  hostname: awx-demo.example.com
```

Deploy the awx-demo.yml 

```
kubectl apply -f awx-demo.yml
```

Wait 15 seconds for it to deploy.

```
kubectl get pods -l "app.kubernetes.io/managed-by=awx-operator"
kubectl get svc -l "app.kubernetes.io/managed-by=awx-operator"
```

WAIT A FEW MINS... 8-10 Minutes

Get the Admin user password:
```
kubectl get secrets
kubectl get secret awx-demo-admin-password -o jsonpath="{.data.password}" | base64 --decode
```

Output will look like this.
```
5opmmSFkMiux9srVeYBLUoCJudeq46LL

```

Expose the deployment:
```
kubectl expose deployment awx-demo --type=LoadBalancer --port=8080
```

Minikube tunnel
On a new session, start the minikube tunnel:

```
minikube tunnel
```

Enable AWX to be access via the Internet:

```
kubectl port-forward svc/awx-demo-service --address 0.0.0.0 30886:80
```
Now visit https://your_ip:high_port


