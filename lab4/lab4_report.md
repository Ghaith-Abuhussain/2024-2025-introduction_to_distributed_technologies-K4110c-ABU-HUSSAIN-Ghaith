University: [ITMO University](https://itmo.ru/ru/)  
Faculty: [FICT](https://fict.itmo.ru)  
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)  
Year: 2024/2025  
Group: К4110с  
Author: ABU HUSSAIN Ghaith  
Lab: Lab4  
Date of create: 10.01.2025  
Date of finished: 16.01.2025  

# Lab4
### Purpose of the work
Get acquainted with CNI Calico and the IPAM Plugin function, learn the features of CNI and CoreDNS.

### Progress
1.  Run `minikube` with the `CNI=Calico` plugin and check.
2.  Specify `lable` for previously launched nodes and assign IP addresses.
3.  Create `deployment` with 2 container replicas.
4.  Create a service to access pods and start port forwarding mode and connect to containers through a web browser, check `Container name`  and `Container IP`.
5.  Ping pods.

### Getting the job done
#### 1. Running Minikube with the `CNI=Calico`.

    minikube start --nodes=2 --cni=calico
![enter image description here](https://github.com/Ghaith-Abuhussain/2024-2025-introduction_to_distributed_technologies-K4110c-ABU-HUSSAIN-Ghaith/blob/main/lab4/images/1.%20deploy%202%20nodes%20in%20minikube%20cluster%20with%20cni_calico.PNG?raw=true)  
We can check the created nodes:

    kubectl get nodes
![enter image description here](https://github.com/Ghaith-Abuhussain/2024-2025-introduction_to_distributed_technologies-K4110c-ABU-HUSSAIN-Ghaith/blob/main/lab4/images/2.%20check%20the%20deployment%20of%20the%20two%20nodes.PNG?raw=true)  
We can see that the nodes are created successfully.
#### 2. Specify the labels for containers
To label nodes using default labeling, assign a label to each of them, for example, by geographical location:

    kubectl label nodes minikube-m02 zone=south
    kubectl label nodes minikube zone=north

![enter image description here](https://github.com/Ghaith-Abuhussain/2024-2025-introduction_to_distributed_technologies-K4110c-ABU-HUSSAIN-Ghaith/blob/main/lab4/images/3.%20labeling%20nodes%20without%20yaml%20file%20to%20check%20the%20IPAM.PNG?raw=true)  
We can check the labeling by describing the nodes:

    kubectl describe node minikube
![enter image description here](https://github.com/Ghaith-Abuhussain/2024-2025-introduction_to_distributed_technologies-K4110c-ABU-HUSSAIN-Ghaith/blob/main/lab4/images/4.%20describe%20node%20minikube%20for%20IPAM%20labeling.PNG?raw=true)  

        kubectl describe node minikube-m02
![enter image description here](https://github.com/Ghaith-Abuhussain/2024-2025-introduction_to_distributed_technologies-K4110c-ABU-HUSSAIN-Ghaith/blob/main/lab4/images/5.%20describe%20node%20minikube-m02%20for%20IPAM%20labeling.PNG?raw=true)  
Now, we want to address the IP address, So, we delete the `default-ipv4-ippool`:

    kubectl delete ippool default-ipv4-ippool
![enter image description here](https://github.com/Ghaith-Abuhussain/2024-2025-introduction_to_distributed_technologies-K4110c-ABU-HUSSAIN-Ghaith/blob/main/lab4/images/6.%20delete%20default%20ippool.PNG?raw=true)  
Now, we create 2 yaml files for the ippool configurations. 
file: ippool_north.yaml  

    apiVersion: projectcalico.org/v3
    kind: IPPool
    metadata:
      name: zone-north-ippool
    spec:
      cidr: 192.168.1.0/24
      ipipMode: Always
      natOutgoing: true
      nodeSelector: zone == "north"

  
file: ippool_south.yaml

    apiVersion: projectcalico.org/v3
    kind: IPPool
    metadata:
      name: zone-south-ippool
    spec:
      cidr: 192.168.0.0/24
      ipipMode: Always
      natOutgoing: true
      nodeSelector: zone == "south"

To apply these manifests, you must first install `calicoctl`.

    curl -L https://github.com/projectcalico/calico/releases/download/v3.29.1/calicoctl-linux-amd64 -o calicoctl
    chmod +x ./calicoctl
    sudo mv calicoctl /usr/local/bin/calicoctl
![enter image description here](https://github.com/Ghaith-Abuhussain/2024-2025-introduction_to_distributed_technologies-K4110c-ABU-HUSSAIN-Ghaith/blob/main/lab4/images/7.%20install%20calicoctl.PNG?raw=true)  
Let us check the version of `calicoctl`:
![enter image description here](https://github.com/Ghaith-Abuhussain/2024-2025-introduction_to_distributed_technologies-K4110c-ABU-HUSSAIN-Ghaith/blob/main/lab4/images/8.%20check%20calicoctl%20version.PNG?raw=true)  
Now, we apply the manifest with `calicoctl`:

    calicoctl create -f ippool_north.yaml --allow-version-mismatch
    calicoctl create -f ippool_south.yaml --allow-version-mismatch
![enter image description here](https://github.com/Ghaith-Abuhussain/2024-2025-introduction_to_distributed_technologies-K4110c-ABU-HUSSAIN-Ghaith/blob/main/lab4/images/10.%20create_ippool_north.PNG?raw=true)  
![enter image description here](https://github.com/Ghaith-Abuhussain/2024-2025-introduction_to_distributed_technologies-K4110c-ABU-HUSSAIN-Ghaith/blob/main/lab4/images/11.%20create_ippool_south.PNG?raw=true)  
Now, we can check the created ippools:

    calicoctl get ippool -o wide --allow-version-mismatch
![enter image description here](https://github.com/Ghaith-Abuhussain/2024-2025-introduction_to_distributed_technologies-K4110c-ABU-HUSSAIN-Ghaith/blob/main/lab4/images/12.%20check%20created%20ippools%20from%20yaml%20files.PNG?raw=true)  
#### 3. Create `deployment` with 2 container replicas.
We created the manifest file:

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: cni-test-deployment
    spec:
      replicas: 2
      selector:
        matchLabels:
          app: cni-test-deployment
      template:
        metadata:
          labels:
            app: cni-test-deployment
        spec:
          containers:
            - name: cni-container
              image: ifilyaninitmo/itdt-contained-frontend:master
              ports:
              - containerPort: 3000
              env:
                - name: REACT_APP_USERNAME
                  value: "GhaithAbuHussain"
                - name: REACT_APP_COMPANY_NAME
                  value: "itmo"
We apply this manifest:

    kubectl apply -f deployment.yaml
![enter image description here](https://github.com/Ghaith-Abuhussain/2024-2025-introduction_to_distributed_technologies-K4110c-ABU-HUSSAIN-Ghaith/blob/main/lab4/images/13.%20create_deployment.PNG?raw=true)  
We can check the deployment and created pods:

    kubectl get deployments
    kubectl get pods

![enter image description here](https://github.com/Ghaith-Abuhussain/2024-2025-introduction_to_distributed_technologies-K4110c-ABU-HUSSAIN-Ghaith/blob/main/lab4/images/14.%20check%20for%20deployments%20and%20pods.PNG?raw=true)  
Everything runs correctly.
#### 4. Create `service` and start port-forwarding.
We create the `service.yaml` manifest.

    apiVersion: v1
    kind: Service
    metadata:
      name: cni-service
    spec:
      selector:
        app: cni-test-deployment
      ports:
        - protocol: TCP
          port: 3000         
      type: ClusterIP   
And we apply this service:

    kubectl apply -f service.yaml

![enter image description here](https://github.com/Ghaith-Abuhussain/2024-2025-introduction_to_distributed_technologies-K4110c-ABU-HUSSAIN-Ghaith/blob/main/lab4/images/15.%20apply%20service.PNG?raw=true)  
 Now, we forward the service on port 3000:
 

    minikube kubectl -- port-forward service/cni-service 3000:3000

 ![enter image description here](https://github.com/Ghaith-Abuhussain/2024-2025-introduction_to_distributed_technologies-K4110c-ABU-HUSSAIN-Ghaith/blob/main/lab4/images/16.%20forward%20service.PNG?raw=true)  
We can check if the service working correctly by calling `localhost:3000` from the browser:
![enter image description here](https://github.com/Ghaith-Abuhussain/2024-2025-introduction_to_distributed_technologies-K4110c-ABU-HUSSAIN-Ghaith/blob/main/lab4/images/17.%20call%20for%20service%20on%20localhost_3000.PNG?raw=true)  
From the previous image we see that the pod is `cni-test-deployment-6f764f7c68-dxdq2` with the IP `192.168.1.65`. We can check the created pods and their nodes:

    kubectl get pods -o wide

![enter image description here](https://github.com/Ghaith-Abuhussain/2024-2025-introduction_to_distributed_technologies-K4110c-ABU-HUSSAIN-Ghaith/blob/main/lab4/images/18.%20check%20which%20pod%20got%20the%20ip%20address.PNG?raw=true)  
#### 5. Pinging
Using  `kubectl exec`, we enter any the pod `cni-test-deployment-6f764f7c68-9qh8l` which has the IP `192.168.0.1` and is assigned to the node `minikube-m02`. Then we ping the address `192.168.1.65` which is the address of the pod `cni-test-deployment-6f764f7c68-dxdq2` which is assigned to the node `minikube`. We can see the result in the following image:
![enter image description here](https://github.com/Ghaith-Abuhussain/2024-2025-introduction_to_distributed_technologies-K4110c-ABU-HUSSAIN-Ghaith/blob/main/lab4/images/19.%20test_ping_1.PNG?raw=true)  
We also do the same by entering the pod `cni-test-deployment-6f764f7c68-dxdq2` and ping the address `192.168.0.1` of the pod `cni-test-deployment-6f764f7c68-9qh8l` which is assigned to the node `minikube-m02`. We can see the result in the following image:
![enter image description here](https://github.com/Ghaith-Abuhussain/2024-2025-introduction_to_distributed_technologies-K4110c-ABU-HUSSAIN-Ghaith/blob/main/lab4/images/20.%20test_ping_2.PNG?raw=true)  
We can see as an overall result that the ping process between the nodes was successful.
Here we can see the schema of the lab work:
![enter image description here](https://github.com/Ghaith-Abuhussain/2024-2025-introduction_to_distributed_technologies-K4110c-ABU-HUSSAIN-Ghaith/blob/main/lab4/schema.PNG?raw=true)  
