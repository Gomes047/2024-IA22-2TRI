# 2024-IA22-2TRI

Tutorial Completo: Criando uma API com Node.js e TypeScript
Objetivo
Neste tutorial, vamos criar uma API simples usando Node.js e TypeScript. A API permitirá criar, listar, atualizar e deletar usuários. Utilizaremos SQLite como banco de dados para armazenar as informações dos usuários.
Passo 1: Preparando o Ambiente
1.1. Instalando o Node.js
Antes de começar, você precisa ter o Node.js instalado em seu computador. O Node.js é uma plataforma que permite executar JavaScript fora do navegador.
Windows/Mac/Linux: Você pode baixar e instalar o Node.js a partir do site oficial: nodejs.org.
1.2. Instalando o Visual Studio Code (VSCode)
O Visual Studio Code é um editor de código-fonte que recomendamos para esse projeto.
Windows/Mac/Linux: Você pode baixar o VSCode a partir do site oficial: code.visualstudio.com.
Passo 2: Configurando o Projeto
2.1. Criar o Diretório do Projeto
Abra o VSCode.
Abra o terminal no VSCode (Ctrl + ``).
Crie um novo diretório para o projeto e navegue até ele:
bash
Copiar código
mkdir meu-projeto
cd meu-projeto


2.2. Inicializar o Projeto com npm
Execute o comando abaixo para criar um arquivo package.json, que irá gerenciar as dependências do projeto:

```bash
Copiar código
npm init -y
```

2.3. Instalar Dependências
Instale as bibliotecas necessárias para o projeto:

Copiar código

```BASH
npm install express cors sqlite3 sqlite
npm install --save-dev typescript nodemon ts-node @types/express @types/cors
```


express: Framework para criar o servidor.
cors: Middleware para habilitar CORS (Cross-Origin Resource Sharing).
sqlite3 e sqlite: Bibliotecas para interagir com o banco de dados SQLite.
typescript: Compilador TypeScript.
nodemon: Ferramenta que reinicia automaticamente o servidor quando arquivos são alterados.
ts-node: Permite executar TypeScript diretamente.
@types/express e @types/cors: Tipos TypeScript para Express e CORS.
2.4. Configurar o TypeScript
Crie um arquivo de configuração do TypeScript com:
bash
Copiar código
npx tsc --init


Abra o arquivo tsconfig.json gerado e faça as seguintes alterações:
json
Copiar código
```json
{
  "compilerOptions": {
    "target": "ES2017",
    "module": "commonjs",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  }
}
```


outDir: Diretório onde os arquivos compilados serão armazenados.
rootDir: Diretório onde o código fonte está localizado.
2.5. Adicionar Scripts no package.json
Adicione um script no seu package.json para iniciar o servidor:
json
Copiar código
```json
"scripts": {
  "dev": "nodemon src/app.ts"
}
```

"dev": Script que inicia o servidor em modo de desenvolvimento usando nodemon.
Passo 3: Criar o Servidor
3.1. Criar o Arquivo Principal do Servidor
Crie um diretório src e dentro dele, crie um arquivo app.ts:

Copiar código
```bash
mkdir src
touch src/app.ts
```

Abra o src/app.ts e adicione o seguinte código para criar o servidor básico:
typescript
Copiar código
```typescript
import express from 'express';
import cors from 'cors';

const port = 3333;
const app = express();

app.use(cors());
app.use(express.json());

app.get('/', (req, res) => {
  res.send('Hello World');
});

app.listen(port, () => {
  console.log(`Server running on port ${port}`);
});
```

3.2. Iniciar o Servidor
No terminal, execute:
bash
Copiar código
```bash
npm run dev
```

O servidor deve iniciar e exibir a mensagem: Server running on port 3333.
Abra seu navegador e acesse http://localhost:3333. Você deve ver a mensagem Hello World.
Passo 4: Configurar o Banco de Dados
4.1. Criar o Arquivo de Conexão com o Banco de Dados
No diretório src, crie um arquivo database.ts e adicione o seguinte código:
typescript
Copiar código
```typescript
import { open, Database } from 'sqlite';
import sqlite3 from 'sqlite3';

let instance: Database | null = null;

export async function connect() {
  if (instance) return instance;

  const db = await open({
     filename: './src/database.sqlite',
     driver: sqlite3.Database
   });
 
 await db.exec(`
   CREATE TABLE IF NOT EXISTS users (
     id INTEGER PRIMARY KEY AUTOINCREMENT,
     name TEXT,
     email TEXT
   )
 `);

 instance = db;
 return db;
}
```

O código acima cria uma tabela users no banco de dados SQLite, se ela não existir.
Passo 5: Adicionar Funcionalidades da API
5.1. Criar Rotas para Manipulação de Usuários
No arquivo src/app.ts, adicione as seguintes rotas para criar, listar, atualizar e deletar usuários:
typescript
Copiar código
```typescript
import express from 'express';
import cors from 'cors';
import { connect } from './database';

const port = 3333;
const app = express();

app.use(cors());
app.use(express.json());

app.get('/', (req, res) => {
  res.send('Hello World');
});

app.post('/users', async (req, res) => {
  const db = await connect();
  const { name, email } = req.body;

  const result = await db.run('INSERT INTO users (name, email) VALUES (?, ?)', [name, email]);
  const user = await db.get('SELECT * FROM users WHERE id = ?', [result.lastID]);

  res.json(user);
});

app.get('/users', async (req, res) => {
  const db = await connect();
  const users = await db.all('SELECT * FROM users');
  res.json(users);
});

app.put('/users/:id', async (req, res) => {
  const db = await connect();
  const { name, email } = req.body;
  const { id } = req.params;

  await db.run('UPDATE users SET name = ?, email = ? WHERE id = ?', [name, email, id]);
  const user = await db.get('SELECT * FROM users WHERE id = ?', [id]);

  res.json(user);
});

app.delete('/users/:id', async (req, res) => {
  const db = await connect();
  const { id } = req.params;

  await db.run('DELETE FROM users WHERE id = ?', [id]);

  res.json({ message: 'User deleted' });
});

app.listen(port, () => {
  console.log(`Server running on port ${port}`);
});
```
5.2. Testar as Rotas
Utilize o Postman ou uma ferramenta similar para testar suas rotas.
Criar Usuário: Envie um POST para http://localhost:3333/users com o corpo:
json
Copiar código
```json
{
  "name": "John Doe",
  "email": "johndoe@mail.com"
}
```

Listar Usuários: Envie um GET para http://localhost:3333/users.
Atualizar Usuário: Envie um PUT para http://localhost:3333/users/:id com o corpo:
json
Copiar código
```json
{
  "id": 1,
  "name": "Jane Doe",
  "email": "janedoe@mail.com"
}
```

Listando os usuários
Adicione a rota /users ao servidor.
adicione ao app.ts
copiar código
```ts
app.get('/users', async (req, res) => {
  const db = await connect();
  const users = await db.all('SELECT * FROM users');

  res.json(users);
});
```

Editando um usuário
Adicione a rota /users/:id ao servidor.
adicione ao app.ts
copiar código
```ts
app.put('/users/:id', async (req, res) => {
  const db = await connect();
  const { name, email } = req.body;
  const { id } = req.params;

  await db.run('UPDATE users SET name = ?, email = ? WHERE id = ?', [name, email, id]);
  const user = await db.get('SELECT * FROM users WHERE id = ?', [id]);

  res.json(user);
});
```

Deletando um usuário
Adicione a rota /users/:id ao servidor.
adicione ao app.ts
copiar código
```ts
app.delete('/users/:id', async (req, res) => {
  const db = await connect();
  const { id } = req.params;

  await db.run('DELETE FROM users WHERE id = ?', [id]);

  res.json({ message: 'User deleted' });
});
```

Passo 6: Criar a Pasta Public e Adicionar Arquivos

6.1 Criar a Pasta Public

No diretório raiz do seu projeto, crie uma nova pasta chamada public. Esta pasta será usada para armazenar arquivos estáticos, como o HTML e o CSS.

bash
Copiar código
```bash
mkdir public
```
6.2 Criar o Arquivo index.html

Dentro da pasta public, crie um arquivo chamado index.html e adicione o seguinte código:

html
Copiar código
```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
  <link rel="stylesheet" href="estilo.css">
</head>

<body>
  <form>
    <input type="text" name="name" placeholder="Nome">
    <input type="email" name="email" placeholder="Email">
    <button type="submit">Cadastrar</button>
  </form>

  <table>
    <thead>
      <tr>
        <th>Id</th>
        <th>Name</th>
        <th>Email</th>
        <th>Ações</th>
      </tr>
    </thead>
    <tbody>
      <!--  -->
    </tbody>
  </table>

  <script>
    const form = document.querySelector('form')

    form.addEventListener('submit', async (event) => {
      event.preventDefault()

      const name = form.name.value
      const email = form.email.value

      await fetch('/users', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ name, email })
      })

      form.reset()
      fetchData()
    })

    const tbody = document.querySelector('tbody')

    async function fetchData() {
      const resp = await fetch('/users')
      const data = await resp.json()

      tbody.innerHTML = ''

      data.forEach(user => {
        const tr = document.createElement('tr')
        tr.innerHTML = `
          <td>${user.id}</td>
          <td>${user.name}</td>
          <td>${user.email}</td>
          <td>
            <button class="excluir">excluir</button>
            <button class="editar">editar</button>
          </td>
        `

        const btExcluir = tr.querySelector('button.excluir')
        const btEditar = tr.querySelector('button.editar')

        btExcluir.addEventListener('click', async () => {
          await fetch(`/users/${user.id}`, { method: 'DELETE' })
          tr.remove()
        })

        btEditar.addEventListener('click', async () => {
          const name = prompt('Novo nome:', user.name)
          const email = prompt('Novo email:', user.email)

          await fetch(`/users/${user.id}`, {
            method: 'PUT',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ name, email })
          })

          fetchData()
        })

        tbody.appendChild(tr)
      })
    }

    fetchData()
  </script>
</body>

</html>
```
6.3 Criar o Arquivo estilo.css

Ainda na pasta public, crie um arquivo chamado estilo.css e adicione o seguinte código:
css
Copiar código
```css
body {
  font-family: Arial, sans-serif;
  margin: 0;
  padding: 0;
  background-color: #f4f4f4;
}

form {
  background-color: #fff;
  padding: 20px;
  margin: 20px auto;
  max-width: 500px;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

input[type="text"],
input[type="email"] {
  width: calc(100% - 22px);
  padding: 10px;
  margin: 10px 0;
  border: 1px solid #ddd;
  border-radius: 4px;
  box-sizing: border-box;
}

button {
  background-color: #007bff;
  color: #fff;
  border: none;
  padding: 10px 15px;
  margin: 10px 0;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
}

button:hover {
  background-color: #0056b3;
}

table {
  width: 100%;
  border-collapse: collapse;
  margin: 20px auto;
  max-width: 800px;
  background-color: #fff;
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

th, td {
  padding: 12px;
  text-align: left;
  border-bottom: 1px solid #ddd;
}

th {
  background-color: #007bff;
  color: #fff;
}

tr:hover {
  background-color: #f1f1f1;
}

.excluir {
  background-color: #dc3545;
  margin-right: 5px;
}

.editar {
  background-color: #ffc107;
}

.excluir:hover {
  background-color: #c82333;
}

.editar:hover {
  background-color: #e0a800;
}
```

Acesse a Aplicação: Abra seu navegador e vá para http://localhost:3333. Você deverá ver seu formulário e tabela estilizados.

Teste as Funcionalidades: Use o formulário para adicionar, editar e excluir usuários e verifique se tudo está funcionando conforme o esperado.

Deletar Usuário: Envie um DELETE para http://localhost:3333/users/:id.
Conclusão
Parabéns! Você criou com sucesso uma API básica utilizando Node.js e TypeScript, conectada a um banco de dados SQLite. Agora, você pode expandir e personalizar essa API conforme necessário para atender às suas necessidades específicas.