## 0. Environment for testing 
  - OS: Ubuntu16.04
  - Ansible: >2.4.0.0
  - Template: >2.9
  - Python-netaddr should be installed
  ### Accessing through ssh is required to make a secure channel to every target nodes from local or a dedicated host.

   * VM naming rule for creation

     - Master: [account]-k8s-master-[#n]

     - Worker: [account]-k8s-worker-[#n]

     - RR: [account]-k8s-rr[#n]

   * how to exchange key to access without from local to VMs

     i.g) for i in {1..3};

            do ssh-copy-id -i deploy@[account]-k8s-master-[#n].pg1.krane.9rum.cc;

            done;

     i.g) ssh-copy-id deploy@kmaster.pg1.krane.9rum.cc 
   
   * can delegate the role of deployment to a specific vm
   
## 1. Do not set any proxy settings on all hosts
   - left it without adding any proxy options at /etc/environment, .profile, bashrc
     just use bare VM itself
     
## 2. Set proxy-value at a few files in below
   - inventory/group_vars/all.yaml (go to line 96)
   
     http_proxy: "http://proxy.daumkakao.io:3128"
     
     https_proxy: "http://proxy.daumkakao.io:3128"
     
   - roles/kubespray-defaults/defaults/main.yaml (go to line 193)
   
     appends every IPs of all hosts at the end of no_proxy
     
     (i.g, 127.0.0.1,localhost,10.195.22.225~~~~~~)
     
     replace value("none") to operating system what you wish to run (i.g, bootstrap_os: ubuntu)

   - inventory/group_vars/all.yml (go to line 54)

     "upstream_dns_servers:

      - 10.20.30.40"

## 3. Add a single task to enable swapoff option at the end of file.
   - roles/bootstrap-os/tasks/bootstrap-ubuntu.yaml
     
     "- name: swapoff
     
        action: command swapoff -a "

## 4. Internal loadbalancers for apiservers
   - go to line 30 to comment out
     if high availability is required
     Let's enable internal load balancer

     set to true
     "loadbalancer_apiserver_localhost: true"

## 5. Make a file("inventory/inventory.cfg") with directives in below

    [all]
    
    [kube-master]
    
    [etcd]
    
    [kube-node]
    
    [k8s-cluster:children]
    
    [calico-rr]
    
    [rack0]
    
    [rack0:vars]
    cluster_id="1.0.0.1"


    *** Example ***
    
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
    rr

    k8s-master1
    
    k8s-master2
    
    k8s-master3
    
    k8s-worker1
    
    k8s-worker2
    
    k8s-worker3

    [rack0:vars]
    cluster_id="1.0.0.1"
    
    =================================================================


## Use two option in below to tear down
  
  - substitue cluster.yml for reset.yml and do it again.
  
  - delete whole vm...and recreate it again. (do a step #0)
  
## Enjoy~ Kubernetes where The Mine is inside in

## Example of commands
  - ansible-playbook -i inventory/inventory.cfg cluster.yml -b -v --ask-vault-pass -u deploy --flush-cache
  
  - ansible-playbook -i inventory/inventory.cfg reset.yml -b -v --ask-vault-pass -u deploy --flush-cache  
  
  ### You may want to add **worker** nodes to your existing cluster
  
  - ansible-playbook -i inventory/inventory.cfg scale.yml -b -v --ask-vault-pass -u deploy --flush-cache
  
    (remember! add new node spec into inventory.cfg...just add without changing anything!, you can see text message while scrolling)
    
    TASK [kubernetes/node : include] 
    
    *****************************************************************************************************
    
    Thursday 21 December 2017  11:48:19 +0900 (0:00:00.078)       0:04:59.967 *****
    
    included: /Users/ted.j/Kakaowork/Ted-K8S/kubespray/roles/kubernetes/node/tasks/install_host.yml for k8s-worker1, k8s-worker2, k8s-worker3, k8s-worker4
    
