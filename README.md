# Automação do deploy com Ansible

Este repositório contém a automação do deploy do site da Vaga Lume feito
com [Ansible][ansible] para um servidor rodando apache.

Existe somente um script de deploy, e o que muda é o endereço onde irá ser
realizada a implantação, da sua máquina local até o servidor de
produção.

Essa automação foi criada para ser usada com GoCD.

## Dependências

É necessário instalar o [Ansible][ansible] no seu computador para utilizar as
automações aqui definidas. Para isso, você pode seguir as instruções encontradas
na [documentação oficial][ansible-install].

O Ansible é um mecanismo de automação de TI que possui diversos usos:

- gestão da configuração dos seus servidores
- implantação automática de aplicações
- orquestração de serviços
- provisionamento em nuvem
- etc

Se você quiser testar o deploy localmente, irá precisar do [Docker][docker] instalado.
Para isso, siga os passos definidos na [documentação oficial][docker-install] na sua
máquina.

O Docker é um jeito de criar aplicações dentro de containers, onde todas
as dependências já estão instaladas na imagem. Nós criamos uma imagem
com o apache já instalado, assim não é necessário instalá-lo na sua
máquina.

## Como fazer o deploy localmente

Fizemos uma imagem de docker com apache e ssh justamente para fazer esse
deploy localmente. Ela está definida em [associacao-vagalume/apache-rsync][apache-rsync]
e pode ser encontrada no [Docker Hub][apache-rsync-image].

Preparação:

1. Baixe a imagem de docker

        docker pull pevangelista/apache-rsycn

2. Crie um par de chaves ssh, caso não tenha

        ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

3. Adicione a seguinte linha no seu arquivo de hosts:

    127.0.0.1   development

4. Tenha o conteúdo do site localizado em `../_site`

5. Garanta que o site possua o arquivo `../_site/VERSAO.txt`

Para fazer o deploy localmente, siga os seguintes passos:

1. Crie um container de docker temporário

        docker run --rm -d \
          -p 8080:80 -p 2222:22 \
          -e SSH_PUBKEY="$(cat ~/.ssh/id_rsa.pub)" \
          --name development pevangelista/apache-rsync

2. Execute o playbook ansible, onde `<VERSAO>` é o conteúdo do arquivo
   `../_site/VERSAO.txt`

        GO_PIPELINE_LABEL=<VERSAO> ansible-playbook playbook_deploy.yml

3. Acesse o site em http://development:8080

## Servidores / Inventories

Este repositório define os seguintes servidores:

- **development** - Um servidor local com acesso ssh na porta 2222 e
  usuário `ssh_user`
- **acceptance** - O servidor que está disponível na pipeline do Go,
  acessível na porta padrão com usuário `ssh_user`
- **staging** - O servidor de staging da Vaga Lume, com a configuração
  protegida por senha
- **production** - O servidor de produção da Vaga Lume, com a
  configuracão protegida por senha.

Para ter acesso a staging e produção, será necessário configurar o
acesso por meio de chave diretamente.

Se nenhum inventory for definido, o ansible irá utilizar `development`
por padrão. Para escolher outro inventory, execute o comando acima
adicionando a opção `-i inventory_<ambiente>`.

[ansible]: https://www.ansible.com/
[ansible-install]: http://docs.ansible.com/ansible/latest/intro_installation.html
[apache-rsync]: https://github.com/associacao-vagalume/apache-rsync
[apache-rsync-image]: https://hub.docker.com/r/pevangelista/apache-rsync
[docker]: https://www.docker.com/get-docker#/overview
[docker-install]: https://docs.docker.com/engine/installation/
