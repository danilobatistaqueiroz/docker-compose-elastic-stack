**Logstash:** Componente de processamento de dados do Elastic Stack que recebe, transforma e envia dados para o Elasticsearch.

**Beats:** Plugs de dados leves e de propósito único que podem enviar dados de centenas ou milhares de máquinas para o Logstash ou para o Elasticsearch.


# ELK Stack vs Elastic Stack

ELK é constituído de Elasticsearch+Logstash+Kibana

Elastic Stack é Beats+Elasticsearch+Logstash+Kibana


### propósito do projeto
Filebeat + Logstash para importar um conjunto de logs do Apache.  
E outro Filebeat que envia ao Logstash arquivo csv com cidades.  
Usando o padrão grok.  



For example, and especially if installing Elasticsearch on the cloud, it is a good best practice to bind Elasticsearch to either a private IP or localhost:

sudo vim /etc/elasticsearch/elasticsearch.yml
```yml
network.host: "localhost"
http.port:9200
``` 

Creating an Elasticsearch Index  
Indexing is the process of adding data to Elasticsearch.   

**the three main configuration sections in a Logstash configuration file**, each responsible for different functions and using different Logstash plugins.  


curl -XPUT 'localhost:5044'  -H 'Content-Type: application/json' -d '
{
  "id": 4,
  "username": "john",
  "last_login": "2018-01-25 12:34:56"
}
'

curl -XPUT 'localhost:5044'  -H 'Content-Type: application/json' -d 'ola'



O ideal é fechar o elasticsearch para acesso externo:  
`/etc/elasticsearch/elasticsearch.yml`  
network.host: "localhost" http.port:9200 cluster  
initial_master_nodes: ["<PrivateIP"]  



logstash.conf
```
input {
	http {
		
	}
	file {
		path => "/csv/worldcitiespop.csv"
		start_position => beginning
		tag => "file"
	}
	beats {
		port => 5044
		tag => "beats"
	}
	tcp {
		port => 5000
		tag => "tcp"
	}
}
filter {
	if "file" in [tag]{
		csv {
        	columns => ["country","city","accentCity","region","population"]
   		}		
		mutate {
			add_field => {
	    		"location" => "%{column6},%{column7}"
  			} 
			remove_field => [ "message", "@version","@timestamp","host","path","column6","column7" ]	
		}
	}
}
output {
	stdout { 

	}
	if "file" in [tag]{
		elasticsearch {
			index => "cities"
			hosts => "elasticsearch:9200"
			user => "elastic"
			password => "changeme"
			ecs_compatibility => disabled
		}	
	} else {
		elasticsearch {
			hosts => "elasticsearch:9200"
			user => "elastic"
			password => "changeme"
			ecs_compatibility => disabled
		}	
	}
}
```



reiniciar apenas um dos serviços:  
`docker-compose restart logstash`

consultar os logs de um serviço:  
`docker-compose logs logstash`



# create and delete Indexes

```bash
curl -XPUT 'localhost:9200/twitters/_doc/1?pretty' -H 'Content-Type: application/json' -d'
{
    "@timestamp": "2021-05-18T15:57:27.541Z",
    "ip": "225.14.117.190",
    "extension": "twt",
    "destinations": "3",
    "user": "danilo",
    "twitte": "vamos filtra essa bagaça"
}
'
#deleta um indice
curl -XDELETE 'localhost:9200/cities'

#deleta um documento
curl -XDELETE 'localhost:9200/twitters/_doc/1'

#listar os indices
curl 'localhost:9200/_cat/indices?v'
```