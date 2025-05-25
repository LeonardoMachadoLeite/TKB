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

### Preparing tools - Rust and Spin

rustup target add wasm32-wasip1
spin new tkb-wasm -t http-rust
    Description []: My first Wasm app
    HTTP path [/...]: /tkb
tree .\ /f
spin build

### Build an OCI image and push it to an OCI registry

docker build --platform wasi/wasm --provenance=false -t nigelpoulton/k8sbook:wasm-0.2 .
docker inspect nigelpoulton/k8sbook:wasm-0.2
docker push nigelpoulton/k8sbook:wasm-0.2

### Build and configure a new multi-node Kubernetes cluster for Wasm

k3d cluster create wasm --image ghcr.io/deislabs/containerd-wasm-shims/examples/k3d:v0.11.1 -p "5005:80@loadbalancer" --agents 2
kubectl get nodes
docker exec -i k3d-wasm-agent-1 ash
ps | grep containerd
ls /bin | grep shim
cat /var/lib/rancher/k3s/agent/etc/containerd/config.toml
containerd --config /var/lib/rancher/k3s/agent/etc/containerd/config.toml config dump | grep spin
exit
kubectl label nodes k3d-wasm-agent-1 wasm=yes
kubectl get nodes --show-labeles | grep wasm=yes
kubectl apply -f rc-spin.yml
kubectl get runtimeclass

### Deploy and test the app

kubectl apply -f app.yml
kubectl get pods -o wide
curl http://localhost:5005/tkb

### Clean up

k3d cluster delete wasm
kubectl delete -f app.yml
kubectl delete runtimeclass rc-spin
docker rmi nigelpouton/k8sbook:wasm-0.1

## Chapter 10 - Service discovery

kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl get deploy -n kube-system -l k8s-app=kube-dns
kubectl get svc -n kube-system -l k8s-app=kube-dns
cat /etc/resolv.conf
kubectl apply -f ./service-discovery/sd-example.yml
kubectl get all --namespace dev
kubectl get all --namespace prod
kubectl exec -it jump --namespace dev -- bash
cat /etc/resolv.conf
apt-get update && apt-get install curl -y
curl ent:8080
curl ent.prod.svc.cluster.local:8080
kubectl logs -n kube-system coredns-668d6bf9bc-7bsrh
kubectl get svc kube-dns -n kube-system
kubectl get endpointslice -n kube-system -l k8s-app=kube-dns
kubectl run -it dnsutils --image registry.k8s.io/e2e-test-images/jessie-dnsutils:1.7
nslookup kubernetes
kubectl attach dnsutils -c dnsutils -i -t
kubectl get svc kubernetes
kubectl delete pod -n kube-system -l k8s-app=kube-dns

### Clean up

kubectl delete pod dnsutils
kubectl delete -f ./service-discovery/sd-example.yml

## Chapter 11 - Storage

kubectl get sc
kubectl describe sc standard
kubectl get pv
kubectl get pvc
kubectl apply -f ./storage/lke-pvc-test.yml
kubectl delete pvc pvc-test
kubectl apply -f ./storage/lke-sc-wait-keep.yml
kubectl apply -f ./storage/lke-pvc-wait-keep.yml
kubectl apply -f ./storage/lke-app.yml
kubectl describe pod volpod

### Clean up

kubectl delete pod volpod
kubectl delete pvc pvc-wait-keep
kubectl delete pv pvc-279f09e083254fa9
kubectl delete sc block-wait-keep

## Chapter 12 - ConfigMaps and Secrets

### Configmaps

kubectl create configmap testmap1 --from-literal shortname=SAFC --from-literal longname="Sunderland Association Football Club"
kubectl describe cm testmap1
kubectl create cm testmap2 --from-file ./configmaps/cmfile.txt
kubectl get cm
kubectl get cm testmap2 -o yaml
kubectl apply -f ./configmaps/fullname.yml
kubectl apply -f ./configmaps/singlemap.yml
kubectl apply -f ./configmaps/podenv.yml
kubectl exec envpod --env | grep NAME
kubectl apply -f ./configmaps/podstartup.yml
kubectl logs startup-pod -c args1
kubectl describe pod startup-pod
kubectl delete pod startup-pod
kubectl apply -f ./configmaps/podvol.yml
kubectl edit cm multimap
kubectl exec cmvol -- ls /etc/name
kubectl exec cmvol -- cat /etc/name/Country

### Secrets

kubectl create secret generic creds --from-literal user=nigelpouton --from-literal pwd=Password123
kubectl get secret creds -o yaml
kubectl apply -f ./configmaps/tkb-secret.yml

### Clean up

kubectl get pods
kubectl get cm
kubectl get secrets
kubectl delete pods cmvol envpod secret-pod
kubectl delete cm multimap test-config testmap1 testmap2
kubectl delete secrets creds tkb-secret

## Chapter 13 - StatefulSets

kubectl apply -f ./statefulsets/dd-kind-sc.yml
kubectl get sc
kubectl apply -f ./statefulsets/headless-svc.yml
kubectl get svc
kubectl apply -f ./statefulsets/sts.yml
kubectl get sts --watch
kubectl get pvc
kubectl get pods
kubectl apply -f ./statefulsets/jump-pod.yml
kubectl exec -it jump-pod -- bash
dig SRV dullahan.default.svc.cluster.local
kubectl apply -f ./statefulsets/sts.yml
kubectl get pods
kubectl delete pod tkb-sts-0
kubectl get pods --watch
kubectl describe pod tkb-sts-0

kubectl delete pvc webroot-tkb-sts-0
kubectl delete -f ./statefulsets/sts.yml

## Extra

kubectl apply -f ./statefulsets/postgres.yml
nvim %WINDIR%\System32\Drivers\Etc\Hosts