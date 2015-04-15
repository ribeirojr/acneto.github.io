---
layout: post
title:  "Node.JS acessando MongoDB"
date:   2015-01-01 18:00
categories: node javascript mongodb nosql 
---

<center>*Nesse post irei mostrar como escrever uma aplicação Node.JS básica e depois evolui-la para usar MongoDB de maneira modular, dividindo a responsabilidade entre camadas.*</center>

#### O que é Node.JS?

Primeiro é importante entender que Node.JS é uma *plataforma* para aplicações JavaScript e não um *framework*. Porém existem frameworks construidos para trabalhar com Node - como o Express - que tem como objetivo facilitar a vida do desenvolvedor na construção de aplicativos web.

#### Asynchronous I/O

Node.JS tem como objetivo ser performático, por isso ele se baseia no conceito de *asynchronous I/O* e *event loop*. Por exemplo: quando o browser envia vários requests para uma aplicação Node, essas requisições são tratadas pelo event loop e se algum desses requests envolve I/O (uma consulta no banco de dados, por exemplo) sua aplicacão não ficará bloqueada até o banco de dados terminar de processar, ela continuará ativa processando os requests que sobraram. É dessa forma que aplicações Nodes geralmente são mais rapidas que as escritas em plataformas tradicionais, chamadas de *blocking I/O*. 

![]({{ site.url }}/assets/nodejs/event-loop.png)

<p/>
#### Instalando Node.JS

Instalar as ferramentas é bastante simples e depende da plataforma que você está usando. No caso do Mac OS basta fazer o download do pacote `pkg` no [site do projeto][nodejs-site] e instalar como qualquer aplicação. Caso você tenha receba algum erro de dependencia no seu SO verifique se você tem o [XCode][xcode-site] instalado na sua máquina. Quando você instala o Node, também será instalado o [npm][npm-site], que é basicamente um gerenciador de pacotes específico dessa plataforma.

Você pode verificar se o Node e o `npm` estão instalados basta executar no terminal:

{% highlight console %}
$ node -v
v0.10.29

$ npm -v
1.4.14
{% endhighlight %}

Agora podemos escrever nossa primeira aplicação Node. Dentro de um diretorio vazio, crie um arquivo `app.js` com o conteúdo abaixo:

{% highlight javascript %}

var http = require('http'); // import do módulo http

// cria um servidor para atender os requests
// devolve o response para o navegador
http.createServer(function (req, res) { 
  res.writeHead(200, {'Content-Type': 'text/plain'});
  res.end('RESPOSTA DO SERVIDOR!');
}).listen(1337, '127.0.0.1');

console.log('SERVIDOR RODANDO...');

{% endhighlight %}

No terminal, execute:

{% highlight console %}
$ node app.js
{% endhighlight %}

Abra o navegador e aponte para: http://127.0.0.1:1337

#### Arquitetura da plataforma

A figura abaixo mostra a arquitetura da plataforma Node e como os módulos se relacionam. Como vimos acima, criamos uma aplicação usando o módulo `http` que já vem por default quando instalamos a plataforma. Futuramente usaremos o `npm` para baixarmos outros módulos.

![]({{ site.url }}/assets/nodejs/core-node.png)

#### MongoDB <p/>

Vamos evoluir nossa aplição `app.js` para que passe a se comunicar com um banco de dados e persista algum dado. Nesse post iremos usar o banco [MongoDB][mongodb-site]. O MongoDB é um banco de dados open-source do tipo [NoSQL][nosql-wiki]. Por vários motivos esse banco vem sendo muito popular na comunidade Node.JS. 

Para nossa aplicação funcionar, precisamos:

* Instalar o MongoDB no ambiente;
* Instalar o driver que faz a comunicação `Node <-> MongoDB`;
* Instalar uma biblioteca de mapeamento ODM (Object Data Mapping) chamada Mongoose;
* Criar os arquivos `.js` necessários

<p/>
#### Instalando o MongoDB

Se você usa Mac OS, basta [seguir os passos do site do projeto][install-mongodb-macos]. Em seguida crie um diretorio `/data/db`:

{% highlight console %}
$ mkdir /data/db
{% endhighlight %}

E execute `sudo mongod`:

{% highlight console %}
$ sudo mongod
MongoDB starting : pid=11714 port=27017 dbpath=/data/db 64-bit host=MacBookPro.local
2015-01-31T17:35:03.527-0200 [initandlisten] db version v2.6.3
.
.
.
2015-01-31T17:35:04.630-0200 [initandlisten] waiting for connections on port 27017
{% endhighlight %}

Agora que o MongoDB está instalado e rodando, precisaremos instalar o driver de comunicação. Dentro do diretório do arquivo `app.js` execute:

{% highlight console %}
$ npm install mongodb
.
.
.
mongodb@1.4.29 node_modules/mongodb
├── kerberos@0.0.8 (nan@1.5.1)
└── bson@0.2.18 (nan@1.5.1)
{% endhighlight %}

E para o Mongoose:

{% highlight console %}
$ npm install mongoose
.
.
.
mongoose@3.8.22 node_modules/mongoose
├── regexp-clone@0.0.1
├── sliced@0.0.5
├── muri@0.3.1
├── hooks@0.2.1
├── mpath@0.1.1
├── mpromise@0.4.3
├── ms@0.1.0
├── mquery@0.8.0 (debug@0.7.4)
└── mongodb@1.4.28 (bson@0.2.18, kerberos@0.0.7)
{% endhighlight %}

Toda aplicação Node possui um arquivo chamado `package.json` que é resposavel por declarar as dependencias do projeto, versão, etc. Para gerar esse arquivo automaticamente execute o comando abaixo e tecle enter para todas as perguntas.

{% highlight console %}
$ npm init
{% endhighlight %}

{% highlight json %}
// package.json
{
  "name": "nodejs-blog-post",
  "version": "0.0.0",
  "main": "app.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "mongoose": "^3.8.22",
    "mongodb": "^1.4.29"
  },
  "devDependencies": {}
}
{% endhighlight %}

#### Modelando e salvando objetos com Mongoose

Para entender como vamos modelar nossos objetos com Mongoose precisamos entender o conceito de `Schema` e `Models`. De forma muito simplificada seria:

* [Schema][mongoose-schema]: define a estutura que queremos mapear.
* [Models][mongoose-models]: são instâncias do Schema, seriam os objetos que queremos salvar e recuperar do banco.

Vamos imaginar que após nossa modelagem, temos um objeto `Pessoa`. Então nosso `Schema` seria algo do tipo:

{% highlight javascript %}
var Mongoose = require('mongoose');
// schema
var pessoaSchema = new Schema({
   nome:String,
   telefone:Number,
   _enabled:Boolean
});

// gerando um Model a partir do Schema
var pessoaModel = Mongoose.model('Pessoa', pessoaSchema);
{% endhighlight %}

Precisamos agora estruturar o diretório da nossa aplicação para que nosso código fique modular. Vamos criar as seguintes camadas: `service`, `models` e `database`:

![]({{ site.url }}/assets/nodejs/estrutura-projeto.png)

A nível de código, a aplicação ficou assim:

![]({{ site.url }}/assets/nodejs/estrutura-projeto2.png)

Abaixo segue o código de cada arquivo com comentários:

{% highlight javascript %}
// database.js
var mongoose = require('mongoose'); // importa o módulo mongoose
mongoose.connect('mongodb://localhost/test'); // conecta no banco 'test'

// exporta os objetos
exports.Mongoose = mongoose;
exports.conexaoDB = mongoose.connection;
{% endhighlight %}

{% highlight javascript %}
// pessoa_model.js
var conexaoDB = require('../database/database'); 

// cria o Schema
var pessoaSchema = new conexaoDB.Mongoose.Schema({
   nome:String,
   telefone:Number,
   _enabled:Boolean
});

// cria um Model e exporta
exports.Pessoa = conexaoDB.Mongoose.model('Pessoa', pessoaSchema); 
{% endhighlight %}

{% highlight javascript %}
// pessoa_service.js
var database = require('../database/database');
var model = require('../models/pessoa_model');

// método para criar e salvar uma Pessoa
exports.criarSalvarPessoa = function criarSalvarPessoa() {

  // instanciamos um objeto Pessoa e populamos
  var pessoa = new model.Pessoa({
    nome: "Joao Silva",
    telefone: 123456,
    idade: true
  });
   
  // o Model possui um método save()
  pessoa.save(function(err, pessoa) {
    if (err) 
        return console.error(err);
    else {
      // gravado com sucesso!
      console.log("PERSISTIDO NO BANCO." + pessoa);
      database.conexaoDB.close();
    }
  });

};
{% endhighlight %}

{% highlight javascript %}
...
// app.js
// cria um servidor para atender os requests
http.createServer(function (req, res) { 
  res.writeHead(200, {'Content-Type': 'text/plain'});
  res.end('RESPOSTA DO SERVIDOR!');

  // chama nosso serviço
  pessoaService.criarSalvarPessoa();

}).listen(1337, '127.0.0.1');
{% endhighlight %}

Agora execute `$ node app.js` novamente e aponte para http://127.0.0.1:1337. A saída no console será:

<p/>
{% highlight console %}
SERVIDOR RODANDO...
PERSISTIDO NO BANCO.{ __v: 0,
  nome: 'Joao Silva',
  telefone: 123456,
  _id: 54cd43ddb0ce37593545194c }
{% endhighlight %}

Para confirmar, entre no shell do MongoDB:

{% highlight console %}
$ mongo
MongoDB shell version: 2.6.3
connecting to: test
>
{% endhighlight %}

Você pode explorar os registros fisicamente gravados através de um `show collections`:

{% highlight console %}
MongoDB shell version: 2.6.3
connecting to: test
> show collections
pessoas
system.indexes
>
{% endhighlight %}

E por último, pode executar um `find()` na `collection`.

{% highlight console %}
MongoDB shell version: 2.6.3
connecting to: test
> show collections
pessoas
system.indexes
> db.pessoas.find();
{ "_id" : ObjectId("54cd5777a417019c38673508"), "nome" : "Joao Silva", "telefone" : 123456, "__v" : 0 }
{% endhighlight %}

Bem, é isso. [O código do projeto está no Github][nodejs-mongodb].

Recomendo a leitura da documentação dos [comandos do MongoDB][mongodb-documentation] e como usa-los [através do Mongoose][mongoose-guide].

[nodejs-mongodb]: https://github.com/acneto/nodejs-mongodb
[mongoose-guide]:http://mongoosejs.com/docs/guide.html
[mongodb-documentation]:http://docs.mongodb.org/manual/crud/
[mongodb-site]:http://www.mongodb.org
[nosql-wiki]:http://en.wikipedia.org/wiki/NoSQL
[mongoose-schema]: http://mongoosejs.com/docs/guide.html
[mongoose-models]: http://mongoosejs.com/docs/models.html
[install-mongodb-macos]:http://docs.mongodb.org/manual/tutorial/install-mongodb-on-os-x/
[nodejs-site]:http://nodejs.org
[npm-site]:http://npmjs.com
[xcode-site]:https://developer.apple.com/xcode/