docker exec -it postgresql psql

docker exec -it postgresql psql -U myuser -h localhost -p 5432 mpes

docker exec -it postgresql psql -U myuser -h localhost -p 5432 gera_certidao




CREATE DATABASE gera_certidao;

psql -U myuser -h localhost -p 5432 gera_certidao

CREATE USER user_geracertidao WITH PASSWORD 'g3rac3rtida0';

GRANT ALL PRIVILEGES ON DATABASE mpes TO user_geracertidao;

psql -U myuser -h localhost -p 5432 gera_certidao

CREATE TABLE certidao (
  id SERIAL PRIMARY KEY,
  numero VARCHAR(255) NOT NULL,
  data_emissao TIMESTAMP NOT NULL,
  tipo VARCHAR(255) NOT NULL,
  conteudo TEXT NOT NULL
);


\d certidao

GRANT SELECT, UPDATE ON DATABASE "gera_certidao" TO user_geracertidao;

GRANT SELECT ON ALL TABLES IN DATABASE "gera_certidao" TO user_geracertidao;

GRANT SELECT ON ALL TABLES IN SCHEMA public TO user_geracertidao;


\du
