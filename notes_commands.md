# Commands

## Sidecar example

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
