# GitOps na Prática - Deploy de Online Boutique com Kubernetes e ArgoCD
Este projeto demonstra a prática de GitOps implementando o deploy da aplicação Online Boutique (conjunto de microserviços) em um cluster Kubernetes local usando Rancher Desktop, gerenciado via ArgoCD.

## 📋 Funcionalidades

* Deploy automatizado de microserviços usando GitOps.

* Gerenciamento de infraestrutura como código (IaC).

* Sincronização automática entre repositório Git e cluster Kubernetes.

* Interface web do ArgoCD para monitoramento e controle.

* Aplicação Online Boutique funcionando com frontend acessível via port-forward.

## 🛠️ Tecnologias Utilizadas

* Kubernetes (via Rancher Desktop).

* ArgoCD - GitOps continuous delivery tool.

* Docker - Containerização.

* GitHub - Versionamento e fonte de verdade.

* Kubectl - CLI para Kubernetes.

## ⚙️ Pré-requisitos
* Rancher Desktop com Kubernetes habilitado.

* Kubectl configurado.

* ArgoCD instalado no cluster.

* Conta no GitHub.

* Git instalado.

* Docker funcionando localmente.

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
Este comando baixa e aplica todos os manifestos necessários para ter o ArgoCD funcionando como controlador GitOps no seu cluster.

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
Para instalar no windows, vamos usar o Invoke-WebRequest no Power shell.

Invoke-WebRequest é um cmdlet nativo do PowerShell para fazer requisições HTTP/HTTPS, similar ao curl no Linux/Mac.

Primeiro vamos pegar a versão mais recente do argocd e armazenar na variável ```$version```.
```powershell
$version = (Invoke-RestMethod https://api.github.com/repos/argoproj/argo-cd/releases/latest).tag_name
```
* ``` Invoke-RestMethod ```
   * Faz uma requisição à API do GitHub.

   * Consulta o endpoint de latest release do ArgoCD.

   * Retorna automaticamente como objeto (não como texto puro).

* ``` https://api.github.com/repos/argoproj/argo-cd/releases/latest ```
   * Endpoint oficial da API do GitHub.

   * Retorna informações da última versão estável do ArgoCD.

* ``` .tag_name ```
   * Acessa a propriedade que contém o número da versão. (Exemplo: "v2.8.4", "v2.9.0")

* ```$version =```
   * Armazena o resultado na variável $version para uso posterior
 
Substitua ``` $version ``` no comando abaixo pela versão do Argo CD que você deseja baixar:

```powershell
$url = "https://github.com/argoproj/argo-cd/releases/download/" + $version + "/argocd-windows-amd64.exe"
$output = "argocd.exe"

Invoke-WebRequest -Uri $url -OutFile $outputkubectl 
```
Agora precisamos mover o arquivo para o seu PATH.

```powersheell
[Environment]::SetEnvironmentVariable("Path", "$env:Path;C:\Path\To\ArgoCD-CLI", "User")
```
Depois de concluir as instruções acima, você agora poderá executar ``` argocd ``` comandos, para fazer um teste rode o comando:

```powershell
argocd version
```
Vai retornar algo parecido com isso.

```powershell
argocd: v3.1.9+8665140
  BuildDate: 2025-10-17T22:07:41Z
  GitCommit: 8665140f96f6b238a20e578dba7f9aef91ddac51
  GitTreeState: clean
  GoVersion: go1.24.6
  Compiler: gc
  Platform: windows/amd64
```

# Etapa 3 Acessar o ArgoCD localmente
## 3.1 Port Forwarding
Para acessar o argocd precisamos fazer um port forwarding do Kubernetes que cria uma conexão segura e temporária entre uma máquina local e um recurso específico (normalmente um Pod ou Service) dentro de um cluster Kubernetes. Isso permite acesso local a aplicações ou serviços executando dentro do cluster como se estivessem rodando localmente, sem expô-los externamente. É utilizado principalmente para desenvolvimento, depuração e testes.

```powershell
kubectl port-forward svc/argocd-server -n argocd 8080:443
```
* recomendo usar outro terminal para rodar esse comando, pois ele vai ficar rodando a aplicação.

Para acessar o argocd coloque o endereço ``` https://localhost:8080 ``` no seu navegador 

<img width="1790" height="636" alt="Captura de tela 2025-10-26 152553" src="https://github.com/user-attachments/assets/d969042f-08c6-487f-ad88-12f4993dc519" />

## 3.2 Fazendo login no argocd
Precisamos fazer login no argocd, o usuário padrão dele é ``` admin ``` e a senha temos que descobrir usando o comando:

**Para linux**
```bash
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d
```

**Como o windows não reconhece base64, temos que decodificar usando:**

```powershell
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | %{ [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($_)) }
```
* Decodificação do base64
  * % → Alias para ForEach-Object

  * [System.Convert]::FromBase64String($_) → Converte base64 para bytes

  * [System.Text.Encoding]::UTF8.GetString() → Converte bytes para texto legível
 
Descobrindo a senha podemos fazer o acesso no argocd com o comando:

```powershell
argocd login localhost:8080
```
Ele vai pedir o nome de usuário e depois a senha que descobrimos anteriormente, Depois ele estabelece a conexão.

``` powershell
'admin:login' logged in successfully
Context 'localhost:8080' updated
```
Podemos entrar pelo interface gráfica, colocando o nome de usuário e a senha.

<img width="1894" height="546" alt="Captura de tela 2025-10-26 153457" src="https://github.com/user-attachments/assets/8ef13e56-c83a-4cd2-8bf3-7803d57e5ccb" />

# Etapa 4 Criando a Aplicação no ArgoCD.
Agora que o cluster Kubernetes está integrado ao ArgoCD, o próximo passo é criar a aplicação. Para isso, precisamos conectar um repositório Git contendo os manifestos Kubernetes da nossa aplicação. O ArgoCD monitorará continuamente este repositório e automaticamente implantará e sincronizará as alterações no cluster sempre que detectar atualizações no código-fonte.

Para criarmos a aplicação usamos o comando:

```powershell
argocd app create online-boutique `
   --repo https://github.com/italolinux/GitOps-ProjetodeKubernetes.git `
   --path k8s `
   --dest-server https://kubernetes.default.svc `
   --dest-namespace default
```
**OBS:** Para o linux é o mesmo comando só trocar ``` ` ``` por ``` \ ```.

| Parâmetro          | Significado                                       |
| ------------------ | ------------------------------------------------- |
| `--repo`           | Link do repositório Git (com `.git`)              |
| `--path`           | Caminho dentro do repositório onde estão os YAMLs |
| `--dest-server`    | Cluster onde a app será implantada                |
| `--dest-namespace` | Namespace dentro do cluster                       |

Vamos verificar se a nossa aplicação foi criada.
```powershell
argocd app list
```
Retorna:

```powershell
NAME                    CLUSTER                         NAMESPACE  PROJECT  STATUS     HEALTH   SYNCPOLICY  CONDITIONS  REPO                                                          PATH  TARGET
argocd/online-boutique  https://kubernetes.default.svc  default    default  OutOfSync  Missing  Manual      <none>      https://github.com/italolinux/GitOps-ProjetodeKubernetes.git  k8s
```
Como podemos observar o status da nossa aplicação está ``` **OutOfSync** ``` e a health está ``` **Missing** ```, precisamos fazer o sync da aplicação com o argocd.

```powershell
argocd app sync online-boutique
```
* O que acontece durante o ``` sync ```:
    
    *  Conecta ao repositório Git e busca os manifests mais recentes.

    * Compara o estado atual do cluster com o estado desejado no Git.

    * Aplica as diferenças no cluster Kubernetes.

    * Cria/atualiza todos os recursos definidos no online-boutique.yaml.
      
# Etapa 5 Acessar o Front-end da aplicação
Igual para acessar o argocd tivemos que fazer um port forwarding, faremos para o online-boutique.
Para isso vamos ver os serviços da aplicação.

```powershell
kubectl get svc
```

Vai retornar.

```powershell
NAME                    TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)        AGE
adservice               ClusterIP      10.43.211.236   <none>          9555/TCP       6m13s
cartservice             ClusterIP      10.43.34.72     <none>          7070/TCP       6m13s
checkoutservice         ClusterIP      10.43.215.186   <none>          5050/TCP       6m13s
currencyservice         ClusterIP      10.43.160.249   <none>          7000/TCP       6m13s
emailservice            ClusterIP      10.43.75.157    <none>          5000/TCP       6m13s
frontend                ClusterIP      10.43.255.5     <none>          80/TCP         6m13s
frontend-external       LoadBalancer   10.43.60.228    192.168.127.2   80:31850/TCP   6m13s
kubernetes              ClusterIP      10.43.0.1       <none>          443/TCP        47h
paymentservice          ClusterIP      10.43.33.93     <none>          50051/TCP      6m13s
productcatalogservice   ClusterIP      10.43.12.72     <none>          3550/TCP       6m13s
recommendationservice   ClusterIP      10.43.208.3     <none>          8080/TCP       6m13s
redis-cart              ClusterIP      10.43.116.21    <none>          6379/TCP       6m13s
shippingservice         ClusterIP      10.43.196.210   <none>          50051/TCP      6m13s
```
Precisamos dar o port forward no ``` LoadBalancer ``` com o nome de ``` frontend-external. ```

```powershell
kubectl port-forward svc/frontend-external 80:80
```
Agora é só acessar no navegador o endereço ``` http://localhost ```.

<img width="1890" height="899" alt="Captura de tela 2025-10-26 162957" src="https://github.com/user-attachments/assets/94408451-9e95-4534-8737-1d2aca72c0ba" />

# Etapa 6 Desafio extra Criando a aplicação com um repositório privado.
Para criar nossa aplicação usando um repositório privado, vou usar o método com HTTPS + token do GitHub.
## 6.1 Crie um token de acesso pessoal no GitHub

* Acesse: https://github.com/settings/tokens

* Clique em Generate new token → Fine-grained token

* Dê permissão de Read access to contents

* Copie o token (exemplo: ghp_xxxxxxx)

## 6.2 Adicione o repositório ao ArgoCD

Rode este comando :

```powershell
argocd repo add https://github.com/italolinux/meu-repo.git `
  --username italolinux `
  --password ghp_xxxxxxx `
  --insecure
```
* Substitua ``` meu-repo.git ``` pelo nome real do seu repositório.

* No ``` --username ``` coloque seu nome de usuário do github.

* Substitua ``` ghp_xxxxxxx ``` pelo token real.

* A flag ``` --insecure ``` evita problemas de certificado no ambiente local.

Para verificar .

```powershell

argocd repo list
```
Retorna.

```powershell
TYPE  NAME  REPO                                                          INSECURE  OCI    LFS    CREDS  STATUS      MESSAGE  PROJECT
git         https://github.com/italolinux/GitOps-ProjetodeKubernetes.git  false     false  false  false  Successful
```
Como podemos ver os status está ``` Successful ``` comprovando a conexão.

Agora é só seguir as etapas [4](# Etapa 4 Criando a Aplicação no ArgoCD.) e [5](# Etapa 5 Acessar o Front-end da aplicação) de criação e acesso a aplicação. 
