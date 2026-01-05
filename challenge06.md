#### Question
We are working on an application that will be deployed on multiple containers within a pod on Kubernetes cluster. There is a requirement to share a volume among the containers to save some temporary data. The Nautilus DevOps team is developing a similar template to replicate the scenario. Below you can find more details about it.  

1.  Create a pod named  `volume-share-nautilus`.  
  
2.  For the first container, use image  `fedora`  with  `latest`  tag only and remember to mention the tag i.e  `fedora:latest`, container should be named as  `volume-container-nautilus-1`, and run a  `sleep`  command for it so that it remains in running state. Volume  `volume-share`  should be mounted at path  `/tmp/blog`.  
  
3.  For the second container, use image  `fedora`  with the  `latest`  tag only and remember to mention the tag i.e  `fedora:latest`, container should be named as  `volume-container-nautilus-2`, and again run a  `sleep`  command for it so that it remains in running state. Volume  `volume-share`  should be mounted at path  `/tmp/apps`.  
      
4.  Volume name should be  `volume-share`  of type  `emptyDir`.  
    
5.  After creating the pod, exec into the first container i.e  `volume-container-nautilus-1`, and just for testing create a file  `blog.txt`  with any content under the mounted path of first container i.e  `/tmp/blog`.  
   
6.  The file  `blog.txt`  should be present under the mounted path  `/tmp/apps`  on the second container  `volume-container-nautilus-2`  as well, since they are using a shared volume.

#### Solution

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: volume-share-nautilus
  name: volume-share-nautilus
spec:
  volumes:
  - name: volume-share
    emptyDir: {}
  containers:
  - image: fedora:latest
    name: volume-container-nautilus-1
    command: ["/bin/sh"]
    args: ["-c", "sleep infinity"]
    volumeMounts:
    - mountPath: /tmp/blog
      name: volume-share
    resources: {}
  - image: fedora:latest
    name: volume-container-nautilus-2
    command: ["/bin/sh"]
    args: ["-c", "sleep infinity"]
    volumeMounts:
    - mountPath: /tmp/apps
      name: volume-share
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```bash
k exec -it po/volume-share-nautilus -c volume-container-nautilus-1 -- /bin/sh
k exec -it po/volume-share-nautilus -c volume-container-nautilus-2 -- /bin/sh
```