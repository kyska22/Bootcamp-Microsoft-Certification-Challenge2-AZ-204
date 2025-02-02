# Projeto de Validação de CPF com Azure Functions

Este projeto implementa um microsserviço serverless no Azure para validar CPFs.

## Ferramentas
Antes de iniciar, precisamos ter as seguintes ferramentas instaladas:
 - Visual Studio Code: Editor de código.
 - .NET SDK 8: O SDK para construir o aplicativo.
 - Azure CLI: Ferramenta de linha de comando para interagir com o Azure.
 - Azure Functions Core Tools: Ferramenta para desenvolver e executar funções localmente.
 - Conta no Azure: Para criar e gerenciar recursos de nuvem.


## Estrutura do Projeto

- **Azure Functions**: Função que valida o CPF.
- **Armazenamento (opcional)**: Para persistência de logs.
- **Application Insights (opcional)**: Para monitoramento e logs.

## Passos para Construção

1. Criação do projeto com Azure Functions:
   ```bash
   func init ValidacaoCPF --dotnet
   cd ValidacaoCPF
   func new --template "Function" --name ValidarCPF
  
Isso criará um novo projeto de Azure Functions com a função ValidarCPF.
   
2. Instalação de Dependências:

Instale as dependências necessárias para validar o CPF. Para isso, adicione um pacote de validação usando uma expressão regular para o CPF. Execute o comando no terminal para adicionar o pacote de validação:
  ``` bash
  dotnet add package System.Text.RegularExpressions
 

3. Código da Função

Arquivo ValidarCPF.cs dentro da pasta Functions

```
using System;
using System.Text.RegularExpressions;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.Extensions.Logging;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;

public static class ValidarCPF
{
    [FunctionName("ValidarCPF")]
    public static async Task<IActionResult> Run(
        [HttpTrigger(AuthorizationLevel.Function, "get", "post")] HttpRequest req,
        ILogger log)
    {
        log.LogInformation("Validando CPF.");

        // Obter o CPF da query string ou do corpo da requisição
        string cpf = req.Query["cpf"];

        if (string.IsNullOrEmpty(cpf))
        {
            return new BadRequestObjectResult("Por favor, forneça um CPF.");
        }

        // Realizar validação
        bool valido = Validar(cpf);

        if (valido)
        {
            return new OkObjectResult($"CPF {cpf} é válido.");
        }
        else
        {
            return new BadRequestObjectResult($"CPF {cpf} é inválido.");
        }
    }

    // Função para validar o CPF
    private static bool Validar(string cpf)
    {
        // Remover caracteres não numéricos
        cpf = Regex.Replace(cpf, @"[^\d]", "");

        // Verificar se o CPF tem 11 dígitos
        if (cpf.Length != 11)
            return false;

        // Verificar se o CPF possui todos os dígitos iguais (ex: 111.111.111-11)
        var regex = new Regex(@"^(\d)\1{10}$"); // Ex: 111.111.111-11 é inválido
        if (regex.IsMatch(cpf))
            return false;

        // Cálculo de validação de dígitos verificadores
        int[] digitos = new int[11];
        for (int i = 0; i < 11; i++)
            digitos[i] = int.Parse(cpf[i].ToString());

        // Primeiro dígito verificador
        int soma1 = 0;
        for (int i = 0; i < 9; i++)
            soma1 += digitos[i] * (10 - i);
        int resto1 = soma1 % 11;
        if (resto1 < 2)
            resto1 = 0;
        else
            resto1 = 11 - resto1;

        // Segundo dígito verificador
        int soma2 = 0;
        for (int i = 0; i < 10; i++)
            soma2 += digitos[i] * (11 - i);
        int resto2 = soma2 % 11;
        if (resto2 < 2)
            resto2 = 0;
        else
            resto2 = 11 - resto2;

        // Comparar com os dígitos verificadores calculados
        return digitos[9] == resto1 && digitos[10] == resto2;
    }
}


4. Testando a Função Localmente
```
func start

Acesse http://localhost:7071/api/ValidarCPF?cpf=12345678909 para testar a função localmente. Ela retornará se o CPF fornecido é válido ou não.

5. Deployment para o Azure

 - Autenticação no Azure:
```
az login

 - Criar uma Função no Azure:
```
az functionapp create --resource-group <nome-do-grupo-de-recursos> --consumption-plan-location <região> --runtime dotnet --functions-version 4 --name <nome-do-app-de-função> --storage-account <nome-da-conta-de-armazenamento>

 - Publicar a Função:
```
func azure functionapp publish <nome-do-app-de-função>

6. Monitoramento e Logging

 - Ativar Application Insights:
Ao criar o App Service para a função no Azure, você pode habilitar o Application Insights. Isso permitirá monitorar a saúde da sua função, erros e outros logs.

 - Ver Logs via Azure Portal:
Acesse o portal do Azure e no menu da sua Function App, clique em "Logs" para ver a execução da função.

## Conclusão
Agora temos um microsserviço serverless no Azure para validação de CPFs, utilizando o Azure Functions. Este serviço pode escalar automaticamente e tem baixo custo operacional devido ao modelo serverless, onde você paga apenas pela execução. O sistema é fácil de manter e pode ser estendido conforme a necessidade.
