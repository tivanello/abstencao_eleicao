# Comandos básicos – Ambiente Virtual (Python) + Git/GitHub (Windows)

## 1) Ambiente virtual (venv)

### Criar o ambiente virtual (na raiz do projeto)
    py -m venv venv

### Ativar o ambiente
PowerShell:
    .\venv\Scripts\Activate.ps1

CMD:
    venv\Scripts\activate

### Atualizar pip (recomendado)
    py -m pip install --upgrade pip

### Instalar dependências (quando você já tem `requirements.txt`)
    py -m pip install -r requirements.txt

### Gerar `requirements.txt` (para “subir o ambiente” do jeito certo)
    py -m pip freeze > requirements.txt

### Desativar o ambiente
    deactivate

### Importante: não subir a pasta `venv/` para o Git
No `.gitignore`, garanta:
    venv/
    .venv/

---

## 2) Git – Criar repositório local e conectar no GitHub

### Entrar na pasta do projeto
    cd C:\projetos\nome_do_projeto

### Inicializar o Git (primeira vez)
    git init

### Ver status do repositório
    git status

### Definir branch principal como main
    git branch -M main

### Conectar no repositório remoto (GitHub)
    git remote add origin https://github.com/USUARIO/NOME_DO_REPO.git

### Verificar remotos configurados
    git remote -v

---

## 3) Primeiro envio (push) para o GitHub

### Adicionar tudo, commitar e enviar
    git add .
    git commit -m "Primeiro commit"
    git push -u origin main

---

## 4) Fluxo do dia a dia (máquina → GitHub)

### Ver o que mudou (antes de commitar)
    git status

### Ver diferenças (o que foi alterado)
    git diff

### Adicionar alterações e commitar
    git add .
    git commit -m "Descrição clara da alteração"

### Enviar para o GitHub
    git push

---

## 5) Verificar atualizações (GitHub ↔ máquina)

### Ver se há commits no GitHub que você ainda não tem (sem alterar nada local)
    git fetch
    git status

Se aparecer algo como “your branch is behind”, você está desatualizado localmente.

### Trazer atualizações do GitHub para a sua máquina (padrão)
    git pull

### Trazer atualizações do GitHub sem “sujar” o histórico (recomendado)
    git pull --rebase

### Ver se sua máquina tem commits que ainda não foram enviados (antes do push)
    git status

Se aparecer “your branch is ahead”, você tem commits locais pendentes de `git push`.

---

## 6) Criar pasta no repositório (GitHub)

### Regra do Git
Git NÃO versiona pasta vazia.  
Para a pasta existir no GitHub, ela precisa ter pelo menos 1 arquivo.

### Criar a pasta `notebooks` e garantir que ela suba (mesmo vazia)
PowerShell:
    New-Item -ItemType Directory notebooks
    New-Item -ItemType File notebooks\.gitkeep
    git add notebooks\.gitkeep
    git commit -m "Cria pasta notebooks"
    git push

### Se a pasta já existe e tem arquivos (ex.: .ipynb)
    git add notebooks
    git commit -m "Adiciona notebooks"
    git push

---

## 7) Se algo não aparece no GitHub (pasta/arquivo)

### Ver se está sendo ignorado pelo `.gitignore`
    git status --ignored

### Descobrir exatamente qual regra está ignorando um arquivo/pasta
    git check-ignore -v notebooks
    git check-ignore -v notebooks\arquivo.ipynb

### Forçar adicionar mesmo ignorado (usar com cuidado)
    git add -f notebooks
    git commit -m "Adiciona notebooks (forçado)"
    git push

---

## 8) Caso comum: push rejeitado (remoto tem coisas que você não tem)

### Resolver trazendo do GitHub e depois enviando
    git pull --rebase origin main
    git push -u origin main

---

## 9) Comandos rápidos de consulta

### Ver branch atual
    git branch

### Ver histórico resumido
    git log --oneline --decorate -10

### Ver o que está “staged” (pronto para commit)
    git diff --staged
