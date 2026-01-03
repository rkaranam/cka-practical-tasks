#### Question

We encountered an issue with our Nginx and PHP-FPM setup on the Kubernetes cluster this morning, which halted its functionality. Investigate and rectify the issue:  
  
The pod name is  `nginx-phpfpm`  and configmap name is  `nginx-config`. Identify and fix the problem.  
 
Once resolved, copy  `/home/thor/index.php`  file from the  `jump host`  to the  `nginx-container`  within the nginx document root. After this, you should be able to access the website using  `Website`  button on the top bar.

#### Solution

Following pod having issue with volumes and volumeMounts. Fix it

```yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"labels":{"app":"php-app"},"name":"nginx-phpfpm","namespace":"default"},"spec":{"containers":[{"image":"php:7.2-fpm-alpine","name":"php-fpm-container","volumeMounts":[{"mountPath":"/usr/share/nginx/html","name":"shared-files"}]},{"image":"nginx:latest","name":"nginx-container","volumeMounts":[{"mountPath":"/var/www/html","name":"shared-files"},{"mountPath":"/etc/nginx/nginx.conf","name":"nginx-config-volume","subPath":"nginx.conf"}]}],"volumes":[{"emptyDir":{},"name":"shared-files"},{"configMap":{"name":"nginx-config"},"name":"nginx-config-volume"}]}}
  creationTimestamp: "2026-01-03T12:58:05Z"
  labels:
    app: php-app
  name: nginx-phpfpm
  namespace: default
  resourceVersion: "1771"
  uid: 5be0fc1b-3d5d-4138-97ed-505b5b5ac84d
spec:
  containers:
  - image: php:7.2-fpm-alpine
    imagePullPolicy: IfNotPresent
    name: php-fpm-container
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /usr/share/nginx/html
      name: shared-files
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-bv2g4
      readOnly: true
  - image: nginx:latest
    imagePullPolicy: Always
    name: nginx-container
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/www/html
      name: shared-files
    - mountPath: /etc/nginx/nginx.conf
      name: nginx-config-volume
      subPath: nginx.conf
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-bv2g4
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: kodekloud-control-plane
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - emptyDir: {}
    name: shared-files
  - configMap:
      defaultMode: 420
      name: nginx-config
    name: nginx-config-volume
  - name: kube-api-access-bv2g4
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
          expirationSeconds: 3607
          path: token
      - configMap:
          items:
          - key: ca.crt
            path: ca.crt
          name: kube-root-ca.crt
      - downwardAPI:
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
            path: namespace
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2026-01-03T12:58:05Z"
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2026-01-03T12:58:16Z"
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2026-01-03T12:58:16Z"
    status: "True"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2026-01-03T12:58:05Z"
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: containerd://52116a60d387e838c83289226ca6367133afbe59b161a732876ef19f2ad41ed1
    image: docker.io/library/nginx:latest
    imageID: docker.io/library/nginx@sha256:ca871a86d45a3ec6864dc45f014b11fe626145569ef0e74deaffc95a3b15b430
    lastState: {}
    name: nginx-container
    ready: true
    restartCount: 0
    started: true
    state:
      running:
        startedAt: "2026-01-03T12:58:15Z"
  - containerID: containerd://85991a48ce76cfacaf9f4361e39f6db34ffb0de45139b06bd2e0d815f91d53a6
    image: docker.io/library/php:7.2-fpm-alpine
    imageID: docker.io/library/php@sha256:2e2d92415f3fc552e9a62548d1235f852c864fcdc94bcf2905805d92baefc87f
    lastState: {}
    name: php-fpm-container
    ready: true
    restartCount: 0
    started: true
    state:
      running:
        startedAt: "2026-01-03T12:58:09Z"
  hostIP: 172.17.0.2
  phase: Running
  podIP: 10.244.0.5
  podIPs:
  - ip: 10.244.0.5
  qosClass: BestEffort
  startTime: "2026-01-03T12:58:05Z"
```

The issue is **misaligned shared volume mount paths between Nginx and PHP-FPM**.  
Both containers must mount the **same directory path** for PHP files, otherwise Nginx will not see the PHP files that PHP-FPM serves.

### What’s wrong

-   **PHP-FPM** mounts the shared volume at:
    
    ```
    /usr/share/nginx/html    
    ```
    
-   **Nginx** mounts the same volume at:
    
    ```
    /var/www/html    
    ```
  
These paths **must be identical**.

#### ✅ Fixed Pod Spec (recommended)

Standard practice:

-   Use `/var/www/html` for **both** Nginx and PHP-FPM
    
-   Keep ConfigMap volume with `subPath` as-is
    

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-phpfpm
  labels:
    app: php-app
spec:
  containers:
  - name: php-fpm-container
    image: php:7.2-fpm-alpine
    volumeMounts:
    - name: shared-files
      mountPath: /var/www/html

  - name: nginx-container
    image: nginx:latest
    volumeMounts:
    - name: shared-files
      mountPath: /var/www/html
    - name: nginx-config-volume
      mountPath: /etc/nginx/nginx.conf
      subPath: nginx.conf

  volumes:
  - name: shared-files
    emptyDir: {}
  - name: nginx-config-volume
    configMap:
      name: nginx-config
```

----------

## ✅ Summary of Fix

-   ✔ Same volume (`shared-files`)    
-   ✔ Same mount path (`/var/www/html`)    
-   ✔ ConfigMap mounted correctly using `subPath`
    

This ensures:

-   Nginx can read PHP files    
-   PHP-FPM processes the same files    
-   No volumeMount mismatch issues
    
### Copy file from local to Kubernetes container    

To copy a file from your local machine to a Kubernetes container, you use the  `kubectl cp`  command. This command works similarly to the standard Linux  `cp`  command but is designed for transferring files to and from remote pods.

#### Prerequisites

-   You must have  `kubectl`  installed and configured to connect to your Kubernetes cluster.
-   You need the name of the destination Pod. Use  `kubectl get pods`  to find it.
-   The  `tar`  utility needs to be available in the container's image for  `kubectl cp`  to work under the hood. Most standard images include it, but minimal/distroless images might not.

#### Basic Command Syntax

The general syntax for copying a file from your local machine to a Pod is:

```bash
kubectl cp <local-file-path> <pod-name>:<destination-path>
```

Examples

-   **Copy a single file:**  
    To copy a local file named  `config.yaml`  to the  `/etc/config`  directory of a pod named  `my-app-pod`:

    ```bash
    kubectl cp config.yaml my-app-pod:/etc/config/config.yaml
    ```
    
    Note that the destination path must include the filename.
-   **Specify a namespace:**  
    If your pod is in a namespace other than the default, use the  `-n`  flag:

    ```bash
    kubectl cp -n my-namespace config.yaml my-app-pod:/etc/config/config.yaml
    ```
    
-   **Specify a container in a multi-container pod:**  
    If a pod runs multiple containers, use the  `-c`  flag to specify the target container:
    
    ```bash
    kubectl cp config.yaml my-app-pod:/etc/app/config.yaml -c app-container-name
    ```
    
-   **Copy an entire directory:**  
    The  `kubectl cp`  command handles directories recursively by default. To copy a local folder named  `app-data`  to a pod:
    
    ```bash
    kubectl cp app-data my-app-pod:/var/www/app-data
    ```

```bash
k cp /home/thor/index.php nginx-phpfpm:/var/www/html -c nginx-container
```