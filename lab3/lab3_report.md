University: [ITMO University](https://itmo.ru/ru/)
Faculty: [FICT](https://fict.itmo.ru)
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)
Year: 2024/2025
Group: К4110с
Author: ABU HUSSAIN Ghaith
Lab: Lab3
Date of create: 09.01.2025
Date of finished: 16.01.2025

# Lab3
### Purpose of the work
Get acquainted with certificates and "secrets" in Minikube, rules for secure data storage in Minikube.

### Progress
1.  Create `configMap` with the variables`REACT_APP_USERNAME` and`REACT_APPCOMPANY_NAME`
2.  Create with 2 container replicas `replicaSet` and pass variables `REACT_APP_USERNAME`and`REACT_APPCOMPANY_NAME` , via `configMap`.
3.  Enable and generate a TLS certificate `minikube addons enable ingress`, import the certificate into minikube.
4.  Create `ingress` in minikube.
5.  Enter the `hosts` FQDN and IP address of ingress and go to the browser.
6.  Log in to the web application using your own FQDN using HTTPS and check for a certificate.

### Getting the job done
#### 1. Creation of`configMap` with variables,`REACT_APP_USERNAME`and`REACT_APPCOMPANY_NAME`.
We create the `configmap.yaml` file with the content:

    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: react-config
      namespace: default
    data:
      REACT_APP_USERNAME: "GhaithAbuHussain"
      REACT_APP_COMPANY_NAME: "itmo"
Then we apply the command: 

    kubectl apply -f configmap.yaml
to apply the config map:
![enter image description here](https://github.com/Ghaith-Abuhussain/2024-2025-introduction_to_distributed_technologies-K4110c-ABU-HUSSAIN-Ghaith/blob/main/lab3/images/2.%20create_configmap.PNG?raw=true)

#### 2. Create a `replicaSet` with 2 replicas of the container `ifilyaninitmo/itdt-contained-frontend:master` and pass the variables `REACT_APP_USERNAME`, `REACT_APP_COMPANY_NAME` using the previously created `configMap`
In the previous lab works we used `deployment` which ensure higher-level abstraction for managing ReplicaSets and pods. The Replicasets ensures a specific number of pod replicas are running and low-level resource for managing pods directly.
We write the manifest`replicaset.yaml`:

    apiVersion: apps/v1
    kind: ReplicaSet
    metadata:
      name: react-app-replicaset
    spec:
      replicas: 2
      selector:
        matchLabels:
          app: react-app
      template:
        metadata:
          labels:
            app: react-app
        spec:
          containers:
          - name: react-app-container
            image: ifilyaninitmo/itdt-contained-frontend:master
            envFrom:
            - configMapRef:
                name: react-config
            ports:
            - containerPort: 3000

Here we specify the object type `ReplicaSet`, and the number of `replicas = 2`. In the template specifications, specify the name of the container and the image we need. We also indicate the name of our `envFrom: configMapRef``congifMap`.
We apply the manifest and check the replicaset and the pods

    kubectl apply -f replicaset.yaml
    kubectl get replicaset
    kubectl get pods
![enter image description here](https://github.com/Ghaith-Abuhussain/2024-2025-introduction_to_distributed_technologies-K4110c-ABU-HUSSAIN-Ghaith/blob/main/lab3/images/3.%20create_replicaset.PNG?raw=true)
We can describe one of the pods:

    kubectl describe pod react-app-replicaset-bgwqb

![enter image description here](https://github.com/Ghaith-Abuhussain/2024-2025-introduction_to_distributed_technologies-K4110c-ABU-HUSSAIN-Ghaith/blob/main/lab3/images/4.%20describe_pod_replicas.PNG?raw=true)
We can see that the pods are running correctly.
#### 3. Enabling the ingress addon and creating a tsl certificate.
We use this command:

    minikube addons enable ingress
To enable ingress addon.
![enter image description here](https://github.com/Ghaith-Abuhussain/2024-2025-introduction_to_distributed_technologies-K4110c-ABU-HUSSAIN-Ghaith/blob/main/lab3/images/5.%20enable_ingress.PNG?raw=true)
We can check the existence of the `ingress-nginx-controller` using this command:

    kubectl get pods -n ingress-nginx
 ![enter image description here](https://github.com/Ghaith-Abuhussain/2024-2025-introduction_to_distributed_technologies-K4110c-ABU-HUSSAIN-Ghaith/blob/main/lab3/images/6.%20get_ingress_nginx_pods.PNG?raw=true)
Now we need to create the tls certificate, we start by checking the the openssl plugin:
![enter image description here](https://github.com/Ghaith-Abuhussain/2024-2025-introduction_to_distributed_technologies-K4110c-ABU-HUSSAIN-Ghaith/blob/main/lab3/images/7.%20check_open_ssl_existance.PNG?raw=true)
The next step to creating a certificate is to create a private key. It is used for data encryption, certificate signing, and other cryptographic operations. It is executed by the command to generate a private key, specifying the algorithm - RSA.

    openssl genpkey -algorithm RSA -out private.key

![enter image description here](https://github.com/Ghaith-Abuhussain/2024-2025-introduction_to_distributed_technologies-K4110c-ABU-HUSSAIN-Ghaith/blob/main/lab3/images/8.%20generate_private_key.PNG?raw=true)
Next, we need to create a request for signing the certificate, with an explicit indication of the path to the file "openssl.cnf" (without an explicit indication, the program tried to find this file in a non-existent directory).

    openssl req -new -key private.key -out csr.pem
![enter image description here](https://github.com/Ghaith-Abuhussain/2024-2025-introduction_to_distributed_technologies-K4110c-ABU-HUSSAIN-Ghaith/blob/main/lab3/images/9.%20generate%20csr.pem.PNG?raw=true)
We can then verify that the certificate generation request was successful.

    cat csr.pem
![enter image description here](https://github.com/Ghaith-Abuhussain/2024-2025-introduction_to_distributed_technologies-K4110c-ABU-HUSSAIN-Ghaith/blob/main/lab3/images/10.%20display_csr.pem.PNG?raw=true)
Now, we can create the certificate:

    openssl req -x509 -days 365 -key private.key -in csr.pem -out certificate.crt
Here you can specify the number of days `-days 356` that the certificate is valid for, the file with the private key `-key private.key`, the file with the request to generate the key `-in csr.pem`, and the name of the certificate itself `-out certificate.crt`. Here I also had to specify an explicit path to the file "openssl.cnf".
![enter image description here](https://github.com/Ghaith-Abuhussain/2024-2025-introduction_to_distributed_technologies-K4110c-ABU-HUSSAIN-Ghaith/blob/main/lab3/images/11.%20generate_certificate.crt.PNG?raw=true)
The next step is to import the certificate into `minikube`. Kubernetes uses secrets to store certificates. To create it, we need to run the following command:

    kubectl create secret tls my-app-tls --key private.key --cert certificate.crt
![enter image description here](https://github.com/Ghaith-Abuhussain/2024-2025-introduction_to_distributed_technologies-K4110c-ABU-HUSSAIN-Ghaith/blob/main/lab3/images/12.%20create_my_app_tls_secret.PNG?raw=true)
#### 4. Creating service and ingress.
We create the service.yaml manifest:

    apiVersion: v1
    kind: Service
    metadata:
      name: react-app-service
    spec:
      selector:
        app: react-app
      ports:
        - protocol: TCP
          port: 3000
          targetPort: 3000
          nodePort: 30007 
      type: NodePort
the service is from type `NodePort` with the `nodePort: 30007`. The service will run the app `react-app`.
We apply this service:

    kubectl apply -f service.yaml
![enter image description here](https://github.com/Ghaith-Abuhussain/2024-2025-introduction_to_distributed_technologies-K4110c-ABU-HUSSAIN-Ghaith/blob/main/lab3/images/13.%20create_service.PNG?raw=true)
Now, we create the ingress.yaml manifest:

    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: react-app-ingress
      annotations:
        nginx.ingress.kubernetes.io/ssl-redirect: "true"
    spec:
      tls:
        - hosts:
            - react-app.local
          secretName: my-app-tls
      rules:
        - host: react-app.local
          http:
            paths:
              - path: /
                pathType: Prefix
                backend:
                  service:
                    name: react-app-service
                    port:
                      number: 3000
The same secret created in the previous step `secretName: my-app-tls`. The domain name `host: react-app.local` specified when we created the certificate.
We then apply the manifest:

    kubectl apply -f ingress.yaml
![enter image description here](https://github.com/Ghaith-Abuhussain/2024-2025-introduction_to_distributed_technologies-K4110c-ABU-HUSSAIN-Ghaith/blob/main/lab3/images/14.%20create_ingress.PNG?raw=true)
We can check the creation of the ingress:

    kubectl get ingress
![enter image description here](https://github.com/Ghaith-Abuhussain/2024-2025-introduction_to_distributed_technologies-K4110c-ABU-HUSSAIN-Ghaith/blob/main/lab3/images/15.%20display_ingress.PNG?raw=true)
We can see that the `react-app-ingress` took the host `react-app.local` and the address is `192.168.49.2`. The ports are 80 (for http and 443 for https).
Now, we update the hosts for `/etc/hosts`. We need to add the line:
`192.168.49.2				react-app.local`
before the IPV6 capable hosts.
![enter image description here](https://github.com/Ghaith-Abuhussain/2024-2025-introduction_to_distributed_technologies-K4110c-ABU-HUSSAIN-Ghaith/blob/main/lab3/images/16.%20update_etc_hosts.PNG?raw=true)
#### 6. Checking the certificate in the browser
Now we can go to the browser using the specified link `react-app.local`. A familiar interface opens.
![enter image description here](https://github.com/Ghaith-Abuhussain/2024-2025-introduction_to_distributed_technologies-K4110c-ABU-HUSSAIN-Ghaith/blob/main/lab3/images/17.%20calling_for_react_app_local_with_https.PNG?raw=true)
So, from the browser, we can check the certificate:
![enter image description here](https://github.com/Ghaith-Abuhussain/2024-2025-introduction_to_distributed_technologies-K4110c-ABU-HUSSAIN-Ghaith/blob/main/lab3/images/18.%20display_certificate_from_browser.PNG?raw=true)
