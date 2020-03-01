# OpenShift on AWS Installation

Pre-reqs
AWS account Created
Domain set up

This install will create a new VPC, NAT Gateway, Subnets, Route Tables, VPC DHCP Options and VPC Endpoints as part of the installation process.  There is an option to create an OS Cluster in a existing VPC.

Create key pair
```sh
ssh-keygen -t rsa -b 4096 -N '' -f <path>/<filename>
```
Start the ssh-agent process as a background task:
```sh
eval "$(ssh-agent -s)"

Agent pid 63417
```

Add your SSH private key to the ssh-agent:
```sh
ssh-add <path>/<file_name> 

Identity added: /home/<you>/<path>/<file_name> (<computer_name>)
```

Download install media from https://mirror.openshift.com/pub/openshift-v4/clients/ocp/4.3.3/

Download Image Pull secret from your RedHat account
https://cloud.redhat.com/openshift/install/pull-secret

Then run 

```sh
./openshift-install create install-config --dir=~/MyStuff/OS_AWS/demo
```

Complete the prompts as required.

A temp folder is created which has the terraform statefiles and also the install-config.yaml file which can be edited as needed (example specifiying existing VPC and network ranges if so required)

Create the cluster

```sh
./openshift-install create cluster --dir=/Users/sion/MyStuff/OS_AWS/install_config_files  --log-level=info

INFO Consuming Install Config from target directory 
INFO Creating infrastructure resources...         
INFO Waiting up to 30m0s for the Kubernetes API at https://api.openshift-demo.circumflexdevops.co.uk:6443... 
INFO API v1.16.2 up                               
INFO Waiting up to 30m0s for bootstrapping to complete... 
INFO Destroying the bootstrap resources...        
INFO Waiting up to 30m0s for the cluster at https://api.openshift-demo.circumflexdevops.co.uk:6443 to initialize... 
INFO Waiting up to 10m0s for the openshift-console route to be created... 
INFO Install complete!                            
INFO To access the cluster as the system:admin user when using 'oc', run 'export KUBECONFIG=/Users/sion/MyStuff/OS_AWS/install_config_files/auth/kubeconfig' 
INFO Access the OpenShift web-console here: https://console-openshift-console.apps.openshift-demo.circumflexdevops.co.uk 
INFO Login to the console with user: kubeadmin, password:

```

to destroy a cluster run

```sh
$ ./openshift-install destroy cluster \
--dir=<installation_directory> --log-level=info
```