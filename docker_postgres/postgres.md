Seja para rodar sua aplicação de forma isolada, seja para poder errar, seja para evitar instalar um monte de coisa no seu computador que não conseguirá gerir depois, rodar o seu servidor Postgres em um container docker são muitas. Mas é bem fácil de errar e ter dor de cabeça, eu mesmo na primeira vez que me propus a rodar esse conteiner achava que seria simples como instalar o SQL Server no seu PC ou na nuvem do Azure, mas não foi. O lado bom é que já venci esse problema e deixei aqui o docker-compose prontinho para rodar.

# Pré requisitos para esse tutorial
 - Docker instalado

# Vantagens e stuff interessante: 

Um container docker é uma forma de isolar um ambiente de software, garantindo que ele funcione da mesma maneira em qualquer máquina. Um postgres é um sistema gerenciador de banco de dados relacional, muito usado em aplicações web e análise de dados. Juntos, eles podem facilitar o aprendizado e o desenvolvimento de projetos de ciência de dados.

Algumas das vantagens de usar um container docker com postgres são:

- Facilidade de instalação e configuração: você não precisa se preocupar em instalar e configurar o postgres na sua máquina local, basta baixar uma imagem docker pronta e executá-la com um comando simples.
- Portabilidade e reprodutibilidade: você pode compartilhar o seu container docker com outras pessoas, ou transferi-lo para outra máquina, sem perder as configurações e os dados do seu banco. Isso garante que o seu projeto seja reproduzível em qualquer ambiente.
- Segurança e isolamento: o container docker cria uma camada de segurança entre o seu banco de dados e o sistema operacional, evitando que possíveis ataques ou erros afetem o seu trabalho. Além disso, o container docker permite que você crie redes isoladas para conectar o seu banco de dados com outras aplicações, como notebooks ou dashboards.
- Flexibilidade e escalabilidade: o container docker permite que você ajuste os recursos do seu banco de dados de acordo com a sua necessidade, como memória, CPU e armazenamento. Você também pode criar vários containers para distribuir a carga ou testar diferentes versões do postgres.

Chega de bla-bla-bla e vamos ao ponto:

~~~yaml

version: '3.8'

services:
  postgres:
    image: postgres:latest
    container_name: postgresql
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: myuser
      POSTGRES_PASSWORD: mypassword
    volumes:
      - ./db_sql:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    networks:
      - pg-network

  pgadmin:
    image: dpage/pgadmin4:latest
    container_name: pgadmin4
    environment:
      PGADMIN_DEFAULT_EMAIL: admin@pgadmin.com
      PGADMIN_DEFAULT_PASSWORD: adminpassword
    ports:
      - "4321:80"
    networks:
      - pg-network

networks:
  pg-network:
    driver: bridge

~~~

# Registrando o banco

Para registrar o servidor PostgreSQL no PgAdmin usando o Docker Compose que você forneceu, siga estas etapas:

Após executar o docker-compose up -d para iniciar os serviços, aguarde alguns segundos para que os contêineres PostgreSQL e PgAdmin sejam inicializados e estejam em execução.

Acesse o PgAdmin:
Abra um navegador da web e acesse o PgAdmin usando o endereço http://ip_da_maquina_host:4321, substituindo "ip_da_maquina_host" pelo endereço IP da máquina onde o PgAdmin está sendo executado.

Faça login:
Use as credenciais definidas nas variáveis de ambiente PGADMIN_DEFAULT_EMAIL e PGADMIN_DEFAULT_PASSWORD no arquivo docker-compose.yml. No seu caso, o e-mail seria "admin@pgadmin.com" e a senha seria "adminpassword".

Adicione um novo servidor:
Agora, você pode adicionar um novo servidor PostgreSQL ao PgAdmin:

No painel esquerdo, clique com o botão direito em "Servers" e selecione "Create" > "Server..."

![image](https://github.com/pedropberger/tutorials/assets/98188778/eba683d8-e7fc-4212-8d33-f07ec1c85168)

Na guia "General", dê um nome ao servidor em "Name" (por exemplo, "Docker PostgreSQL").

Na guia "Connection", configure as informações de conexão com o servidor PostgreSQL:

"Host name/address": Use o nome do serviço do contêiner PostgreSQL, que é "postgres" no seu caso, já que eles compartilham a mesma rede.
"Port": Geralmente é 5432 por padrão.
"Maintenance database": O nome do banco de dados que você configurou no contêiner PostgreSQL (neste caso, "mydb").
"Username": O nome de usuário que você configurou no contêiner PostgreSQL (neste caso, "myuser").
"Password": A senha que você configurou no contêiner PostgreSQL (neste caso, "mypassword").
Clique em "Save" para adicionar o servidor.

![image](https://github.com/pedropberger/tutorials/assets/98188778/03946a28-7f93-4660-9a25-e41fe7b4c90f)

Acesse o servidor:
Após salvar as configurações, você verá o servidor listado no painel esquerdo do PgAdmin. Clique nele para expandir e visualizar os bancos de dados e outras informações do servidor PostgreSQL.

Certifique-se de que as informações fornecidas no PgAdmin correspondam às configurações definidas em seu arquivo docker-compose.yml para o serviço PostgreSQL. Isso inclui nome do host, porta, nome de usuário, senha e nome do banco de dados.
