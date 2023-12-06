# Instalação do Elasticsearch em Docker

1.  Crie uma rede Docker.

```
docker network create elastic
```

2. Baixe a image do Elasticsearch.

```
docker pull docker.elastic.co/elasticsearch/elasticsearch:8.10.2
```

3. Inicie o _container_ do Elasticsearch.

```
docker run --name es01 --net elastic -p 9200:9200 -it -m 1GB docker.elastic.co/elasticsearch/elasticsearch:8.10.2
```

__Obs.: A opção `-m` ajusta o limite de memória do _container_. Isso anula a necessidade ajustar manualmente o tamanho da JVM.__   

Na saída do comando será exibida a senha do usuário `elastic` e o `token` para efetuar o _enrollment_.

> ✅ Elasticsearch security features have been automatically configured!
> ✅ Authentication is enabled and cluster connections are encrypted.
>
>ℹ️  Password for the elastic user (reset with `bin/elasticsearch-reset-password -u elastic`):
>  DTxmQyPSCgOIdEZt4EHs
>
>ℹ️  HTTP CA certificate SHA-256 fingerprint:
>  2c2a1c2f797a36e18ebcdfb3d82def72f73e34d2e89e0aaa36dc4d5fc0619249
>
>ℹ️  Configure Kibana to use this cluster:
>• Run Kibana and click the configuration link in the terminal when Kibana starts.
>• Copy the following enrollment token and paste it into Kibana in your browser (valid for the next 30 minutes):
>  eyJ2ZXIiOiI4LjEwLjIiLCJhZHIiOlsiMTcyLjE5LjAuMjo5MjAwIl0sImZnciI6IjJjMmExYzJmNzk3YTM2ZTE4ZWJjZGZiM2Q4MmRlZjcyZjczZTM0ZDJlODllMGFhYTM2ZGM0ZDVmYzA2MTkyNDkiLCJrZXkiOiJlcnF0bG9zQkRoT1Z3QTFEc2JTdzpTcXVBdlhGaVN0NmdfejZOZXRwRDV3In0=
>
>ℹ️ Configure other nodes to join this cluster:
>• Copy the following enrollment token and start new Elasticsearch nodes with `bin/elasticsearch --enrollment-token <token>` (valid for the next 30 minutes):
>


4. Use uma variável de ambiente para armazenar a senha.

```
export ELASTIC_PASSWORD="your_password"

```

5. Copie o certificado SSL para um diretório do sistema local:

```
docker cp es01:/usr/share/elasticsearch/config/certs/http_ca.crt .
```

6. Faça uma chamada REST API para o Elasticsearch para assegurar que o _container_ do Elasticsearch esteja em execução.

```
curl --cacert http_ca.crt -u elastic:$ELASTIC_PASSWORD https://localhost:9200
```

### Adicione mais nós

1. Use um nó existente para gerar um _enrollment token_ para o novo nó.

```
docker exec -it es01 /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s node
```
O _enrollment token_ é válido por 30 minutos.

2. Start a new Elasticsearch container. Include the enrollment token as an environment variable.

```
docker run -e ENROLLMENT_TOKEN="<token>" --name es02 --net elastic -it -m 1GB docker.elastic.co/elasticsearch/elasticsearch:8.10
```

3. Use a chamada de API `cat nodes` para verificar se o nó foi adicionado ao _cluster_.

```
curl --cacert http_ca.crt -u elastic:$ELASTIC_PASSWORD https://localhost:9200/_cat/nodes
```

## Execução do Kibana

1. Inicie o _container_ do Kibana.

```
docker run --name kib01 --net elastic -p 5601:5601 docker.elastic.co/kibana/kibana:8.10.2

```

2. Quando o Kibana iniciar, um _link_ único será gerado na saída do terminal. Para obter acesoo ao Kibana, abra este _link_ num navegador _web_. 

# Referências

[Install Elasticsearch with Docker][https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html#run-kibana-docker]
