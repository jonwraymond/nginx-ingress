# Nginx Ingress Controller with Public IP 

Assume you have a bare metal Kubernetes cluster on prem. One of the challenges is to make the pod available to public network. Most common and easy solution is to use NodePort as well as upfront load balancer, which needs another layer of maintenance. Furthermore, as the nubmer of services grows, there will be performance issue related to the iptables filter constraints. 

This repo provides a solution to implement [Nnginx Ingress controller](https://github.com/nginxinc/kubernetes-ingress) with [Multus-CNI](https://github.com/intel/multus-cni) so a public IP will be attached to the Ingress directly. Please refer to above two modules for better understanding of the detail technologies involved. Here I focus on more about how to set it up. 

## Install Multus-CNI

$ git clone https://github.com/intel/multus-cni.git && cd multus-cni
$ kubectl apply -f ./images/multus-daemonset.yml

## Create a namespace 'nginx-ingress' and a service account
$ kubectl apply -f ns-and-sa.yaml

## Create a NetworkAttachmentDefinition in the 'nginx-ingress' namespace:
** Notes ** Please make necessary change to the 'namespace', NIC on host server 'master', ip 'ranges' and 'routes' before you apply it.

$ kubectl apply -f macvlan-ingress.yaml

## Create a secret with a TLS certificate and a key for the default server in Nginx 
$ kubectl apply -f default-server-secret.yaml

** Notes ** you can also create a secret in CLI with one holding a chain of certificates

$ kubectl create secret tls ingress-certificate --key yourkey.key --cert yourcert.pem -n nginx-ingress

## Create a config map for customizing Nginx configuration
$ kubectl apply -f nginx-config.yaml

## Configure rbac
$ kubectl apply -f rbac.yaml

## Create nginx ingress deployment, deployment/nginx-ingress.yaml
** Notes ** you will need to update the ip address, which will be the fixed ip to be assigned to Ingress and must be in the scope defined in macvlan-ingress.yaml

        k8s.v1.cni.cncf.io/networks: '[
          {
            "name": "macvlan-ingress",
            "ips": "10.1.0.101"  
          }
        ]'

$ kubectl apply -f nginx-ingress.yaml

## You are good to go!!! 

## Ingress for an application 
** Notes ** please update the doname name 'myapp.mydomain.com' and service name 'myapp' accordingly

$ kubectl apply -f myapp-ingress.yaml
