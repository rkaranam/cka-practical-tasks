#### Question

Below is a scenario which needs to be configured on Kubernetes cluster. Please find below more details about it.

1.  Create a  `pod`  named  `print-envars-greeting`.
    
2.  Configure spec as, the container name should be  `print-env-container`  and use  `bash`  image.
    
3.  Create three environment variables:
    

a.  `GREETING`  and its value should be  `Welcome to`

b.  `COMPANY`  and its value should be  `Nautilus`

c.  `GROUP`  and its value should be  `Industries`

4.  Use command  `["/bin/sh", "-c", 'echo "$(GREETING) $(COMPANY) $(GROUP)"']`  (please use this exact command), also set its  `restartPolicy`  policy to  `Never`  to avoid crash loop back.
    
5.  You can check the output using  `kubectl logs -f print-envars-greeting`  command.

#### Solution

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: print-envars-greeting
spec:
  containers:
  - image: bash
    name: print-env-container
    resources: {}
    command: ["/bin/sh", "-c", 'echo "$(GREETING) $(COMPANY) $(GROUP)"']
    env:
    - name: GREETING
      value: "Welcome to"
    - name: COMPANY
      value: "Nautilus"
    - name: GROUP
      value: "Industries"
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```