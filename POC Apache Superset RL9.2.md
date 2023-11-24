# INSTALAÇÃO DO APACHE SUPERSET NO ROCKY LINUX 9.2

### CRIAÇÃO DA VM

-   Instancia com 2 cores e 4 GB de RAM e 126GB 
-   Utilizando VM com  Hyper-V
-   Usuário com privilégios de root ou sudo.
- ISO Utilizada:  https://download.rockylinux.org/pub/rocky/9/isos/x86_64/Rocky-9.2-x86_64-minimal.iso

### PROVA DE CONCEITO REALIZADA NA SEGUINTE  CONFIGURAÇÃO

**OS E VERSÃO**
```bash
 cat /etc/os-release
 ```
NAME="Rocky Linux"
VERSION="9.2 (Blue Onyx)"

**VERSÃO DO PYTHON UTILIZADA**
```bash
python -V
```
Python 3.9.16

**VERSÃO DO PIP UTLIZADA**
```bash
pip -V
```
pip 23.3.1

**VERSÃO DO APACHE SUPERSET**
```bash
superset version
```
Superset 3.0.1

**VERSÃO DAS DEPENDÊNCIAS**
```bash
superset --version
```
Python 3.9.16
Flask 2.3.3
Werkzeug 2.3.7

**ATUALIZANDO SISTEMA**
***Configuração do Endereço IP com subnet /23***

**CONFIGURAÇÃO DE IP**
```bash
sudo nmcli connection modify My-New-Connection ipv4.address xxx.xxx.xxx.xxx/23
```
**CONFIGURAÇÃO DO GATEWAY**
```bash
sudo nmcli connection modify My-New-Connection ipv4.gateway xxx.xxx.xxx.xxx
```
**CONFIGURAÇÃO DO DNS**
```bash
sudo nmcli connection modify My-New-Connection ipv4.dns "8.8.8.8"  
 ```
**ATUALIZANDO O ROCKY LINUX**
***Instalação dos Pacotes Necessários***
```bash
sudo yum install gcc gcc-c++ libffi-devel python-devel python-pip openssl-devel cyrus-sasl-devel openldap-devel
```

## PREPARANDO O SISTEMA
**INSTALAÇÃO E ATUALIZAÇÃO** 
```bash
sudo yum install python-pip
pip3 install --upgrade pip
pip install --upgrade setuptools pip
sudo yum install mlocate
```
**FERRAMENTAS UTEIS**
```bash
sudo yum install tar curl nano git wget
```
**SERVIDOR HTTP, PROXY REVERSO**
```bash
sudo yum install nginx -y
```
**SERVIDOR HTTP PYTHON WEB SERVER GATEWAY INTERFACE**
```bash
pip3 install gunicorn flask
```


**INSTALANDO GERENCIADO DE PACOTES HOMEBREW**
***Execute e aperte Enter quando solicitado***
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```
***Entre no arquivo de inicialização '.bashrc' e copie e cole  no final dele todas as variáveis abaixo.***

**USANDO O EDITOR 'NANO' PARA REALIZAR AS ALTERAÇÕES**
```bash
sudo nano ~/.bashrc
```
***Copie as linha de variáveis abaixo e cole no final do arquivo '.bashrc.'***
```bash
export PATH=$PATH:/home/linuxbrew/.linuxbrew/bin
```
**VARIÁVEIS DO  DFLAGS E CFLAGS**
```bash
export LDFLAGS="-L$(brew --prefix openssl)/lib"  
export CFLAGS="-I$(brew --prefix openssl)/include"
```
***Salve e Saia do arquivo com Ctrl + X depois Y***

**APLICANDO AS ALTERAÇÕES DE INICIALIZAÇÃO**
```bash
source ~/.bashrc
```
**INSTALANDO AMBIENTE VIRTUAL COM PYENV**
```bash
brew install pyenv
brew install pyenv-virtualenv
```
***Entre no arquivo de inicialização '.bashrc'  novamente  e copie e cole  no final dele as variáveis abaixo.***

```bash
sudo nano ~/.bashrc
```
***Copie as linha de variáveis abaixo e cole no final do arquivo '.bashrc.'***

**VARIÁVEIS DO PYENV**
```bash
export PYENV_ROOT="$HOME/.pyenv"
export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init --path)"
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
```
***Salve e Saia do arquivo com Ctrl + X depois Y***

**APLICANDO AS ALTERAÇÕES DE INICIALIZAÇÃO**
```bash
source ~/.bashrc
```
**CRIANDO AMBIENTE VIRTUAL  CHAMADO "SUPERSET"**
```bash
pyenv virtualenv superset  
```
## ATIVANDO AMBIENTE VIRTUAL  CHAMADO "SUPERSET"
```bash
pyenv activate superset
```
**ATUALIZANDO PIP NO AMBIENTE VIRTUAL**
```bash
pip3 install --upgrade pip
```
**INSTALANDO DEPENDÊNCIAS NO AMBIENTE VIRTUAL**
```bash
pip install sqlparse=='0.4.3'
pip install Pillow
pip install wheel
pip install gevent
```
```bash
sudo yum install gcc libffi-devel python-devel openssl-devel -y
```
**INSTALANDO O PSYCOPG2**
```bash
pip install psycopg2-binary
```
**INSTALANDO APACHE SUPERSET**
```bash
pip install apache-superset
```
## CRIAR BANCO DE DADOS DO SUPERSET
***Entre no arquivo de inicialização novamente, copie e cole as duas variáveis, no final do arquivo '.bashrc ' .***
```bash
sudo nano ~/.bashrc
```
**VARIÁVEL DO FLASK**
```bash
export FLASK_APP=superset
```
**VARIÁVEL SECRET KEY**
```bash
export SUPERSET_SECRET_KEY=`openssl rand -base64 42`
```
**RECAREGUE AS CONFIGURAÇÕES DE INICIALIZAÇÃO**
```bash
source ~/.bashrc
```
**CRIANDO BANCO DE DADOS**
```bash
superset db upgrade
```
**CRIE O USUÁRIO ADMINISTRADOR  E METADADOS DO SUPERSET**
```bash
superset fab create-admin
```
**CARREGANDO ALGUNS EXEMPLOS EXEMPLOS ( OPCIONAL)**
```bash
superset load_examples
```
**CRIAR FUNÇÕES  E PERMISSÕES PADRÕES**
```bash
superset init
```
## ALTERANDO IDIOMA PARA 'pt_BR'  NO ARQUIVO CONFIG.PY
***Use o nano para entrar no arquivo config.py  vá ate a linha 353 e realizar as configurações abaixo.***
```bash
sudo nano /home/<usuario>/.pyenv/versions/superset/lib/python3.9/site-packages/superset/config.py
```
```bash
BABEL_DEFAULT_LOCALE = "pt_BR"
```
***Modifique para 'False' a variável abaixo. Se encontra na linha 1407:***
```bash
TALISMAN_ENABLED = utils.cast_to_boolean(os.environ.get("TALISMAN_ENABLED", False))
```
**APOS ALTERAR REINICIE AS PERMISSÕES DO SUPERSET**
```bash
superset init
```
## LIBERANDO AS PORTAS DO HTTP DO SUPERSET
``` bash
sudo firewall-cmd --add-port=8081/tcp --permanent 
sudo firewall-cmd --add-port=8088/tcp --permanent 
sudo firewall-cmd --reload
```
**ATIVANDO SUPERSET NA INICIALIZAÇÃO**
***Entre no arquivo de inicialização novamente, copie e cole o comando  abaixo , no final do arquivo '.bashrc ' .***
```bash
sudo nano ~/.bashrc
```
**CRIANDO ACESSO HTTP DO SUPERSET**
``` bash
gunicorn -w 10  -k gevent --timeout 120 -b  0.0.0.0:8081 --limit-request-line 0 --limit-request-field_size 0 --statsd-host localhost:8088 "superset.app:create_app()"
```
**RECAREGUE AS CONFIGURAÇÕES DE INICIALIZAÇÃO**
```bash
source ~/.bashrc
```
## SUPERSET COM POSTGRESQL
```bash
https://github.com/gonnyssondias/apche-superset-com-postgres-no-rocky-linux-v9.2.git
```
