# The following install awx on your windows machine
# TESTING 

# Required
* Install docker on windows 
* Install minikube https://minikube.sigs.k8s.io/docs/start/ minikube version: v1.29.0
* install kubectl on windows https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/

# Start minikube

```
minikube start --addons=ingress --cpus=2 --install-addons=true --kubernetes-version=stable --memory=6g
```

```
kubectl apply -f https://raw.githubusercontent.com/ansible/awx-operator/0.13.0/deploy/awx-operator.yaml
```

Check 

```
kubectl get pods -A
```

Create a deployment file call it vi awx-demo.yml

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
NOTE: to get the encoding working i used a bash terminal.

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

Minikube tunnel On a new session, start the minikube tunnel:

minikube tunnel
Enable AWX to be access via the Internet:

kubectl port-forward svc/awx-demo-service --address 0.0.0.0 30886:80
Now visit https://your_ip:high_port