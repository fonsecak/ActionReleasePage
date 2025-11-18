# 📝 GUIA COMPLETO: Operador PIPE (|) no YAML

## 🎯 O que é o Operador PIPE?

O **pipe `|`** (barra vertical) é um operador YAML que permite escrever **texto em múltiplas linhas**, preservando quebras de linha. É essencial para executar vários comandos shell sequencialmente em GitHub Actions.

---

## 📚 Operadores de Texto Multi-linha no YAML

### 1️⃣ **Pipe `|` (Literal Block Scalar)**
✅ **Preserva quebras de linha**
✅ **Remove indentação**

```yaml
script: |
  echo "Linha 1"
  echo "Linha 2"
  echo "Linha 3"
```

**Resultado:**
```
echo "Linha 1"
echo "Linha 2"
echo "Linha 3"
```

### 2️⃣ **Greater Than `>` (Folded Block Scalar)**
✅ **Junta linhas em uma só**
✅ **Útil para textos longos**

```yaml
description: >
  Este é um texto muito longo
  que será juntado em uma
  única linha.
```

**Resultado:**
```
Este é um texto muito longo que será juntado em uma única linha.
```

### 3️⃣ **Comparação Direta**

```yaml
# SEM pipe - comando único
run: echo "Olá"

# COM pipe - múltiplas linhas
run: |
  echo "Linha 1"
  echo "Linha 2"

# COM > - texto longo
description: >
  Texto muito longo
  em múltiplas linhas
  mas será uma só
```

---

## 💻 Exemplos Práticos em GitHub Actions

### Exemplo 1: Instalação de Dependências

**❌ SEM PIPE (errado para múltiplos comandos):**
```yaml
- name: Instalar dependências
  run: npm install && npm test && npm run build
  # Difícil de ler e manter
```

**✅ COM PIPE (recomendado):**
```yaml
- name: Instalar dependências
  run: |
    npm install
    npm test
    npm run build
  # Fácil de ler, cada comando em uma linha
```

### Exemplo 2: Build com Logs

```yaml
- name: Build completo com logs
  run: |
    echo "=== Iniciando Build ==="
    echo "Data: $(date)"
    echo "Branch: ${{ github.ref }}"
    
    echo "=== Limpando arquivos antigos ==="
    rm -rf dist/
    mkdir -p dist/
    
    echo "=== Instalando dependências ==="
    npm ci
    
    echo "=== Compilando código ==="
    npm run build
    
    echo "=== Build finalizado ==="
    ls -lah dist/
```

### Exemplo 3: Comandos Condicionais

```yaml
- name: Deploy condicional
  run: |
    if [ "${{ github.ref }}" == "refs/heads/main" ]; then
      echo "Branch principal detectada"
      echo "Fazendo deploy para produção..."
      npm run deploy:prod
    else
      echo "Branch secundária"
      echo "Fazendo deploy para staging..."
      npm run deploy:staging
    fi
```

### Exemplo 4: Loop em Shell Script

```yaml
- name: Processar múltiplos arquivos
  run: |
    # Loop para processar arquivos
    for file in src/*.js; do
      echo "Processando: $file"
      npx eslint "$file"
    done
    
    # Listar todos os arquivos processados
    echo "Total de arquivos:"
    ls src/*.js | wc -l
```

### Exemplo 5: Variáveis e Cálculos

```yaml
- name: Análise de tamanho
  run: |
    # Definir variáveis
    BUILD_DIR="dist"
    MAX_SIZE=5000000  # 5MB em bytes
    
    # Calcular tamanho do build
    BUILD_SIZE=$(du -sb $BUILD_DIR | cut -f1)
    
    # Exibir informações
    echo "Tamanho do build: $BUILD_SIZE bytes"
    echo "Tamanho máximo: $MAX_SIZE bytes"
    
    # Verificar se excede o limite
    if [ $BUILD_SIZE -gt $MAX_SIZE ]; then
      echo "❌ ERRO: Build muito grande!"
      exit 1
    else
      echo "✅ Build dentro do limite"
    fi
```

### Exemplo 6: Criação de Arquivos

```yaml
- name: Criar arquivo de configuração
  run: |
    # Criar arquivo usando heredoc
    cat > config.json << EOF
    {
      "name": "Meu App",
      "version": "1.0.0",
      "environment": "production",
      "api_url": "${{ secrets.API_URL }}"
    }
    EOF
    
    # Exibir conteúdo
    cat config.json
    
    # Validar JSON
    jq . config.json
```

### Exemplo 7: Git Operations

```yaml
- name: Operações Git complexas
  run: |
    # Configurar Git
    git config user.name "GitHub Actions Bot"
    git config user.email "bot@github.com"
    
    # Criar nova branch
    git checkout -b release/${{ github.event.inputs.version }}
    
    # Fazer alterações
    echo "${{ github.event.inputs.version }}" > VERSION
    
    # Commit
    git add .
    git commit -m "Release version ${{ github.event.inputs.version }}"
    
    # Push
    git push origin release/${{ github.event.inputs.version }}
    
    # Criar tag
    git tag -a v${{ github.event.inputs.version }} -m "Release ${{ github.event.inputs.version }}"
    git push origin --tags
```

### Exemplo 8: Docker Build e Push

```yaml
- name: Build e Push Docker
  run: |
    # Build da imagem
    echo "Building Docker image..."
    docker build -t myapp:latest .
    docker build -t myapp:${{ github.sha }} .
    
    # Login no Docker Hub
    echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin
    
    # Tag das imagens
    docker tag myapp:latest ${{ secrets.DOCKER_USERNAME }}/myapp:latest
    docker tag myapp:${{ github.sha }} ${{ secrets.DOCKER_USERNAME }}/myapp:${{ github.sha }}
    
    # Push das imagens
    docker push ${{ secrets.DOCKER_USERNAME }}/myapp:latest
    docker push ${{ secrets.DOCKER_USERNAME }}/myapp:${{ github.sha }}
    
    # Logout
    docker logout
```

### Exemplo 9: Testes com Retry Logic

```yaml
- name: Executar testes com retry
  run: |
    MAX_ATTEMPTS=3
    ATTEMPT=0
    
    while [ $ATTEMPT -lt $MAX_ATTEMPTS ]; do
      ATTEMPT=$((ATTEMPT + 1))
      echo "Tentativa $ATTEMPT de $MAX_ATTEMPTS"
      
      if npm test; then
        echo "✅ Testes passaram!"
        exit 0
      else
        echo "❌ Testes falharam"
        if [ $ATTEMPT -lt $MAX_ATTEMPTS ]; then
          echo "Aguardando 10 segundos antes de tentar novamente..."
          sleep 10
        fi
      fi
    done
    
    echo "❌ Testes falharam após $MAX_ATTEMPTS tentativas"
    exit 1
```

### Exemplo 10: Download e Verificação

```yaml
- name: Download e verificação de arquivo
  run: |
    # Download
    wget https://exemplo.com/arquivo.zip
    
    # Verificar checksum
    echo "Calculando checksum..."
    CHECKSUM=$(sha256sum arquivo.zip | cut -d ' ' -f 1)
    EXPECTED="abc123..."
    
    if [ "$CHECKSUM" == "$EXPECTED" ]; then
      echo "✅ Checksum válido"
      
      # Extrair
      unzip arquivo.zip
      
      # Listar conteúdo
      ls -lah
    else
      echo "❌ Checksum inválido!"
      exit 1
    fi
```

---

## 🎨 Boas Práticas com PIPE

### ✅ DO (Faça)

1. **Use comentários** para explicar comandos complexos
```yaml
run: |
  # Limpar cache antigo
  rm -rf .cache/
  
  # Instalar dependências
  npm ci
```

2. **Organize logicamente** os comandos
```yaml
run: |
  # ETAPA 1: Preparação
  mkdir -p dist/
  
  # ETAPA 2: Build
  npm run build
  
  # ETAPA 3: Validação
  npm run validate
```

3. **Use echo** para logs informativos
```yaml
run: |
  echo "=== Iniciando Deploy ==="
  npm run deploy
  echo "=== Deploy Concluído ==="
```

4. **Verifique erros** com condicionais
```yaml
run: |
  if ! npm test; then
    echo "Testes falharam!"
    exit 1
  fi
```

5. **Defina variáveis** para reutilização
```yaml
run: |
  APP_NAME="meu-app"
  VERSION="${{ github.ref_name }}"
  
  echo "Deploying $APP_NAME version $VERSION"
```

### ❌ DON'T (Não Faça)

1. **Não misture pipe com comandos inline**
```yaml
# ❌ Evite isso
run: |
  npm install
run: npm test  # Conflito!
```

2. **Não ignore tratamento de erros**
```yaml
# ❌ Ruim
run: |
  rm -rf /importante/
  # Sem verificação!

# ✅ Bom
run: |
  if [ -d "/importante/" ]; then
    rm -rf /importante/
  fi
```

3. **Não crie comandos muito longos**
```yaml
# ❌ Difícil de manter
run: |
  # 100+ linhas de código shell
  # Considere criar um script separado
```

---

## 🔧 Alternativas ao PIPE Inline

### Opção 1: Script Externo

**Criar arquivo `.github/scripts/deploy.sh`:**
```bash
#!/bin/bash
set -e  # Sair em caso de erro

echo "=== Deploy Script ==="
npm run build
npm run deploy
echo "=== Deploy Concluído ==="
```

**Usar no workflow:**
```yaml
- name: Executar script de deploy
  run: bash .github/scripts/deploy.sh
```

### Opção 2: Composite Action

**Criar `.github/actions/build/action.yml`:**
```yaml
name: Build Action
description: Build completo do projeto
runs:
  using: composite
  steps:
    - run: npm ci
      shell: bash
    - run: npm run build
      shell: bash
    - run: npm test
      shell: bash
```

**Usar no workflow:**
```yaml
- name: Build
  uses: ./.github/actions/build
```

---

## 🎯 Dicas Avançadas

### 1. **Set -e** para parar em erros
```yaml
run: |
  set -e  # Parar script se qualquer comando falhar
  npm install
  npm test
  npm run build
```

### 2. **Set -x** para debug
```yaml
run: |
  set -x  # Exibir comandos sendo executados
  npm run build
```

### 3. **Variáveis de ambiente no bloco**
```yaml
run: |
  export NODE_ENV=production
  export API_URL="${{ secrets.API_URL }}"
  
  npm run build
  echo "Build para $NODE_ENV concluído"
```

### 4. **Funções shell reutilizáveis**
```yaml
run: |
  # Definir função
  log_info() {
    echo "ℹ️  $1"
  }
  
  log_error() {
    echo "❌ $1"
  }
  
  # Usar função
  log_info "Iniciando build"
  npm run build
  log_info "Build concluído"
```

### 5. **Capturar saída de comandos**
```yaml
run: |
  # Capturar versão do Node
  NODE_VERSION=$(node --version)
  echo "Node version: $NODE_VERSION"
  
  # Capturar e verificar
  BUILD_SIZE=$(du -sh dist/ | cut -f1)
  echo "Build size: $BUILD_SIZE"
```

---

## 📊 Resumo Visual

```
┌─────────────────────────────────────────────────────┐
│           COMANDO ÚNICO vs PIPE                     │
├─────────────────────────────────────────────────────┤
│                                                      │
│  SEM PIPE:                                          │
│  run: echo "comando único"                          │
│                                                      │
│  COM PIPE:                                          │
│  run: |                                             │
│    echo "Linha 1"                                   │
│    echo "Linha 2"                                   │
│    echo "Linha 3"                                   │
│                                                      │
│  EQUIVALENTE A:                                     │
│  $ echo "Linha 1"                                   │
│  $ echo "Linha 2"                                   │
│  $ echo "Linha 3"                                   │
│                                                      │
└─────────────────────────────────────────────────────┘
```

---

## 🎓 Conclusão

O operador **pipe `|`** é essencial para:
- ✅ Escrever múltiplos comandos de forma organizada
- ✅ Melhorar legibilidade do código
- ✅ Facilitar manutenção
- ✅ Adicionar comentários e documentação
- ✅ Criar lógica complexa em workflows

**Use sempre que precisar executar mais de um comando shell!**
