Access to Kubernetes Dashboard
==============================

# How to give cluster role to a single user

  1. delete current kubernetes dashboard
      > kubebel delete -f kubernetes-dashboard.yaml

  2. re run to create new dashboard container
      > kubectl create -f kubernetes-dashboard.yaml

  3. execute a command in below for authorization

      > kubectl create rolebinding serviceaccounts-view \
        --clusterrole=view \
        --group=system:serviceaccounts:kube-system \
        --namespace=kube-system

# Meaning of the last command
  : will grant a role to the service acocunt group for that namespace 
    1. Define the namespace for this Grant
    2. Define the name of serviceaccounts
    3. rolebinding (serviceaccounts-view)

    Finally, Grant a role to all service accounts in a namespace


