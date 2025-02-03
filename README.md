# Criando um Sistema de Gerenciamento de Catálogos de Filmes no Azure

Este guia descreve como criar um sistema simples de gerenciamento de catálogos de filmes, semelhante à Netflix, utilizando um frontend, um API Gateway e Azure Functions para comunicação com um banco de dados SQL no Azure.

## 1. Criar uma Conta no Azure
Se ainda não possui uma conta no Azure, registre-se em [https://azure.microsoft.com](https://azure.microsoft.com).

## 2. Criar um Banco de Dados SQL no Azure
1. Acesse o portal do Azure ([https://portal.azure.com](https://portal.azure.com)).
2. No menu esquerdo, clique em **Banco de Dados SQL** e depois em **+ Criar**.
3. Preencha os campos:
   - **Nome do Servidor**: `meu-servidor-filmes`
   - **Grupo de Recursos**: Crie um novo ou use um existente.
   - **Autenticação**: Escolha um método de autenticação (senha ou Active Directory).
   - **Camada de Serviço**: Escolha um plano adequado para sua necessidade.
4. Clique em **Revisar + Criar** e depois em **Criar**.

## 3. Criar a Estrutura do Banco de Dados
1. No portal do Azure, acesse o **Banco de Dados SQL** e abra o **Query Editor**.
2. Execute o seguinte script para criar a tabela de filmes:
   ```sql
   CREATE TABLE Filmes (
       id INT PRIMARY KEY IDENTITY(1,1),
       titulo NVARCHAR(255) NOT NULL,
       descricao NVARCHAR(500),
       ano_lancamento INT,
       genero NVARCHAR(100)
   );
   ```

## 4. Criar as Azure Functions para Comunicação com o Banco de Dados

### Criar um App de Função
1. No portal do Azure, vá até **Funções** e clique em **+ Criar**.
2. Escolha **Plano de Consumo** e selecione a linguagem desejada (Python, C#, Node.js).
3. Nomeie o aplicativo como `api-filmes` e clique em **Criar**.

### Criar uma Function para Buscar Filmes
1. No portal do Azure, acesse **Funções** e clique em **+ Criar Função**.
2. Escolha **Trigger HTTP** e nomeie como `getFilmes`.
3. Substitua o código da função pelo seguinte (exemplo em Python):
   ```python
   import pyodbc
   import os
   import azure.functions as func

   def main(req: func.HttpRequest) -> func.HttpResponse:
       connection_string = os.getenv("SQL_CONNECTION_STRING")
       conn = pyodbc.connect(connection_string)
       cursor = conn.cursor()
       cursor.execute("SELECT id, titulo, descricao, ano_lancamento, genero FROM Filmes")
       filmes = [dict(zip([column[0] for column in cursor.description], row)) for row in cursor.fetchall()]
       return func.HttpResponse(json.dumps(filmes), mimetype="application/json")
   ```
4. Configure a **Connection String** do banco de dados nas configurações do aplicativo de função.

## 5. Criar um API Gateway no Azure API Management
1. No portal do Azure, vá até **API Management** e clique em **+ Criar**.
2. Nomeie o recurso como `api-gateway-filmes` e selecione o grupo de recursos apropriado.
3. Após a criação, vá até **APIs** e clique em **+ Criar API**.
4. Selecione **HTTP** e defina a URL base apontando para as Azure Functions criadas.
5. Configure os endpoints, como `/filmes`, para redirecionar chamadas para `https://<nome-da-funcao>.azurewebsites.net/api/getFilmes`.

## 6. Criar um Frontend em React
### Criar um Aplicativo React
1. Instale o Node.js e crie um novo projeto React:
   ```sh
   npx create-react-app catalogo-filmes
   cd catalogo-filmes
   ```
2. Instale o Axios para requisições HTTP:
   ```sh
   npm install axios
   ```

### Criar um Componente para Listar Filmes
1. No diretório `src`, crie um arquivo `Filmes.js`:
   ```jsx
   import React, { useEffect, useState } from "react";
   import axios from "axios";

   const Filmes = () => {
       const [filmes, setFilmes] = useState([]);

       useEffect(() => {
           axios.get("https://api-gateway-filmes.azure-api.net/filmes")
               .then(response => setFilmes(response.data))
               .catch(error => console.error(error));
       }, []);

       return (
           <div>
               <h1>Catálogo de Filmes</h1>
               <ul>
                   {filmes.map(filme => (
                       <li key={filme.id}>{filme.titulo} ({filme.ano_lancamento})</li>
                   ))}
               </ul>
           </div>
       );
   };

   export default Filmes;
   ```

### Rodar o Frontend Localmente
```sh
npm start
```

## 7. Fazer o Deploy do Frontend no Azure
1. No portal do Azure, vá até **App Services** e clique em **+ Criar**.
2. Escolha **Static Web App** e conecte ao repositório GitHub do projeto React.
3. Defina o comando de build como `npm run build` e o diretório de saída como `build/`.
4. Após a publicação, o frontend estará acessível na URL fornecida pelo Azure.

## 8. Testar o Sistema
- Acesse a URL do frontend e verifique se os filmes são carregados corretamente.
- Utilize o Postman ou cURL para testar as funções diretamente:
  ```sh
  curl "https://api-gateway-filmes.azure-api.net/filmes"
  ```

Agora, seu sistema básico de gerenciamento de catálogos de filmes está rodando no Azure! 🚀

