1. Do not set any proxy settings to all hosts
   - /etc/environment, .profile, bashrc
   - just use bare VM environment

2. Set value of proxy to a few files in below
   - inventory/group_vars/all.yaml (go to line 96)
     http_proxy: "http://proxy.daumkakao.io:3128"
     https_proxy: "http://proxy.daumkakao.io:3128"
     
   - roles/kubespray-defaults/defaults/main.yaml (go to line 193)
     appends every IPs of all hosts at the end of no_proxy
     (i.g, 127.0.0.1,localhost,10.195.22.225~~~~~~)
     replace none value to one of OS (i.g, bootstrap_os: ubuntu)

3. Add a single task at the end
   - roles/bootstrap-os/tasks/bootstrap-ubuntu.yaml
     
     "- name: swapoff
        action: command swapoff -a "

4. make a file("inventory.cfg") with directives
    [all]
    [kube-master]
    [etcd]
    [kube-node]
    [k8s-cluster:children]
    [calico-rr]
    [rack0]
    [rack0:vars]
    cluster_id="1.0.0.1"

    Example)
    Three masters,
    Three etcds on the same server on where masters are running
    Three workers
    One route reflector
    Logical group
    
    =================================================================
    
    [all]
    k8s-master1 ansible_ssh_host=10.195.22.197 ip=10.195.22.197
    k8s-master2 ansible_ssh_host=10.195.22.199 ip=10.195.22.199
    k8s-master3 ansible_ssh_host=10.195.22.204 ip=10.195.22.204
    k8s-worker1 ansible_ssh_host=10.195.22.205 ip=10.195.22.205
    k8s-worker2 ansible_ssh_host=10.195.22.210 ip=10.195.22.210
    k8s-worker3 ansible_ssh_host=10.195.22.212 ip=10.195.22.212
    rr ansible_ssh_host=10.195.22.225 ip=10.195.22.225

    [kube-master]
    k8s-master1
    k8s-master2
    k8s-master3

    [etcd]
    k8s-master1
    k8s-master2
    k8s-master3

    [kube-node]
    k8s-worker1
    k8s-worker2
    k8s-worker3

    [k8s-cluster:children]
    kube-node
    kube-master

    [calico-rr]
    rr

    [rack0]
    k8s-master1
    k8s-master2
    k8s-master3
    rr
    k8s-worker1
    k8s-worker2
    k8s-worker3

    [rack0:vars]
    cluster_id="1.0.0.1"
    =================================================================

* Enjoy Kubernetes
