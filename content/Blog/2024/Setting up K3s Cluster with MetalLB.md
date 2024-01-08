---
notetype: Blog
title: Setting up K3s Cluster with MetalLB
date: 2024-01-08
tags:
  - output/blog
  - Tech/Network
---
Up::[[Engineering MOC]]
Tags:: #Tech/Web 

Hence I added more services into my homelab, I  need a more elegant way to manage those server and services.So,Rebuild my environment from bare VM into a k8s cluster driven one.
I am going to post a series to explain how achieved it.

![image](https://github.com/bintis/xirin/assets/57840704/4583fe4f-bd8b-484f-a0f7-b259fefe7736)



## **Prelude**

In this section, I will show how to build a cluster with the following specifications:

- **Quantity of nodes:** 3
- **Web UI:** Rancher
- **Load Balancer:** Metallb
- **Public Access:** Cloudflare Tunnel
- **Ingress Controller:** Nginx
- **Setup Tool:** k3sup
- **Base System:** Rocky Linux 9.2
- **Package Management:** Helm

## **Preparation:**

Firstly, assume you have three Rocky Linux 9.2 VMs installed in Proxmox. Create an admin user for each and set the IP addresses to static values to avoid IP changes after restarts.

|HostName|IP Address|User|
|---|---|---|
|Node1|10.1.1.101|admin|
|Node2|10.1.1.102|admin|
|Node3|10.1.1.103|admin|

**Before installing k3s, some preparations are necessary:**

1. **Turn Off the Firewall for Convenience:**
    
    Disabling the firewall can ease the installation process, but it's crucial to re-enable it later for security
    
    ```bash
    systemctl disable firewalld
    ```
    
    
2. **Turn Off SELINUX:**
    
    SELINUX can often interfere with installations and services running on the system.
    
    bashCopy code
    
    ```bash
    sudo sed -i -e "s/^SELINUX=enforcing$/SELINUX=disabled/g"    /etc/selinux/config  
    # After modification, reboot VM to confirm 
    getenforce
    ```
    
3. **Turn Off Swap:**
    
    Kubernetes usually requires swap to be turned off for better performance and stability.
    
    
    ```bash
    swapoff -a
    ```
    
4. **Generate SSH Key Pair:**
    
    This step is crucial for enabling SSH access without a password, simplifying node management.
    
    
    ```bash
    ssh-keygen
    ```
    
5. **Transfer Public Key to Servers:**
    
    Copy the public key to other servers for secure passwordless SSH connections.
    
    bashCopy code
    
    ```bash
    ssh-copy-id admin@10.1.1.102 ssh-copy-id admin@10.1.1.103
    ```
    
6. **Add Admin User to Auto Sudo in Every VM:**
    
    Granting admin users the ability to execute sudo commands without a password streamlines various administration tasks.
    
    - **Add User to the Wheel Group:**
        ```bash
        sudo usermod -aG wheel admin
        ```
        
    - **Verify the User's Group Membership:**
        
        ```bash
        groups admin
        ```
        
    - **Ensure that Wheel Group Has sudo Privileges:** This step is to ensure that the admin users can execute commands as superusers.

**Preparation is now complete.**

## **Installation Begin**

### **Install Node1:**

Set up the first node as the master node using k3sup.

```bash
export SERVER_IP=10.1.1.101 export USER=admin  k3sup install --ip $SERVER_IP --user $USER --no-extras --ssh-key /home/admin/.ssh/id_rsa
```

### **Join Node2 and Node3 to Node1:**

Extend the cluster by adding additional nodes.



```bash
# For Node2 
export AGENT_IP=10.1.1.101 
# Server's IP
export SERVER_IP=10.1.1.102  
# Repeat for Node3 with AGENT_IP=10.1.1.103
export USER=admin  k3sup join --ip $AGENT_IP --server-ip $SERVER_IP --user $USER --ssh-key /home/admin/.ssh/id_rsa  
```

### **Test Your Cluster:**

Verify that all nodes are connected and recognized by the cluster.

```bash
export KUBECONFIG=/home/admin/kubeconfig kubectl config use-context default kubectl get node -o wide
```

### **Install Metallb:**

Metallb will provide a network load balancer for your cluster.You loadbalancer service will obtain IP address same as your home network one.

```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.12/config/manifests/metallb-native.yaml
```

### **Configure Parameters:**

Create configuration files for Metallb to define the range of IP addresses it can use.



```bash
# Create IPAddressPool.yaml
cat << EOF > IPAddressPool.yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: pool
  namespace: metallb-system
spec:
  addresses:
  - 10.1.1.70-10.1.1.99
EOF

# Create metallb.yaml
cat << EOF > metallb.yaml
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: config
  namespace: metallb-system
EOF

kubectl apply -f metallb.yaml IPAddressPool.yaml

```

### **Install Nginx:**

Nginx will act as the ingress controller, managing access to your cluster's services.

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.7.1/deploy/static/provider/baremetal/deploy.yaml
```

### **Configure Helm & Install Rancher:**

Set up Helm, a package manager for Kubernetes, and use it to install Rancher for cluster management.


```
# Install Helm
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 && chmod 700 get_helm.sh && ./get_helm.sh

# Set KUBECONFIG
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml

# Add Helm repo for Rancher
helm repo add rancher-latest https://releases.rancher.com/server-charts/latest

# Create namespace for Rancher
kubectl create namespace cattle-system

# Install Rancher with Helm
helm install rancher rancher-latest/rancher --namespace cattle-system --set hostname=rancher.yourdomain.cc --set bootstrapPassword=YourPasswordHere --set ingress.tls.source=secret 

# Expose Rancher and Portainer via LoadBalancer
kubectl expose deployment rancher -n cattle-system --type=LoadBalancer --name=rancher-lb --port=443

```

### **Set Environment Variable for Future Use with kubectl and Helm:**

Add KUBECONFIG to your .bashrc for easy future access to your cluster.
```bash
echo 'export KUBECONFIG=/etc/rancher/k3s/k3s.yaml' >> ~/.bashrc
```



<a rel="license" href="http://creativecommons.org/licenses/by-nc-nd/4.0/"><img alt="クリエイティブ・コモンズ・ライセンス" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-nd/4.0/88x31.png" /></a><br />この 作品 は <a rel="license" href="http://creativecommons.org/licenses/by-nc-nd/4.0/">クリエイティブ・コモンズ 表示 - 非営利 - 改変禁止 4.0 国際 ライセンス</a>の下に提供されています。

@Bintis 著作权，不许抄。
