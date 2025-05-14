# Commands

## Chapter 4 - Pods

### Startup

kubectl apply -f ./pods/initsidecar.yml
kubectl get pods
kubectl get services
kubectl get pod -o "custom-columns=NAME:.metadata.name,INIT:.spec.initContainers[*].name, CONTAINERS:.spec.containers[*].name"
kubectl describe pod git-sync
kubectl get svc svc-sidecar

### Shutdown

kubectl delete pod git-sync
kubectl delete service svc-sidecar
kubectl delete -f ./pods/initsidecar.yml

## Chapter 5 - Namespaces

kubectl api-resources
kubectl get namespaces
kubectl describe ns default
kubectl get svc -n kube-system / kubectl get svc --namespace kube-system

### Managing a namespace

kubectl create ns hydra
kubectl apply -f ./namespaces/shield-ns.yml
kubectl delete ns hydra
kubectl config set-context --current --namespace shield
kubectl apply -f ./namespaces/app.yml

### Clean up

kubectl delete ns shield
kubectl config set-context --current --namespace default

## Chapter 6 - Deployments

### Creating the Deployment

kubectl apply -f ./deployments/deploy.yml
kubectl get pods
kubectl get deploy
kubectl describe deploy hello-deploy
kubectl get rs
kubectl describe rs hello-deploy-{HASH}
kubectl apply -f ./deployments/lb.yml

### Scaling the app

kubectl get deploy hello-deploy
kubectl scale deploy hello-deploy --replicas 5
kubectl apply -f ./deployments/deploy.yml
kubectl rollout status deploy hello-deploy

### Rollout and Rollback

kubectl describe rs hello-deploy-{HASH}
kubectl rollout pause deploy hello-deploy
kubectl rollout resume deploy hello-deploy
kubectl rollout history deployment hello-deploy
kubectl describe deploy hello-deploy
kubectl rollout undo deployment hello-deploy --to-revision=1

### Clean up

kubectl delete -f ./deployments/deploy.yml
kubectl delete -f ./deployments/lb.yml

## Chapter 7 - Services

### Working wih Services imperatively

kubectl apply -f ./services/deploy.yml
kubectl get deploy svc-test
kubectl expose deployment svc-test --type=LoadBalancer
kubectl get svc -o wide
kubectl describe svc svc-test
kubectl delete svc svc-test

### Working wih Services the declarative way

kubectl apply -f ./services/lb.yml
kubectl get svc svc-lb -o wide
kubectl describe svc svc-lb
kubectl get endpointslices
kubectl describe endpointslice svc-lb-24rfg

### Clean up

kubectl delete -f ./services/deploy.yml ./services/lb.yml

## Chapter 8 - Ingress

### Intalling NGINX Ingress Controller

kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.12.0/deploy/static/provider/cloud/deploy.yaml
kubectl get pods -n ingress-nginx -l app.kubernetes.io/name=ingress-nginx

### Configuring Ingress

kubectl get ingressclass
kubectl describe ingressclass nginx
kubectl apply -f ./ingress/app.yml
kubectl apply -f ./ingress/ig-all.yml
kubectl get ing
kubectl describe ing mcu-all

### Clean up

kubectl delete -f ./ingress/ig-all.yml
kubectl delete -f ./ingress/app.yml

## Chapter 9 - Wasm

rustup target add wasm32-wasip1
