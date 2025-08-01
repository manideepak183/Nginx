This Repository includes installation of Kuberenetes from scratch using WSL ubuntu distro and deploying a sample nginx application and accessing the application on your local endpoint.

Prerequisites needed for this K8's installation & deployment. 
=
1. Windows10/11 compatible to install WSL.
2. internet connectivity to download all binaries, packages and dependancies.
3. sudo access to your Windoes machine
4. make sure you have atleast 4 CPU and 8GB RAM 

High level steps for deployment.
=
1. Installing Ubuntu distro on your windows using WSL
2. Setup docker and other pre-req's on your ubuntu.
3. Setup multi-node kubernetes cluster using KIND type.
4. install add-ons like calico for networking and a nginx ingress controller.
5. Setup your sample application with deployments, services, ingress.




Installation of Ubuntu distro using WSL(If you already have Ubuntu instance, you can skip this Ubuntu setup)
=
1. Download WSl2 on your windows machine.
2. Reboot your system
3. open CMD and run wsl --list --online
4. choose any OS distribution according to your requirement. Here we choose Ubuntu-22.04.
5. install ubuntu distribution using "wsl --install Ubuntu-22.04"
6. create your user and pass along with the prompt for your linux machine.
7. after logging in to your ububtu VM, you can check resources from "free -h" "lscpu"




Pre-requisites for your K8s cluster.
=
1. Install docker with below command.
     sudo apt update
     sudo apt install -y docker.io
     sudo usermod -aG docker $USER
     newgrp docker
2. Enable and restart your docker
     sudo systemctl enable docker
     sudo service docker start
3. After installing your docker, you can run basic docker commands to check if everything is working.
     docker ps -a
     docker version
4. try pulling small busybox image from internet.
     docker pull busybox.
5. If you are not able to pull, might be issue with your DNS server.
   add these 2 lines to your /etc/resolv.conf (entries for google and cloudflare DNS servers)
     nameserver 8.8.8.8
     nameserver 1.1.1.1
7. try pulling again, it should pull succesfully.
8. try running a sample debug container with below command to verify docker installation.
     docker run busybox
9. Install helm as well in your machine.
     curl -O https://get.helm.sh/helm-v3.16.2-linux-amd64.tar.gz
     tar -xvzf helm-v3.16.2-linux-amd64.tar.gz
     cd linux-amd64/
     chmod +x helm
     mv helm /usr/local/bin/.




Setup multi-node kubernetes cluster using KIND, Kubernetes in docker.
=
1. Here we are installing Kubernetes cluster on our ubuntu using Kind.
2. Download latest Kind availiable release from internet to your machine.
     curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.23.0/kind-linux-amd64
     chmod +x ./kind
     Here we are moving binary to PATH to recognise binary globally on your VM.
     sudo mv ./kind /usr/local/bin/kind 
     kind version
4. install kubectl of required version.
     curl -LO "https://dl.k8s.io/release/v1.31.0/bin/linux/amd64/kubectl"
     chmod +x kubectl
     sudo mv kubectl /usr/local/bin/
     kubectl version
5. Now we are good to create multi node cluster.
6. create a manifest for cluster with nodes like below content as kind-cluster.yaml(manifest present in Kubernetes directory).
7. Now run below comamnd to create a kind cluster
     kind create cluster --name mani --config kind-cluster.yaml
8. It takes some time to download latest node images. once done you can check your cluster.
     kubectl get nodes -owide
9. This will create 3 docker containers in your VM.
    manideepak@ANAPARTM01:~/WSL$ docker ps -a
    CONTAINER ID   IMAGE                  COMMAND                  CREATED         STATUS         PORTS                       NAMES
    a7e473dec6a8   kindest/node:v1.30.0   "/usr/local/bin/entr…"   2 minutes ago   Up 2 minutes   127.0.0.1:39913->6443/tcp   mani-control-plane
    35f6bfc4092c   kindest/node:v1.30.0   "/usr/local/bin/entr…"   2 minutes ago   Up 2 minutes                               mani-worker
    d26404f512b4   kindest/node:v1.30.0   "/usr/local/bin/entr…"   2 minutes ago   Up 2 minutes                               mani-worker2
10. You can execute into container shell to verify containers.
    docker exec -it mani-control-plane cat /etc/os-release
11. By this your Cluster installation is completed.




Install add-ons like calico for networking and a nginx ingress controller.
=
1. Calico is used as network plugin here for advanced routed using network policies, BG Protocols.
2. apply manifest present for Calico.
3. wait for calico controller and calico pods to up.
       kubectl get po -n kube-system -w
4. Now that Calico is installed, we can proceed with nginx controller deployment.
5. apply manifest file for Nginx controller.
       kubectl apply -f nginx-controller.yaml
6. if nginx controller pod is in pending state due to node selctors. label any of master or worker node with below label.
        kubectl label node mani-control-plane ingress-ready=true




Deploying sample static Nginx app including postgres:
=
1. create a namespace for more security.
     kubectl create ns my-app
2. create my-app deployment and postgres stateful set
     kubectl apply -f my-app.yaml my-postgres-ss.yaml
3. expose these deployments with services, clusterIP service for deployment, and headless service for postgres.
     kubectl apply -f my-app-service.yaml my-postgres-service.yaml
4. Now create your Ingress pointing to my-app-service.
     kubectl apply -f my-ingress.yaml
5. The service created to access ingress controller is Node port. So out nginx application can be accessed from WSL like below.
   here 31487 is the Node port exposed and 172.18.0.3 is the IP of master node. 
    curl -H "Host: demo.local" http://172.18.0.3:31487
6. If you want to access it from your local browser following steps should be followed.
7. add below entry of "my-app-local" into your Windows hosts file.
     location: C:\Windows\System32\drivers\etc
     ENTRY: 127.0.0.1 my-app-local
    <img width="499" height="119" alt="image" src="https://github.com/user-attachments/assets/31d86ec7-a68a-45c7-a67f-a82a664c6aae" />
9. run below port-forward command to expose your svc to external world.
    kubectl port-forward -n ingress-nginx svc/ingress-nginx-controller 8080:80
    <img width="641" height="280" alt="image" src="https://github.com/user-attachments/assets/22dfb763-d066-401b-bdaf-d65ec4ed03c8" />

     
    
       



     

   

   







