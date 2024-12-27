# Rika's crazy kubernetes cluster
## why
space intentionally left blank

## base setup
OS: Rocky
- install the tools (kubelet, kubeadm, kubectl)
- create a wireguard network on (all) nodes
- load and persist br_netfilter
- - `modprobe br_netfilter && echo 'br_netfilter' > /etc/modules-load.d/k8s.conf`
- - `sysctl -w net.bridge.bridge-nf-call-iptables = 1 && echo 'net.bridge.bridge-nf-call-iptables = 1' >> /etc/sysctl.conf`
- init the cluster with `kubeadm init --apiserver-advertise-address <ip of wireguard endpoint> --control-plane-endpoint <your domain of choice> --apiserver-cert-extra-sans <also add a wildcard for your domain here> --node-name <node name> --pod-network-cidr 10.224.0.0/12 --upload-certs`
- save the output for later
- if running only one or limited amount of nodes remove taint from controlplane nodes to also use them as workers `kubectl taint nodes <node name> node-role.kubernetes.io/control-plane:NoSchedule-`
- TODO add more nodes

## setting up the infress(es)
- add the traefik repo to helm `helm repo add traefik https://traefik.github.io/charts`
- create a namespace for them
- add a traefik ingress to handle external requests `helm install --set ingressClass.name=traefik-external --set 'service.externalIPs={<the ip to listen on>}' -f traefik/values.yml traefik-external traefik/traefik -n ingress`
- add a second traefik ingress to handle only listen on the wireguard interface `helm install --set ingressClass.name=traefik-internal --set 'service.externalIPs={<the ip to listen on>}' -f traefik/values.yml traefik-internal traefik/traefik -n ingress`
- TODO probably should create my own chart so I only have to create the relevant service accounts for traefik once

## setting up storage and PVs (WIP)
- look into rook (config for one node cluster)

## plans
- [ ] log collection (alloy + loki)
- [ ] metrics collection (prometheus + grafana)
- [ ] hosting stuff (need ideas)
- [ ] SSO with keycloak??
- honestly I just wanna see some numbers move :3