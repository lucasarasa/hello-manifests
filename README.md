# 🚀 Repositório Hello Manifests – GitOps com ArgoCD

Este repositório contém os manifests Kubernetes para a aplicação `hello-app`, utilizados para deploy automatizado via GitOps com ArgoCD.

---

## 📌 Objetivo

* Armazenar os manifests Kubernetes da aplicação `hello-app`
* Automatizar o deploy da aplicação via ArgoCD
* Facilitar a integração com pipelines CI/CD

---

## 🧱 Estrutura do Repositório

```
hello-manifests/
├── .github/
│   └── workflows/
│       └── verificar-pr-deploy.yaml  # Workflow do GitHub Actions para validação de PRs
├── manifests/
│   ├── deployment.yaml               # Manifesto de Deployment da aplicação
│   └── service.yaml                  # Manifesto de Service da aplicação
└── README.md                         # Este documento
```

---

## ⚙️ Tecnologias Utilizadas

* Kubernetes
* ArgoCD
* GitOps

---

## 📄 Manifests Kubernetes

### Deployment (`deployment.yaml`)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: hello-app
  name: hello-app
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello-app
  template:
    metadata:
      labels:
        app: hello-app
    spec:
      containers:
      - image: SEU-USUARIO/SUA-IMAGEM:TAG
        name: hello-app
        ports:
        - containerPort: 8080
```

### Service (`service.yaml`)

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: hello-app
  name: hello-app
  namespace: default
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 8080
  selector:
    app: hello-app
```

---

## 🤖 Deploy via ArgoCD

Este repositório é monitorado pelo ArgoCD para sincronizar automaticamente os manifests com o cluster Kubernetes.

## 🔐 Acesso via SSH no ArgoCD

### 🛠️ Pré-requisitos: instalação e configuração do ArgoCD já concluídas

---

### 🌐 1. Expor o ArgoCD localmente (acesso via navegador)

```bash
kubectl port-forward svc/argocd-server -n argocd 8081:443
```

Acesse via browser: [https://localhost:8081](https://localhost:8081)

---

### 🔑 2. Obter senha inicial do usuário `admin`

#### 👉 No **Linux**:

```bash
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d
```

#### 👉 No **Windows** (PowerShell):

```powershell
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | ForEach-Object { [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($_)) }
```

---

### 🔐 3. Fazer login no ArgoCD via CLI

```bash
argocd login localhost:8081
```

Usuário: `admin` Senha: a senha obtida no passo anterior

---

### ▶️ 4. Criar a aplicação (repositório público)

Se o repositório `hello-manifest` for **público**, o ArgoCD pode acessá-lo diretamente. Para criar a aplicação, use o comando:

```bash
argocd app create hello-app \
  --repo https://github.com/SEU-USUARIO/SEU-REPOSITORIO.git \
  --path manifests \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default
```

### 🔁 5. Sincronizar a aplicação manualmente

```bash
argocd app sync hello-app
```

> Ou faça isso pela interface gráfica

---

### 🔐 4.1. Criar a aplicação (repositório privado)

1. Gere um par de chaves SSH com:

```bash
ssh-keygen -t rsa -b 4096 -C "argocd@acesso" -f argocd-ssh-key
```

> Pressione Enter nas perguntas para manter os padrões. Isso criará dois arquivos: `argocd-ssh-key` (privada) e `argocd-ssh-key.pub` (pública).

2. Adicione a chave pública (`argocd-ssh-key.pub`) no GitHub:

   * Acesse o repositório `hello-app` ou `hello-manifest`
   * Vá em **Settings > Deploy Keys > Add deploy key**
   * Cole a chave pública e marque **Allow read access**

3. Crie uma `Secret` no cluster com a chave privada:

Crie um arquivo `repo-secret.yaml`:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: repo-ssh-creds
  namespace: argocd
  labels:
    argocd.argoproj.io/secret-type: repository
type: Opaque
stringData:
  url: git@github.com:SEU-USUARIO/SEU-REPOSITORIO.git
  sshPrivateKey: |
    -----BEGIN OPENSSH PRIVATE KEY-----
    SUA_CHAVE_PRIVADA_AQUI
    -----END OPENSSH PRIVATE KEY-----
```

Aplique com:

```bash
kubectl apply -f repo-secret.yaml
```

4. O ArgoCD poderá acessar o repositório via SSH.

Veja os detalhes no [README do projeto de manifests](https://github.com/lucasarasa/hello-manifests/blob/main/README.md)

---

## ✅ Validação automática de Pull Requests

Este repositório utiliza um workflow do GitHub Actions para validar alterações feitas via Pull Request no branch `main`. O objetivo é garantir que o arquivo `deployment.yaml` contenha a imagem da aplicação corretamente antes que as alterações sejam aplicadas no cluster.

### Workflow: `Verificar PR de deploy`

```yaml
name: Verificar PR de deploy

on:
  pull_request:
    branches:
      - main

jobs:
  validate-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout do PR
        uses: actions/checkout@v3

      - name: Verificar imagem usada no deployment.yaml
        run: |
          echo "Imagem encontrada:"
          grep "image:" manifests/deployment.yaml || echo "Não encontrou linha 'image:'"

      - name: Mostrar imagem atualizada
        run: |
          grep image: manifests/deployment.yaml
```

Este workflow é executado automaticamente sempre que um Pull Request é aberto ou atualizado para o branch `main`, verificando se a imagem foi corretamente definida no manifesto de deployment.

---

## 📂 Relacionamento com o Repositório de Código

Este repositório trabalha em conjunto com o repositório `hello-app`, que contém o código da aplicação e a pipeline CI/CD.

Link para o repositório de código: [https://github.com/lucasarasa/hello-app](https://github.com/lucasarasa/hello-app)

---

## 👨‍💻 Autor

**Lucas Sarasa**\
🔗 [LinkedIn](https://www.linkedin.com/in/lucassarasa)