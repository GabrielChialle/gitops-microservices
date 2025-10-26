# Implantação do Projeto GitOps na Prática

## Pré-requisitos

- **Minikube instalado e em execução** (`minikube start`)
- **Kubectl configurado** (`kubectl get nodes` funcionando)
- **ArgoCD instalado no cluster**
- **Conta no GitHub com repositório público**
- **Git instalado**
- **Docker funcionando localmente**

---

## Etapa 1 – Fork e repositório GitHub

1.1. Realizar o fork do repositório oficial:
   - [https://github.com/GoogleCloudPlatform/microservices-demo](https://github.com/GoogleCloudPlatform/microservices-demo)

1.2. Criar um repositório no GitHub com:
   - Apenas o arquivo `release/kubernetes-manifests.yaml`
   - Estrutura recomendada:

![Estrutura pastas](https://github.com/user-attachments/assets/ad4e28b7-074a-43fb-88a5-50ef28664717)

1.3. Clonar o repositório localmente:
```
git clone https://github.com/SEU_USUARIO/gitops-microservices.git
cd gitops-microservices
```

1.4. Copiar o manifesto do projeto para o repositório:
```
cp path/para/microservices-demo/release/kubernetes-manifests.yaml k8s/online-boutique.yaml
```

1.5. Fazer o commit e enviar para o GitHub:
```
git add .
git commit -m "Adicionando manifesto do Online Boutique"
git push origin main
```

---

## Etapa 2 – Instalar ArgoCD no cluster local

2.1. Criar o namespace do ArgoCD:
```
kubectl create namespace argocd
```

2.2. Aplicar o manifesto de instalação do ArgoCD:
```
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

2.3. Verificar se os pods foram criados corretamente:
```
kubectl get pods -n argocd
```

Todos os pods devem aparecer com o status Running.

---

## Etapa 3 – Acessar ArgoCD localmente

3.1. Fazer o port-forward para acessar a interface web:
```
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

3.2. Acessar no navegador:

URL: `https://localhost:8080`

<img width="1366" height="768" alt="Captura de tela de 2025-10-26 12-28-13" src="https://github.com/user-attachments/assets/45eedb33-74f5-47f1-b74c-8f9abd2a96fe" />

Credenciais padrão:

   - Usuário: `admin`
   - Senha: executar o comando abaixo para obter:
```
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo
```

---

## Etapa 4 – Criar o App no ArgoCD

4.1. No painel do ArgoCD, clique em "New App".

<img width="1366" height="768" alt="Captura de tela de 2025-10-26 12-30-02" src="https://github.com/user-attachments/assets/12f2352d-c5d3-4e65-8926-a669ee998057" />

4.2. Preencha com as seguintes informações:

   - Application Name: `online-boutique`
   - Project: `default`
   - Sync Policy: `automática (com prune e self-heal)`
   - Repository URL: `https://github.com/SEU_USUARIO/gitops-microservices.git`
   - Revision: `HEAD`
   - Path: `k8s`
   - Cluster URL: `https://kubernetes.default.svc`
   - Namespace: `online-boutique`

Assim ficará o yaml:
```
project: default
source:
  repoURL: https://github.com/SEU-USUARIO/gitops-microservices.git
  path: k8s
  targetRevision: HEAD
  directory:
    recurse: true
    jsonnet: {}
destination:
  server: https://kubernetes.default.svc
  namespace: online-boutique
syncPolicy:
  automated:
    prune: true
    selfHeal: true
    enabled: true
  syncOptions:
    - CreateNamespace=true
```

4.3. Clique em "Create".

4.4. Após criar, clique em "Sync" → "Synchronize" para aplicar o manifesto no cluster.

<img width="1366" height="768" alt="Captura de tela de 2025-10-26 13-01-01" src="https://github.com/user-attachments/assets/85ca8330-7e50-4887-973b-f57f2eb243da" />

Durante o processo de sincronização, o status “Progressing” indica que os pods estão sendo criados.

---

## Etapa 5 – Acessar o front-end

O frontend roda como um ClusterIP, então precisamos expor a aplicação via port-forward.

5.1. Identificar o serviço do frontend:
```
kubectl get svc -n online-boutique
```

5.2. Fazer o port-forward:
```
kubectl port-forward -n online-boutique svc/frontend 8081:80
```

Acessar no navegador:

URL: `http://localhost:8081`

<img width="1366" height="768" alt="Captura de tela de 2025-10-26 13-06-55" src="https://github.com/user-attachments/assets/18afa5cc-ae65-42c4-9ae4-34b5c70c502a" />


Se a página do Online Boutique aparecer, o projeto foi implantado com sucesso 🎉

---

## Conclusão
Com o ambiente configurado, o ArgoCD gerencia automaticamente o estado do cluster a partir do repositório Git.
