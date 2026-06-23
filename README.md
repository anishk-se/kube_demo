# Prequistes Installation
1. install docker desktop

2. install kind:
# Detect architecture and download the binary
[ $(uname -m) = x86_64 ] && curl -Lo ./kind https://k8s.io
[ $(uname -m) = aarch64 ] && curl -Lo ./kind https://k8s.io

# Make the binary executable
chmod +x ./kind

# Move it to a directory in your executable PATH
sudo mv ./kind /usr/local/bin/kind

3. install kubectl:
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl