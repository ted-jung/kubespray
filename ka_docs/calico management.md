Calico Management
=====================

# There are three options for Calico network configuration.
  Route Reflector(hence RR), which is one of options for container networking using Calico.

  It is configured well through Kubespray with just one RR.

  As cluster grows highly according to the grow of Service, 
  then we need to set another RR that has another new worker nodes.
  
  Pl, check the status of peering between nodes after put new workers into current cluster.

# Pre condition
  ## Status of peering option
    full mesh(peering between nodes) should not be enabled on.
    > calicoctl config set nodeToNodeMesh off
    
    > calicoctl config get nodeToNodeMesh
      off

  ## AS number
    > calicoctl config get asNumber
      64512 (this is default in kubespray)

# How to check the running status of calico

  ## Master node
    > calicoctl node status
     => you should see the peered RR right after the command.
        IPv4 BGP status
        +--------------+---------------+-------+----------+-------------+
        | PEER ADDRESS |   PEER TYPE   | STATE |  SINCE   |    INFO     |
        +--------------+---------------+-------+----------+-------------+
        | 10.175.22.54 | node specific | up    | 05:21:15 | Established |
        | 10.175.22.47 | node specific | up    | 05:22:38 | Established |
        +--------------+---------------+-------+----------+-------------+

  ## Worker node
    > calicoctl node status
     => the same result as master node


  ## If rows are not the exact number of RR, then do below command to fill up or fix it.

    >  lets suppose, there two RR.
      RR1. 10.175.22.54
      RR2. 10.175.22.47

   Note> Pl, do below lines per on each node(masters, workers).
   
         replace the peerIP with IP of RR as much the number of RR.
         
         replace the name with every workers/master
         
    >  cat << EOF | calicoctl create -f -
        apiVersion: v1
        kind: bgpPeer
        metadata:
          peerIP: ${10.175.22.47} <- peerIP
          scope: node
          node: ks-master1 <- name of worker/master
        spec:
          asNumber: 64512
        EOF
      
  ###  Wonderful beginning ~~
  ###  Checking the status of the BGP peers
    > sudo calicoctl node status
        +--------------+---------------+-------+----------+-------------+
        | PEER ADDRESS |   PEER TYPE   | STATE |  SINCE   |    INFO     |
        +--------------+---------------+-------+----------+-------------+
        | 10.175.22.54 | node specific | up    | 05:21:15 | Established |
        | 10.175.22.47 | node specific | up    | 05:22:38 | Established |
        +--------------+---------------+-------+----------+-------------+
    
    > calicoctl get bgpPeer --node=ks2-worker1
        SCOPE   PEERIP         NODE          ASN
        node    10.175.22.47   ks2-worker1   64512
        node    10.175.22.54   ks2-worker1   64512


  ## Extra Command
    if some node whichever do not required is listed up, then delete it.
    > calicoctl delete bgpPeer aa:bb::ff --scope=node --node=node1
