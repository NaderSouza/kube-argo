# Projeto kube-argo

## Objetivo

Deploy Local de Kubernetes com ArgoCD e Aplicação `kube-argo`

Este guia descreve como configurar clusters locais utilizando **Minikube** e **Kind**, instalar o **ArgoCD**, e implantar a aplicação do repositório [kube-argo](https://github.com/NaderSouza/kube-argo) no Kubernetes.

## Para o desenvolvimento e implementação descritos neste guia, segui as seguintes premissas:

- Utilizar Minikube e Kind para configurar clusters Kubernetes locais de forma prática e reproduzível.
- Realizar a instalação do ArgoCD para gerenciar e automatizar implantações no Kubernetes.
- Implantar a aplicação disponibilizada no repositório kube-argo diretamente no cluster Kubernetes configurado.
- Garantir que todas as etapas sejam facilmente replicáveis para ambientes de desenvolvimento e testes.

## Tecnologias relacionadas

- [ArgoCD](https://argo-cd.readthedocs.io/)
- [Docker](https://www.docker.com/)
- [Kind](https://kind.sigs.k8s.io/)
- [Kubernetes](https://kubernetes.io/)
- [Minikube](https://minikube.sigs.k8s.io/docs/)
- [Nginx](https://nginx.org/)

## Configuração da Aplicação

Para que possamos continuar daqui para frente, precisamos ter o seguinte instalado:

- Um cluster Kubernetes
- kubectl instalado

## 1. Criar Clusters Locais

### 1.1. Minikube

```bash
# Iniciar o cluster Minikube
minikube start --driver=docker

# Verificar o status do cluster
kubectl cluster-info

```

### 1.2. Kind

```bash
# Criar arquivo de configuração kind-config.yaml
(Usar o kind-config.yaml como exemplo caso esteja usando minikube e o cluster localmente)

# Criar o cluster Kind
kind create cluster --config kind-config.yaml

# Verificar o cluster
kubectl get nodes

#Aplicar o deplyment
kubectl apply -f nginx-deployment.yaml
```

### 2. Instalar o ArgoCD

```bash
# Criar namespace do ArgoCD
kubectl create namespace argocd

# Instalar o ArgoCD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Verificar os pods do ArgoCD
kubectl get pods -n argocd
```

### 3. Configurar o ArgoCD CLI

```bash
# Baixar e instalar o CLI do ArgoCD
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
rm argocd-linux-amd64

# Expor o serviço do ArgoCD localmente
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Obter a senha inicial do ArgoCD
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d

# Fazer login no ArgoCD CLI
argocd login localhost:8080 --username admin --password <SENHA>

```

### 4. Configurar e Sincronizar a Aplicação kube-argo

```bash
# Adicionar o cluster Kind ao ArgoCD
argocd cluster add kind-kind --insecure --in-cluster --upsert

# Criar a aplicação no ArgoCD
argocd app create kube-argo \
  --repo https://github.com/NaderSouza/kube-argo \
  --path . \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default

# Sincronizar a aplicação
argocd app sync kube-argo
```

### 5. Verificar os Recursos no Cluster

```bash
# Listar os pods
kubectl get pods

# Verificar os serviços
kubectl get svc

# Confirmar o ConfigMap
kubectl get configmap nginx-config

# Listar todas as aplicações no ArgoCD
argocd app list


```

### 6. Acessar a replica

```bash

# Verifique o Pod:
kubectl get pods


# Verificar a replica criada
kubectl port-forward pod/<nome-do-pod> 9090:80

```

### 7. Acessar o ArgoCD no Navegador

### https://localhost:8080

### 8. Deletar

```bash

# Deletar cluster:
kind delete cluster --name <nome-do-cluster>


# Deletar namespace
kubectl delete namespace <nome-do-namespace>

```
