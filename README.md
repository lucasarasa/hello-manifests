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
├── manifests/
│   ├── deployment.yaml     # Manifesto de Deployment da aplicação
│   └── service.yaml        # Manifesto de Service da aplicação
└── README.md               # Este documento
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

### Criando a aplicação no ArgoCD

Use o comando abaixo para criar a aplicação no ArgoCD:

```bash
argocd app create hello-app \
  --repo https://github.com/SEU-USUARIO/SEU-REPOSITORIO.git \
  --path manifests \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default
```

### Sincronizando a aplicação

Após criar a aplicação, sincronize os manifests com o cluster:

```bash
argocd app sync hello-app
```

---

## 🔐 Acesso via SSH no ArgoCD

### ▶️ Usando repositório público (mais simples)

Se o repositório `hello-manifest` for **público**, o ArgoCD pode acessá-lo diretamente. Para criar a aplicação, use o comando:

```bash
argocd app create hello-app \
  --repo https://github.com/SEU-USUARIO/SEU-REPOSITORIO.git \
  --path manifests \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default
```

Em seguida, sincronize:

```bash
argocd app sync hello-app
```

### 🔐 Usando repositório **privado** com chave SSH

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

Agora, o repositório está configurado para ser acessado de forma segura pelo ArgoCD e sincronizar automaticamente os manifests com o cluster

---

## 📂 Relacionamento com o Repositório de Código

Este repositório trabalha em conjunto com o repositório `hello-app`, que contém o código da aplicação e a pipeline CI/CD.

Link para o repositório de código: [https://github.com/lucasarasa/hello-app](https://github.com/lucasarasa/hello-app)

---

## 👨‍💻 Autor

**Lucas Sarasa**\
🔗 [LinkedIn](https://www.linkedin.com/in/lucassarasa)