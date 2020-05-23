
# OpenTracing Support

In this example we deploy the NGINX or NGINX Plus Ingress Controller and a simple web application. Then we enable OpenTracing and use a tracer (Jaeger) for tracing the requests that go through NGINX or NGINX Plus to the web application.

## Prerequisites

The default Ingress Controller images don’t include the OpenTracing module required for this example. 
Step1: git clone https://github.com/nginxinc/kubernetes-ingress/tree/master/build 
Step2: create docker image in by using this repo
       ```
       $ make clean
       $ make DOCKERFILE=DockerfileWithOpentracing PREFIX=YOUR-PRIVATE-REGISTRY/nginx-ingress
       ```
       PS: I already created an one, you can get it by using this dockerhub repo: wangsiming519/nginx-ingress:opentracing_1.7.0


## Step 1 - Deploy Ingress Controller and testing App

There is a quick demo app which use nginx sample

```
kubectl apply -f app-nginx-deployment.yaml
kubectl apply -f demo-ingress.yaml
```

And you can use below command to check
```
IC_IP=<your ingress svc IP>
IC_HTTPS_PORT=80
curl --resolve nginx.kube.com:$IC_HTTPS_PORT:$IC_IP http://nginx.kube.com
```



## Step 2 - Deploy a Tracer

1. Use the [all-in-one dev template](https://github.com/jaegertracing/jaeger-kubernetes#development-setup) to deploy Jaeger in the default namespace. **Note:** This template should be only used for development or testing.
   ```
   kubectl create -f https://raw.githubusercontent.com/jaegertracing/jaeger-kubernetes/master/all-in-one/jaeger-all-in-one-template.yml
   ```

2. Wait for the jaeger pod to be ready:
   ```
   $ kubectl get pod

   NAME                      READY     STATUS   
   jaeger-6c996dbcd9-j5jzf   1/1       Running
   ```

## Step 3 - Enable OpenTracing
1. Update the ConfigMap with the keys required to load OpenTracing module with Jaeger and enable  OpenTracing for all Ingress resources.
   ```
   kubectl apply -f nginx-ingress-controller-config.yaml
   ```

## Step 4 - Test Tracing
1. Make a request to the app. 
   
   **Note:** $IC_HTTPS_PORT and $IC_IP env variables should have been set from the Prerequisites step in the complete-example installation instructions.
   ```
   curl --resolve cafe.example.com:$IC_HTTPS_PORT:$IC_IP https://cafe.example.com:$IC_HTTPS_PORT/coffee --insecure
   ```
1. Forward a local port to the Jaeger UI port on the Jaeger pod:
   ```
   kubectl port-forward <YOUR_JAEGER_POD> 16686:16686
   ``` 
1. Open Jaeger dashboard in your browser available via http://localhost:16686. Search for the traces by specifying the name of the service to `nginx-ingress` and clicking `Find Traces`. You will see:
