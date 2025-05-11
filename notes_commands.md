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
kubectl describe rs hello-deploy-7fd99bf846
kubectl apply -f ./deployments/lb.yml

### Scaling the app

kubectl get deploy hello-deploy
kubectl scale deploy hello-deploy --replicas 5
kubectl apply -f ./deployments/deploy.yml
kubectl rollout status deploy hello-deploy

### Rollout and Rollback

kubectl describe rs hello-deploy-7fbfb4b557
kubectl rollout pause deploy hello-deploy
kubectl rollout resume deploy hello-deploy
kubectl rollout history deployment hello-deploy
kubectl describe deploy hello-deploy
kubectl rollout undo deployment hello-deploy --to-revision=1

### Clean up

kubectl delete -f ./deployments/deploy.yml
kubectl delete -f ./deployments/lb.yml
