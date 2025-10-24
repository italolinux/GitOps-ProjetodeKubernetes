# boutique-online
Este projeto demonstra a prática de GitOps implementando o deploy da aplicação Online Boutique (conjunto de microserviços) em um cluster Kubernetes local usando Rancher Desktop, gerenciado via ArgoCD.

## 🚀 Objetivo

Executar um conjunto de microserviços em Kubernetes local controlado por GitOps com ArgoCD, utilizando um repositório público no GitHub como fonte de verdade para os manifests Kubernetes.

## 📋 Funcionalidades

Deploy automatizado de microserviços usando GitOps

Gerenciamento de infraestrutura como código (IaC)

Sincronização automática entre repositório Git e cluster Kubernetes

Interface web do ArgoCD para monitoramento e controle

Aplicação Online Boutique funcionando com frontend acessível via port-forward

## 🛠️ Tecnologias Utilizadas

Kubernetes (via Rancher Desktop)

ArgoCD - GitOps continuous delivery tool

Docker - Containerização

GitHub - Versionamento e fonte de verdade

Kubectl - CLI para Kubernetes

## 📁 Estrutura do Projeto

gitops-microservices/
└── k8s/
    └── online-boutique.yaml
