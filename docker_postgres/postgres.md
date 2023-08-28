


# Registrando o banco:

Para registrar o servidor PostgreSQL no PgAdmin usando o Docker Compose que você forneceu, siga estas etapas:

Após executar o docker-compose up -d para iniciar os serviços, aguarde alguns segundos para que os contêineres PostgreSQL e PgAdmin sejam inicializados e estejam em execução.

Acesse o PgAdmin:
Abra um navegador da web e acesse o PgAdmin usando o endereço http://ip_da_maquina_host:4321, substituindo "ip_da_maquina_host" pelo endereço IP da máquina onde o PgAdmin está sendo executado.

Faça login:
Use as credenciais definidas nas variáveis de ambiente PGADMIN_DEFAULT_EMAIL e PGADMIN_DEFAULT_PASSWORD no arquivo docker-compose.yml. No seu caso, o e-mail seria "admin@pgadmin.com" e a senha seria "adminpassword".

Adicione um novo servidor:
Agora, você pode adicionar um novo servidor PostgreSQL ao PgAdmin:

No painel esquerdo, clique com o botão direito em "Servers" e selecione "Create" > "Server..."



Na guia "General", dê um nome ao servidor em "Name" (por exemplo, "Docker PostgreSQL").

Na guia "Connection", configure as informações de conexão com o servidor PostgreSQL:

"Host name/address": Use o nome do serviço do contêiner PostgreSQL, que é "postgres" no seu caso, já que eles compartilham a mesma rede.
"Port": Geralmente é 5432 por padrão.
"Maintenance database": O nome do banco de dados que você configurou no contêiner PostgreSQL (neste caso, "mydb").
"Username": O nome de usuário que você configurou no contêiner PostgreSQL (neste caso, "myuser").
"Password": A senha que você configurou no contêiner PostgreSQL (neste caso, "mypassword").
Clique em "Save" para adicionar o servidor.

Acesse o servidor:
Após salvar as configurações, você verá o servidor listado no painel esquerdo do PgAdmin. Clique nele para expandir e visualizar os bancos de dados e outras informações do servidor PostgreSQL.

Certifique-se de que as informações fornecidas no PgAdmin correspondam às configurações definidas em seu arquivo docker-compose.yml para o serviço PostgreSQL. Isso inclui nome do host, porta, nome de usuário, senha e nome do banco de dados.
