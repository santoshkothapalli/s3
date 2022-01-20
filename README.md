# How to create a Kubernetes Cluster on GCP and Deploy the Jenkins containers...

Step 0 - Create a Google Kubernetes Cluster from Console
- Search for Kubernetes Cluster in the Google Console
- Click on Create to create a new Cluster 
- Select GKE Standard 
- Provide the name of the cluster , region , version of the GKE
- Select Node Pool and provide the number of nodes to required in the cluster
- Leave other parameters as default and create the clutser..

Step 1 - 

Once the cluster is created successfully in the console ,  Open the Cloud shell and try to connect to the clutser

gcloud container clusters get-credentials <<Name of the clutsre>> --zone <<Zone>> --project <<Name of the project>>

- Once the authentication is successful 
Check if the nodes are created accordingly.

- Use the following command to set shortcut 
alias k=kubectl
k get no -o wide

The should list the nodes like below

AME                                       STATUS   ROLES    AGE    VERSION            INTERNAL-IP   EXTERNAL-IP     OS-IMAGE                             KERNEL-VERSION   CONTAINER-RUNTIME
gke-cluster-3-default-pool-c88e7214-797v   Ready    <none>   157m   v1.21.5-gke.1302   10.128.0.1   34.68.0.0     Container-Optimized OS from Google   5.4.144+         containerd://1.4.8
gke-cluster-3-default-pool-c88e7214-phrm   Ready    <none>   157m   v1.21.5-gke.1302   10.128.0.2   34.68.0.1     Container-Optimized OS from Google   5.4.144+         containerd://1.4.8
gke-cluster-3-default-pool-c88e7214-zznn   Ready    <none>   157m   v1.21.5-gke.1302   10.128.0.3   34.68.0.2     Container-Optimized OS from Google   5.4.144+         containerd://1.4.8


Step 2 — Installing Jenkins on Kubernetes
- There are 3 steps to be performed to set up Jenkins on the Kubernetes Container successfully

2.1. Create a Deployment in a specific name (e.g. have used namespace as jenkins)
Create a file called jenkins.yaml with the below content 

This creates a deployment with one replica , uses the image jenkins/jenkis:lts and exposes the container on ports 8080 and 50000
Inorder to store the pipelines that are created it needs to mount the volume at /var/jenkins_vol. The selector app=jenkins will be used for creating the services.

apiVersion: apps/v1
kind: Deployment
metadata:
  name: jenkins
spec:
  replicas: 1
  selector:
    matchLabels:
      app: jenkins
  strategy: {}
  template:
    metadata:
      labels:
        app: jenkins
    spec:
      containers:
      - image: jenkins/jenkins:lts
        name: jenkins
        ports:
        - name: http-port
          containerPort: 8080
        volumeMounts:
        - name: jenkins-vol
          mountPath: /var/jenkins_vol
        resources: {}
      volumes:
      - name: jenkins-vol
        emptyDir: {}
        
Command used for Deployment:
k apply -f jenkins.yaml

Once the command execution is successful validate the deployment 
k get deploy jenkins -n jenkins 

NAME      READY   UP-TO-DATE   AVAILABLE   AGE
jenkins   1/1     1            1           158m

Also validate the pods created
k get po -n jenkins

NAME                       READY   STATUS    RESTARTS   AGE
jenkins-794699f9bc-dltqx   1/1     Running   0          158m


2.2 Create a Service of Type NodePort
- Create a file called jenkins-service.yaml with the below content 

apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  name: jenkins
  namespace: jenkins
spec:
  type: NodePort
  ports:
  - port: 8080
    targetPort: 8080
    nodePort: 30000
  selector:
    app: jenkins
   
Command used for Creating Service:
k apply -f jenkins-service.yaml

Once the command execution is successful validate the services created
NAME      TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
jenkins   NodePort   10.48.20.113   <none>        8080:30000/TCP   151m

Cluster IP is the IP address that can be used internally to connect , as you can see internally service is exposed on 8080 and externally on port 30000

2.3. Lets validate the below if Jenkins in deployed on our cluster successfully..

- Retrieve the name of the pod 
- k get po -n jenkins
NAME                       READY   STATUS    RESTARTS   AGE
jenkins-794699f9bc-dltqx   1/1     Running   0          163m
- Pass the name to get the logs from the conatiner

k logs jenkins-794699f9bc-dltqx -n jenkins

If everything is successful , you be should be able to see the below message..

************************************************************
*************************************************************
*************************************************************

Jenkins initial setup is required. An admin user has been created and a password generated.

Please use the following password to proceed to installation:

367ad298ce4443a5b351b4adab52dd

This may also be found at: /var/jenkins_home/secrets/initialAdminPassword

*************************************************************
*************************************************************
*************************************************************

Lets Log into the jenkins container to get the initialAdminPassword..

Command : 
k exec -it jenkins-794699f9bc-dltqx -n jenkins -- /bin/sh
#cat /var/jenkins_home/secrets/initialAdminPassword

Take the External IP Address of any of the nodes from the Step 1 and try to access the Jenkins web console

http://34.68.0.0:30000

In case you are not able to open , check the Firewall rules of your cluster and modify accordingly to access port 30000.










   

