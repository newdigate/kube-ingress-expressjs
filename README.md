# creating a secure website 
how to setup a secure expressjs site using kubernetes

## pre-requisites
* a domain name
* access to a kubernetes accounts
  * I'm using [civo](https://civo.com) - signup and get $250 credit (expires in 2 months)
* [kubectl](https://kubernetes.io/docs/tasks/tools/) client 

## instructions
* create and connect to your cluster
  * in your kubernetes service provider website, create a new cluster
    * in [civo](https://civo.com), when creating the new cluster, you can specify [cert-manager](https://cert-manager.io/docs/) to be installed under applications > architecture 
  * download the kubernetes config for the new cluster (store it securely)
  * copy to ~/.kube/config or local development working path and set environment variable KUBECONFIG
  ```
  > export KUBECONFIG=$(PWD)/civo-clustername-config
  ```
* DNS setup
  * login to your domain name service provider website 
    * point the nameservers for your domain to your kubernetes service providers nameservers
  * login to your kubernetes service provider website
    * add your domain name to your kubernetes service providers DNS management 
    * add a CNAME for a sub-domain and point the CNAME to the public IP address of your cluster
  
* create an expressjs deployment
  <details> <summary>click to expand expressjs-deployment.yaml</summary>
 
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
  
  ``` sh
  > kubectl apply -f expressjs-deployment.yaml
  ```
* deploy a service
  <details> <summary>click to expand expressjs-service.yaml</summary>
 
  ``` yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: expressjs
  spec:
    ports:
    - name: http
      port: 8080
      protocol: TCP
      targetPort: 8080
    selector:
      app: expressjs
    sessionAffinity: None
    type: ClusterIP
  ```
 
  </details>
  
  ```
  > kubectl apply -f expressjs-service.yaml
  ```
