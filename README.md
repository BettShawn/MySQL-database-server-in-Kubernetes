# Deploy a MySQL database server in Kubernetes - Static
In this tutorial, we are going to learn how to deploy a MySQL database server in a Kubernetes Cluster set up in a local machine, this is one of many ways to persist data in Kubernetes.

Kubernetes is a tool for automating **deployment**, scaling, and management of containerized applications.

Get familiar with some terminologies and Kubernetes objects that will be used through this tutorial:

**Docker Image**: A collection of files that packs together all the necessities needed to set up a completely functional container.

**Container**: An instance of an image, a running image;

**Node**: A Kubernetes Object, a virtual machine that runs a container and provide resources;

**Kubernetes Cluster**: A collection of nodes and configurations to manage them;

**Pod**: A Kubernetes object, a running container, the smallest deployable units of computing that can be created and managed in kubernetes;

**Deployment**: A Kubernetes Object, that monitors set of pods, it make sure that those pods are running and make sure to restart pods if they are down;

**Service**: A Kubernetes Object that provides a way to access a running container(pod);

**Persistent Volume (PV)**: A Kubernete object, is a piece of storage in the cluster;

**Persistent Volume Claim (PVC)**: A request for the Persistent Volume storage;

Kubernetes **Config** file: A file that tells Kubernetes about the different Objects to be created. It's written in YAML syntax (.yaml).

Technically, we will create a Deployment that will manage a Pod running a container of a MySQL docker image, then we will create a Service that will permit access to the pod. This pod will request for storage (using Persistent Volume Claim) to a storage resource (Persistent Volume).

A Persistent Volume can be created statically or dynamically. In the next phase of this tutorial, we learn how to do it statically.

Prerequisites:

(a) Docker in our machine;

(b) Local Kubernetes Cluster via Minikube;

    *Minikube is a tool that makes it easy to run Kubernetes locally. Minikube runs a single-node Kubernetes cluster for users looking to try out Kubernetes or develop with it day-to-day.*


1. Build a Persistent Volume (PV);

  First, create a working directory and navigate in:

     mkdir mysql-kube
     cd mysql-kube/

  I created a simple bash script to automate this

    #!/bin/bash
    
    echo "**** Script Begin"
    
    mkdir mysql-kube
    cd mysql-kube/
    
    echo "**** Script End"
    
![image](https://github.com/BettShawn/MySQL-database-server-in-Kubernetes/assets/51289343/ecfc86fc-2ec6-4575-96fd-0e9d32f6e9ca)

  Create a yaml file named mysql-pv.yaml, put in the following:
    
    apiVersion: v1
    kind: PersistentVolume
    metadata:
      name: mysql-pv
      labels:
        type: local
    spec:
      storageClassName: manual
      capacity:
        storage: 5Gi
      accessModes:
        - ReadWriteOnce
      hostPath:
        path: "/mnt/data"

![image](https://github.com/BettShawn/MySQL-database-server-in-Kubernetes/assets/51289343/8d911049-a90f-4f57-91f6-826f523f1f44)
Save and close the file.

This yaml file once applied in kubernetes, will provision a Persistent Volume, for the MySQL database server Pod. The persistent volume will not depend on the pod's lifecycle. This means that anytime the pod restarts due to a crash or a malfunction, the provisioned storage will survive.

2. Build a Persistent Volume Claim (PVC)
   
  In the working directory mysql-kube/, create a file named mysql-pvc.yaml, put the following:

    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: mysql-pv-claim
    spec:
      storageClassName: manual
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 5Gi

  ![image](https://github.com/BettShawn/MySQL-database-server-in-Kubernetes/assets/51289343/cd0429d4-e9cc-4ae0-8da1-54a326fe24d5)

  Up there, we created the file that will provision a storage when applied, this file on the other hand will create a Persistent Volume Claim that will be used by the     MySQL Pod to request for that provisioned storage.

3. MySQL pod's deployment
  Here you are going to create a file named mysql-deployment.yaml in the same directory, mysql-kube/. Create the file and put the code below:

        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: mysql
        spec:
          selector:
            matchLabels:
              app: mysql
          strategy:
            type: Recreate
          template:
            metadata:
              labels:
                app: mysql
            spec:
              containers:
                - image: mysql:8.0
                  name: mysql
                  env:
                    - name: MYSQL_ROOT_PASSWORD
                      value: password
                  ports:
                    - containerPort: 3306
                      name: mysql
                  volumeMounts:
                    - name: mysql-persistent-storage
                      mountPath: /var/lib/mysql
              volumes:
                - name: mysql-persistent-storage
                  persistentVolumeClaim:
                    claimName: mysql-pv-claim
   ![image](https://github.com/BettShawn/MySQL-database-server-in-Kubernetes/assets/51289343/481fc2bd-596c-4cd0-b22a-267d2454af53)

  This file will create a deployment object to manage a Pod running a container of MySQL docker image and in its specifications, there is a reference to the Persistent     Volume Claim that the pod will use to request for the Persistent Volume.
  
  ![image](https://github.com/BettShawn/MySQL-database-server-in-Kubernetes/assets/51289343/0f47e236-669e-4b6f-810f-865fe164cc6d)

  Before applying this deployment file, create a service object that will permit other pods to access the MySQL database pod that will be created.
  Still in the mysql-kube/ directory, create a yaml file named mysql-service.yaml and put the code below:

    apiVersion: v1
    kind: Service
    metadata:
      name: mysql
    spec:
      ports:
        - port: 3306
      selector:
        app: mysql
      clusterIP: None
      
  ![image](https://github.com/BettShawn/MySQL-database-server-in-Kubernetes/assets/51289343/40be742b-c441-4436-a253-115f08766031)

  Save the file, make sure that our Kubernetes cluster is up and running. Open the terminal and navigate to mysql-kube/ run the following:

    minikube start

  ![image](https://github.com/BettShawn/MySQL-database-server-in-Kubernetes/assets/51289343/8ba09432-d18d-46ca-b32f-7b1b0d16edd9)

  Step 1: Create the Persistent Volume

    kubectl apply -f mysql-pv.yaml

  Step 2: Create the Persistent Volume Claim

    kubectl apply -f mysql-pvc.yaml

  Step 3: Create the Deployment

    kubectl apply -f mysql-deployment.yaml

  Step 4: Create the Service

    kubectl apply -f mysql-service.yaml

  This sequence of commands created a Persistent Volume, a Persistent Volume Claim, a Deployment that manages a Pod running a container of a mysql docker image and a       Service that permits access to that Pod.

  Check if your kubernetes objects were successfully created with:

  _Deployment_

    kubectl get deployments
    
  ![image](https://github.com/BettShawn/MySQL-database-server-in-Kubernetes/assets/51289343/835f71d6-b7cf-47c1-a11a-cd9c32797077)

_Pod_

    kubectl get pods

  ![image](https://github.com/BettShawn/MySQL-database-server-in-Kubernetes/assets/51289343/8a586580-d0cb-46a4-962e-f14116ca06bc)

_Service_

    kubectl get services

  ![image](https://github.com/BettShawn/MySQL-database-server-in-Kubernetes/assets/51289343/9d9a4a43-fc4b-47c4-a6f0-0a0f58657337)

  We did great so far, now run a test to create a Pod running a MySQL container that connects to the MySQL database server Pod as a client;

    kubectl run -it --rm --image=mysql:8.0 --restart=Never mysql-client -- mysql -h mysql --password="password"

  ![image](https://github.com/BettShawn/MySQL-database-server-in-Kubernetes/assets/51289343/dac2e309-20d5-4a36-9445-4d15b8721664)

  This command runs the MySQL container in an interactive mode, which allows you to execute commands at the time of running the container.

  A MySQL shell will open and we could create new databases, new tables, insert data to tables and do more SQL commands.
  
  ![image](https://github.com/BettShawn/MySQL-database-server-in-Kubernetes/assets/51289343/b6d89f85-91a9-43d4-9bb6-a30b6044eb4e)

4. Conclusion
  With this, we learnt through Kubernetes Objects how to deploy a MySQL database server in a Kubernetes Cluster using a static method of provisioning storage.

  Also we tested how to connect a client to that deployed server, by executing SQL commands when running the container in interactive mode.

