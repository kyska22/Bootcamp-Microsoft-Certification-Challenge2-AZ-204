# Projeto: Deploy de API na Azure com Azure DevOps e Docker

## Visão Geral
Este projeto descreve a implementação de uma API utilizando ASP.NET e o deploy na Azure com o auxílio de Azure Functions, Docker e Azure DevOps. A API será desenvolvida no Visual Studio Code.

## Ferramentas Utilizadas
- Visual Studio Code
- Azure DevOps
- Azure Functions
- Docker
- ASP.NET Core

## Estrutura do Projeto
```
/Deploy_API_Azure_Project
│
├── src
│   └── API
│       ├── Controllers
│       │   └── WeatherForecastController.cs
│       ├── Dockerfile
│       ├── Program.cs
│       └── Startup.cs
│
├── .devops
│   └── pipelines
│       └── azure-pipelines.yml
│
└── README.md
```

## 1. Criando a API com ASP.NET

### 1.1 Inicializando o Projeto
No Visual Studio Code, inicialize um projeto ASP.NET Core:
```bash
dotnet new webapi -o API
```

### 1.2 Criando o Controller
```csharp
using Microsoft.AspNetCore.Mvc;

namespace API.Controllers
{
    [ApiController]
    [Route("[controller]")]
    public class WeatherForecastController : ControllerBase
    {
        [HttpGet]
        public IEnumerable<string> Get()
        {
            return new string[] { "Sunny", "Cloudy", "Rainy" };
        }
    }
}
```

## 2. Docker

### 2.1 Criando o Dockerfile
```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:7.0 AS base
WORKDIR /app
COPY . .
ENTRYPOINT ["dotnet", "API.dll"]
```

### 2.2 Build e Run
```bash
docker build -t api-image .
docker run -d -p 8080:80 api-image
```

## 3. Azure Functions
Criar uma Azure Function para processar solicitações específicas.

## 4. Azure DevOps

### 4.1 Pipeline de CI/CD
Arquivo `azure-pipelines.yml`:
```yaml
trigger:
- main

pool:
  vmImage: 'ubuntu-latest'

steps:
- task: UseDotNet@2
  inputs:
    packageType: 'sdk'
    version: '7.x'

- script: |
    dotnet build --configuration Release
  displayName: 'Build Project'

- task: Docker@2
  inputs:
    containerRegistry: '$(dockerRegistryServiceConnection)'
    repository: '$(dockerRepository)'
    command: 'buildAndPush'
    Dockerfile: '**/Dockerfile'
    tags: |
      $(Build.BuildId)
```

## 5. Deploy na Azure

1. Criar um Serviço de App na Azure.
2. Conectar o serviço ao repositório do Azure DevOps.
3. Configurar as variáveis de ambiente necessárias.

## Conclusão
Este projeto oferece uma visão prática sobre como criar, containerizar e realizar o deploy de uma API ASP.NET na Azure utilizando Azure DevOps.

