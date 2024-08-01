# Iniciando um projeto Node.js com TypeScript

Crie um diretório para o projeto e acesse-o pelo vscode, abra o terminal e siga os passos abaixo.

```bash
npm init -y
npm install express cors sqlite3 sqlite
npm install --save-dev typescript nodemon ts-node @types/express @types/cors
npx tsc --init
mkdir src
touch src/app.ts
```

## Configuranado o `tsconfig.json`

Mude a linha ```"outDir": "./",``` para ```"outDir": "./dist",``` e adicione a linha ```"rootDir": "./src",```, seu arquivo de configuração do compilador do TypeScript ficará mais ou menos assim.

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

## Configurando o `package.json`

Adicione o seguinte script ao seu `package.json`

```json
"scripts": {
  "dev": "nodemon src/app.ts"
}
```

## Criando arquivo inicial do servidor

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

## Inicializando o servidor

```bash
npm run dev
```

Se tudo ocorrer bem, você verá a mensagem `Server running on port 3333` no terminal.

## Testando o servidor

Abra o navegador e acesse `http://localhost:3333`, você verá a mensagem `Hello World`.

## Configurando o banco de dados

Crie um arquivo `database.ts` dentro da pasta `src` e adicione o seguinte código.

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

## Adicionando o banco de dados ao servidor

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

app.listen(port, () => {
  console.log(`Server running on port ${port}`);
});
```

## Testando a inserção de dados

Abra o Postman e faça uma requisição POST para `http://localhost:3333/users` com o seguinte corpo.

```json
{
  "name": "John Doe",
  "email": "johndoe@mail.com"
}
```

Se tudo ocorrer bem, você verá a resposta com o usuário inserido.

```json
{
  "id": 1,
  "name": "John Doe",
  "email": "johndoe@mail.com"
}
```

## Listando os usuários

Adicione a rota `/users` ao servidor.

```typescript
app.get('/users', async (req, res) => {
  const db = await connect();
  const users = await db.all('SELECT * FROM users');

  res.json(users);
});
```

## Editando um usuário

Adicione a rota `/users/:id` ao servidor.

```typescript
app.put('/users/:id', async (req, res) => {
  const db = await connect();
  const { name, email } = req.body;
  const { id } = req.params;

  await db.run('UPDATE users SET name = ?, email = ? WHERE id = ?', [name, email, id]);
  const user = await db.get('SELECT * FROM users WHERE id = ?', [id]);

  res.json(user);
});
```

## Deletando um usuário

Adicione a rota `/users/:id` ao servidor.

```typescript
app.delete('/users/:id', async (req, res) => {
  const db = await connect();
  const { id } = req.params;

  await db.run('DELETE FROM users WHERE id = ?', [id]);

  res.json({ message: 'User deleted' });
});
```