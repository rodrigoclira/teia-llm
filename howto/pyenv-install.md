# Instalação do pyenv


```bash
# Dependências (Ubuntu/Debian)
sudo apt install -y build-essential libssl-dev zlib1g-dev libbz2-dev \
  libreadline-dev libsqlite3-dev libncursesw5-dev xz-utils tk-dev \
  libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev

# Instalar pyenv
curl https://pyenv.run | bash
```

Adicionar ao ~/.bashrc:

```
export PYENV_ROOT="$HOME/.pyenv"
export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init -)"
```

Uso

```
# Ver versões disponíveis
pyenv install --list | grep " 3\."

# Instalar Python 3.12
pyenv install 3.12.10

# Listar versões instaladas
pyenv versions

# Definir versão local (só para a pasta atual)
cd /home/rcls/codigos/github/ifpe/teia-llm
pyenv local 3.12.10

# Criar venv com a versão certa
python -m venv venv312
source venv312/bin/activate
pip install gensim
```

O pyenv local cria um arquivo .python-version na pasta — só afeta aquele diretório. O sistema continua usando o Python padrão normalmente.

Depois de criar o venv, é só selecionar o novo kernel no VS Code (canto inferior direito do notebook).