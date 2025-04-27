## UPGRADING CONTROLPLANE

```bash
sudo nano /etc/apt/sources.list.d/kubernetes.list

Change the version from 1.31 to 1.32

sudo apt update

sudo apt-cache madison kubeadm
   kubeadm | 1.32.4-1.1 | https://pkgs.k8s.io/core:/stable:/v1.32/deb  Packages
   kubeadm | 1.32.3-1.1 | https://pkgs.k8s.io/core:/stable:/v1.32/deb  Packages
   kubeadm | 1.32.2-1.1 | https://pkgs.k8s.io/core:/stable:/v1.32/deb  Packages
   kubeadm | 1.32.1-1.1 | https://pkgs.k8s.io/core:/stable:/v1.32/deb  Packages
   kubeadm | 1.32.0-1.1 | https://pkgs.k8s.io/core:/stable:/v1.32/deb  Packages
```

**Upgrade kubeadm:**

```bash
# replace x in 1.32.x-* with the latest patch version
sudo apt-mark unhold kubeadm && \
sudo apt-get update && sudo apt-get install -y kubeadm='1.32.x-*' && \
sudo apt-mark hold kubeadm

kubeadm version
```

**Upgrade controlplane componenets**

```bash
sudo kubeadm upgrade plan

sudo kubeadm upgrade apply v1.32.4

kubectl drain controlplane --ignore-daemonsets

sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && sudo apt-get install -y kubelet='1.32.4-1.1' kubectl='1.32.4-1.1' && \
sudo apt-mark hold kubelet kubectl


sudo systemctl daemon-reload
sudo systemctl restart kubelet

kubectl uncordon controlplane
```

## UPGRADING WORKERNODES

```bash
sudo nano /etc/apt/sources.list.d/kubernetes.list

Change the version from 1.31 to 1.32

sudo apt update

sudo apt-mark unhold kubeadm && sudo apt-get update && sudo apt-get install -y kubeadm='1.32.4-1.1' && sudo apt-mark hold kubeadm

sudo kubeadm upgrade node

kubectl drain <node01|node02> --ignore-daemonsets  ( On controlplane node )

sudo apt-mark unhold kubelet kubectl
sudo apt-get update && sudo apt-get install -y kubelet='1.32.4-1.1' kubectl='1.32.4-1.1'
sudo apt-mark hold kubelet kubectl

sudo systemctl daemon-reload
sudo systemctl restart kubelet

kubectl uncordon <node01|node02> ( On Controlplane node )
```

*Repeat the same process on all worker nodes*