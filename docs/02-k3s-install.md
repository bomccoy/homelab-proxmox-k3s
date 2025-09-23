# K3s Install

## Master Node
```bash
curl -sfL https://get.k3s.io | sh -
```

Check status:
```bash
sudo kubectl get nodes
```

Get join token:
```bash
sudo cat /var/lib/rancher/k3s/server/node-token
```

## Worker Node(s)
```bash
curl -sfL https://get.k3s.io | K3S_URL=https://<MASTER_IP>:6443 K3S_TOKEN=<TOKEN> sh -
```

Verify cluster:
```bash
kubectl get nodes
```
