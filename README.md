k8s-master
=========

An ansible role to configure a kubernetes master on a Red Hat Enterprise Linux based system.

Requirements
------------

N/A

Role Variables
--------------

###Defaults:
####kube-apiserver settings
* k8s_apiserver_listen_ip: 0.0.0.0
* k8s_apiserver_port: ''                   #default 8080 and commented out in conf
* k8s_kubelet_port: ''                     #default 10250 and commented out in conf
* k8s_etcd_urls: 'http://kubernetes.local:2379' #can be comma separated list if clustered etcd setup
* k8s_service_network: 192.168.22.0/24
* k8s_admission_control: 'NamespaceLifecycle,NamespaceExists,LimitRanger,SecurityContextDeny,ResourceQuota'

####Optional advanced apiserver settings
* k8s_auth_mode: ''
* k8s_auth_policy_file: ''
* k8s_token_auth_file: ''
* k8s_apiserver_secure_port: '0'
* k8s_tls_cert_file: ''
* k8s_tls_private_key_file: ''
* k8s_client_ca_file: ''
* k8s_apiserver_additional_args: ''        #pass more apiserver switches here as a string
* k8s_controller_manager_additional_args: ''
* k8s_kube_proxy_additional_args: ''
* k8s_scheduler_additional_args: ''

####etcd/flannel network settings
* etcd_server_url: http://0.0.0.0
* etcd_port: 2379
* etcd_key: /kube01/network
* flannel_backend_network: 172.16.0.0/12
* flannel_subnet_length: '24' #applies to kubernetes node backend subnet length

####kubernetes general settings
* k8s_master_url: http://kubernetes.local
* k8s_master_port: 8080
* k8s_allow_privileged: false
* k8s_log_level: 0
* k8s_logtostderr: true

####install cockpit and kubernetes plugin?  
****Please note, this will rip out the default RHEL cockpit to be able to install the cockpit-kubernetes package and dependencies from the CentOS7 Extras repository****
* k8s_cockpit: true

####is the master also a node/minion?
* k8s_mst_is_node: false

If you'd like to run a kubernetes node on your master, add the k8s-node role.
The only extra variable you'll probably have to define is the kubelet_hostname_override: localhost and switch k8s_mst_is_node: true.

###Variables:
* k8s_mst_packages:
    - curl
    - etcd
    - kubernetes-master
    - kubernetes-node
    - flannel

* cockpit_kubernetes_pkg: 
    - cockpit-kubernetes
    - cockpit

####For ripping and replacing RHEL cockpit due to dependencies
* cockpit_default:
    - cockpit-shell
    - cockpit-bridge
    - cockpit-ws
    - cockpit
    - cockpit-networkmanager
    - cockpit-storaged
    - cockpit-docker


Dependencies
------------

N/A

Example Playbooks
----------------
Bare minimum, where 192.168.122.20 is the IP address of the desired master server.

    - hosts: kubernetes_master
      vars:
        k8s_etcd_urls: http://192.168.122.20:2379
        k8s_master_url: http://192.168.122.20
      roles:
         - k8s-master

Bare minimum while running a node on the master as well (applied k8s-node role) where 192.168.122.20 is the IP address of the desired master server.

    - hosts: kubernetes_master
      vars:
        k8s_etcd_urls: http://192.168.122.20:2379
        k8s_master_url: http://192.168.122.20
        k8s_mst_is_node: true
        #node settings 
        kubelet_hostname_override: localhost
      roles:
        - k8s-master
        - k8s-node


License
-------

MIT

Author Information
------------------

Andrew J. Huffman
