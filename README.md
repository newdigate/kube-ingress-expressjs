# creating a secure website 
how to setup a secure expressjs site using kubernetes

## pre-requisites
* a domain name
* access to an online kubernetes service provider
  * I'm using [civo](https://civo.com)
    * signup with civo and get $250 credit (expires in 2 months)
* [kubectl](https://kubernetes.io/docs/tasks/tools/) client 

## instructions
* **create** and **connect** to your cluster
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
  
* create an expressjs **deployment**
  * using alexellis2/service:0.3.5 container image to deploy a simple nodejs application webapp using expressjs  
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
  
* create a **service**
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
  
  ``` sh
  > kubectl apply -f expressjs-service.yaml
  ```
  
* create an **issuer**
  <details> <summary>click to expand expressjs-issuer.yaml</summary>
 
  ``` yaml
  apiVersion: cert-manager.io/v1
  kind: Issuer
  metadata:
    name: letsencrypt-prod
  spec:
    acme:
      email: name@gmail.com
      server: https://acme-v02.api.letsencrypt.org/directory
      privateKeySecretRef:
        name: expressjs-secret-issuer-account-key
      solvers:
      - http01:
          ingress:
            class: traefik
   ```

   </details>
  
  ``` sh
  > kubectl apply -f expressjs-issuer.yaml
  ```
  
 * create an **ingress**
   * replace `expressjs.example.com` and `wildcard-example-com-tls` with your own domain name 
 
      <details> <summary>click to expand expressjs-ingress.yaml</summary>

     ``` yaml
     apiVersion: networking.k8s.io/v1
     kind: Ingress
     metadata:
       name: express
       annotations:
         cert-manager.io/issuer: letsencrypt-prod
         kubernetes.io/ingress.class: "traefik"
     spec:
       tls:
       - hosts:
         - expressjs.example.com
         secretName: wildcard-example-com-tls
       rules:
       - host: "expressjs.example.com"
         http:
           paths:
           - pathType: Prefix
             path: "/"
             backend:
               service:
                 name: expressjs
                 port:
                   number: 8080
     ```

    </details>
  
    ``` sh
    > kubectl apply -f expressjs-ingress.yaml
    ```
