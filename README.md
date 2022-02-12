# kube-ingress-expressjs
how to setup an https expressjs site

## pre-requisites
you will need:
* a domain name
  * [name.com](http://name.com/)
* access to a kubernetes accounts
  * [civo](https://civo.com)
  * [linode](https://linode.com) 

## instructions
<details> <summary>point your domain's nameservers to your kubernetes providers nameservers </summary>

     <details> <summary> for <a href='https://civo.com'>civo</a> </summary>

       ```
       ns0.civo.com
       ns1.civo.com
       ```
      
     </details>
 
</details>

2. create an expressjs deployment
<details>
 
``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  generation: 1
  name: expressjs
  labels:
   app: expressjs
spec:
  replicas: 1
  selector:
    matchLabels:
      app: expressjs
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
    type: RollingUpdate
  template:
    metadata:
      name: expressjs
      labels:
        app: expressjs
    spec:
      containers:
      - name: expressjs
        image: alexellis2/service:0.3.5
        imagePullPolicy: Always
        resources:
          limits:
            cpu: 50m
            memory: 128Mi
          requests:
            cpu: 50m
            memory: 128Mi
        ports:
        - containerPort: 8080
          protocol: TCP
        readinessProbe:
          failureThreshold: 3
          httpGet:
            path: /health
            port: 8080
            scheme: HTTP
          initialDelaySeconds: 2
          periodSeconds: 2
          successThreshold: 1
          timeoutSeconds: 1
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      securityContext: {}
      terminationGracePeriodSeconds: 30
```
</details>
