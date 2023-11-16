
# Instalação do Apache Superset no Rocky Linux 9.1

## Criação da VM

-   Instancia com 2 cores e 4 GB de RAM e 126GB 
-   Utilizando VM com  Hyper-V
-   Usuário com privilégios de root ou sudo.
- ISO Utilizada:  Rocky-9.1-20221215.1-x86_64-minimal

## Prova de Conceito Realizada na Seguinte  Configuração
```bash
#OS e Versão
 cat /etc/os-release
NAME="Rocky Linux"
VERSION="9.2 (Blue Onyx)"
```
```bash
#Versão do Python Utilizada
python -V
Python 3.9.16
```
```bash
#Versão do Pip Utlizada
pip -V
pip 23.3.1
```
```bash
#Versão do Apache Superset
superset version
Superset 3.0.1

#Versão das dependências
superset --version
Python 3.9.16
Flask 2.3.3
Werkzeug 2.3.7
```
## Atualizando Sistema
**Configuração do Endereço IP com subnet /23**
```bash
#Configuração de IP 
sudo nmcli connection modify My-New-Connection ipv4.address xxx.xxx.xxx.xxx/23
```
**Configuração do Gateway**
```bash
#Configuração do Gateway 
sudo nmcli connection modify My-New-Connection ipv4.gateway xxx.xxx.xxx.xxx
```
**Configuração do DNS**
```bash
#Configuração do DNS
sudo nmcli connection modify My-New-Connection ipv4.dns "8.8.8.8"  
 ```
**Atualizando o Rocky Linux**
```bash
#Atualização dos Pacotes  
sudo yum update -y 
```
## Preparando o Sistema
```bash
#Instalação e Atualização 
sudo yum install python-pip
pip3 install --upgrade pip
pip install --upgrade setuptools pip
sudo yum install mlocate
sudo updatedb
```
**Ferramentas Uteis**
```bash
sudo yum install tar curl nano git wget
```
**Instalando Gerenciado de Pacotes Homebrew**
```bash
#Execute e aperte Enter quando solicitado
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Entre no arquivo de inicialização.
sudo nano ~/.bashrc
# Copie a linha abaixo e cole no final do arquivo .bashrc.

export PATH=$PATH:/home/linuxbrew/.linuxbrew/bin
*Salve e Saia do arquivo com Ctrl + X

# Agora aplique as alterações
source ~/.bashrc
```
**Definindo as Variáveis  DFLAGS e CFLAGS para que determinados pacotes Python sejam compilados corretamente.**
```bash
export LDFLAGS="-L$(brew --prefix openssl)/lib"  
export CFLAGS="-I$(brew --prefix openssl)/include"
```

**Instalando Servidores WEB **
```bash
# Servidor HTTP, proxy reverso
sudo yum install nginx -y

# Servidor HTTP Python Web Server Gateway Interface
pip3 install gunicorn flask
```
**Instalando Ambiente Virtual com PYENV**
```bash
# Atualizando Brew
brew update

# Instalando pyenv
brew install pyenv
brew install pyenv-virtualenv

# Criando Ambiente Virtual "Superset"
pyenv virtualenv superset  

# Variaveis do pyenv
export PYENV_ROOT="$HOME/.pyenv"
export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init --path)"
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"

# Ativando Ambiente Virtual "Superset"
pyenv activate superset

# Atualizando PIP no Ambiente Virtual
pip3 install --upgrade pip

# Caso o pip nao associe o camaniho
sudo ln -s /usr/bin/python3 /usr/bin/python
```
**Instalando Dependências no Ambiente Virtual**
```bash
pip install sqlparse=='0.4.3'
pip install Pillow
pip3 install wheel
pip install gevent
sudo dnf install -y gcc gcc-c++ libffi-devel python-devel python-pip openssl-devel openldap-devel
```
**Instalando Apache Superset**
```bash
pip install apache-superset
```
**Criar Banco de Dados do Superset**
```bash
# Ativando Variavel Flask
export FLASK_APP=superset

# Ativando Secret Key
export SUPERSET_SECRET_KEY=`openssl rand -base64 42`

# Recaregue a configuração
source .bashrc

# Criando Banco de Dados
superset db upgrade
```
**Criar Usuário Administrador  e Meta dados do Superset**
```bash
superset fab create-admin
```
**Carregando alguns exemplos exemplos ( Opcional)**
```bash
superset load_examples
```
**Criar funções  e permissões padrões**
```bash
superset init
```
**Alterando Idioma do Superset**
```bash
#Alterando Idioma para Portugues Brasileiro no Config.py linha 350.
sudo nano /home/<usuario>/.pyenv/versions/superset/lib/python3.9/site-packages/superset/config.py
BABEL_DEFAULT_LOCALE = "pt_BR"
```
**Observações: Caso a logon do Superset fique em loop**
```bash
# Verifique se as Variaveis do Arquivo Config.py são as Seguintes: 

TALISMAN_ENABLED = utils.cast_to_boolean(os.environ.get("TALISMAN_ENABLED", False))

# Apos Alterar Reinicie o Superset
superset init
```
**Criando acesso http do Superset**
``` bash
sudo firewall-cmd --add-port=8081/tcp --permanent 
sudo firewall-cmd --add-port=8088/tcp --permanent 
sudo firewall-cmd --reload
```
**Verificando Dependências Instaladas** 
```bash
pip list

Package                   Version
------------------------- ------------
alembic                   1.12.1
amqp                      5.1.1
apache-superset           3.0.1
apispec                   6.3.0
async-timeout             4.0.3
attrs                     23.1.0
Babel                     2.13.1
backoff                   2.2.1
bcrypt                    4.0.1
billiard                  4.1.0
blinker                   1.7.0
Brotli                    1.1.0
cachelib                  0.10.2
celery                    5.3.4
certifi                   2023.7.22
cffi                      1.16.0
click                     8.1.7
click-didyoumean          0.3.0
click-option-group        0.5.6
click-plugins             1.1.1
click-repl                0.3.0
colorama                  0.4.6
convertdate               2.4.0
cron-descriptor           1.4.0
croniter                  2.0.1
cryptography              39.0.2
Deprecated                1.2.14
deprecation               2.1.0
dnspython                 2.4.2
email-validator           1.3.1
exceptiongroup            1.1.3
Flask                     2.3.3
Flask-AppBuilder          4.3.9
Flask-Babel               2.0.0
Flask-Caching             1.11.1
Flask-Compress            1.14
Flask-JWT-Extended        4.5.3
Flask-Limiter             3.5.0
Flask-Login               0.6.3
Flask-Migrate             3.1.0
Flask-SQLAlchemy          2.5.1
flask-talisman            1.1.0
Flask-WTF                 1.2.1
func-timeout              4.3.5
geographiclib             2.0
geopy                     2.4.0
gevent                    23.9.1
greenlet                  3.0.1
gunicorn                  21.2.0
h11                       0.14.0
hashids                   1.3.1
hijri-converter           2.3.1
holidays                  0.23
humanize                  4.8.0
idna                      3.4
importlib-metadata        6.8.0
importlib-resources       6.1.0
isodate                   0.6.1
itsdangerous              2.1.2
Jinja2                    3.1.2
jsonschema                4.19.2
jsonschema-specifications 2023.7.1
kombu                     5.3.2
korean-lunar-calendar     0.3.1
limits                    3.6.0
Mako                      1.2.4
Markdown                  3.5.1
markdown-it-py            3.0.0
MarkupSafe                2.1.3
marshmallow               3.20.1
marshmallow-sqlalchemy    0.26.1
mdurl                     0.1.2
msgpack                   1.0.7
nh3                       0.2.14
numpy                     1.23.5
ordered-set               4.1.0
outcome                   1.3.0.post0
packaging                 23.2
pandas                    1.5.3
paramiko                  3.3.1
parsedatetime             2.6
pgsanity                  0.2.9
Pillow                    10.1.0
pip                       23.3.1
polyline                  2.0.1
prison                    0.2.1
prompt-toolkit            3.0.39
psycopg2                  2.9.9
pyarrow                   12.0.1
pycparser                 2.21
Pygments                  2.16.1
PyJWT                     2.8.0
PyMeeus                   0.5.12
PyNaCl                    1.5.0
pyparsing                 3.1.1
PySocks                   1.7.1
python-dateutil           2.8.2
python-dotenv             1.0.0
python-geohash            0.8.5
pytz                      2023.3.post1
PyYAML                    6.0.1
redis                     4.6.0
referencing               0.30.2
rich                      13.6.0
rpds-py                   0.12.0
selenium                  4.9.1
setuptools                68.2.2
shortid                   0.1.2
simplejson                3.19.2
six                       1.16.0
slack-sdk                 3.23.0
sniffio                   1.3.0
sortedcontainers          2.4.0
SQLAlchemy                1.4.50
SQLAlchemy-Utils          0.38.3
sqlparse                  0.4.4
sshtunnel                 0.4.0
tabulate                  0.8.10
trio                      0.23.1
trio-websocket            0.11.1
typing_extensions         4.8.0
tzdata                    2023.3
urllib3                   2.0.7
vine                      5.0.0
wcwidth                   0.2.9
Werkzeug                  2.3.7
wrapt                     1.15.0
wsproto                   1.2.0
WTForms                   3.1.1
WTForms-JSON              0.3.5
XlsxWriter                3.0.9
zipp                      3.17.0
zope.event                5.0
zope.interface            6.1

```

**Criando acesso http do Superset**
``` bash
gunicorn -w 10  -k gevent --timeout 120 -b  0.0.0.0:8081 --limit-request-line 0 --limit-request-field_size 0 --statsd-host localhost:8088 "superset.app:create_app()"

```
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEyNjAxNzc0MTBdfQ==
-->