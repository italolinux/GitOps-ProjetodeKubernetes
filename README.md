# GitOps na Prática - Deploy de Online Boutique com Kubernetes e ArgoCD
Este projeto demonstra a prática de GitOps implementando o deploy da aplicação Online Boutique (conjunto de microserviços) em um cluster Kubernetes local usando Rancher Desktop, gerenciado via ArgoCD.

## 📋 Funcionalidades

* Deploy automatizado de microserviços usando GitOps

* Gerenciamento de infraestrutura como código (IaC)

* Sincronização automática entre repositório Git e cluster Kubernetes

* Interface web do ArgoCD para monitoramento e controle

* Aplicação Online Boutique funcionando com frontend acessível via port-forward

## 🛠️ Tecnologias Utilizadas

* Kubernetes (via Rancher Desktop)

* ArgoCD - GitOps continuous delivery tool

* Docker - Containerização

* GitHub - Versionamento e fonte de verdade

* Kubectl - CLI para Kubernetes

## ⚙️ Pré-requisitos
* Rancher Desktop com Kubernetes habilitado

* Kubectl configurado

* ArgoCD instalado no cluster

* Conta no GitHub

* Git instalado

* Docker funcionando localmente

Para a instalação do Rancher Desktop no Windows, necessita do WSL2.

# Etapa 1 – Fork e repositório GitHub
O princípio central do GitOps estabelece o Git como a fonte única e definitiva para toda a configuração de infraestrutura e aplicações. Esta abordagem garante rastreabilidade, controle de versão e consistência em todos os ambientes.

## 1.1 - Criando o Fork
Um fork é um novo repositório que compartilha configurações de código e visibilidade com o repositório "upstream" original. Os forks geralmente são usados para iterar ideias ou alterações antes de serem propostas de volta para o repositório upstream, como em projetos código aberto ou quando um usuário não tem acesso de gravação ao repositório upstream.

Para esse projeto vamos criar o Fork do repositório:
```
https://github.com/GoogleCloudPlatform/microservices-demo
```
Acessando o repositório acima, no canto superior direito da página, clique em Criar Fork.

<img width="787" height="387" alt="criando fork" src="https://github.com/user-attachments/assets/9d57ba8a-52f0-4038-accf-c43bd9e99d86" />

## 1.2 Criando o repositório no github
você irá estruturar um repositório Git que armazenará toda a configuração declarativa da aplicação Online Boutique. Este repositório servirá como base para todo o processo de deployment automatizado.

* No GitHub, clique em New repository.
* Escolha um nome. (EX: GitOps-ProjetodeKubernetes).
* Deixe o repositório público para que o ArgoCD possa fazer a sincronização mais pra frente.
* Crie o repositório.
* Adicione o conteúdo do arquivo [release/kubernetes-manifests.yaml](https://github.com/GoogleCloudPlatform/microservices-demo/blob/main/release/kubernetes-manifests.yaml) a pasta **k8s** com nome **online-boutique.yaml**.

## 1.3 Estrutura do Projeto
```
GitOps-ProjetodeKubernetes/
└── k8s/
    └── online-boutique.yaml
```
# Etapa 2 Instalar ArgoCD no cluster local
O ArgoCD é uma ferramenta open-source essencial para GitOps e Continuous Delivery, especializando-se exclusivamente na etapa de deployment (CD) em Kubernetes. Como um operador nativo do Kubernetes, ele estende a plataforma através de Custom Resources e Controllers para gerenciar aplicações de forma declarativa, sincronizando automaticamente o estado do cluster com a configuração armazenada no Git. Sua abordagem focada no CD - sem tentar resolver todo o fluxo de CI/CD - permite que equipes adotem entregas contínuas com maior maturidade, segurança e qualidade, tornando-se peça fundamental nas engenharias de software mais avançadas do mundo.

## 2.1 Criação do namespace do Argocd
Para instalar o ArgoCD como operador no Kubernetes, o primeiro passo é criar um namespace dedicado chamado argocd. Execute o comando abaixo para preparar o ambiente:
``` powershell
kubectl create namespace argocd
```
Este namespace isolará todos os recursos do ArgoCD, seguindo as melhores práticas de organização e segurança do cluster Kubernetes.

## 2.2 Instalando o argocd.
Agora vamos instalar o ArgoCD como um operador no Kubernetes:
``` powershell
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
Este comando baixa e aplica todos os manifestos necessários para ter o ArgoCD funcionando como controlador GitOps no seu cluster

Vamos ver se os pods do ArgoCD foram criados com sucesso:

``` powershell
kubectl get pods -n argocd
```
Deve retornar:
``` powershell
NAME                                               READY   STATUS    RESTARTS   AGE
argocd-application-controller-0                    1/1     Running   0          98s
argocd-applicationset-controller-79887fd6-d7zdj    1/1     Running   0          98s
argocd-dex-server-655b4448b5-kzhrv                 1/1     Running   0          98s
argocd-notifications-controller-77fd6f9885-hrtq8   1/1     Running   0          98s
argocd-redis-7fdcfb697b-df7mk                      1/1     Running   0          98s
argocd-repo-server-7d969c8c68-n95gv                1/1     Running   0          98s
argocd-server-58f6cdcccd-schqw                     1/1     Running   0          98s
```
Este output do ``` kubectl get pods ``` demonstra que a instalação do ArgoCD foi bem-sucedida, com todos os sete componentes principais em execução no cluster Kubernetes. Cada pod possui uma função específica: o ``` argocd-application-controller-0 ``` é o cérebro que sincroniza as aplicações entre o Git e o cluster; o ``` argocd-server ``` fornece a API e interface web; o ``` argocd-repo-server ``` gerencia os repositórios Git; enquanto componentes como ``` argocd-redis ``` e ``` argocd-dex-server ``` fornecem cache e autenticação.

## 2.3 Instalando Argocd CLI 
O ArgoCD CLI é uma ferramenta complementar que permite interagir com o ArgoCD diretamente pelo terminal, oferecendo controle total sobre aplicações, projetos e configurações através de comandos simples. Enquanto a interface web proporciona uma visão gráfica intuitiva, o CLI agiliza operações rotineiras, automações e integrações em pipelines CI/CD.

Para instalar a versão mais recente no Linux:
``` bash
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
rm argocd-linux-amd64
```
Para instalar no windows, vamos usar o PowerShell
