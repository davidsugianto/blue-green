# Example Blue Green deployment using Istio

- Download istio: `curl -L https://istio.io/downloadIstio | sh -` o `curl -L https://git.io/getLatestIstio | ISTIO_VERSION=1.5.1 sh -`

**NOTE** Since we are running Istio with Minikube, we need to make one change before going ahead with the next step – changing the Ingress Gateway service from type LoadBalancer to NodePort. Memory required so running istio 8GB and cpu 4 core.

- Start Kubernetes on Local: `minikube start -p cluster_name` or `minikube start -p kube-blue-green --memory='8092mb' --cpus=4 --vm-driver='virtualbox'`
- install Helm package manager: `kubectl create -f helm/helm-rbac.yaml` and `helm init --service-account=tiller`
- Testing Helm: `helm version`
- Add repo istio: `helm repo add istio.io https://storage.googleapis.com/istio-release/releases/1.5.1/charts/`
- Create namespace istio-system: `kubectl create namespace istio-system`
- Istio init: `helm template install/kubernetes/helm/istio-init --name istio-init --namespace istio-system | kubectl apply -f -`
- Jobs check: `kubectl -n istio-system wait --for=condition=complete job --all`
- CRDs check: `kubectl get crds | grep 'istio.io\|certmanager.k8s.io' | wc -l` or  `kubectl get crds | grep 'istio.io' | wc -l`
- Enter to directory istio package: `cd istio-1.5.1/`
- Install istio: `helm template install/kubernetes/helm/istio --name istio --namespace istio-system | kubectl apply -f -`
- After install check pods and service: `kubectl get pods -n istio-system` and `kubectl get svc -n istio-system`
- Install kiali: `kubectl apply -f kiali/kia-secrets.yaml` and `helm template install/kubernetes/helm/istio --name=istio --namespace=istio-system --set kiali.enabled=True | kubectl apply -f -`

### Install App Deployment and Service
- App service: `kubectl apply -f service.yaml`
- Blue deployment: `kubectl apply -f blue-deployment.yaml`
- Green deployment: `kubectl apply -f green-deployment.yaml`

### Configure Blue Green Deployment
- Install app gateway: `kubectl apply -f app-gateway.yaml`
- Create destination rule: `kubectl apply -f des-rule.yaml`
- Install virtual service: `kubectl apply -f virtual-service.yaml`