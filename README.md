# Kubernetes Observability using Grafana and Prometheus with Istio

![Kubernetes Observability using Grafana and Prometheus with Istio](/architecture-diagram/KubernetesObservabilityGrafanaPrometheus.png)

---

## Grafana and Prometheus Architecture

![Grafana and Prometheus Architecture](/architecture-diagram/Kubernetes%20Observability%20using%20Grafana%20and%20Prometheus%20with%20Istio.png)

---

## Istio Architecture

[Istio Service Mesh Architecture](https://istio.io/latest/docs/ops/deployment/architecture/arch.svg)

---

### Setup Local Kubernetes Environment (KinD) with LoadBalancer (Metallb)

Please refer to the following github repo for setting up a local kubernetes environment using KinD and LoadBalancer using Metallb.

[Create Multi-Node Local Kubernetes Cluster (KinD) with LoadBalancer (Metallb)](https://github.com/NaumanMunir9/Create-Multi-Node-Local-Kubernetes-Cluster--KinD--with-LoadBalancer--Metallb-)

---

## Project WorkFlow

### Intall Istioctl

```shell
brew install istioctl
```

---

### Getting Started

#### "Demo" configuration profile for Istio

For this application, we use the demo [configuration profile](https://istio.io/latest/docs/setup/additional-setup/config-profiles/). It’s selected to have a good set of defaults for testing, but there are other profiles for production or performance testing.

```shell
istioctl install --set profile=demo -y
```

---

### Deploying the application

```shell
k label namespace default istio-injection=enabled
```

---

### Deploy the sample application

```shell
k apply -f https://raw.githubusercontent.com/istio/istio/release-1.18/samples/bookinfo/platform/kube/bookinfo.yaml
```

---

```shell
k get all
```

To confirm that the Bookinfo application is running, send a request to it by a curl command from some pod, for example from ratings:

```shell
k exec "$(k get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}')" -c ratings -- curl -sS productpage:9080/productpage | grep -o "<title>.*</title>"
```

---

### Open the application to outside traffic

The Bookinfo application is deployed but not accessible from the outside. To make it accessible, you need to create an Istio Ingress Gateway, which maps a path to a route at the edge of your mesh.

#### Associate this application with the Istio gateway:

```shell
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.18/samples/bookinfo/networking/bookinfo-gateway.yaml
```

#### Ensure that there are no issues with the configuration

```shell
istioctl analyze
```

---

### Determining the ingress IP and ports

#### Set the ingress host and ports

```shell
export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].port}')
export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].port}')
```

#### Set GATEWAY_URL

```shell
export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT
```

#### Ensure an IP address and port were successfully assigned to the environment variable

```shell
echo "$GATEWAY_URL"
```

---

### Verify external access

Confirm that the Bookinfo application is accessible from outside by viewing the Bookinfo product page using a browser.

Run the following command to retrieve the external address of the Bookinfo application

```shell
echo "http://$GATEWAY_URL/productpage"
```

Paste the output from the previous command into your web browser and confirm that the Bookinfo product page is displayed.

---

### View the dashboard

Istio integrates with several different telemetry applications. These can help you gain an understanding of the structure of your service mesh, display the topology of the mesh, and analyze the health of your mesh.

Use the following instructions to deploy the Kiali dashboard, along with Prometheus, Grafana, and Jaeger.

#### Install Kiali, Grafana and Prometheus along with other addons

```she
kubectl apply -f istio-samples-addons -R
kubectl rollout status deployment/kiali -n istio-system
```

---

### k6 - open-source load testing tool

[Grafana k6](https://k6.io/docs/) is an open-source load testing tool that makes performance testing easy and productive for engineering teams. k6 is free, developer-centric, and extensible.

#### Paste the following code in a file named "average-load.js"

```js
import http from 'k6/http';

export const options = {
  stages: [
    { duration: '2m', target: 100 }, // traffic ramp-up from 1 to 100 users over 5 minutes.
    { duration: '5m', target: 200 }, // stay at 200 users for 5 minutes
    { duration: '2m', target: 100 }, // ramp-down to 100 users
  ],
};

export default () => {
  const urlRes = http.get('http://172.19.255.201/productpage');
};

```

```shell
k6 run average-load.js
```

---

### Access Prometheus and Grafana Dashboards

```shell
k get all
```
