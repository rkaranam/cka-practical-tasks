#### Question
An application deployed on the Kubernetes cluster requires an update with new features developed by the Nautilus application development team. The existing setup includes a deployment named  `nginx-deployment`  and a service named  `nginx-service`. Below are the necessary changes to be implemented without deleting the deployment and service:

1.) Modify the service nodeport from  `30008`  to  `32165`

2.) Change the replicas count from  `1`  to  `5`

3.) Update the image from  `nginx:1.17`  to  `nginx:latest`

#### Solution

```bash
k set image deployment/nginx-deployment nginx-container=nginx:latest
k scale deploy/nginx-deployment --replicas=5
k rollout status deploy/nginx-deployment
```