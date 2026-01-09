#### Question

Create a new service account with the name `pvviewer`. Grant this Service account access to `list` all PersistentVolumes in the cluster by creating an appropriate cluster role called `pvviewer-role` and ClusterRoleBinding called `pvviewer-role-binding`.  
Next, create a pod called `pvviewer` with the image: `redis` and serviceAccount: `pvviewer` in the default namespace.

#### Solution

Pods authenticate to the API Server using ServiceAccounts. If the serviceAccount name is not specified, the default service account for the namespace is used during a pod creation.  
  
Reference:  `https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/`

Now, create a service account  `pvviewer`:

```sh
kubectl create serviceaccount pvviewer
```

To create a clusterrole:

```sh
kubectl create clusterrole pvviewer-role --resource=persistentvolumes --verb=list
```

To create a clusterrolebinding:

```sh
kubectl create clusterrolebinding pvviewer-role-binding --clusterrole=pvviewer-role --serviceaccount=default:pvviewer
```

Solution manifest file to create a new pod called  `pvviewer`  as follows:

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: pvviewer
  name: pvviewer
spec:
  containers:
  - image: redis
    name: pvviewer
  # Add service account name
  serviceAccountName: pvviewer
```

#### Question

Create a PriorityClass named `low-priority` with a value of 50000. A pod named `lp-pod` exists in the namespace `low-priority`. Modify the pod to use the priority class you created. Recreate the pod if necessary.

#### Solution

Use  `kubectl apply`  to create the PriorityClass, then edit the existing pod to add the priority class.

1.  Create the priority class manifest file  `pc.yaml`

```bash
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: low-priority
value: 50000
globalDefault: false
description: "Low priority class"
```

2.  Apply the manifest file

```bash
kubectl apply -f pc.yaml
```

3.  Inspect the pod definition to be able to recreate it:

```bash
kubectl get pod lp-pod -n low-priority -o yaml
```

4.  Create and update the new yaml  `lp-pod.yaml`  for the pod after adding the priority class:

```yaml
# vi lp-pod.yaml

apiVersion: v1
kind: Pod
metadata:
  name: lp-pod
  namespace: low-priority
spec:
  priorityClassName: low-priority
  containers:
  - name: nginx
    image: nginx
```

4.  Delete and recreate the pod using the configured priority class

```bash
kubectl delete pod lp-pod -n low-priority
kubectl apply -f lp-pod.yaml
```

#### Question

We have deployed a new pod called  `np-test-1`  and a service called  `np-test-service`. Incoming connections to this service are not working. Troubleshoot and fix it.  
Create NetworkPolicy, by the name  `ingress-to-nptest`  that allows incoming connections to the service over port  `80`.

Important: Don't delete any current objects deployed.

#### Solution

Solution manifest file to create a network policy  `ingress-to-nptest`  as follows:

```yaml
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: ingress-to-nptest
  namespace: default
spec:
  podSelector:
    matchLabels:
      run: np-test-1
  policyTypes:
  - Ingress
  ingress:
  - ports:
    - protocol: TCP
      port: 80
```

#### Question

A PersistentVolumeClaim named  `app-pvc`  exists in the namespace  `storage-ns`, but it is not getting bound to the available PersistentVolume named  `app-pv`.

Inspect both the PVC and PV and identify why the PVC is not being bound and fix the issue so that the PVC successfully binds to the PV. Do not modify the PV resource.

#### Solution

Use  `kubectl describe`  and check the PVC and PV details to identify the issue.

1.  Inspect both resources:

```bash
kubectl describe pv app-pv
kubectl describe pvc app-pvc -n storage-ns
```

2.  Note that there is accessModes mismatch between PV and PVC objects. Flush the PVC in a yaml file to be able to update it:

```bash
kubectl get pvc app-pvc -n storage-ns -o yaml > pvc.yaml
```

3.  Update the access mode on  `pvc.yaml`:

```bash
# vi pvc.yaml

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-pvc
  namespace: storage-ns
spec:
  accessModes:
    - ReadWriteOnce        # fix access mode to match PV object
  resources:
    ...
```

4.  Delete and re-create the pvc object

```bash
kubectl delete pvc -n storage-ns app-pvc
kubectl apply -f pvc.yaml
```

4.  Verify the status of the PVC is showing as  `Bound`:

```bash
kubectl get pvc app-pvc -n storage-ns
```

#### Question

We have created a new deployment called `nginx-deploy`. Scale the deployment to 3 replicas. Has the number of replicas increased? Troubleshoot and fix the issue.

#### Solution

Use the command  `kubectl scale`  to increase the replica count to 3.  

```sh
kubectl scale deploy nginx-deploy --replicas=3
```

The  `controller-manager`  is responsible for scaling up pods of a replicaset. If you inspect the control plane components in the  `kube-system`  namespace, you will see that the  `controller-manager`  is not running.  

```sh
kubectl get pods -n kube-system
```

The command running inside the  `controller-manager`  pod is incorrect.  
After fix all the values in the file and wait for  `controller-manager`  pod to restart.  
  
Alternatively, you can run  `sed`  command to change all values at once:

```
sed -i 's/kube-contro1ler-manager/kube-controller-manager/g' /etc/kubernetes/manifests/kube-controller-manager.yaml
```

This will fix the issues in  `controller-manager`  yaml file.  
  
At last, inspect the deployment by using below command, you should see 3/3 under  `READY`  if the fix above was properly performed. Example:

```sh
controlplane ~ ➜  kubectl get deploy
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deploy   3/3     3            3           6m2s
```

#### Question

Create a Horizontal Pod Autoscaler (HPA)  `api-hpa`  for the deployment named  `api-deployment`  located in the  `api`  namespace.  
The HPA should scale the deployment based on a custom metric named  `requests_per_second`, targeting an average value of 1000 requests per second across all pods.  
Set the minimum number of replicas to 1 and the maximum to 20.

Note: Deployment named  `api-deployment`  is available in api namespace. Ignore errors due to the metric  `requests_per_second`  not being tracked in  `metrics-server`


#### Solution

Under /root/ folder you will find a yaml file  `api-hpa.yaml`. Update the yaml file as per task given.

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-hpa
  namespace: api
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api-deployment
  minReplicas: 1
  maxReplicas: 20
  metrics:
  - type: Pods
    pods:
      metric:
        name: requests_per_second
      target:
        type: AverageValue
        averageValue: "1000"
```

Use below command

```bash
kubectl create -f api-hpa.yaml
```

#### Question

Configure the  `web-route`  to split traffic between  `web-service`  and  `web-service-v2`.The configuration should ensure that 80% of the traffic is routed to  `web-service`  and 20% is routed to  `web-service-v2`.

**Note:**  `web-gateway`,  `web-service`, and  `web-service-v2`  have already been created and are available on the cluster.


#### Solution

Copy the below YAML file to the terminal and create a HTTP Route.

```yaml
kubectl create -n default -f - <<EOF
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: web-route
  namespace: default
spec:
  parentRefs:
    - name: web-gateway
      namespace: default
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /
      backendRefs:
        - name: web-service
          port: 80
          weight: 80
        - name: web-service-v2
          port: 80
          weight: 20
EOF
```

#### Question

While preparing to install a CNI plugin on your Kubernetes cluster, you typically need to confirm the cluster-wide Pod network CIDR. Identify the Pod subnet configured for the cluster (the value specified under `podSubnet` in the kubeadm configuration). Output this CIDR in the format `x.x.x.x/x` to a file located at `/root/pod-cidr.txt`.


#### Solution

To identify the cluster-wide Pod subnet, inspect the  `kubeadm-config`  ConfigMap, which contains the  `ClusterConfiguration`  used during  `kubeadm init`:

```bash
kubectl -n kube-system get configmap kubeadm-config -o yaml | grep podSubnet
# podSubnet: 172.17.0.0/16
```

Save just the CIDR value to  `/root/pod-cidr.txt`:

```bash
kubectl -n kube-system get configmap kubeadm-config -o yaml \
  | awk '/podSubnet:/{print $2}' > /root/pod-cidr.txt
```

Verify:

```bash
cat /root/pod-cidr.txt
# 172.17.0.0/16
```

Note:

Be careful not to confuse  **Cluster PodCIDR**  vs  **Node PodCIDR**:

-   Cluster = Entire pool  
    Cluster PodCIDR (big range: 172.17.0.0/16)  
    Find it with :  `kubectl get cm kubeadm-config -n kube-system -o yaml`
    
-   Node = Single slice  
    Node PodCIDR (small slice: 172.17.0.0/24)  
    Find it with :  `kubectl get node <name> -o jsonpath='{.spec.podCIDR}'`