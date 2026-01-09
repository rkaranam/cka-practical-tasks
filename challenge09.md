#### Question
They want to set up the Jenkins server on Kubernetes cluster. Below you can find more details about the task:

1) Create a namespace  `jenkins`

2) Create a Service for jenkins deployment. Service name should be  `jenkins-service`  under  `jenkins`  namespace, type should be  `NodePort`, nodePort should be  `30008`

3) Create a Jenkins Deployment under  `jenkins`  namespace, It should be name as  `jenkins-deployment`  , labels  `app`  should be  `jenkins`  , container name should be  `jenkins-container`  , use  `jenkins/jenkins`  image , containerPort should be  `8080`  and replicas count should be  `1`.  

Make sure to wait for the pods to be in running state and make sure you are able to access the Jenkins login screen in the browser before hitting the  `Check`  button.

#### Solution

**jenkins-deployment**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"    
  creationTimestamp: "2026-01-06T18:32:00Z"
  generation: 1
  labels:
    app: jenkins
  name: jenkins-deployment
  namespace: jenkins
  resourceVersion: "1794"
  uid: ca2e950c-ef2f-4633-b3c3-5eb63df3bfc5
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: jenkins
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: jenkins
    spec:
      containers:
      - image: jenkins/jenkins
        imagePullPolicy: Always
        name: jenkins-container
        ports:
        - containerPort: 8080
          protocol: TCP
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
 ```

**jenkins-service**

```yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: "2026-01-06T18:25:01Z"
  labels:
    app: jenkins-service
  name: jenkins-service
  namespace: jenkins
  resourceVersion: "2354"
  uid: e455b30b-2d18-4c9e-b60f-8caf57c27c95
spec:
  clusterIP: 10.96.2.72
  clusterIPs:
  - 10.96.2.72
  externalTrafficPolicy: Cluster
  internalTrafficPolicy: Cluster
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - nodePort: 30008
    port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: jenkins
  sessionAffinity: None
  type: NodePort
status:
  loadBalancer: {}
```