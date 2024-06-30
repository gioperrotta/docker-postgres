## 1. Verificar a instalação e criar conta no Docker Hub
Para verificar se o Docker foi instalado, abra um terminal e execute o seguinte comando:
### docker --version
Isso deve retornar a versão do Docker em sua máquina.

## 2. Executar um container
Execute o seguinte comando para executar o container:
### docker run hello-world

## 3. Listar imagens
Agora vamos visualizar todas as imagens baixadas, execute o comando a seguir:
### docker images

## 4. Listar containers
Lista os containers que estão em execução no nosso ambiente, para isso execute este comando:
### docker ps
    . para listar todos os containers em execução ou não.
###  docker ps -a

## 5. Pausar um container
Agora, caso o container estivesse em execução, poderíamos pausar nosso container executando o seguinte comando com o seu ID ou o nome de container:
### docker stop <ID ou nome do container>

## 7. Remover um container
Após pausar o container, você também poderá remove-lo, para isso você pode executar o comando abaixo junto com o ID ou nome de container:
### docker rm <ID ou nome do container>

## 7. Remover uma imagem
Por fim, vamos aprender a remover a imagem utilizada pelo nosso container, isto vai liberar espaço do nosso ambiente. 
### docker image rm <ID ou nome da imagem>

## 8. Mais comandos Docker
Estes são apenas alguns comandos básicos, você pode descobrir mais comandos e suas opções usando a propriedade –help no terminal.
### docker --help
### docker images --help
### docker ps --help


Preparando o ambiente com Docker
Para começar, garanta que você está com o Docker instalado, abra seu terminal e execute o comando a seguir. Caso ainda não tenha o Docker instalado, no artigo primeiros passos com Docker, explico como realizar a instalação.

docker --version
O resultado esperado é a versão do Docker instalado, semelhante a este print:


A segunda etapa é criar uma rede para nossos Containers, geralmente gosto de adotar essa estratégia para manter os containers de um projeto, dentro de uma rede especifica para ele, rode o seguinte comando:

docker network create local-network
Após executar o comando, você deve ver um ID alfanúmerico no terminal, semelhante a este:


Configurando o PostgreSQL com Docker
Agora, para iniciar nossa configuração, vamos começar criando um arquivo Docker Compose para definir o serviço do PostgreSQL.

Crie um diretório em algum local da sua preferência e crie um arquivo chamado nomeado como docker-compose.yml e adicione o seguinte conteúdo:

version: "3.8"
services:
  dcs-postgres:
    image: postgres:15.3-alpine
    container_name: dcs-postgres
    restart: always
    environment:
      POSTGRES_PASSWORD: password
      POSTGRES_USER: postgres
    ports:
      - 5432:5432
    volumes:
      - ./data/postgres:/var/lib/postgresql/data

networks:
  default:
    name: local-network
    external: true
Vamos conferir cada propriedade e entender o que fizemos.

image	Definimos a imagem Docker e versões do Postgre que será utilizada.
container_name	Definimos um nome para o container.
restart	Definimos a estratégia de reinicialização, com o valor always, o serviço sempre será reiniciado caso alguma falha aconteça.
environment	Definimos duas variáveis para serem atribuídas na inicialização do Postgres, senha e usuário de acesso.
ports	Definimos a porta publica e interna do nosso container para conexão.
volumes	Definimos o local onde os arquivos do Postgres devem ser espelhados, caso nosso serviço seja interrompido, os dados armazenados serão preservados no nosso diretório local data/postgres.
networks	Definimos o nome da rede default em que os serviços deste Compose serão executados, informando o nome da rede em name e dizendo que é uma rede externa. Isso é importante para mantermos os containers na mesma rede e compartilhar informações entre si.
Uma informação extra, caso você esteja usando um Macbook com M1 ou M2, recomendo que você também informe a propriedade platform com o valor linux/x86_64 no Compose.

Isto ajuda na compatibilidade da imagem Docker com o Apple Silicon, um problema que eu tive logo que comecei a trabalhar com os M2.


Agora com o serviço definido, podemos executar nosso Compose, no terminal execute:

docker-compose -f docker-compose.yml up
Após rodar o comando, o Docker vai fazer o download da imagem do PostgreSQL, iniciar o container e o serviço do PostgreSQL.

No final, você deve ver uma imagem semelhante a esta.


Agora o que resta é testar a conexão.

Podemos usar alguma IDE para conectar ao servidor, o instalador do pgAdmin é uma opção, entretando você pode usar o container pré-configurado do pgAdmin web, uma alternativa para quem não quer instalar o executável.

Vamos configurar o serviço do pgAdmin no Docker Compose.

Configurando o pgAdmin via Docker
Para usar o pgAdmin com Docker, vamos utilizar o container disponível aqui no DockerHub, para isso, vamos definir configurar um novo serviço no Compose.

Atualize seu arquivo yml com o conteúdo a seguir:

version: "3.8"
services:
  dcs-postgres:
    image: postgres:15.3-alpine
    container_name: dcs-postgres
    restart: always
    environment:
      POSTGRES_PASSWORD: password
      POSTGRES_USER: postgres
    ports:
      - 5432:5432
    volumes:
      - ./data/postgres:/var/lib/postgresql/data
  dcs-pgadmin:
    image: dpage/pgadmin4:latest
    container_name: dcs-pgadmin
    restart: always
    depends_on:
      - dcs-postgres
    environment:
      PGADMIN_DEFAULT_EMAIL: user@localhost.com
      PGADMIN_DEFAULT_PASSWORD: password
    ports:
      - 8080:80
    volumes:
      - ./data/pgadmin:/var/lib/pgadmin

networks:
  default:
    name: local-network
    external: true
O que fizemos agora foi adicionar um novo serviço chamado dcs-pgadmin, usando praticamente as mesmas propriedades do serviço anterior, com pequenas modificações, vamos as novas proriedades:

depends_on	Definimos que este serviço depende do serviço dcs-postgres, ou seja, este novo serviço será iniciado somente após serviço do PostgreSQL já ter iniciado.
environment	Definimos propriedades para autenticação no pgAdmin. Quando você abrir o pgAdmin no seu navegador, será solicitado um e-mail e senha.
ports	Definimos que a porta 80 do container será acessível pela porta 8080 em nosso ambiente local.
Agora para testar, pare o container atual (CTRL+C) no seu terminal e execute novamente o comando para subir o compose.

O resultado no seu terminal deve ser semelhante a este:


Para testar se o serviço do pgAdmin está funcionando acesse http://localhost:8080 no seu navegador, haverá uma tela de autenticação, você deverá informar o usuário e senha que defimos no Composer e se preferir você poderá trocar o idioma da aplicação.

A tela deve ser semelhante a esta:


Autenticado, agora vamos adicionar nosso servidor e testar a conexão.

Adicionando servidor no pgAdmin
Para adicionar o pgAdmin, na aba Object Explorer, clique com o botão direito no item Servers, após na opção Registrar e Servidor. Se preferir, também há um atalho nos links rápidos.

Menu registrar um novo servidor PostgreSQL
Uma janela deve abrir para você configurar o serviço, na aba General informe um nome para identificar sua conexão, neste tutorial chamei de Server Projeto A.

Aba general - Registrando um novo servidor PostgreSQL
Mova para a aba Conexão e no campo host name informe o nome do container do Postgre, no campo username o usuário, a senha no respectivo campo e clique em Salvar para continuar.

Estes valores foram definidas no Compose.

Aba conexão - Registrando um novo servidor PostgreSQL
Existem também outras configurações úteis, mas para este cenário, vamos configurar o essencial para começar a testar o PostgreSQL.

Após salvar, se você seguiu todas as etapas até aqui, o resultado é um Dashboard com as informações do banco de dados que você está conectado.


Por fim, agora vamos criar nosso database, clique com botão direito em Bancos de dados


Informe um nome para seu banco de dados e clique em Salvar para continuar.


Finalizado, na aba Object Explorer, você deverá ver o seu novo banco de dados listado.  


Pronto! Chegamos ao fim e agora você tem um ambiente com PostgreSQL e o pgAdmin web.

Agora, sempre que precisar trabalhar com estas ferramentas, basta abrir o terminal na pasta que utilizou para adicionar o docker-compose.yml e executar este comando:

docker-compose -f docker-compose.yml up
Próximos passos
Agora que concluímos a configuração do PostgreSQL e o pgAdmin com Docker Compose, podemos reutilizar este ambiente sem precisar ter novas instalações, basta o Docker. 😌

O próximo passos é você adicionar outros serviços no Compose que possam ser úteis para seu fluxo de trabalho, por exemplo, um serviço com Redis para cache, um serviço com um sistema de filas, como RabbitMQ e muitos outros.

A principal vantagem dessa abordagem, é que você poderá padronizar o seu ambiente e utilizar diferentes versões do mesmo software trocando apenas o número da versão.

Além disso, você poderá compartilhar este Docker Compose com outros Devs, algo muito útil quando trabalhamos em uma equipe e todos precisam usar a mesma versão.

Espero que o conteúdo tenha te ajudado, mas se tiver qualquer dúvida ou sugestão, pode deixar nos comentários a baixo, ficarei feliz em poder ajudar.

Até mais, sucesso!