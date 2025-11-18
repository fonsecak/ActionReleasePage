# 🚀 GitHub Actions - Release e Deploy Pages

Projeto com GitHub Actions configuradas para criar releases automatizadas e fazer deploy no GitHub Pages.

## 📁 Estrutura do Projeto

```
.github/
├── workflows/
│   ├── release.yml      # Action para criar releases com ZIP
│   └── static.yml       # Action para deploy no GitHub Pages
├── GUIA-GITHUB-ACTIONS.md   # 📚 Guia completo de GitHub Actions
└── GUIA-PIPE-YAML.md        # 📝 Guia sobre operador PIPE (|)
```

## 🎯 O que são GitHub Actions?

GitHub Actions é uma plataforma de **CI/CD** (Integração Contínua/Entrega Contínua) que automatiza tarefas no seu repositório.

### 🔄 Workflows Disponíveis

#### 1️⃣ **Release com Pacote ZIP** (`release.yml`)

**Função:** Cria uma release no GitHub com um arquivo ZIP do código.

**Quando executa:** Manualmente (workflow_dispatch)

**Como usar:**
1. Vá em `Actions` no GitHub
2. Selecione "📦 Criar Release com Pacote ZIP"
3. Clique em "Run workflow"
4. Digite a versão (ex: `v1.0.0`)
5. Clique em "Run workflow"

**O que faz:**
- ✅ Faz checkout do código
- ✅ Cria arquivo ZIP (excluindo .git e .github)
- ✅ Cria tag e release no GitHub
- ✅ Anexa o ZIP na release

#### 2️⃣ **Deploy GitHub Pages** (`static.yml`)

**Função:** Publica automaticamente o site no GitHub Pages.

**Quando executa:**
- Automaticamente em push na branch `main`
- Manualmente via Actions tab

**O que faz:**
- ✅ Faz checkout do código
- ✅ Configura GitHub Pages
- ✅ Faz upload dos arquivos
- ✅ Publica no GitHub Pages

**URL do site:** `https://<usuario>.github.io/<repositorio>/`

## 📚 Documentação Completa

### 📖 Guias Disponíveis

1. **[GUIA-GITHUB-ACTIONS.md](./GUIA-GITHUB-ACTIONS.md)**
   - O que são GitHub Actions
   - Estrutura de workflows
   - Componentes (name, on, jobs, steps)
   - Gatilhos (push, PR, schedule, manual)
   - Jobs paralelos vs sequenciais
   - Actions reutilizáveis mais usadas
   - Secrets e variáveis
   - Exemplos práticos completos
   - Boas práticas

2. **[GUIA-PIPE-YAML.md](./GUIA-PIPE-YAML.md)**
   - O que é o operador PIPE `|`
   - Como escrever múltiplas linhas
   - Diferença entre `|` e `>`
   - Exemplos práticos
   - Comandos condicionais
   - Loops em shell
   - Git operations
   - Docker build
   - Boas práticas

## 🎓 Conceitos Principais

### 📌 Anatomia de um Workflow

```yaml
name: Nome do Workflow          # Nome exibido no GitHub

on:                             # QUANDO executar
  push:                         # Em push
  workflow_dispatch:            # Ou manualmente

jobs:                           # O QUE executar
  nome-do-job:                  # Nome do trabalho
    runs-on: ubuntu-latest      # Onde executar
    steps:                      # Passos do trabalho
      - name: Passo 1
        run: echo "Olá"
```

### 🎬 Gatilhos (Triggers)

| Gatilho | Quando Executa | Exemplo |
|---------|----------------|---------|
| `push` | Em commits | Push na branch main |
| `pull_request` | Em PRs | Abertura de PR |
| `workflow_dispatch` | Manual | Botão "Run workflow" |
| `schedule` | Agendado | Todo dia às 2h |
| `release` | Ao criar release | Publicação de release |

### 💻 Comandos Múltiplos com PIPE

**Sem PIPE (comando único):**
```yaml
run: echo "Uma linha"
```

**Com PIPE (múltiplas linhas):**
```yaml
run: |
  echo "Linha 1"
  echo "Linha 2"
  npm install
  npm test
```

### 🔐 Usando Secrets

**Configurar:**
1. Settings → Secrets and variables → Actions
2. New repository secret
3. Nome: `MEU_SECRET`, Valor: `valor-secreto`

**Usar:**
```yaml
steps:
  - name: Usar secret
    env:
      TOKEN: ${{ secrets.MEU_SECRET }}
    run: echo "Token configurado"
```

## 🛠️ Actions Mais Usadas

### 1. Checkout (Clonar repositório)
```yaml
- uses: actions/checkout@v4
```

### 2. Setup Node.js
```yaml
- uses: actions/setup-node@v4
  with:
    node-version: '20'
    cache: 'npm'
```

### 3. Setup Python
```yaml
- uses: actions/setup-python@v5
  with:
    python-version: '3.11'
    cache: 'pip'
```

### 4. Upload de Artefatos
```yaml
- uses: actions/upload-artifact@v4
  with:
    name: meu-arquivo
    path: dist/
```

### 5. Deploy para GitHub Pages
```yaml
- uses: actions/deploy-pages@v4
```

## 📊 Exemplos de Uso

### Exemplo 1: CI/CD Simples

```yaml
name: CI/CD

on: [push]

jobs:
  test-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Instalar e testar
        run: |
          npm ci
          npm test
      
      - name: Build
        run: npm run build
      
      - name: Deploy
        if: github.ref == 'refs/heads/main'
        run: npm run deploy
```

### Exemplo 2: Testes em Múltiplas Versões

```yaml
name: Testes

on: [push, pull_request]

jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest]
        node: [18, 20]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}
      - run: npm ci
      - run: npm test
```

### Exemplo 3: Deploy Agendado

```yaml
name: Deploy Diário

on:
  schedule:
    - cron: '0 2 * * *'  # Todo dia às 2h

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: |
          echo "Deploy agendado"
          npm run build
          npm run deploy
```

## 🎯 Boas Práticas

### ✅ Faça

- ✅ Use cache para dependências
- ✅ Defina versões específicas de actions
- ✅ Use secrets para dados sensíveis
- ✅ Adicione condicionais para otimizar
- ✅ Use nomes descritivos e emojis
- ✅ Comente código complexo
- ✅ Teste localmente quando possível

### ❌ Não Faça

- ❌ Não commite secrets no código
- ❌ Não use versões `latest` em produção
- ❌ Não execute comandos destrutivos sem condições
- ❌ Não ignore erros de build/teste
- ❌ Não crie workflows muito complexos

## 🔍 Variáveis Úteis do GitHub

```yaml
${{ github.repository }}      # nome/repositorio
${{ github.ref_name }}        # main, develop
${{ github.sha }}             # SHA do commit
${{ github.actor }}           # Usuário que acionou
${{ github.event_name }}      # push, pull_request
${{ github.workspace }}       # Diretório de trabalho
```

## 🚨 Troubleshooting

### Problema: Workflow não executa

**Solução:**
- Verifique se está em `.github/workflows/`
- Verifique sintaxe YAML (indentação!)
- Verifique gatilhos (`on:`)

### Problema: Permissões negadas

**Solução:**
```yaml
permissions:
  contents: write
  pages: write
```

### Problema: Secret não funciona

**Solução:**
- Verifique o nome do secret
- Use `${{ secrets.NOME_CORRETO }}`
- Secrets são case-sensitive

### Problema: Build falha

**Solução:**
```yaml
- run: |
    set -e  # Parar em erro
    npm ci
    npm test
```

## 📚 Recursos Adicionais

- 📖 [Documentação Oficial GitHub Actions](https://docs.github.com/en/actions)
- 🛠️ [Marketplace de Actions](https://github.com/marketplace?type=actions)
- 📝 [GUIA-GITHUB-ACTIONS.md](./GUIA-GITHUB-ACTIONS.md) - Guia completo
- 📝 [GUIA-PIPE-YAML.md](./GUIA-PIPE-YAML.md) - Guia do operador PIPE

## 🤝 Contribuindo

Sinta-se livre para:
- 🐛 Reportar bugs
- 💡 Sugerir melhorias
- 📝 Melhorar documentação
- 🔧 Enviar PRs

## 📝 Licença

Este projeto é open source e está disponível sob a licença MIT.

---

**Desenvolvido com ❤️ usando GitHub Actions**
