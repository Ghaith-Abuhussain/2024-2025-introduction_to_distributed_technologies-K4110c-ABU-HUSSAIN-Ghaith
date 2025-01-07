University: [ITMO University](https://itmo.ru/ru/)
Faculty: [FICT](https://fict.itmo.ru)
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)
Year: 2024/2025
Group: К4110с
Author: ABU HUSSAIN Ghaith
Lab: Lab1
Date of create: 07.01.2025
Date of finished: 09.01.2025

# Lab1
In this lab we created a pod with the name vault in the local worker node. The pod will run a container with the name *vault* and image *hashicorp/vault:latest*.
To create the pod we used the .yaml file *vault-pod.yaml* and the code to create such a pod is:

    apiVersion: v1
    kind: Pod
    metadata:
      name: vault
      labels:
        app: vault
    spec:
      containers:
      - name: vault
        image: hashicorp/vault:latest
        ports:
        - containerPort: 8200
          protocol: TCP
        env:
        - name: VAULT_DEV_ROOT_TOKEN_ID
          value: "ghaith"
        - name: VAULT_DEV_LISTEN_ADDRESS
          value: "0.0.0.0:8200"
        command: ["vault", "server", "-dev"]

and we used the command:

    kubectl apply -f vault-pod.yaml
  to apply the pod.
  After that we used the command:
  

    minikube kubectl -- expose pod vault --type=NodePort --port=8200
to create a service as it is indicated in the lab proceedure but the service with the name *vault* was already created:

    Error from server (AlreadyExists): services "vault" already exists
  After that, to access the container, we used the following command:
  

    minikube kubectl -- port-forward service/vault 8200:8200
  which will forward the port from our computer to the container. 
  After that we access Vault at [http://localhost:8200](http://localhost:8200/)
  then we logged in using the token which was *ghaith*. We found the token using the pod log after executing this command:
  

    kubectl logs vault
We can find from the output of this command that the root token is `ghaith`:

    Unseal Key: mcA6Z6BqEGmmwray97dT0/1VsSDIos3CaiXifeA6W3E=
    Root Token: ghaith
Here we attach the diagram of the containers and services:
![enter image description here](https://github.com/Ghaith-Abuhussain/2024-2025-introduction_to_distributed_technologies-K4110c-ABU-HUSSAIN-Ghaith/blob/main/lab1/diagram.PNG?raw=true)
After that we logged in to the vault using the *ghaith* token:
![enter image description here](https://github.com/Ghaith-Abuhussain/2024-2025-introduction_to_distributed_technologies-K4110c-ABU-HUSSAIN-Ghaith/blob/main/lab1/vault.PNG?raw=true)
