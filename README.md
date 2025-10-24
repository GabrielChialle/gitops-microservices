
# GitOps na Prática - Online Boutique

### Objetivo: Executar um conjunto de microserviços (Online Boutique) em Kubernetes local usando Rancher Desktop, controlado por GitOps com ArgoCD, a partir de um repositório público no GitHub.

## Índice

1. [Pré-requisitos](#pré-requisitos)
2. [Instalando o Rancher Desktop](#2-instalando-o-rancher-desktop)
3. [Fork e criando reposiório no GitHub](#3-fork-e-criando-repositório-no-github)
    - 3.1 [Fork do repositório oficial](#31-fork-do-repositório-oficial)
    - 3.2 [Criando repositório no GitHub](#32-criando-repositório-no-github)
4. [Instalando ArgoCD no Cluster local](#4-instalando-argocd-no-cluster-local)
   - 4.1 [Acessar o ArgoCD localmente](#41-acessar-o-argocd-localmente)
5. [Criar o App no ArgoCD](#5-criar-o-app-no-argocd)
6. [Resultado final: Acessar o front-end](#6-resultado-final-acessar-o-front-end)
7. [EXTRAS: Modificar o arquivo online-boutique.yml e acessar e monitorar um repositório privado com ArgoCD](#7-extras-modificar-o-arquivo-online-boutiqueyml-e-acessar-e-monitorar-um-repositório-privado-com-argocd)
   - 7.1 [Modificar o arquivo online-boutique.yml](#71-modificar-o-arquivo-online-boutiqueyml)
   - 7.2 [Acessar e monitorar um repositório privado com ArgoCD](#72-acessar-e-monitorar-um-repositório-privado-com-argocd)

## 1. Pré-requisitos
- Para que seja possível utilizar o Rancher Desktop e suas dependências, é necessário possuir o WSL (Windows Subsystem for Linux) instalado em sua máquina.
- Para instalar o WSL, clique no link [Como Instalar o WSL](https://learn.microsoft.com/pt-br/windows/wsl/install) e siga o passo a passo.
- Git, clique no link [Git](https://git-scm.com/install/), selecione seu SO, baixe e instale o programa.

## 2. Instalando o Rancher Desktop
- Faça o download atráves do [Rancher Desktop](https://rancherdesktop.io/) e instale o programa.
- Após instalado, vá até **_Preferences_** > **_Container Engine_**, e selecione o _**dockerd (moby)**_.
- Em seguida vá até **_Kubernetes_** e habilite o Kubernetes caso não esteja ligado.

![Engine Rancher](https://github.com/user-attachments/assets/3f9c24f3-bf39-4681-ba57-289c2be2694d)

![Kubernetes Rancher](https://github.com/user-attachments/assets/0264a93e-7727-43e6-bd61-4260d2d72a39)

## 3. Fork e criando repositório no GitHub
### 3.1. Fork do repositório oficial
- Acesse o repositório [microservices-demo](https://github.com/GoogleCloudPlatform/microservices-demo)
- Clique em **_fork_** e em seguida **_Create fork_**.

### 3.2. Criando repositório no GitHub
- Vá até seu perfil no GitHub, clique em **_Repositories_** e **_New_**.
- Preencha os campos e clique em **_Create repository_**

![Exemplo repositorio](https://github.com/user-attachments/assets/dcb663b5-29be-45c3-9fa4-86a3ccbb9efb)

> [!IMPORTANT]
> A visibilidade do repositório deve estar como **_public_**, caso contrário, os próximos passos não funcionarão corretamente.

- Abra o Git bash em sua máquina, clone e abra o repositório criado.
- Para clonar clique em **_Code_** no repositório criado, copie a URL e rode o comando:

```
git clone https://github.com/<SEU-USUARIO>/<SEU-REPOSITÓRIO>.git
```

- Com o repositório aberto em um editor de texto ou IDE de sua preferência, crie a seguinte estrutura de pastas e arquivos:

```bash
seu-repositório/
├── k8s/
     ├── online-boutique.yml
```

![Estrutura pastas](https://github.com/user-attachments/assets/ad4e28b7-074a-43fb-88a5-50ef28664717)

- Acesse o arquivo [kubernetes-manifests.yaml](https://github.com/GoogleCloudPlatform/microservices-demo/blob/main/release/kubernetes-manifests.yaml) e copie **todo** o conteúdo para o seu arquivo **online-boutique.yml**
- Salve todas as alterações e envie para o seu repositório com os seguintes comandos:

```
git add .
```

```
git commit -m "Initial commit"
```

```
git push origin main
```

## 4. Instalando ArgoCD no Cluster local
- Com o Rancher Desktop devidamente instalado e rodando, abra o Git bash e digite os seguintes comandos:
```
kubectl create namespace argocd
``` 

```
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
``` 

### 4.1. Acessar o ArgoCD localmente
- Para acessarmos o ArgoCD, devemos fazer um port-forward para a interface web.
- Rode o seguinte comando para realizar o port-forward:
```
kubectl port-forward svc/argocd-server -n argocd 8080:443
```
- Agora, ao acessar o [localhost:8080](http://localhost:8080), aparecerá a tela inicial do ArgoCD.

![Tela ArgoCD](https://github.com/user-attachments/assets/1a07c4b7-b6e8-4a8f-8c18-c32fa7bf6dc7)

- Para logar, devemos utilizar o usuário e senha padrões:
- Usuário: admin
- Para obter a senha, rode o seguinte comando e copie a saída:
```
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 --decode
```

![Dashboard ArgoCD](https://github.com/user-attachments/assets/269417ca-7235-4938-a363-ffa16c0c1247)

## 5. Criar o App no ArgoCD

- Acesse a página do ArgoCD, e clique em **_New App_**.
- Clique em **_Edit as YAML_** e cole o seguinte código:

```
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: online-boutique
spec:
  destination:
    namespace: argocd
    server: https://kubernetes.default.svc
  source:
    path: k8s/
    repoURL: git@github.com:SEU-USUARIO/SEU-REPOSITÓRIO-PRIVADO.git
    targetRevision: HEAD
    directory:
      include: online-boutique.yml
  sources: []
  project: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
      enabled: true
    syncOptions:
      - CreateNamespace=true
```
> [!IMPORTANT]
> Substitua "SEU-USUARIO" e "SEU-REPOSITÓRIO" com os seus dados, caso contrário, não funcionará.

## 6. Resultado final: acessar o Front-end
- Para acessarmos o front-end da aplicação, devemos fazer um port-forward para o site.
- Rode o seguinte comando para realizar o port-forward:
```
kubectl port-forward svc/frontend-external -n argocd 3000:80
```
- Após isso, basta ir até o [localhost:3000](http:localhost:3000) e poderemos ver a página inicial da aplicação, e também outras páginas.

![Tela online boutique](https://github.com/user-attachments/assets/c5baa48f-8ab4-4457-8acd-59b49a8e8c47)

![Tela oculos](https://github.com/user-attachments/assets/1b2c66fe-4380-4c2c-89d2-f96183651e92)

![Tela carrinho](https://github.com/user-attachments/assets/612d6832-fbaa-4a86-9729-877d8f522b9a)

## 7. EXTRAS: Modificar o arquivo online-boutique.yml e acessar e monitorar um repositório privado com ArgoCD
### 7.1. Modificar o arquivo online-boutique.yml
**Podemos aumentar de réplicas de um microserviço para garantir alta disponibilidade e aumentar o nível de tolerância a falhas da aplicação.**
- Para isso, abra o arquivo **_online-boutique.yml_**, e, na seção do **_emailservice_**, altere o número de réplicas para quantidade desejada.

![Replicas exemplo](https://github.com/user-attachments/assets/13e3cd36-b134-4439-b541-7dbe4043eba8)

> [!IMPORTANT]
> Não esqueça de salvar o arquivo.

- Após isso basta enviar as alterações para o Git com os seguintes comandos:

```
git add .
```
```
git commit -m "feat: add replicas to emailservice"
```
```
git push origin main
```
- Podemos verificar as alterações recentes no site do ArgoCD.
- Para isso vá até sua aplicação e clique em **_Refresh_**.

![Exemplo refresh argocd](https://github.com/user-attachments/assets/6e101719-74f2-4697-8313-e6c90d6d6e10)

- Também podemos verificar o número de replicas no Git bash com o comando: 
```
kubectl get deployments -n argocd
```

**Antes da alteração do número de réplicas:**

![Antes réplicas](https://github.com/user-attachments/assets/4248e76c-4ad5-426d-ab95-bd9e9cf0e0f9)

**Depois da alteração do número de réplicas:**

![Depois réplicas](https://github.com/user-attachments/assets/53f11e9e-14d5-40ee-bae7-373b5bdca715)

## 7.2. Acessar e monitorar um repositório privado com ArgoCD
- Vá até seu perfil no GitHub, clique em **_Repositories_** e **_New_**.
- Preencha os campos e clique em **_Create repository_**

![Private repo](https://github.com/user-attachments/assets/9936f6cf-4763-4acd-a971-f04216969023)

>[!IMPORTANT]
> Dessa vez devemos mudar a visibilidade do repositório para private!

>[!IMPORTANT]
> Utilize a mesma estrutura de pastas e arquivos do repositório público criado anteriormente!

- Agora, devemos criar uma chave pública e uma chave privada para utilizarmos, para isso, rode o seguinte comando no bash:
```
 ssh-keygen -t ed25519 -C "argocd-deploy-key" -f ./argocd-deploy-key -N ""
```

> [!IMPORTANT]
> Com o comando acima criaremos duas chaves ssh utilizando o algoritmo ed25519. <br>
> Utilizamos o -C para adicionar um comentário ao final da chave pública. <br>
> O -f para definir o caminho e nome das chaves. <br>
> Por fim -N "" para não precisarmos de uma senha para acessar a chave privada.

- Após geradas ambas as chaves, acesse seu GitHub e vá até seu projeto > **_Settings_** > **_Deploy keys_** > **_Add deploy key_**.
- Pegue a valor da sua chave pública com o seguinte comando e cole no campo **_Key_**:
```
cat argocd-deploy-key.pub
```
- Clique em _**Add key**_.

> [!IMPORTANT]
> Você deve estar no diretório no qual suas chaves foram criadas!

![Exemplo deploy key](https://github.com/user-attachments/assets/2eb41ff4-4412-4376-8ba9-b4dbb5f1c083)

- Agora acesse o site do ArgoCD e vá até _**Settings**_ > _**Repositories**_ > _**Connect Repo**_.
- Pegue o valor da sua chave privada com o seguinte comando:
```
cat argocd-deploy-key
```
- Preencha as informações com os seus dados e clique em _**Connect**_.

![Exemplo private key](https://github.com/user-attachments/assets/20503fc9-8594-4245-9ecd-b34651e5dd6e)

- Após isso é possivel ver a conexão com o status _**Successful**_.

![Exemplo status](https://github.com/user-attachments/assets/02ff445a-7235-4873-884c-ad889a0d8626)

- Agora, basta criar um app utilizando o mesmo template utilizado anteriormente, mas alterando a _**repoURL**_ para o repositório privado criado.

```
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: online-boutique
spec:
  destination:
    namespace: argocd
    server: https://kubernetes.default.svc
  source:
    path: k8s/
    repoURL: git@github.com:SEU-USUARIO/SEU-REPOSITÓRIO.git
    targetRevision: HEAD
    directory:
      include: online-boutique.yml
  sources: []
  project: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
      enabled: true
    syncOptions:
      - CreateNamespace=true
```

### Após isso, concluímos todos os passos do projeto e tudo está devidamente configurado e funcionando corretamente.
