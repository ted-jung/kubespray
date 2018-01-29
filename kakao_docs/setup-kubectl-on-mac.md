Install and Set Up kubectl on macOS
====================================

# Install kubectl
  ## If you are on macOS and using Homebrew package manager, you can install with:
    > brew update
    > brew install kubectl
    
  ## Copy config from master node.
    > scp root@{$YOUR_SERVER}:/root/.kube/config ~/.kube/config

  ## Make sure your kubectl is working.
    > kubectl config current-context
    admin-cluster.local

Reference : https://kubernetes.io/docs/tasks/tools/install-kubectl/