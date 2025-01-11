University: [ITMO University](https://itmo.ru/ru/)
Faculty: [FICT](https://fict.itmo.ru)
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)
Year: 2024/2025  
Group: K4110c  
Author: ABU HUSSAIN Ghaith
Lab: Lab2  
Date of create: 08.01.2025  
Date of finished: 16.01.2025

# Lab2
### Purpose of the work
Familiarize yourself with the types of container deployment "controllers," familiarize yourself with network services, and deploy your web application.
### Progress
1.  Create with 2 container replicas.`deployment`
2.  Create a service to access pods.
3.  Run into port forwarding mode and connect to containers through a web browser.`minikube`
4.  Checking on a page in a web browser for the variables , , and .`REACT_APP_USERNAME``REACT_APP_COMPANY_NAME``Container name`
5.  Checking logs for containers.

### Procedure
#### 1. Create a deployment with 2 replicas
At first we created the deployment.yaml file which will hold the manifest to create the deployment.

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: frontend-deployment
    spec:
      replicas: 2
      selector:
        matchLabels:
          app: frontend
      template:
        metadata:
          labels:
            app: frontend
        spec:
          containers:
            - name: frontend-container
              image: ifilyaninitmo/itdt-contained-frontend:master
              env:
                - name: REACT_APP_USERNAME
                  value: "GhaithAbuHussain"
                - name: REACT_APP_COMPANY_NAME
                  value: "itmo"
There are several important aspects this file.

1.  In the parameter `kind``Deployment`, we specify the type of resource that controls the deployment and update of a set of containers.
2.  The value `spec: replicas``2` is specified because that's how many container replicas we need to support.
3.  The parameter `selector:``app: frontend`. It determines which pods belong to a given Deployment. In this case, only pods with the appropriate label will be managed by this Deployment.
4.  The parameter `template``Deployment``app: frontend` which describes the pod template that will be deployed to the cluster. This is where we specify the right label that Kubernetes can use to find and link pods.
5.  In the section, specify the container and the two environment variables:`spec``templates``REACT_APP_USERNAME` and`REACT_APP_COMPANY_NAME`.

We applied the manifest using the command:

    kubectl apply -f deployment.yaml
![enter image description here](https://github.com/Ghaith-Abuhussain/2024-2025-introduction_to_distributed_technologies-K4110c-ABU-HUSSAIN-Ghaith/blob/main/lab2/images/creating_deployments.PNG?raw=true)
The output of this command shows that the deployment is correctly created. We can check for the creation using this command:

    kubectl get deployments
  ![enter image description here](https://github.com/Ghaith-Abuhussain/2024-2025-introduction_to_distributed_technologies-K4110c-ABU-HUSSAIN-Ghaith/blob/main/lab2/images/available_deployments.PNG?raw=true)
We can see that there are 2 ready and available deployments.
We can check the pods also using this command:

    kubectl get pods
  ![enter image description here](https://github.com/Ghaith-Abuhussain/2024-2025-introduction_to_distributed_technologies-K4110c-ABU-HUSSAIN-Ghaith/blob/main/lab2/images/running_pods.PNG?raw=true)
We can check how the created Deployment recreate pods with the recovery of "broken" containers. To do this, get a list of pods and delete them then re-display the list of pods again:
![enter image description here](https://github.com/Ghaith-Abuhussain/2024-2025-introduction_to_distributed_technologies-K4110c-ABU-HUSSAIN-Ghaith/blob/main/lab2/images/check_for_deleting%29pods.PNG?raw=true)
As we can see a new pods have been created instead of the previous ones.
### 2. Creating of service to access pods
We also created the manifest for the service:

    apiVersion: v1
    kind: Service
    metadata:
      name: frontend-service
    spec:
      selector:
        app: frontend
      ports:
        - protocol: TCP
          port: 80         
          targetPort: 3000  
      type: NodePort    
In this manifest, it indicates that the object is a service. - This is the port inside the container where our application runs. This is the port to which the external tarfic is directed. Service type.`kind: service``targetPort: 3000``port: 80``NodePort`
We create the service using this command:

    kubectl apply -f service.yaml
we can check the service creation using this command:

    kubectl get svc frontend-service
 ![enter image description here](https://github.com/Ghaith-Abuhussain/2024-2025-introduction_to_distributed_technologies-K4110c-ABU-HUSSAIN-Ghaith/blob/main/lab2/images/checkservice.PNG?raw=true)
### 3. Forwarding the port
Now we need to let Minikube forward the port from our computer to the container. So, we use this command:

    kubectl port-forward svc/frontend-service 8082:80
Now using the browser, we call for *localhost:8082*. and we can see the result in the following image:
![enter image description here](https://github.com/Ghaith-Abuhussain/2024-2025-introduction_to_distributed_technologies-K4110c-ABU-HUSSAIN-Ghaith/blob/main/lab2/images/Calling_for_localhost.PNG?raw=true)
### 4. Variable Validation
Previously we can see the container name and the container IP in addition to the environment variables (*REACT_APP_USERNAME* and *REACT_APP_COMPANY_NAME*).
Lets delete the pod frontend-deployment-648bd59fcf-m8s7z and see what happens:
![enter image description here](https://github.com/Ghaith-Abuhussain/2024-2025-introduction_to_distributed_technologies-K4110c-ABU-HUSSAIN-Ghaith/blob/main/lab2/images/delete_pod_command.PNG?raw=true)
Now let us call the *localhost:8082*:
![enter image description here](https://github.com/Ghaith-Abuhussain/2024-2025-introduction_to_distributed_technologies-K4110c-ABU-HUSSAIN-Ghaith/blob/main/lab2/images/Calling_for_localhost_after_deleting_the_pod.PNG?raw=true)
As you can see, the name of the container has changed to ***frontend-deployment-648bd59fcf-qg2nr***, while the variables have remained the same. This is because these names are created during the initialization phase of the Deployment object and are written in the manifest. As a result, they do not change even when you change the working container, to change them, you need to update the deployment manifest and recreate pods with new variable values.

As for the variable `container_name`, it directly depends on the name of the working container. Accordingly, if we change the container, the variable also changes.

### 5. Schema
![enter image description here](https://github.com/Ghaith-Abuhussain/2024-2025-introduction_to_distributed_technologies-K4110c-ABU-HUSSAIN-Ghaith/blob/main/lab2/lab2_diagram.PNG?raw=true)
