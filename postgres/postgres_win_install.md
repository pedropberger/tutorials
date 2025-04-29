# Tutorial Rápido: Instalando PostgreSQL e pgAdmin no Windows

Vamos instalar o PostgreSQL (banco de dados relacional open-source) e o pgAdmin (interface gráfica pra gerenciar o Postgres) no Windows. É bem tranquilo, só seguir o passo a passo!
O que são esses componentes?

PostgreSQL: Um banco de dados relacional robusto, usado pra armazenar e gerenciar dados de forma estruturada. É super poderoso pra aplicações web, corporativas, etc.
pgAdmin: Uma ferramenta gráfica que facilita a administração do Postgres. Com ela, você cria bancos, tabelas, roda queries e gerencia usuários sem precisar de linha de comando.

## Pré-requisitos

Windows 10 ou superior (64-bit).
Uns 500 MB de espaço livre.
Conexão com internet pra baixar os instaladores.

## Passo a Passo
1. Baixar o PostgreSQL

Acesse o site oficial: postgresql.org/download/windows.
Clique em "Download the installer" (EnterpriseDB é a opção mais comum).
Escolha a versão mais recente (ex.: 16.x) e baixe o instalador pra Windows x86-64.

2. Instalar o PostgreSQL

Execute o arquivo baixado (ex.: postgresql-16.x.x-windows-x64.exe).
Siga o wizard:
Diretório de instalação: Deixe o padrão (ex.: C:\Program Files\PostgreSQL\16).
Componentes: Marque o PostgreSQL Server e o pgAdmin 4. O Stack Builder é opcional (pule se não precisar de extras).
Diretório de dados: Deixe o padrão (ex.: C:\Program Files\PostgreSQL\16\data).
Senha do superusuário: Crie uma senha forte pro usuário postgres (anote, vai precisar!).
Porta: Use a padrão (5432), a menos que já esteja em uso.
Locale: Deixe como "Default locale" ou escolha [Portuguese, Brazil] se preferir.


Clique em "Next" até instalar. Pode demorar uns minutos.

3. Verificar a Instalação do PostgreSQL

Após a instalação, o serviço do Postgres é iniciado automaticamente.
Pra confirmar, abra o Prompt de Comando e digite:psql -U postgres


Digite a senha que você criou. Se aparecer o prompt postgres=#, tá funcionando!
Digite \q pra sair.

4. Configurar o pgAdmin

O pgAdmin já vem instalado com o Postgres (se você marcou a opção).
Abra o pgAdmin 4 (procure no menu Iniciar).
Na primeira execução, ele abre no navegador (ex.: http://127.0.0.1:5050).
Crie uma senha mestre pro pgAdmin (anote essa também!).

5. Conectar o pgAdmin ao PostgreSQL

No pgAdmin, clique em Add New Server.
Na aba General:
Name: Dê um nome (ex.: "Local Server").


Na aba Connection:
Host name/address: localhost ou 127.0.0.1.
Port: 5432.
Maintenance database: postgres.
Username: postgres.
Password: A senha do superusuário que você criou.


Clique em Save. Se conectar, você verá o servidor na barra lateral.

6. Testar Tudo

No pgAdmin, expanda o servidor, vá em Schemas > public > Tables.
Clique com o botão direito em "Tables" e crie uma tabela de teste.
Ou use a janela Query Tool pra rodar um SQL simples:CREATE TABLE teste (id SERIAL PRIMARY KEY, nome VARCHAR(50));
INSERT INTO teste (nome) VALUES ('Hello World');
SELECT * FROM teste;


Se funcionar, tá tudo certo!

## Dicas Extras

Serviço do Postgres: O serviço roda automaticamente no boot. Pra gerenciar, use o Services do Windows (procure por "PostgreSQL").
Porta em uso?: Se a porta 5432 estiver ocupada, mude durante a instalação ou edite o arquivo postgresql.conf em C:\Program Files\PostgreSQL\16\data.
Esqueceu a senha?: Redefina editando o arquivo pg_hba.conf (mude md5 pra trust, reinicie o serviço, e use psql pra alterar a senha).

## Possíveis Problemas

Erro de conexão: Verifique se o serviço do Postgres tá rodando (no Gerenciador de Tarefas ou Services).
Firewall: Libere a porta 5432 no Firewall do Windows se for acessar de outra máquina.
pgAdmin não abre: Tente acessar http://127.0.0.1:5050 manualmente no navegador.
