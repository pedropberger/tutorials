docker exec -it postgresql psql

docker exec -it postgresql psql -U myuser -h localhost -p 5432 mpes

docker exec -it postgresql psql -U myuser -h localhost -p 5432 Gera_Certidão




CREATE DATABASE Gera_Certidao;

psql -U myuser -h localhost -p 5432 Gera_Certidao

CREATE USER user_pcvt WITH PASSWORD '@30056776';

GRANT ALL PRIVILEGES ON DATABASE Gera_Certidao TO user_pcvt;

psql -U myuser -h localhost -p 5432 Gera_Certidão

CREATE TABLE certidao (
  id SERIAL PRIMARY KEY,
  numero VARCHAR(255) NOT NULL,
  data_emissao TIMESTAMP NOT NULL,
  tipo VARCHAR(255) NOT NULL,
  conteudo TEXT NOT NULL
);


\d certidao

