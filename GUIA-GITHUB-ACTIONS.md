# 📚 GUIA COMPLETO DE GITHUB ACTIONS

## 📖 Índice
1. [O que são GitHub Actions?](#o-que-são-github-actions)
2. [Estrutura Básica de um Workflow](#estrutura-básica)
3. [Componentes Principais](#componentes-principais)
4. [Gatilhos (Triggers)](#gatilhos-triggers)
5. [Jobs e Steps](#jobs-e-steps)
6. [Actions Reutilizáveis](#actions-reutilizáveis)
7. [Secrets e Variáveis](#secrets-e-variáveis)
8. [Exemplos Práticos](#exemplos-práticos)
9. [Boas Práticas](#boas-práticas)

---

## 🎯 O que são GitHub Actions?

**GitHub Actions** é uma plataforma de **CI/CD (Continuous Integration/Continuous Deployment)** integrada ao GitHub que permite automatizar tarefas no seu projeto.

### Para que serve?
- ✅ **Build e testes** automáticos do código
- 🚀 **Deploy** automático para servidores
- 📦 **Criar releases** e pacotes
- 🔍 **Análise de código** (linting, segurança)
- 📝 **Automações** diversas (notificações, issues, etc)

---

## 🏗️ Estrutura Básica de um Workflow

```yaml
# Nome do workflow (aparece na interface do GitHub)
name: Nome do Workflow

# Quando executar (gatilho/trigger)
on:
  push:
    branches: [main]

# O que executar (trabalhos)
jobs:
  nome-do-job:
    runs-on: ubuntu-latest
    steps:
      - name: Primeiro passo
        run: echo "Olá, mundo!"
```

### 📁 Localização dos Arquivos
- Todos os workflows ficam em: `.github/workflows/`
- Extensão: `.yml` ou `.yaml`
- Você pode ter múltiplos workflows

---

## 🧩 Componentes Principais

### 1️⃣ **NAME** (Nome)
```yaml
name: 🚀 Deploy para Produção
```
- Nome exibido na aba "Actions" do GitHub
- Use emojis para facilitar identificação visual

### 2️⃣ **ON** (Gatilhos/Triggers)
```yaml
on:
  push:              # Quando houver push
  pull_request:      # Quando houver PR
  workflow_dispatch: # Execução manual
  schedule:          # Agendamento (cron)
```

### 3️⃣ **JOBS** (Trabalhos)
```yaml
jobs:
  build:           # Nome do job
    runs-on: ubuntu-latest  # Sistema operacional
    steps:         # Lista de passos
      - name: Primeiro passo
        run: echo "Executando..."
```

### 4️⃣ **STEPS** (Passos)
```yaml
steps:
  # Usar uma action pré-construída
  - uses: actions/checkout@v4
  
  # Executar comandos shell
  - run: npm install
  
  # Comando com múltiplas linhas (usando pipe |)
  - run: |
      echo "Linha 1"
      echo "Linha 2"
      npm test
```

---

## 🎬 Gatilhos (Triggers)

### 📌 **PUSH** - Executar em commits
```yaml
on:
  push:
    branches:
      - main
      - develop
    paths:
      - 'src/**'      # Apenas arquivos em src/
      - '**.js'       # Apenas arquivos .js
```

### 📌 **PULL_REQUEST** - Executar em PRs
```yaml
on:
  pull_request:
    types:
      - opened
      - synchronize   # Novos commits no PR
      - reopened
    branches:
      - main
```

### 📌 **WORKFLOW_DISPATCH** - Execução Manual
```yaml
on:
  workflow_dispatch:
    inputs:
      ambiente:
        description: 'Ambiente para deploy'
        required: true
        type: choice
        options:
          - desenvolvimento
          - homologacao
          - producao
      versao:
        description: 'Versão a ser deployada'
        required: true
        type: string
```

### 📌 **SCHEDULE** - Agendamento (Cron)
```yaml
on:
  schedule:
    # Executa todo dia às 2h da manhã (UTC)
    - cron: '0 2 * * *'
    # Executa toda segunda-feira às 9h
    - cron: '0 9 * * 1'
```

**Formato Cron:** `minuto hora dia mês dia-da-semana`
- `*` = qualquer valor
- `*/15` = a cada 15 unidades
- `0 0 * * 0` = todo domingo à meia-noite

### 📌 **RELEASE** - Em criação de release
```yaml
on:
  release:
    types: [published, created]
```

### 📌 **ISSUES** - Em abertura/fechamento de issues
```yaml
on:
  issues:
    types: [opened, closed]
```

---

## 💼 Jobs e Steps

### 🔄 Jobs Paralelos vs Sequenciais

#### Jobs executam em PARALELO por padrão:
```yaml
jobs:
  testes:
    runs-on: ubuntu-latest
    steps:
      - run: npm test
  
  build:
    runs-on: ubuntu-latest
    steps:
      - run: npm run build
```

#### Jobs SEQUENCIAIS usando `needs`:
```yaml
jobs:
  testes:
    runs-on: ubuntu-latest
    steps:
      - run: npm test
  
  build:
    needs: testes    # Só executa se 'testes' passar
    runs-on: ubuntu-latest
    steps:
      - run: npm run build
  
  deploy:
    needs: [testes, build]  # Aguarda múltiplos jobs
    runs-on: ubuntu-latest
    steps:
      - run: npm run deploy
```

### 🖥️ Runners (Ambientes de Execução)

```yaml
runs-on: ubuntu-latest    # Linux (mais comum)
runs-on: windows-latest   # Windows
runs-on: macos-latest     # macOS
runs-on: ubuntu-20.04     # Versão específica
```

### 🎯 Estratégias de Matriz

Execute o mesmo job em múltiplas configurações:

```yaml
jobs:
  teste:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        node: [14, 16, 18, 20]
        include:
          - os: ubuntu-latest
            node: 20
            experimental: true
    steps:
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}
      - run: npm test
```

Isso cria **12 jobs** (3 OS × 4 versões Node.js)!

---

## 🔧 Actions Reutilizáveis

### ⭐ Actions Mais Usadas

#### 1. **actions/checkout** - Clonar repositório
```yaml
- name: Checkout código
  uses: actions/checkout@v4
  with:
    fetch-depth: 0        # Todo histórico Git
    submodules: true      # Incluir submódulos
    token: ${{ secrets.GITHUB_TOKEN }}
```

#### 2. **actions/setup-node** - Configurar Node.js
```yaml
- name: Configurar Node.js
  uses: actions/setup-node@v4
  with:
    node-version: '20'
    cache: 'npm'          # Cache de dependências
```

#### 3. **actions/setup-python** - Configurar Python
```yaml
- name: Configurar Python
  uses: actions/setup-python@v5
  with:
    python-version: '3.11'
    cache: 'pip'
```

#### 4. **actions/cache** - Cache de dependências
```yaml
- name: Cache de dependências
  uses: actions/cache@v4
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-node-
```

#### 5. **actions/upload-artifact** - Salvar arquivos
```yaml
- name: Salvar artefato de build
  uses: actions/upload-artifact@v4
  with:
    name: dist-files
    path: dist/
    retention-days: 30
```

#### 6. **actions/download-artifact** - Baixar arquivos
```yaml
- name: Baixar artefato
  uses: actions/download-artifact@v4
  with:
    name: dist-files
    path: dist/
```

---

## 🔐 Secrets e Variáveis

### 🔑 Secrets (Dados Sensíveis)

**Criar Secret:**
1. Repositório → Settings → Secrets and variables → Actions
2. New repository secret
3. Nome e valor

**Usar Secret:**
```yaml
steps:
  - name: Deploy para servidor
    run: |
      sshpass -p ${{ secrets.SENHA_SERVIDOR }} ssh user@servidor
    env:
      API_KEY: ${{ secrets.API_KEY }}
      DATABASE_URL: ${{ secrets.DATABASE_URL }}
```

### 🌍 Variáveis de Ambiente

**Nível de Workflow:**
```yaml
env:
  NODE_ENV: production
  API_URL: https://api.exemplo.com

jobs:
  build:
    steps:
      - run: echo $NODE_ENV
```

**Nível de Job:**
```yaml
jobs:
  build:
    env:
      BUILD_MODE: production
    steps:
      - run: echo $BUILD_MODE
```

**Nível de Step:**
```yaml
steps:
  - name: Executar testes
    env:
      TEST_ENV: ci
    run: npm test
```

### 📊 Variáveis Padrão do GitHub

```yaml
steps:
  - run: |
      echo "Repositório: ${{ github.repository }}"
      echo "Branch: ${{ github.ref_name }}"
      echo "SHA do commit: ${{ github.sha }}"
      echo "Ator: ${{ github.actor }}"
      echo "Evento: ${{ github.event_name }}"
      echo "Workspace: ${{ github.workspace }}"
```

---

## 💡 Exemplos Práticos

### 📝 Exemplo 1: CI/CD Node.js Completo

```yaml
name: 🔄 CI/CD Node.js

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  NODE_VERSION: '20'

jobs:
  # JOB 1: Linting e formatação
  lint:
    name: 🔍 Análise de Código
    runs-on: ubuntu-latest
    steps:
      - name: 📥 Checkout
        uses: actions/checkout@v4
      
      - name: 🟢 Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      
      - name: 📦 Instalar dependências
        run: npm ci
      
      - name: 🔍 ESLint
        run: npm run lint
      
      - name: 💅 Prettier
        run: npm run format:check

  # JOB 2: Testes
  test:
    name: 🧪 Testes
    needs: lint
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node: [18, 20]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}
          cache: 'npm'
      - run: npm ci
      - run: npm test
      - name: 📊 Upload cobertura
        uses: codecov/codecov-action@v4
        if: matrix.node == 20

  # JOB 3: Build
  build:
    name: 🏗️ Build
    needs: [lint, test]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      - run: npm ci
      - run: npm run build
      - name: 📤 Upload build
        uses: actions/upload-artifact@v4
        with:
          name: dist
          path: dist/

  # JOB 4: Deploy (só na main)
  deploy:
    name: 🚀 Deploy
    needs: build
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://meusite.com
    steps:
      - name: 📥 Download build
        uses: actions/download-artifact@v4
        with:
          name: dist
      - name: 🚀 Deploy para servidor
        run: |
          echo "Fazendo deploy..."
          # Seus comandos de deploy aqui
```

### 📝 Exemplo 2: Python com Testes e Deploy

```yaml
name: 🐍 CI/CD Python

on: [push, pull_request]

jobs:
  test:
    name: 🧪 Testes Python ${{ matrix.python-version }}
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ['3.9', '3.10', '3.11']
    
    steps:
      - uses: actions/checkout@v4
      
      - name: 🐍 Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}
          cache: 'pip'
      
      - name: 📦 Instalar dependências
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt
          pip install pytest pytest-cov
      
      - name: 🧪 Executar testes
        run: |
          pytest --cov=./ --cov-report=xml
      
      - name: 📊 Upload cobertura
        uses: codecov/codecov-action@v4
        if: matrix.python-version == '3.11'

  docker:
    name: 🐳 Build Docker
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: 🏗️ Build imagem Docker
        run: |
          docker build -t meu-app:latest .
      
      - name: 🚀 Push para Docker Hub
        if: github.ref == 'refs/heads/main'
        run: |
          echo ${{ secrets.DOCKER_PASSWORD }} | docker login -u ${{ secrets.DOCKER_USERNAME }} --password-stdin
          docker push meu-app:latest
```

### 📝 Exemplo 3: Backup Automático Agendado

```yaml
name: 💾 Backup Diário

on:
  schedule:
    # Todo dia às 3h da manhã (UTC)
    - cron: '0 3 * * *'
  workflow_dispatch:  # Permite execução manual

jobs:
  backup:
    name: 💾 Fazer Backup
    runs-on: ubuntu-latest
    
    steps:
      - name: 📥 Checkout
        uses: actions/checkout@v4
      
      - name: 💾 Criar backup
        run: |
          DATE=$(date +%Y-%m-%d)
          tar -czf backup-$DATE.tar.gz .
          echo "Backup criado: backup-$DATE.tar.gz"
      
      - name: ☁️ Upload para S3
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_KEY }}
        run: |
          aws s3 cp backup-*.tar.gz s3://meu-bucket/backups/
      
      - name: 📧 Notificar por email
        if: success()
        uses: dawidd6/action-send-mail@v3
        with:
          server_address: smtp.gmail.com
          server_port: 587
          username: ${{ secrets.EMAIL_USER }}
          password: ${{ secrets.EMAIL_PASS }}
          subject: Backup realizado com sucesso
          body: O backup diário foi concluído.
          to: admin@exemplo.com
```

### 📝 Exemplo 4: Múltiplos Comandos com Pipe (|)

O **pipe `|`** permite escrever múltiplas linhas de comando shell:

```yaml
name: 📦 Build e Deploy Complexo

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: 🏗️ Build completo com múltiplos comandos
        run: |
          # Exibir informações do sistema
          echo "=== Informações do Sistema ==="
          uname -a
          node --version
          npm --version
          
          # Limpar builds anteriores
          echo "=== Limpando builds anteriores ==="
          rm -rf dist/
          rm -rf build/
          
          # Instalar dependências
          echo "=== Instalando dependências ==="
          npm ci
          
          # Executar testes
          echo "=== Executando testes ==="
          npm test
          
          # Build de produção
          echo "=== Build de produção ==="
          npm run build
          
          # Verificar arquivos gerados
          echo "=== Arquivos gerados ==="
          ls -lah dist/
          
          # Calcular tamanho do build
          echo "=== Tamanho do build ==="
          du -sh dist/
      
      - name: 🔍 Análise de bundle
        run: |
          # Instalar ferramenta de análise
          npm install -g source-map-explorer
          
          # Analisar bundles
          source-map-explorer dist/bundle.js --html bundle-report.html
          
          # Exibir estatísticas
          echo "Bundle analisado com sucesso"
          
      - name: 🚀 Deploy para múltiplos ambientes
        if: github.ref == 'refs/heads/main'
        env:
          DEPLOY_KEY: ${{ secrets.DEPLOY_KEY }}
        run: |
          # Deploy para staging
          echo "=== Deploy para Staging ==="
          scp -r dist/* user@staging.exemplo.com:/var/www/
          
          # Aguardar 5 segundos
          sleep 5
          
          # Deploy para produção
          echo "=== Deploy para Produção ==="
          scp -r dist/* user@producao.exemplo.com:/var/www/
          
          # Limpar cache do CDN
          echo "=== Limpando cache do CDN ==="
          curl -X POST https://api.cdn.com/purge \
            -H "Authorization: Bearer ${{ secrets.CDN_TOKEN }}"
          
          echo "✅ Deploy concluído com sucesso!"
```

---

## 🎯 Boas Práticas

### ✅ DO (Faça)

1. **Use cache** para dependências
```yaml
- uses: actions/setup-node@v4
  with:
    cache: 'npm'
```

2. **Defina versões específicas** de actions
```yaml
uses: actions/checkout@v4  # ✅ Bom
uses: actions/checkout@main # ❌ Evite
```

3. **Use secrets** para dados sensíveis
```yaml
env:
  API_KEY: ${{ secrets.API_KEY }}  # ✅ Bom
  API_KEY: '12345'                 # ❌ NUNCA faça isso
```

4. **Adicione condicionais** para otimizar
```yaml
- name: Deploy
  if: github.ref == 'refs/heads/main' && success()
  run: npm run deploy
```

5. **Use nomes descritivos**
```yaml
name: 🧪 Testes de Integração         # ✅ Claro
name: Test                            # ❌ Vago
```

### ❌ DON'T (Não Faça)

1. ❌ Não commite secrets no código
2. ❌ Não use `latest` em versões críticas
3. ❌ Não execute comandos destrutivos sem condições
4. ❌ Não ignore falhas de segurança
5. ❌ Não crie workflows muito complexos (divida em múltiplos)

---

## 🔗 Recursos Adicionais

- 📖 [Documentação Oficial](https://docs.github.com/en/actions)
- 🛠️ [Marketplace de Actions](https://github.com/marketplace?type=actions)
- 💬 [GitHub Community](https://github.community/)
- 🎓 [GitHub Learning Lab](https://lab.github.com/)

---

## 📊 Resumo Visual

```
┌─────────────────────────────────────────────────────────┐
│                    GITHUB ACTIONS                        │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  ┌──────────┐  TRIGGER  ┌──────────┐  EXECUTA  ┌────┐ │
│  │  Evento  │ ────────> │ Workflow │ ────────> │Job │ │
│  └──────────┘           └──────────┘           └────┘ │
│      │                                            │     │
│      │                                            ▼     │
│      │  - push                             ┌──────────┐│
│      │  - PR                               │  Steps   ││
│      │  - schedule                         ├──────────┤│
│      │  - manual                           │ Step 1   ││
│      │  - release                          │ Step 2   ││
│      └─────────────────────────────────>   │ Step 3   ││
│                                             └──────────┘│
└─────────────────────────────────────────────────────────┘
```

---

**🎉 Você agora tem conhecimento para criar GitHub Actions poderosas!**
