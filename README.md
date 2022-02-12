# creating a secure website 
how to setup an https expressjs site using kubernetes

## pre-requisites
you will need:
* a domain name
* access to a kubernetes accounts
  * I'm using [civo](https://civo.com) - signup and get $250 credit (expires in 2 months)

## instructions
* create and connect to your cluster
  * create a new cluster
  * download the kubernetes config for the new cluster (store it securely)
  * copy to ~/.kube/config or local development working path and set environment variable KUBECONFIG
  ```
  > export KUBECONFIG=$(PWD)/civo-clustername-config
  ```
* DNS setup
  * login to your domain name providers website and point the nameservers for your domain to your kubernetes service providers nameservers
  * add your domain name to your kubernetes service providers DNS management 
  * add a CNAME for a sub-domain and point the CNAME to the public IP address of your cluster
  
* create an expressjs deployment
  <details> <summary>click to expand .yaml</summary>
 
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
