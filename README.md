# Docker-GLPI-Traefik-Portainer-SSL

[![Docker Hub](https://img.shields.io/docker/v/my-docker-image.svg?style=flat-square)](https://hub.docker.com/r/jonas556440)

[![Imagem GLPI](https://img.shields.io/docker/pulls/my-docker-image.svg?style=flat-square)](https://hub.docker.com/r/jonas556440/glpi)

# Configurando um Ambiente Local Docker com GLPI, Portainer e Traefik (SSL)

Neste tutorial, você aprenderá como configurar um ambiente local Docker com as seguintes ferramentas:

- **GLPI**: Um sistema de gerenciamento de ativos e chamados de código aberto.
- **Portainer**: Uma interface de gerenciamento de contêineres Docker.
- **Traefik**: Um proxy reverso com suporte a SSL para roteamento de tráfego.

<img src="https://glpi-project.org/wp-content/uploads/2022/01/assets-2.png" alt="GLPI" width="600" />

A configuração incluirá a utilização de certificados SSL autoassinados para proteger as conexões HTTPS.

## Requisitos

Para seguir este tutorial, você precisará dos seguintes requisitos:

- Um servidor Ubuntu 22.
- Acesso ao servidor como superusuário ou com privilégios sudo.

## Passo 1: Instalando o Docker

Começaremos instalando o Docker e suas dependências. Execute os seguintes comandos:
```sh
sudo su -
cd /
```

```sh
sudo apt update
sudo apt upgrade -y
sudo add-apt-repository http://us.archive.ubuntu.com/ubuntu jammy-updates multiverse
sudo add-apt-repository http://us.archive.ubuntu.com/ubuntu jammy-backports main restricted universe multiverse
sudo apt install -y lsb-release apt-transport-https ca-certificates curl gnupg software-properties-common -y
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io
sudo systemctl start docker
sudo systemctl enable docker
```

## Passo 2: Instalando o Docker Compose

Agora, instalaremos o Docker Compose para facilitar a criação e gerenciamento de contêineres. Siga as instruções abaixo:

```sh
sudo curl -L https://github.com/docker/compose/releases/download/v2.21.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo docker-compose --version
```

## Passo 3: Criar uma Rede para os Contêineres

Crie uma rede Docker chamada `proxy` para permitir a comunicação entre os contêineres Traefik, GLPI e Portainer. Use o seguinte comando:

```sh
sudo docker network create proxy
```

## Passo 4: Clone o Repositorio
Agora, clone o repositório com os arquivos de configuração necessários para a pasta root raiz de seu servidor Ubuntu:

```sh
cd ~/
git clone https://github.com/jonas556440/Docker-GLPI-Traefik-Portainer-SSL.git
```
Isso criará os diretórios e arquivos necessario em sua pasta root.

## Passo 5: Gerar Certificados SSL Autoassinados

Agora, vamos criar certificados SSL autoassinados para cada subdomínio que você deseja configurar (por exemplo, `glpi.localhost.local`, `portainer.localhost.local`, `traefik.localhost.local`). Siga as instruções interativas e configure os certificados para cada domínio:

```sh
sudo mkdir -p ~/Docker-GLPI-Traefik-Portainer-SSL/Traefik/ssl_certificates
cd ~/Docker-GLPI-Traefik-Portainer-SSL/Traefik/ssl_certificates
openssl req -new -newkey rsa:4096 -days 365 -nodes -x509 -keyout glpi.localhost.local.key -out glpi.localhost.local.crt
openssl req -new -newkey rsa:4096 -days 365 -nodes -x509 -keyout portainer.localhost.local.key -out portainer.localhost.local.crt
openssl req -new -newkey rsa:4096 -days 365 -nodes -x509 -keyout traefik.localhost.local.key -out traefik.localhost.local.crt
```

## Passo 6: Iniciando os Contêineres
Agora, vamos iniciar os contêineres para as diferentes partes do ambiente. Certifique-se de estar no diretório inicial (~) antes de executar esses comandos:

```sh
cd ~/Docker-GLPI-Traefik-Portainer-SSL
docker-compose -f Traefik/docker-compose-traefik.yml up -d
docker-compose -f mariadb_glpi/docker-compose-mariadb.yml up -d
docker-compose -f GLPI/docker-compose-glpi.yml up -d
docker-compose -f Portainer/docker-compose-portainer.yml up -d
```
## Passo 7: Aponte os dominios para o IP do servidor

Certifique-se de que os domínios que você usará com seus certificados SSL autoassinados sejam direcionados para o IP do servidor onde o Docker e seus contêineres estão sendo executados. Isso pode ser feito de duas maneiras:

### Opção 1: Usando um Servidor de DNS Interno

Uma abordagem é configurar um servidor DNS interno em sua rede para resolver os nomes de domínio (por exemplo, glpi.localhost.local, port.localhost.local, traefik.localhost.local) para o IP do servidor onde o Docker está sendo executado.

### Opção 2: Editando o Arquivo /etc/hosts

Outra maneira é adicionar manualmente os mapeamentos de IP e domínio ao arquivo `/etc/hosts` do servidor. Você pode fazer isso executando o seguinte comando:

```bash
sudo nano /etc/hosts
```

Dentro do arquivo, adicione linhas semelhantes a esta para cada domínio que você deseja configurar, substituindo `IP_DO_SERVIDOR` pelo endereço IP real do seu servidor onde o Docker está em execução:

```plaintext
IP_DO_SERVIDOR  glpi.localhost.local
IP_DO_SERVIDOR  port.localhost.local
IP_DO_SERVIDOR  traefik.localhost.local
```

Isso garantirá que os domínios sejam corretamente direcionados para o IP do seu servidor.

## Passo 8: Acessando as Aplicações
Finalmente, você pode acessar as seguintes aplicações:
    
- **GLPI**: https://glpi.localhost.local
- **Portainer**: https://port.localhost.local
- **Traefik**: https://traefik.localhost.local

Você verá avisos de segurança devido ao uso de certificados autoassinados. Você pode aceitar esses avisos para continuar.

Este tutorial ajudará você a configurar um ambiente Docker com GLPI e certificados SSL autoassinados para uma comunicação segura entre os serviços. Certifique-se de ajustar os nomes dos certificados e outros detalhes conforme necessário para atender às suas necessidades. Para obter mais informações e configurações avançadas, consulte a [Wiki][wiki]. 


Este projeto é distribuído sob a licença XYZ. Consulte o arquivo `LICENSE` para obter mais informações.

**Jonas Oliveira** – jonas556440@gmail.com

[npm-image]: https://img.shields.io/npm/v/datadog-metrics.svg?style=flat-square
[npm-url]: https://npmjs.org/package/datadog-metrics
[npm-downloads]: https://img.shields.io/npm/dm/datadog-metrics.svg?style=flat-square
[travis-image]: https://img.shields.io/travis/dbader/node-datadog-metrics/master.svg?style=flat-square
[travis-url]: https://travis-ci.org/dbader/node-datadog-metrics
[wiki]: https://github.com/seunome/seuprojeto/wiki
