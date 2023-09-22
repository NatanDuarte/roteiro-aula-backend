# Aula/Demonstração backend (TDD)

Aula de backend com TDD usando node.js

## Roteiro

### Setup do projeto

* estrutura e dependências:

```shell
mkdir src test views

yarn init -y

# criar gitignore (criei com a extensão do vscode)

yarn add express dotenv
yarn add --dev mocha chai supertest nodemon
```

* Setup do `package.json`, incluir:

```json
"scripts": {
    "dev": "nodemon ./src/app.js"
    "test": "mocha --timeout 10000 ./test/**/*.test.js"
},
```

aqui seria um momento propício para criar um repositório

```shell
git init
git add -A
git commit -m "project setup"
```

### Primeiro teste

```javascript
// test/server.test.js

const chai = require('chai');
const chaiHttp = require('chai-http');
// Importe seu arquivo principal do servidor aqui
const app = require('../app');
const expect = chai.expect;

chai.use(chaiHttp);

describe('Servidor Express', () => {
    it('Deve retornar status code 200 na rota /', (done) => {
        chai.request(app)
            .get('/')
            .end((err, res) => {
                expect(res).to.have.status(200);
                done();
            });
    });
});
```

Rodamos o teste com `yarn test` e... falhou.

O motivo é que ainda não criamos a rota / na aplicação.

Então vamos escrever o mínimo possível para que esse passar.

```javascript
const express = require('express');
const app = express();

app.get('/', (req, res) => {});

module.exports = app;

const port = process.env.PORT || 3000;

app.listen(port, () => {
  console.log(`Servidor rodando na porta ${port}`);
});
```

Executando novamente, temos sucesso!

Agora, vamos melhorar esse teste, além de retornar 200,
deveria retornar a mensagem "Hello World"

```javascript
  it('Deve retornar "Hello, World!" na rota /', (done) => {
        chai.request(app)
            .get('/')
            .end((err, res) => {
                expect(res.text).to.equal('Hello, World!');
                done();
            });
  });
```

testando novamente, teremos uma nova falha.
Vamos incluir o retorno na rota /

```javascript
app.get('/', (req, res) => {
  res.send('Hello, World!');
});
```

Agora, rodamos novamente e devemos obter sucesso.
Vamos realizar uma pequena refatoração? não precisamos, necessáriamente,
de 2 testes separados para validar essa rota. A biblioteca Mocha permite
fazermos mais de uma validação sem deixar o código confuso.

Assim:

```javascript
describe('Meu Servidor Express', () => {
  it('Deve retornar "Hello, World!" na rota /', (done) => {
    chai.request(app)
      .get('/')
      .end((err, res) => {
        expect(res).to.have.status(200);
        expect(res.text).to.equal('Hello, World!');
        done();
      });
  });
});
```

Pronto, agora que temos nossa rota testada e funcional,
seria outro momento interessante para realizar um commit.

### Template engine

vamos criar uma nova rota para exemplificar
o funcionamento de template engines.

vamos instalar a biblioteca de ejs

```shell
yarn add ejs
```

Agora vamos criar um novo teste para a nossa rota de template

```javascript
it('/template deve renderizar a página com a mensagem correta', (done) => {
    chai.request(app)
        .get('/template')
        .end((err, res) => {
            expect(res).to.have.status(200);
            expect(res.text).to.include('<h1>Hello, World!</h1>');
            done();
        });
});
```

Como esperado, o teste falhou. agora vamos incluir
a requisição para o

```javascript
const ejs = require('ejs');

// Configura EJS como a template engine
app.set('view engine', 'ejs');
// Define a pasta onde os arquivos de visualização estão localizados
app.set('views', __dirname + '/views'); 

app.get('/template', (req, res) => {
    // Renderiza o arquivo 'index.ejs'
    res.render('index', { message: 'Hello, World!' });
});
```

Vamos criar um template em /views e chamar de `index.ejs`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Template Engine com EJS</title>
</head>
<body>
    <h1><%= message %></h1>
</body>
</html>
```

Agora os testes devem passar.
