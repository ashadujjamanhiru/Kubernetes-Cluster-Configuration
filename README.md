# Kubernetes-Cluster-Configuration
# Kubernetes Cluster Master-and-Worker Node on Ubuntu-20.4 #
# Describe Kubernetes Cluster Master and Worker Node, First off all we create a Two ec2 instances One for Master Node and One [1] Worker Node #

# Update ec2 Machine*

    apt-get update -y

# Set Hostname your Host Machine *

    sudo hostnamectl set-hostname kubernetes-master

# Disable Swap Memory #

    swapoff -a
   
# Check It's Disable or not #
   
    free -m
   
# Install Docker Engine on your Host Machine #  

    apt install docker -y
    apt install docker.io -y
  
# Added a jason file in Docker Directory # vi /etc/docker/daemon.json

{
"exec-opts": ["native.cgroupdriver=systemd"]
}

# Reload Deamon #

    sudo systemctl daemon-reload

# Start and Enable Docker Engine #

    systemctl start docker && systemctl enable docker

# Check Status Docker Service #

    systemctl status docker

# Now Install Kubernetes

    sudo apt install apt-transport-https curl
    
    curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add
    
    sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
    
    sudo apt install -y kubeadm kubelet kubectl kubernetes-cni

# Enable Kubelet

    systemctl enable kubelet
  
# Note: This Step Follow All Two Machine, Master and WorkerNode1 #



# Task to do in only Master node #

    kubeadm init --pod-network-cidr=10.244.0.0/16

    mkdir -p $HOME/.kube
    
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    
    sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Deploy a pod network #

    kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
  
# Check All pods #

    kubectl get pods -A
  
 # Now Join Kubernetes cluster
# Run these from the worker node only: Example

kubeadm join 192.168.2.121:6443 --token p8r1a2.msnjnqjh36ft443w \
      --discovery-token-ca-cert-hash sha256:33e34bfcbe5d0e1a9a58757e1534eb90ef1076625e9d15c652425d921e4a8e91

# Check Node Ready or Not Ready #

    kubectl get node
  
  


# Deploy Nginx in kubernetes #

    vi nginx-deployment.yml
  
# Copy and Paste yml file #

apiVersion: apps/v1 #for k8s versions before 1.9.0 use apps/v1beta2  and before 1.8.0 use extensions/v1beta1
kind: Deployment
metadata:
  name: frontend
spec:
  selector:
    matchLabels:
      app: guestbook
      tier: frontend
  replicas: 3
  template:
    metadata:
      labels:
        app: guestbook
        tier: frontend
    spec:
      containers:
      - name: php-redis
        image: nginx
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
        env:
        - name: GET_HOSTS_FROM
          value: dns
          #If your cluster config does not include a dns service, then to
          #instead access environment variables to find service host
          #info, comment out the 'value: dns' line above, and uncomment the
          #line below:
          #value: env
        ports:
        - containerPort: 80


# Now Deploy Nginx #

    kubectl apply -f nginx-deployment.yml

# Again nginx service deploy on kubernetes #

    vi service.yml

# Copy and Paste yml file #


apiVersion: v1
kind: Service
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  #comment or delete the following line if you want to use a LoadBalancer
  type: NodePort
  #if your cluster supports it, uncomment the following to automatically create
  #an external load-balanced IP for the frontend service.
  #type: LoadBalancer
  ports:
  - port: 80
  selector:
    app: guestbook
    tier: frontend


# Now Deploy Nginx service #

    kubectl apply -f service.yml

# Show service port #

    kubectl get services --all-namespaces




# Kubernetes kubeadm init fails due to dial tcp 127.0.0.1:10248: connect: connection refused
# Solve url

https://stackoverflow.com/questions/54728254/kubernetes-kubeadm-init-fails-due-to-dial-tcp-127-0-0-110248-connect-connecti?answertab=active#tab-top

***If RuN time Error: unknown service runtime.v1alpha2.RuntimeService
Solve Link:
https://kubernetes.io/docs/setup/production-environment/container-runtimes/



