# Estrutura do Projeto

```
azure-netflix-catalog/
│   README.md
│
├── backend/
│   ├── functions/
│   │   ├── get_catalog.py
│   │   ├── update_catalog.py
│   │   ├── add_movie.py
│   │   ├── delete_movie.py
│   ├── requirements.txt
│   ├── host.json
│   ├── local.settings.json
│
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   ├── pages/
│   │   ├── App.js
│   │   ├── index.js
│   ├── public/
│   ├── package.json
│
├── infrastructure/
│   ├── terraform/
│   │   ├── main.tf
│   │   ├── variables.tf
│   ├── docker/
│   │   ├── Dockerfile
```

# Código das Azure Functions

## backend/functions/get_catalog.py
```python
import azure.functions as func
import json
import redis
from azure.cosmos import CosmosClient
import os

def main(req: func.HttpRequest) -> func.HttpResponse:
    redis_client = redis.Redis(host=os.getenv("REDIS_HOST"), port=6379, db=0)
    cache_key = "catalog"
    cached_data = redis_client.get(cache_key)

    if cached_data:
        return func.HttpResponse(cached_data, mimetype="application/json")
    
    cosmos_client = CosmosClient(os.getenv("COSMOS_ENDPOINT"), os.getenv("COSMOS_KEY"))
    database = cosmos_client.get_database_client("NetflixDB")
    container = database.get_container_client("Catalog")
    
    items = list(container.read_all_items())
    response_data = json.dumps(items)
    redis_client.set(cache_key, response_data, ex=600)
    
    return func.HttpResponse(response_data, mimetype="application/json")
```

## backend/functions/add_movie.py
```python
import azure.functions as func
import json
from azure.cosmos import CosmosClient
import os

def main(req: func.HttpRequest) -> func.HttpResponse:
    cosmos_client = CosmosClient(os.getenv("COSMOS_ENDPOINT"), os.getenv("COSMOS_KEY"))
    database = cosmos_client.get_database_client("NetflixDB")
    container = database.get_container_client("Catalog")
    
    req_body = req.get_json()
    new_movie = {
        "id": req_body["id"],
        "title": req_body["title"],
        "genre": req_body["genre"],
        "year": req_body["year"]
    }
    container.create_item(new_movie)
    
    return func.HttpResponse(json.dumps({"message": "Movie added successfully"}), mimetype="application/json")
```

# README.md
```markdown
# Gerenciador de Catálogos da Netflix - Azure

Este projeto é um sistema de gerenciamento de catálogos de filmes e séries utilizando Azure Functions, Cosmos DB, Redis e Storage Account.

## Tecnologias Utilizadas
- **Azure Functions** para backend serverless
- **Azure Cosmos DB** para banco de dados
- **Azure Redis Cache** para cacheamento
- **Azure Storage Account** para imagens
- **API Gateway (Azure API Management)** para gerenciamento de requisições
- **React.js** no frontend

## Como Rodar o Projeto

### Backend
```sh
cd backend
pip install -r requirements.txt
func start
```

### Frontend
```sh
cd frontend
npm install
npm start
```

### Deploy para Azure
```sh
az functionapp deploy --src-path backend/
```
```

Agora é só rodar e testar! 🚀
