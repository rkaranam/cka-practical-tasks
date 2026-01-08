### Question
A Deployment named  `webapp-deploy`  is running in the  `ingress-ns`  namespace and is exposed via a Service named  `webapp-svc`.

Create an Ingress resource called  `webapp-ingress`  in the same namespace that will route traffic to the service. The Ingress must:

-   Use  `pathType: Prefix`
-   Route requests sent to path  `/`  to the backend service
-   Forward traffic to port  `80`  of the service
-   Be configured for the host  `kodekloud-ingress.app`

Test app availablility using the following command:

```
curl -s http://kodekloud-ingress.app/
```

### Solution

Make sure the path is set correctly and the pathType is Prefix. Use host-based routing

1.  Checkout the resources in the  `ingress-ns`  namespace:

```bash
ssh cluster1-controlplane
kubectl get deployment webapp-deploy -n ingress-ns
kubectl get svc webapp-svc -n ingress-ns
```

2.  Create the Ingress YAML file:

```yaml
# webapp-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: webapp-ingress
  namespace: ingress-ns
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: kodekloud-ingress.app
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: webapp-svc
            port:
              number: 80
```

3.  Apply the Ingress resource:

```bash
kubectl apply -f webapp-ingress.yaml
```

4.  Test access to the app:

```bash
curl http://kodekloud-ingress.app/
```



### Question

From student-node  `ssh cluster1-controlplane`  to solve this question.  
  
  
Create a deployment named  `logging-deployment`  in the namespace  `logging-ns`  with 1 replica, with the following specifications:

1) The main container must be named  `app-container`, use the image  `busybox`, and start by creating a log directory  `/var/log/app`  and run the below command to simulate generating logs:

```
while true; do 
  echo "Log entry" >> /var/log/app/app.log
  sleep 5
done
```

2) Add a co-located container named  `log-agent`  that also uses the  `busybox`  image and runs these commands:

```
touch /var/log/app/app.log
tail -f /var/log/app/app.log
```

3) Both containers must share the same  `emptyDir`  volume mounted at  `/var/log/app`.

4)  **Use any consistent label**  for the Deployment selector and the Pod template labels (the exact key/value is up to you, as long as they match).

`log-agent`  logs should display the entries logged by the main  `app-container`.

### Solution

1.  Create the Deployment YAML file, for example  `logger-deployment.yaml`  :

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: logging-deployment
  namespace: logging-ns
spec:
  replicas: 1
  selector:
    matchLabels:
      app: logger
  template:
    metadata:
      labels:
        app: logger
    spec:
      volumes:
        - name: log-volume
          emptyDir: {}
      containers:
        - name: app-container
          image: busybox
          command: ["sh", "-c", "mkdir -p /var/log/app; while true; do echo 'Log entry' >> /var/log/app/app.log; sleep 5; done"]
          volumeMounts:
            - name: log-volume
              mountPath: /var/log/app
        - name: log-agent
          image: busybox
          command: ["sh", "-c", "touch /var/log/app/app.log; tail -f /var/log/app/app.log"]
          volumeMounts:
            - name: log-volume
              mountPath: /var/log/app
```

2.  Apply the deployment:

```bash
kubectl apply -f logger-deployment.yaml
```

3.  Verify that the  `log-agent`  container displays log entries:

```bash
kubectl logs -n logging-ns deployment/logging-deployment -c log-agent --tail=50
```

You should see repeated  `Log entry`  lines in the output.

### Question

Create a new deployment called nginx-deploy, with image nginx:1.16 and 1 replica. Next, upgrade the deployment to version 1.17 using rolling update.

### Solution

Explore the  `--record`  option while creating the deployment while working with the deployment definition file. Then make use of the  `kubectl apply`  command to create or update the deployment.

To create a deployment definition file  `nginx-deploy`:

```sh
kubectl create deployment nginx-deploy --image=nginx:1.16 --dry-run=client -o yaml > deploy.yaml
```

To create a resource from definition file and to record:

```sh
kubectl apply -f deploy.yaml --record
```

To view the history of deployment  `nginx-deploy`:

```sh
kubectl rollout history deployment nginx-deploy
```

To upgrade the image to next given version:

```sh
kubectl set image deployment/nginx-deploy nginx=nginx:1.17 --record
```

To view the history of deployment  `nginx-deploy`:

```sh
kubectl rollout history deployment nginx-deploy
```

### Question

Create a new user called  `john`. Grant him access to the cluster using a csr named  `john-developer`. Create a role  `developer`  which should grant John the permission to  `create, list, get, update and delete pods`  in the  `development`  namespace . The private key exists in the location:  `/root/CKA/john.key`  and csr at  `/root/CKA/john.csr`.  

`Important Note`: As of kubernetes 1.19, the CertificateSigningRequest object expects a  `signerName`.  
  
Please refer to the documentation to see an example. The documentation tab is available at the top right of the terminal.

### Solution

Solution manifest file to create a CSR as follows:

```yaml
---
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: john-developer
spec:
  signerName: kubernetes.io/kube-apiserver-client
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1ZEQ0NBVHdDQVFBd0R6RU5NQXNHQTFVRUF3d0VhbTlvYmpDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRApnZ0VQQURDQ0FRb0NnZ0VCQUt2Um1tQ0h2ZjBrTHNldlF3aWVKSzcrVVdRck04ZGtkdzkyYUJTdG1uUVNhMGFPCjV3c3cwbVZyNkNjcEJFRmVreHk5NUVydkgyTHhqQTNiSHVsTVVub2ZkUU9rbjYra1NNY2o3TzdWYlBld2k2OEIKa3JoM2prRFNuZGFvV1NPWXBKOFg1WUZ5c2ZvNUpxby82YU92czFGcEc3bm5SMG1JYWpySTlNVVFEdTVncGw4bgpjakY0TG4vQ3NEb3o3QXNadEgwcVpwc0dXYVpURTBKOWNrQmswZWhiV2tMeDJUK3pEYzlmaDVIMjZsSE4zbHM4CktiSlRuSnY3WDFsNndCeTN5WUFUSXRNclpUR28wZ2c1QS9uREZ4SXdHcXNlMTdLZDRaa1k3RDJIZ3R4UytkMEMKMTNBeHNVdzQyWVZ6ZzhkYXJzVGRMZzcxQ2NaanRxdS9YSmlyQmxVQ0F3RUFBYUFBTUEwR0NTcUdTSWIzRFFFQgpDd1VBQTRJQkFRQ1VKTnNMelBKczB2czlGTTVpUzJ0akMyaVYvdXptcmwxTGNUTStsbXpSODNsS09uL0NoMTZlClNLNHplRlFtbGF0c0hCOGZBU2ZhQnRaOUJ2UnVlMUZnbHk1b2VuTk5LaW9FMnc3TUx1a0oyODBWRWFxUjN2SSsKNzRiNnduNkhYclJsYVhaM25VMTFQVTlsT3RBSGxQeDNYVWpCVk5QaGhlUlBmR3p3TTRselZuQW5mNm96bEtxSgpvT3RORStlZ2FYWDdvc3BvZmdWZWVqc25Yd0RjZ05pSFFTbDgzSkljUCtjOVBHMDJtNyt0NmpJU3VoRllTVjZtCmlqblNucHBKZWhFUGxPMkFNcmJzU0VpaFB1N294Wm9iZDFtdWF4bWtVa0NoSzZLeGV0RjVEdWhRMi80NEMvSDIKOWk1bnpMMlRST3RndGRJZjAveUF5N05COHlOY3FPR0QKLS0tLS1FTkQgQ0VSVElGSUNBVEUgUkVRVUVTVC0tLS0tCg==
  usages:
  - digital signature
  - key encipherment
  - client auth
```

To approve this certificate, run:  `kubectl certificate approve john-developer`

Next, create a role  `developer`  and rolebinding  `developer-role-binding`, run the command:

```sh
kubectl create role developer --resource=pods --verb=create,list,get,update,delete --namespace=development

kubectl create rolebinding developer-role-binding --role=developer --user=john --namespace=development
```

To verify the permission from kubectl utility tool:

```sh
kubectl auth can-i update pods --as=john --namespace=development
```

### Question

Create an nginx pod called `nginx-resolver` using the image `nginx` and expose it internally with a service called `nginx-resolver-service`. Test that you are able to look up the service and pod names from within the cluster. Use the image: `busybox:1.28` for dns lookup. Record results in `/root/CKA/nginx.svc` and `/root/CKA/nginx.pod`

### Solution

Use the command  `kubectl run`  and create a nginx pod and busybox pod. Resolve it, nginx service and its pod name from  `busybox`  pod.

To create a pod  `nginx-resolver`  and expose it internally:

```sh
kubectl run nginx-resolver --image=nginx
kubectl expose pod nginx-resolver --name=nginx-resolver-service --port=80 --target-port=80 --type=ClusterIP
```

To create a pod  `test-nslookup`. Test that you are able to look up the service and pod names from within the cluster:

```sh
kubectl run test-nslookup --image=busybox:1.28 --rm -it --restart=Never -- nslookup nginx-resolver-service
kubectl run test-nslookup --image=busybox:1.28 --rm -it --restart=Never -- nslookup nginx-resolver-service > /root/CKA/nginx.svc
```

Get the IP of the  `nginx-resolver`  pod and replace the dots(.) with hyphon(-) which will be used below.

```sh
kubectl get pod nginx-resolver -o wide
kubectl run test-nslookup --image=busybox:1.28 --rm -it --restart=Never -- nslookup <P-O-D-I-P.default.pod> > /root/CKA/nginx.pod
```

### Question

**Steps to install Calico CNI/Cillium/Flannel CNI Plugin and verify in CKA**
