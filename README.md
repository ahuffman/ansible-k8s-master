# ahuffman.k8s-master

An ansible role to configure a kubernetes master on a Red Hat Enterprise Linux based system.

This role can deploy either a SSL Secured kubernetes master (kube-apiserver) or an insecured kubernetes master.  See example playbooks for the minimum required variables to deploy a cluster in a insecure or secure fashion.  It will also open the required system firewall ports with firewalld (default RHEL7/CentOS7).

## Requirements

If the kubernetes master will also be a cluster member (node/minion), you will want to make use of the [`k8s-node`](https://galaxy.ansible.com/ahuffman/k8s-node/) role.   
It is critical that you properly set the hostname and domain of the kubernetes master properly.  You may want to use the [`ahuffman.hosts`](https://galaxy.ansible.com/ahuffman/hosts/) role, or an alternative to accomplish this.


## Role Variables

### Defaults:
Found in [`defaults/main.yml`](defaults/main.yml)
#### k8s-master Role settings:
`k8s_cockpit`: true - Required if you'd like cockpit and the cockpit-kubernetes plugin to be installed   
`k8s_mst_is_node`: false - Change to `true` if you plan on making the master a cluster member (node/minion) as well.  You'll also need to make use of the [`k8s-node`](https://galaxy.ansible.com/ahuffman/k8s-node/) role to properly configure your node.   
`k8s_secure_master`: false - Change to `true` if you'd like to generate certificates and communicate over secured channels.   
`k8s_firewalld`: true - Whether or not to open the required firewall ports with firewalld.  If you're managing your system's firewall ports outside of this role, set to false.  
`k8s_docker_storage_setup`: false - Whether or not to configure docker's container storage pool.  See settings below.   
 

#### Etcd/flannel Network Settings:
`etcd_port`: 2379 - The port of your etcd server.  As mentioned above, you most likely won't need to change this setting.   
`etcd_key`: /kube01/network - The key where your cluster's network settings will be stored in etcd.  Change as needed per cluster.   
`flannel_backend_network`: 172.16.0.0/12 - The network for backend pod/kube-proxy communications.   
`flannel_subnet_length`: '24' - The kubernetes node backend subnet length (i.e. The size of network slices assigned to kubernetes nodes.)   

#### Kube-apiserver Settings:
`k8s_kubelet_port`: '10250' - The port on which kubernetes kubelets are expected to communicate with the apiserver.   
`k8s_service_network`: 192.168.22.0/24 - Modify this setting to define what range of IPs your deployed kubernetes services will serve on.   
`k8s_admission_control`: 'NamespaceLifecycle,NamespaceExists,LimitRanger,SecurityContextDeny,ServiceAccount,ResourceQuota' - Modify this if you wish to not use one of the default kubernetes admission controllers.   

#### Optional Advanced kube-apiserver Settings:
`k8s_auth_mode`: '' - Use this setting to apply one of the authorization modes to the apiserver's configuration.   
`k8s_auth_policy_file`: '' - The location of your authorization method's policy file.   
`k8s_token_auth_file`: '' - The locaion of your token authorization file.   
`k8s_apiserver_additional_args`: '' - Any additional kube-apiserver options you wish to apply to the kube-apiserver service.  See `man kube-apiserver` for all available options.   
`k8s_controller_manager_additional_args`: '' - Any additional kube-controller-manager options you wish to apply to the kube-controller-manager service.  See `man kube-controller-manager` for all available options.   
`k8s_kube_proxy_additional_args`: '' - Any additional kube-proxy options you wish to apply to the kube-proxy service.  See `man kube-proxy` for all available options.   
`k8s_scheduler_additional_args`: - Any additional kube-scheduler options you wish to apply to the kube-scheduler service.  See `man kube-scheduler` for all available options.   

#### Kubernetes General Settings:
`k8s_apiserver_insecure_port`: 8080 - The port where insecure communication with the kube-apiserver will take place.   
`k8s_allow_privileged`: false - Whether or not to allow the execution of privileged containers.   
`k8s_log_level`: 0 - The verbosity of kubernetes logs.   
`k8s_logtostderr`: true - Whether or not to log to standard error.   

#### Docker storage setup options:   
`k8s_docker_storage_disk`: '' - Used with the k8s_docker_storage_setup option above.  Provide an unformatted device such as '/dev/sdb'.  This assumes it is a clean server deployment.  If you've already started the docker engine, then you'll have to cleanup the default storage pool.   
`k8s_docker_storage_vg`: vg_docker - The volume group to use/create for docker storage.   
       `k8s_docker_storage_options`:    
         - AUTO_EXTEND_POOL = true   
         - POOL_AUTOEXTEND_THRESHOLD   
See 'man docker-storage-setup` for all available options.  You can add whichever best suit your environment, but the defaults here should work well for you.   

#### Secure kube-apiserver Settings: - only applies if `k8s_secure_master: true`.
`k8s_apiserver_secure_port`: 6443 - Port to server kube-apiserver secured communications.   
`k8s_apiserver_cert_path`: /etc/kubernetes/certs - Where to store the server's certificates and keys.   
`k8s_apiserver_ca_key_filename`: ca.key - What filename to name your CA key.   
`k8s_apiserver_ca_cert_filename`: ca.crt - What filename to name your CA certificate.   
`k8s_apiserver_server_key_filename`: server.key - What filename to name your kube-apiserver's key.   
`k8s_apiserver_server_csr_filename`: server.csr - What filename to name your kube-apiserver's certificate signing request.   
`k8s_apiserver_server_cert_filename`: server.cert - What filename to name your kube-apiserver's certificate.   
`k8s_proxy_kubeconfig_filename`: proxy.kubeconfig - What filename to name your kube-proxy service's kubeconfig.   
`k8s_scheduler_kubeconfig_filename`: scheduler.kubeconfig - What filename to name your kube-scheduler service's kubeconfig.   
`k8s_controller_manager_kubeconfig_filename`: controller-manager.kubeconfig - What filename to name your kube-controller-manager service's kubeconfig.   
`k8s_apiserver_dns_names`:
  - kubernetes
  - kubernetes.default
  - kubernetes.default.svc
  - kubernetes.default.svc.cluster.local

`k8s_apiserver_dns_names` is a list of the Subject Alternative Names to use during certificate generation.  You should override this variable at the `host_vars` level, and list any possible DNS name that the server will need to serve with.  It's recommended to append your additional server names to this list.   

`k8s_apiserver_additional_ips`: [] - List to add any additional IP address the kube-apiserver will serve on.  This will be used to create IP Subject Alternative Names during certificate generation.  By default the `k8s-master` role will make use of the `ansible_default_ipv4` IP Address, so only add additional addresses other than the `ansible_default_ipv4` address.

### Variables:
Found in [`vars/main.yml`](vars/main.yml)

     k8s_mst_packages:
       - etcd
       - kubernetes-master
       - kubernetes-node
       - flannel
       - openssl #for certificate generation

     cockpit_kubernetes_pkg: 
       - cockpit-kubernetes
       - cockpit

For ripping and replacing RHEL cockpit due to dependencies:

      cockpit_default:
        - cockpit-shell
        - cockpit-bridge
        - cockpit-ws
        - cockpit
        - cockpit-networkmanager
        - cockpit-storaged
        - cockpit-docker

For opening the appropriate firewall ports on the master server (based upon secure or insecure config).  As you'll see, these refer back to what is set in (or over-rided) [`defaults/main.yml`](defaults/main.yml).  
 
      k8s_firewall_ports_secure:
        - '{{ etcd_port }}/tcp' #etcd
        - '{{ k8s_apiserver_secure_port }}/tcp' #kube-apiserver
        - '{{ k8s_kubelet_port }}/tcp' #kubelet

      k8s_firewall_ports_insecure:
        - '{{ etcd_port }}/tcp' #etcd
        - '{{ k8s_apiserver_insecure_port }}/tcp' #kube-apiserver
        - '{{ k8s_kubelet_port }}/tcp' #kubelet


### Viewing your Kubernetes Cluster in Cockpit
If you set `k8s_cockpit: true` the Cockpit service and `cockpit-kubernetes` Cockpit plug-in will be installed on your master.   

You can access your Cockpit via a web-browser by visiting:  https://YOUR_MASTER_HOSTNAME:9090

Once you've logged into the Cockpit service, you can click on the "Cluster" tab at the top of the page to view information about your cluster.


Example Playbooks
----------------
### Insecured kubernetes master

    - hosts: kubernetes_master
      roles:
         - ahuffman.k8s-master

### Insecured kubernetes master, running a node on the master as well (applied [`k8s-node`](https://galaxy.ansible.com/ahuffman/k8s-node/) role)

    - hosts: kubernetes_master
      vars:
        k8s_mst_is_node: true
      roles:
        - ahuffman.k8s-master
    
    - hosts: kubernetes_master
      vars:
        k8s_docker_storage_setup: true
        k8s_docker_storage_disk: /dev/sdb
        k8s_master_fqdn: kubmst01.foobar.com
      roles:
        - ahuffman.k8s-node

### Secured kubernetes master

    - hosts: kubernetes_master
      vars:
        k8s_secure_master: true
        k8s_apiserver_dns_names:
          - kubernetes
          - kubernetes.default
          - kubernetes.default.svc
          - kubernetes.default.svc.cluster.local
          #Adding additional SANs
          - foobar01
          - foobar01.foobar.org
          - foobar
          - foobar.foobar.org
      roles:
        - ahuffman.k8s-master

### Secured kubernetes master, running a secured node on the master as well (applied [`k8s-node`](https://galaxy.ansible.com/ahuffman/k8s-node/) role) 

    - hosts: kubernetes_master
      vars:
        k8s_secure_master: true
        k8s_mst_is_node: true
        k8s_apiserver_dns_names:
          - kubernetes
          - kubernetes.default
          - kubernetes.default.svc
          - kubernetes.default.svc.cluster.local
          #Adding additional SANs
          - foobar01
          - foobar01.foobar.org
          - foobar
          - foobar.foobar.org
      roles:
        - ahuffman.k8s-master

    - hosts: kubernetes_master
      vars:
        #k8s-node settings see k8s-node - https://galaxy.ansible.com/ahuffman/k8s-node/ for more details on settings
        k8s_secure_node: true
        k8s_docker_storage_setup: true
        k8s_docker_storage_disk: /dev/sdb
        k8s_master_fqdn: kubmst01.foobar.com
      roles:
        - ahuffman.k8s-node


License
-------

[MIT](LICENSE)

Author Information
------------------

[Andrew J. Huffman](https://github.com/ahuffman)
