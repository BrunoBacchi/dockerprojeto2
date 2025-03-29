# Exercícios Docker 2 🔥

> **IMPORTANTE**: Para que o projeto seja concluído com sucesso você necessita ter o Linux instalado em sua máquina, pois o criador deste projeto utilizou o linux para isso, pode ser qualquer um de seu gosto. O que foi utilizado aqui é o Fedora 41.

### Estágios do Projeto: 👨‍💻

- Rodar o wordpress local;
- Criar a VPC, EC2
- Criou o RDS
- Instalou o Docker na EC2
- Rodou o Wordpress na EC2
- Criou um script de inicialização no User Data e o testou
- Criou o auto-scaling group e balanceador de Carga
- Criou regras de scaling
- Monitoramento no Cloudwatch

### Primeiro 💻 - Rodar em wordpress local:
<div>
    <details align=¨left¨>
    <summary></summary>

-  Para rodar o wordpress local, como avisado anteriormente você necessitará estar logado em uma maquina linux e faça os seguintes comandos: 

```
sudo dnf update && sudo dnf upgrade -y
```
- Estes comandos buscarão atualizações na máquina e as instalarão.

## Baixando o Docker e o Dockercompose na máquina.

- Aqui está o passo a passo da instação utilizando a documentação oficial e simples, baixe e depois de instalado retorne ao projeto.

[Docker e Dockercompose instalação oficial](https://docs.fedoraproject.org/en-US/quick-docs/installing-docker/)

## Rodando o wordpress localmente

### 1-  Criar uma abstração dos volumes e redes (Opcional).

```
----------------------------------------------------------------------------------------

|- NETWORK
|-- NAME 'tunel'

----------------------------------------------------------------------------------------

|- VOLUMES
|-- NAME wordp
|-- NAME dbm

----------------------------------------------------------------------------------------
```
- Projeto: foi definido o nome da rede, será 'tunel' e será criado dois volumes, um para armazenar os arquivos do site(wordp), outro os arquivos referentes ao banco de dados(dbm).

### 2- Criar volumes para arquivos do site e do banco de dados.

```
1- docker volume create wordp

2- docker volume create dbm

3- docker volume ls

----------------------------------------------------------------------------------------
```
- 1- Cria um volume para o Wordpress.
- 2- Cria um volume para o Banco de dados mysql.
- 3- Listas os volumes.

### 3- Criar uma rede para permitir uma conexão entre o banco e o site.

```
1- docker network create tunel

2- docker network ls

----------------------------------------------------------------------------------------
```
- 1- Cria uma rede com o nome 'tunel'.
- 2- Lista as redes.

### 4 - Criar pastas para armazenar arquivos referentes ao projeto.

```
1- mkdir projetinho
2- cd projetinho

----------------------------------------------------------------------------------------
```
- 1- Cria uma diretório com o nome 'projetinho'
- 2- Entra no diretório especificado (Por mais leigo que seja, fazer uma estrutura para um projeto é essencial).

### 5 - Checar a documentação  do docker-hub.

```
https://hub.docker.com/_/wordpress
```

### 6- Criar uma abstração do banco de dados (Opcional).

```
|- DB NAME 'projetinho'
|-- DB USER 'toguro'
|-- DB PASSWORD '0311'
```

### 7- Criar um compose seguindo a documentação acima.

nano `docker-compose.yml`: 
```
services:
  web:
    image: wordpress
    restart: always
    ports:
      - "80:80"
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: toguro
      WORDPRESS_DB_PASSWORD: 0311
      WORDPRESS_DB_NAME: projetinho
    volumes:
      - wordp:/var/www/html
    networks:
      - tunel

  db:
    image: mysql:8.0
    restart: always
    environment:
      MYSQL_DATABASE: projetinho
      MYSQL_USER: toguro
      MYSQL_PASSWORD: 0311
      MYSQL_RANDOM_ROOT_PASSWORD: '1'
    volumes:
      - dbm:/var/lib/mysql
    networks:
      - tunel

networks:
  tunel:
    driver: bridge

volumes:
  wordp:
  dbm:

```

### 8- Executar o compose, e após o teste apagar ele.

```
1- docker compose up -d

2- docker compose down

----------------------------------------------------------------------------------------
```
- 1- Executa o arquivo 'docker-compose.yml'
- 2- Exclui os containers gerados(Caso não tenha criado os volumes e a rede antes de executar, o mesmo irá criar as redes serão excluidas más os volumes permanecerão)

### 9 - Resultado

- Entre no localhost:8080 da sua máquina e verá a página no ar.

</div>

## Etapa inicial 🤖 - Criando a topologia de rede, os grupos de segurança em teste sem o ALB e ASG:
<div>
    <details align=¨left¨>
    <summary></summary>

```
- EM VPC
- Vamos criar um VPC padrão e dar o nome de projetinho

- Em EC2 > Security groups
- Vamos criar um com nome de toguro usando a VPC que criamos

- Criando o banco de dados RDS


Especificações: 

- RDS com MySQL, sem Multi-AZ e instâncias db.t3.micro

--------------------------------------------------------------------------------------

|- NAME DB 'db-wordpress'
|-- NAME USER 'toguro'
|-- PASSWORD '40028922'
|--- DATABASE NAME 'db_projetinho'

--------------------------------------------------------------------------------------

O que foi escrito acima em:

- Escrevendo RDS No buscador e clicando em Aurora and RDS
- Crie um banco de dados
- Marque a opção MYSQL
- Em modelos marque o nivel gratuito
- Marque o unico disponivel, single-AZ
- Em configurações nomeie o banco de dados como db-wordpress
- coloque a senha sem assentos, que seja facil de lembrar
- Após a criação do banco de dados RDS > escolha o seu banco > security > altere a regra de entrada pro grupo de segurança que vai estar sua ec2.
```

## Criando uma EC2 instancia

```
- Em inicio rápido marque o ubuntu
- Configurações de rede: marque o VPC que criamos em publico
- Atribuia um IP public automaticamente, habilite
- selecione um grupo de segurança existente
- no final em Dados se usuário Copie e cole o que está à baixo para lá:
--------------------------------------------------------------------------------------
#!/bin/bash

 sudo apt update -y
 sudo apt upgrade -y
 
 sudo apt install -y ca-certificates curl gnupg wget
 
 sudo install -m 0755 -d /etc/apt/keyrings
 curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
 sudo chmod a+r /etc/apt/keyrings/docker.gpg
 
 echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
   $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
   sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
 
 sudo apt update -y
 sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
 
 sudo apt install -y mysql-client
 
 sudo usermod -aG docker $USER
 
 newgrp docker
 
 sudo systemctl enable docker
 sudo systemctl start docker
-----------------------------------------------------------------------------------

```

## Entre na EC2 via SSH

## Depois de entrar, baixar o mysql-client para testar a conectividade com o banco de dados como está abaixo:

```
sudo apt install -y mysql-client

-------------------------------------------------------------------------------------

mysql -h [ENDEREÇO_DO_BANCO] -u [USUÁRIO] -p -e "SHOW DATABASES;"
```

## Depois, escreva: nano docker-compose.yml

```
Abrirá o nano para editar, marque: 
------------------------------------------------------------------------
services:
  web:
    image: wordpress
    restart: always
    ports:
      - "80:80"
    environment:
      WORDPRESS_DB_HOST: db-wordpress.c98i000mqf2o.us-east-1.rds.amazonaws.com
      WORDPRESS_DB_USER: toguro
      WORDPRESS_DB_PASSWORD: 40028922
      WORDPRESS_DB_NAME: db_projetinho
    networks:
      - tunel

networks:
  tunel:
    driver: bridge
--------------------------------------------------------------------------
```
## Dado isto, copie o endereço IPv4 público da instância e entre em uma nova guia.

## Feito!

</details>
</div>


## Etapa final 🌐 - Aplicando WordPress com auto-scaling group, balanceador de carga, cloudwatch e regra de scaling 
<div>
    <details align=¨left¨>
    <summary></summary>
        
```
- 1: Entre na AWS, na barra de pesquisa, pesquise por VPC 
- 2: Vá em Criar VPC e mude apenas o nome do projeto. 
- 3: Volte e vá para ¨Suas VPCs" e marque a VPC que você criou 
- 4: Vá em mapas de recursos, passe o mouse em uma subnet pública, e clique na flecha que aparecerá a direita.
- 5: Vá em ações no começo da página 
- 6: Clique em editar configurações de sub-rede 
- 7: Marque a opção de habilidar o enredeço IPv4 púlbico de atribuição automática.
- 8: Salve e faça o mesmo com outra subnet pública que está em VPCs.
```
        
### Criação de security groups 
```
- Vá na barra de pesquisa novamente, digite EC2 e clique.
- No catálogo à esquerda procure por Security Groups e clique.
- Vá em criar um security group.
- Será necessário criar 4 grupos de segurança, um para cada item, como RDS (banco de dados), WebServer, EFS e ALB (Recomendo usar um sg-alb, sg-rds, sg-webserver, sg-efs para que você consiga se localizar, cada um tem uma regra de entrada e saída diferente)
- Em VPC marque a VPC que foi criada.
- Em descrição, escreva como se estivesse explicando o que está fazendo para você no futuro.
``` 
### Definição de Regras de entrada e Saída do WebServer, ALB, EFS e RDS. 

```
- Web Server
- Regra de entrada e saída não marque nada

- RDS (Mysql)
- Regra de entrada: marque MYSQL/AURORA e Origem Marque o Security group do WebServer e descreva isso em description
- Regras de saída: Marque todo o tráfego, e origem marque 0.0.0.0/0

- EFS
- Regras de entrada: Marque NDS e em Origem marque o Secuirity Group do Web Server e intere isso em descrição
- Regras de saída: Marque todo o tráfego, e origem marque 0.0.0.0/0

- ALB
- Regras de entrada: Marque HTTP, e origem marque 0.0.0.0/0
- Regras de saída: Marque HTTP, e origem Marque o Security group do WebServer e intere isso em descrição

- Volte em security group em Web Server e edite o grupo de segurança para adicionar mais regras de entrada e saida

- Editando Web Server:
- Regras de entrada: Marque HTTP e em Origem marque o Secuirity Group do ALB e intere isso em descrição
- Regras de entrada: Marque SSH e em Origem marque o IP da sua máquina e intere isso em descrição

- Regras de saída: marque MYSQL/AURORA, e origem marque o security group do RDS e intere isso em descrição
- Regras de saída: HTTP, e origem marque o security group do ALB e intere isso em descrição
- Regras de saída: marque todo o tráfego, e origem marque 0.0.0.0/0
```
## Criando O EFS 
```
- Na barra de busca digite EFS e entre
- Crie um novo EFS
- Vá em personalizar onde está na caixa à baixo.
- O nome é opcional, mas recomendo deixar um como EFS-projeto
- Marque como regional
- Habilite backups automáticos
- Em gerenciamento de ciclo de vida, marque ¨Nenhum" em Transição para infrequent Acess e Transição para Archive
- Em Configuração de performance, marque intermitente
- Aperte para ir na próxima aba e selecione o grupo de segurança do EFS que foi criado antes.
- Após isso avance até terminar.
```

## Criação do RDS - Banco de dados 
```
- Pesquise na barra de busca por Aurora and RDS e clique

- Vá em criar banco de dados, está um pouco abaixo
- Em opções de mecanismo: marque o MYSQL
- Abaixo em versão de mecanismo: marque a última versào.

- Em MODELOS: Marque a opçào ¨Nível gratuíto¨
- Em DISPONIBILIDADE E DURABILIDADE: Marque a opção Implementação de instância de banco de dados Single-AZ
- Em CONFIGURAÇÕES: mude o Identificador, coloque db-wordpress, será o banco de dados do wordpress
- Em CONFIGURAÇÕES DE CREDENCIAIS: o usuário será o usuário que a gente vai logar, deixo como toguro
- Marque AUTOGERENCIADA
- Em senhas, escolha uma forte, porém fácil e sem assentos.

- Em CONFIGURAÇÕES INSTÂNCIA, marque o tipo para db.t3.micro
- Em CONFIGURAÇÃO ADICIONAL DE ARMAZENAMENTO > Limite máximo de armazenamento > Marque 25 em vez de 1000
- Em CONECTIVIDADE verifique se a VPC está correta
- Em GRUPO DE SEGURANÇA DE VPC (FIREWALL) Marque o grupo de segurança do RDS que foi criado
- Antes de finalizar a criação RDS, defina um nome pro banco de dados em CONFIGURAÇÃO ADICIONAL à baixo do site e guarde essa informação: exemplo: db_wordpress

- Vá e finalize, criar banco de dados, ele vai demorar para subir
```

## Armazenando o endereço do RDS e o ponto de montagem EFS

```
 - RDS

 - Vá para Aurora and RDS
 - Marque o banco de dados criado e vá para Segurança e conexão
 - Copie o Endpoint e guarde.

 - EFS

 - Vá para EFS
 - Em nome clique no seu EFS
 - Vá em anexar no canto superior direito
 - Copie e guarde o código do assistente de montagem do EFS 
```

## Alterando o userdata e o docker-compose.yml para que tenha nossas informações

`userdata`
```
#!/bin/bash

sudo yum update -y
sudo yum install -y docker wget amazon-efs-utils

sudo service docker start
sudo systemctl enable docker.service
sudo usermod -aG docker ec2-user

sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

sudo mkdir -p /wordpress <<<<<< trocar o nome da pasta pra um de sua vontade
sudo mount -t efs -o tls fs-0a69c979ffa96bd6a:/ /wordpress  <<<<<<< fazer o mesmo aqui

if mountpoint -q /wordpress; then
    echo "EFS montado com sucesso em /wordpress"
else
    echo "Falha ao montar EFS"
    exit 1
fi

wget -O /home/ec2-user/docker-compose.yml https://raw.githubusercontent.com/Daijinpala/AVA4_24032025/main/POTATO%20SCRIPT/docker-compose.yml

sudo chown ec2-user:ec2-user /home/ec2-user/docker-compose.yml

cd /home/ec2-user
sudo docker-compose up -d
```

`docker-compose.yml`
```
services:
  web:
    image: wordpress
    restart: always
    ports:
      - "80:80"
    environment:
      WORDPRESS_DB_HOST: seu_endpoint_do_banco_de_dados
      WORDPRESS_DB_USER: o_usuario_do_seu_rds
      WORDPRESS_DB_PASSWORD: sua_senha
      WORDPRESS_DB_NAME: o_nome_da_sua_database
    volumes:
      - /home/ec2-user/pastadesuapreferencia:/var/www/html
    networks:
      - tunel

networks:
  tunel:
    driver: bridge
```

## Criando um modelo de execução Launch Template
```
- Na barra de busca procure por modelos de execução
- Criar modelo de execução
- Coloque o nome como TemplateWebServer e em sua descrição de enfase que é EC2 com docker e docker compose rodando o wordpress
- Marque a opção de orientação sobre o auto scaling

- Mais à baixo em Início rápido, marque o Amazon Linux
- Em instância marque o t2.micro
- Em configurações de rede não marque uma sub-rede específica mas marque o grupo de segurança do web server

- Em Tags marque as Tags que é utilizado na trilha de AWS
- Em deetalhes avançados coloque o arquivo no user-data, não esqueça de trocar o ponto e montagem pelo seu em EFS
- Em dados de usuário coloque o ¨userdata¨ que está em >Alterando o userdata e o docker-compose.yml para que tenha nossas informações< na explicação anterior
```

## Criando o ASG com o ALB 

```
 - Volte para EC2
 - Procure no lado esquerdo o auto scaling group
 - Criar auto scaling group
 - Colocar um nome (ex: ASG-WebServer) e escolher o launch template que criamos anteriormente
 - Em zonas de disponibilidade e sub-nets escolha as sub-nets pulbicas de zonas diferentes
 - Distruibição de disponibilidade deixe mem melhor esforço equilibrado
 - Avance

 - Em balanceamento de carga deixe em anexar um novo balanceador de carga
 - EM anexar um novo balanceador de carga marque application load balancer
 - Deixe o o nome do balanceador de carga como: LoadBalanceWordPress
 - Marque >internet-fancing< em esquema de balanceador de carga
 - Em verificações de integridade, ative a vaixinha de verificações de integridade do Elastic Load balancing
 - Avance

 - Troque a quantidade de máquinas minimas e maximas conforme o pedido do cliente
 - Futuramente você pode trocar a politicá de escalabilidade por uma personalizada do CloudWhatch
 - O tipo de métrica marque média de utilização da CPU
 - O valor do destino marque como 80
 - Habilite o monitoramento do cloudwatch
 - Avance

 - Crie uma tag personalizada para saber quais são as instancias criadas pelo ASG (Exemplo: Name - ASGWordpress)
 - Entre no seu loadbalancer, se você criou ele pelo ASG ele vai vir como padrão o grupo de segurança do seu webserver troque pelo o do ALB criado anteriormente, marque o seu load balance, vá em segurança > editar > edite o grupo de segurança sg_webserver para sg_alb
 - Após terminar os testes reduza o numero de maquinas minimas e maximas para 0 no ASG (Auto Scaling Group), ele mesmo vai encerrar as instancias, pode demorar
```

## Testing

```
- EM EC2 - Verifique a criação das ec2 e suas zonas de disponibilidade
- Cole o ¨user-data¨ de antes e execute ele já na máquina pelo ssh da instancia(Ele não está executando pelo userdata más executa com a gente fazendo pelo ssh)
- Entre usando o DNS do seu load balance pelo navegador
- Acesse as métricas pelo CloudWatch
- Feito!


</div>
