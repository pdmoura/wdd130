# 📂 Documentação: Automação Git Multi-Contas e Chaves SSH (Do Zero)

Este guia documenta como configurar o Git do zero em qualquer máquina para gerenciar infinitos clientes/contas do GitHub de forma isolada. O Git selecionará a identidade (`user.name` / `user.email`) e a chave SSH corretas automaticamente, baseado na URL do repositório remoto, permitindo salvar as pastas em qualquer local do computador.

---

## 🛠️ Passo 1: Configuração Inicial da Máquina Nova

Ao instalar o Git em um computador novo, você precisa abrir o arquivo de configuração global para adicionar os seus dados padrão e o comando automatizador (alias). Escolha uma das opções abaixo para abrir o arquivo.

### Opção A: Abrir pelo Bloco de Notas (Recomendado para Windows)

Execute um dos dois comandos abaixo no seu terminal do PowerShell:

```powershell
# Alternativa 1: Usando a barra normal (mais simples)
notepad $HOME/.gitconfig

# Alternativa 2: Usando a variável nativa do Windows
notepad $env:USERPROFILE\.gitconfig
```

> Como salvar: após colar o código, pressione `Ctrl + S` e feche a janela do Bloco de Notas.

### Opção B: Abrir direto no terminal (interface Vim / padrão multiplataforma)

Execute o comando abaixo para abrir o editor embutido do terminal:

```bash
git config --global --edit
```

1. Pressione `i` para entrar no modo de edição.
2. Quando terminar, pressione `Esc`.
3. Digite `:wq` e pressione `Enter` para salvar e sair.

## Conteúdo a ser colado no arquivo

Cole a estrutura abaixo substituindo os dados iniciais pelos seus dados padrão de uso:

```ini
[user]
    name = SEU_NOME_PADRAO
    email = SEU_EMAIL_PADRAO
[core]
    ignorecase = false

# ==========================================================
# 🤖 SCRIPT DE AUTOMAÇÃO (RODAR APENAS UMA VEZ NA MÁQUINA)
# ==========================================================
[alias]
    novo-cliente = "!f() { \
        username=$1; \
        email=$2; \
        key_name=$3; \
        config_path=\"$HOME/.gitconfig-$username\"; \
        echo \"[user]\n    name = $username\n    email = $email\n[core]\n    sshCommand = \\\"ssh -i $HOME/.ssh/$key_name -F /dev/null\\\"\" > \"$config_path\"; \
        git config --global --add \"includeIf.hasconfig:remote.*.url:git@github.com:$username/**.path\" \"$config_path\"; \
        echo \"Automação criada com sucesso para o cliente $username!\"; \
    }; f"
```

## 🚀 Fluxo de trabalho: adicionando um novo cliente

Sempre que um novo cliente chegar, execute estes 4 passos rápidos no seu terminal. Não é necessário abrir ou editar nenhum arquivo de texto manualmente.

### 1. Gerar a nova chave SSH customizada

Substitua os placeholders pelo e-mail e pelo nome que deseja dar ao arquivo da chave:

```powershell
ssh-keygen -t ed25519 -C "EMAIL_DO_CLIENTE" -f "$HOME/.ssh/NOME_DA_CHAVE_key"
```

Pressione `Enter` em todas as confirmações para deixar a chave sem senha.

### 2. Copiar a chave pública para o GitHub do cliente

Exiba o conteúdo do arquivo público gerado:

```powershell
cat "$HOME/.ssh/NOME_DA_CHAVE_key.pub"
```

Copie todo o texto exibido e cole nas configurações de SSH do GitHub:

`https://github.com/settings/keys`

### 3. Rodar o comando automatizador

Execute o comando criado no Passo 1, passando os 3 parâmetros do cliente na ordem correta (mantenha as aspas duplas):

```powershell
git novo-cliente "USUARIO_GITHUB_DO_CLIENTE" "EMAIL_DO_CLIENTE" "NOME_DA_CHAVE_key"
```

Este comando cria o arquivo de configuração isolado do cliente e o vincula ao Git global instantaneamente.

### 4. Vincular o repositório local e fazer o push

Dentro da pasta do projeto do cliente (em qualquer local do seu disco), certifique-se de usar a URL SSH padrão do GitHub e envie as alterações:

```powershell
git remote set-url origin git@github.com:USUARIO_GITHUB_DO_CLIENTE/NOME_DO_REPOSITORIO.git
git push -u origin main
```

## 🔍 Como o Git se comporta nos bastidores?

Sempre que o comando `git push` for disparado, o Git lê a URL remota do repositório.

Se o link contiver `git@github.com:USUARIO_GITHUB_DO_CLIENTE/`, ele ignora os seus dados padrões globais e carrega dinamicamente o arquivo `~/.gitconfig-USUARIO_GITHUB_DO_CLIENTE`, aplicando o nome, e-mail e a chave SSH exata daquele cliente de forma totalmente transparente e segura.
