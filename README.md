<h1 style="text-align: center;">Kubernetes</h1>

1. Install Docker Desktop
   - Download and install from https://www.docker.com/products/docker-desktop

2. Install kind:
```bash
[ $(uname -m) = x86_64 ] && curl -Lo ./kind https://k8s.io/kind/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```

3. Install kubectl:
```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
```