
- [Instalando o ansible](#instalando-o-ansible)
  - [Requisitos:](#requisitos)
  - [instalação via pip com virtualenv](#instalação-via-pip-com-virtualenv)
  - [instalação das dependencias via pip](#instalação-das-dependencias-via-pip)
- [Inventário](#inventário)
  - [Criando o arquivo de inventário](#criando-o-arquivo-de-inventário)
  - [Adicionando variáveis no group_vars](#adicionando-variáveis-no-group_vars)
  - [Criptografia com ansible-vault](#criptografia-com-ansible-vault)
    - [Criptografia em variáveis](#criptografia-em-variáveis)
    - [Criptografia em Arquivos](#criptografia-em-arquivos)
    - [Carregamento de variáveis e arquivos criptografados:](#carregamento-de-variáveis-e-arquivos-criptografados)
- [Verificando a documentação dos módulos](#verificando-a-documentação-dos-módulos)
- [Executando comandos ad-hoc](#executando-comandos-ad-hoc)
- [Criando playbook](#criando-playbook)
- [Roles](#roles)


## Instalando o ansible

### Requisitos:
- Máquina com sistema operacional linux e o python3 instalado.

### instalação via pip com virtualenv
 - Criar um virtualenv

```bash
mkdir -p ~/virtualenvs ;  python3 -m venv ~/virtualenvs/ansible-tutorial
```

 - Ativando o virtualenv

```bash
source ~/virtualenvs/ansible-tutorial/bin/activate

```


- Atualizando o pip

```bash
pip install --upgrade pip

```

- Instalando o ansible

```bash
pip install ansible

```

### instalação das dependencias via pip

Instalando as dependencias necessárias para execução dos exemplos neste repositório:

- Faça um clone do repositório:
```
git clone git@github.com:ps-corp-platform/infra-automation-basics.git

```
- Ente no diretório clonado e instale as dependencias:


```bash
cd  infra-automation-basics.git
pip install -r ansible/requirementes.txt

```

## Inventário

### Criando o arquivo de inventário

- Cria a estrutura de diretório que iremos trabalhar

```bash
mkdir -p ~/ansible-tutorial/{inventory,roles}

```

```
/home/user/ansible-tutorial/
├── inventory
└── roles

```

- Crie um arquivo dentro do diretório criado para o inventário:

```
touch ~/ansible-tutorial/inventory/hosts
```

Adicione o conteúdo  abaixo no arquivo:

```
localhost ansible_connection=local

[tutorial]
localhost

[prod]
prd-server1
prd-server2

[dev]
dev-server1
dev-server2

[prod:vars]
ntp=ntp.prod.intranet

[dev:vars]

ntp=ntp.dev.intranet

[all:vars]
ansible_python_interpreter=/usr/bin/python3

```

- Execute o comando abaixo para listar o inventário:

```
ansible-inventory -i  ~/ansible-tutorial/inventory/hosts --list

```
saida do comando acima
```
{
    "_meta": {
        "hostvars": {
            "dev-server1": {
                "ansible_python_interpreter": "/usr/bin/python3",
                "ntp": "ntp.dev.intranet"
            },
            "dev-server2": {
                "ansible_python_interpreter": "/usr/bin/python3",
                "ntp": "ntp.dev.intranet"
            },
            "localhost": {
                "ansible_connection": "local",
                "ansible_python_interpreter": "/usr/bin/python3"
            },
            "prd-server1": {
                "ansible_python_interpreter": "/usr/bin/python3",
                "ntp": "ntp.prod.intranet"
            },
            "prd-server2": {
                "ansible_python_interpreter": "/usr/bin/python3",
                "ntp": "ntp.prod.intranet"
            }
        }
    },
    "all": {
        "children": [
            "tutorial",
            "dev",
            "prod",
            "ungrouped"
        ]
    },
    "tutorial": {
        "hosts": [
            "localhost"
        ]
    },
    "dev": {
        "hosts": [
            "dev-server1",
            "dev-server2"
        ]
    },
    "prod": {
        "hosts": [
            "prd-server1",
            "prd-server2"
        ]
    }
}

```

### Adicionando variáveis no group_vars


- Criar diretório do grupo:

```bash
mkdir -p ~/ansible-tutorial/inventory/group_vars/tutorial
```
- Criar arquivo de variável:

```bash

echo "myvar: teste" > ~/ansible-tutorial/inventory/group_vars/tutorial/my-vars.yaml

```

Verifica variável carregada apenas para os hots do grupo tutorial:


```
            "dev-server2": {
                "ansible_python_interpreter": "/usr/bin/python3",
                "ntp": "ntp.dev.intranet"
            },
            "localhost": {
                "ansible_connection": "local",
                "ansible_python_interpreter": "/usr/bin/python3",
                "myvar": "teste"
            },

```

### Criptografia com ansible-vault

#### Criptografia em variáveis
- Criptografar string usando ansible-vault

```
ansible-vault encrypt_string minhacredencial

```
- Digite e confirme a senha que será utilizada para criptografia e posteriormente será exibido o conteudo criptografado:

- Copie o conteúdo e adicione nos arquivos de variáveis de inventário conforme abaixo:

deverá ficar da seguinte forma:

```yaml

mysecret: !vault |
            $ANSIBLE_VAULT;1.1;AES256
            37353132313061393836383835666235343230633434373938616237336330396562333162396436
            6434303464353739653430666665356330383830623438630a373432326434333737623639386163
            32336339643863353566663833626263636631336661373132646134643761333438623239353966
            3839353837316130370a303532383635653933316133366233396362663933306133316336386266
            6638

```
#### Criptografia em Arquivos
- Execute o comando abaixo e coloque a senha

```bash
 ansible-vault encrypt ~/ansible-tutorial/inventory/group_vars/tutorial/my-vars.yaml
```

- Veja o conteúdo criptografado.
```
 cat ~/ansible-tutorial/inventory/group_vars/tutorial/my-vars.yaml
$ANSIBLE_VAULT;1.1;AES256
32303861613166353230666139366138623330666466346462303162613366633362333065303239
6465646462306164613137326132386336353362663939340a396431356535363462623466303365
36613838636665356565353531613733323766633831656166656363386633343934383135393832
3334653634376239310a373266363530343033636162626230316638353136343233386365306234
3537
```

#### Carregamento de variáveis e arquivos criptografados:

Para que o ansible consiga carregar o conteúdo das variáveis e arquivos criptografados é necessário informar a senha, ou um arquivo contendo a senha:

- Informar a senha durante execução:
```bash
ansible -i  ~/ansible-tutorial/inventory/hosts "localhost" -m ping --ask-vault-password
```

- Informar arquivo contendo a senha:

```bash
ansible -i  ~/ansible-tutorial/inventory/hosts "localhost" -m ping --vault-password-file ~/.vault-password
```

- Informar uma senha padrão para utilização do ansible durante criptografia e descriptografia:

```
### Via variável de ambiente
ANSIBLE_VAULT_PASSWORD_FILE=~/.vault-password

```

cat /etc/ansible/ansible.cfg
```
#arquivo global de configuração do ansible
[defaults]
deprecation_warnings=False
vault_password_file=~/.ansible-vault

```



## Verificando a documentação dos módulos

- Listar os módulos disponíveis

```bash
ansible-doc -l
```

- Exibir a documentação de um módulo específico


```bash
#ansible-doc module_name
ansible-doc copy
```



## Executando comandos ad-hoc

Usando o modulo shell para executar comandos

```bash
#ansible -i inventory -m module_name -a module_parameters
ansible -i  ~/ansible-tutorial/inventory/hosts 'localhost' -m shell -a "uname -r"
```

Usando o modulo copy

```bash
#ansible -i inventory -m module_name -a module_parameters
ansible -i  ~/ansible-tutorial/inventory/hosts 'localhost' -m copy -a "content='ansible-tutorial-adhoc' dest=/tmp/ansible-tutorial.txt"
```

## Criando playbook

- Crie o arquivo de playbook conforme abaixo:
```
touch ~/ansible-tutorial/playbook.yaml
```

- Adicione conteúdo abaixo ao arquivo e salve:

```yaml
---
- name: Primeiro Playbook de teste
  hosts: tutorial

  tasks:
    - name: copia um arquivo
      copy:
        content: ansible-tutorial-playbook
        dest: /tmp/ansible-tutorial.txt

    - name: copia arquivos em loop
      copy:
        content: ansible-tutorial-playbook
        dest: /tmp/ansible-tutorial.txt

```
- Executando o primeiro playbook

```bash
ansible-playbook -i  ~/ansible-tutorial/inventory/hosts ~/ansible-tutorial/playbook.yaml
```

##  Roles

- Criando o esqueleto de uma role:

```bash
ansible-galaxy init role_name
```

- Usando role de outros repositorios:

```bash
ansible-galaxy install git+https://github.com/geerlingguy/ansible-role-apache.git,0b7cd353c0250e87a26e0499e59e7fd265cc2f2
```