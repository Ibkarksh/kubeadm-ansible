# Install K8S cluster using kubeadm tool 
## Structure
This reposiotry consists of 2 roles:
1. masters: for the masters installation
2. workers: for workers installation
## Prerequisites:
1. Remote user is a member of sudo group
```bash
usermod -aG sudo vagrant
```
2. Make sure the system doesn't ask for the password when sudo is used
```bash
sudo visudo
## Add the following line in the bottom of file
$USER ALL=(ALL) NOPASSWD: ALL
```
3. Vagrant with libvirt 
4. private ssh key to access all the nodes
```bash
ssh-keygen #procced with the requests by ENTER

# Alternativ to ssh-copy-id by adding the public key to authorized_keys
# You can add the content of public key into Vagrantfile
  cat << EOF >> /home/vagrant/.ssh/authorized_keys
# Add here the public key
EOF
```
5. Choose the required  used subnet & number of machines by changing the variable *myVMs*:
## hosts
> You have to enter a record per node in the hosts file and specify the group either masters or workers
```bash
[etcd]
[haproxy-keepalive]
[masters]
master-01 ansible_host=192.168.160.5  private_ip=192.168.160.5
[workers]
worker-01 ansible_host=192.168.160.3  private_ip=192.168.160.3
worker-02 ansible_host=192.168.160.4  private_ip=192.168.160.4
```
## ansible.cfg
It contains the configurations of the playbook
```bash
[defaults]
# Location of inventory file which simulates hosts file on the scale of playbbok
inventory=./hosts
# The remote user which you can access the nodes through it
remote_user=vagrant
# Use private ssh key & the public key exists on all nodes in authorized_hosts file 
ask_pass = False
private_key_file=./id_rsa
# Increse the timeout if network I/O is limited
timeout=60	
[privilege_escalation]
## Add ubuntu user to sudo group 
become=True
become_method=sudo
#become_user=root
```
## Installation Playbook
```bash
ansible-playbook -i hosts main.yaml
```
## Installation Nginx-Ingress using helm chart
```bash
helm repo add nginx-stable https://helm.nginx.com/stable
helm repo update
helm install ingress-controller nginx-stable/nginx-ingress --set controller.kind=daemonset --namespace ingress-nginx --create-namespace
```
## Deploying Juice shop workload
You can find under deployments/ a manifest file which will help you in the installation of:
1. Deployment of 2 replicas
2. Service to expose the deployment
3. Ingress Object to expose the deployment
```bash
#Apply this command on master node
kubectl apply -f deployments/juice-shop.yaml 
```
---
## Notes:
### Important Variables
> This Playbbok is tested for Almalinux/8
- VIP: The virtual IP wich will be floating between masters. In case of 1 master it can be the ip of master
- sources: The allowed subnet to communicate with Kube-API server
- ifname: The interface name which will used for communication between nodes

