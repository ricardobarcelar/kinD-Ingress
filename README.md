### 1. Instalar KinD

```
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.22.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```
##### 1.1. A Instalação do KinD para executar ingress controller depende de uma configuração específica no nó Control Plane:

```
  - role: control-plane
    kubeadmConfigPatches:
    - |
      kind: InitConfiguration
      nodeRegistration:
        kubeletExtraArgs:
          node-labels: "ingress-ready=true"
    extraPortMappings:
    - containerPort: 80
      hostPort: 80
      protocol: TCP
    - containerPort: 443
      hostPort: 443
      protocol: TCP
```

### 2. Instalar Kubectl

```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
```

### 3. Criação do Cluster Kubernetes usando KinD

```
kind create cluster --name devops --config ./kindcluster.yml
```

##### 3.1. Para verificar se o cluster está funcionando corretamente é possível fazê-lo pelos seguintes comandos:


```
kind get clusters
kubectl config get-contexts
kind delete cluster --name devops
```

### 4. Instalação do Ingress Controller no Cluster

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
```

##### 4.1. Para verificar se o ingress controller roda adequadamente entre com o comando:

```
kubectl get pods -n ingress-nginx
```

### 5. Exemplo de deployment com Ingress

[deployment.yaml](deployment.yaml)

##### 5.1. No manifesto será necessário permitir a atribuição dinâmica da imagem. Para isso é possível usar as variáveis do gitlab:

```
image: "{{CI_REGISTRY_IMAGE}}:{{CI_COMMIT_REF_NAME}}"
```

##### 5.2. No gitlab-ci basta incluir o comando no estágio de build:

```
deploy:
  stage: deploy
  script:
    - kubectl apply -f deployment.yml
```



