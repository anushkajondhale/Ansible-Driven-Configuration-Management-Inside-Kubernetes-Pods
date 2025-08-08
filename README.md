🚀 Ansible Cluster in Docker Containers & Kubernetes Pods
🧭 Overview
This project demonstrates how to set up an Ansible cluster in two environments:

Docker Containers — Running Ansible Master and Nodes with SSH trust.

Kubernetes Pods — Deploying the same setup in a K8s cluster.

Key Highlights:

✅ Build a custom Docker image with Ansible & SSH

✅ Create Ansible Master and Nodes

✅ Establish SSH trust for passwordless automation

✅ Deploy identical setup on Kubernetes

✅ Verify using ansible -m ping

🔹 Phase 1: Ansible Cluster with Docker Containers
🛠 Step 1: Build a Custom Docker Image
Create a Dockerfile:

dockerfile
Copy code
FROM ubuntu:20.04
ENV DEBIAN_FRONTEND=noninteractive

RUN apt update && \
    apt install -y python3 python3-pip ssh sudo ansible iputils-ping && \
    apt clean

# Create ansible user
RUN useradd -m ansible && \
    echo "ansible:ansible" | chpasswd && \
    usermod -aG sudo ansible && \
    mkdir -p /home/ansible/.ssh && \
    chmod 700 /home/ansible/.ssh && \
    chown -R ansible:ansible /home/ansible/.ssh

# Configure SSH server
RUN mkdir /var/run/sshd && \
    echo "PermitRootLogin yes" >> /etc/ssh/sshd_config && \
    echo "PasswordAuthentication yes" >> /etc/ssh/sshd_config

EXPOSE 22
CMD ["/usr/sbin/sshd", "-D"]
🛠 Step 2: Build & Run Containers
bash
Copy code
docker build -t ansible-base .
docker run -dit --name master ansible-base
docker run -dit --name node1 ansible-base
docker run -dit --name node2 ansible-base
🔐 Step 3: Set Up SSH Trust (Master → Nodes)
bash
Copy code
docker exec -it master bash
su - ansible
ssh-keygen -t rsa
Find container IPs:

bash
Copy code
docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' node1
docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' node2
Copy SSH keys:

bash
Copy code
ssh-copy-id ansible@<node1-ip>
ssh-copy-id ansible@<node2-ip>
📁 Step 4: Configure Ansible Inventory & Test
Create inventory.ini:

ini
Copy code
[nodes]
node1 ansible_host=<node1-ip> ansible_user=ansible
node2 ansible_host=<node2-ip> ansible_user=ansible
Run ping test:

bash
Copy code
ansible -i inventory.ini nodes -m ping
☸️ Phase 2: Ansible Cluster in Kubernetes Pods
🛠 Step 1: Push Docker Image to Docker Hub
bash
Copy code
docker tag ansible-base <your-dockerhub-username>/ansible-k8s
docker push <your-dockerhub-username>/ansible-k8s
🧾 Step 2: Create Pod YAML Files
master.yaml:

yaml
Copy code
apiVersion: v1
kind: Pod
metadata:
  name: master
spec:
  containers:
    - name: master
      image: <your-dockerhub-username>/ansible-k8s
      ports:
        - containerPort: 22
      securityContext:
        privileged: true
Create similar YAMLs for node1.yaml and node2.yaml, changing only metadata.name.

Apply:

bash
Copy code
kubectl apply -f master.yaml
kubectl apply -f node1.yaml
kubectl apply -f node2.yaml
📡 Step 3: SSH Trust Between Pods
bash
Copy code
kubectl exec -it master -- bash
su - ansible
ssh-keygen -t rsa
List pod IPs:

bash
Copy code
kubectl get pods -o wide
Copy SSH keys:

bash
Copy code
ssh-copy-id ansible@<node1-pod-ip>
ssh-copy-id ansible@<node2-pod-ip>
📁 Step 4: Inventory & Ping in Kubernetes
inventory.ini:

ini
Copy code
[nodes]
node1 ansible_host=<node1-pod-ip> ansible_user=ansible
node2 ansible_host=<node2-pod-ip> ansible_user=ansible
Test:

bash
Copy code
ansible -i inventory.ini nodes -m ping
✅ Final Result
🎉 You now have a fully functional Ansible Cluster:

🐳 Inside Docker containers

☸️ Inside Kubernetes pods

🔧 Built from scratch with manual configuration

⚡ Ready for automated deployments & orchestration

