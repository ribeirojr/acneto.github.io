---
layout: post
title:  "Código Modular em JavaScript"
date:   2015-01-01 15:00
categories: javascript patterns 
---

<center>*Nesse post irei mostrar como organizar seu código JavaScript com os padrões Module Pattern e Revealing Module Pattern. Esses padrões ajudam a modularizar seu código evitando que seu código fique espalhado por todo o sistema. Iremos começar explorando a criação de objetos básicos, declaração de variáveis, namespaces e por fim apresentar os padrões citados.* </center>

#### Colisões globais

Quando você cria funções, variáveis ou objetos, por default tudo é armazenado no global scope. Sendo assim, quando você cria uma função X em um arquivo qualquer, essa função pode ser sobrescrita por outro arquivo que tenha uma função com o mesmo nome. O exemplo seguinte mostra exatamente isso. A função que realmente será executada será a do último arquivo carregado, nesse caso fileTwo.js.

{% highlight html %}
<html>
<body onload="myFunctionSameName(3, 2);">
RESULT: <div id="result"></div>
</body>
<script type="text/javascript" src="fileOne.js"></script>
<script type="text/javascript" src="fileTwo.js"></script>
</html>
{% endhighlight %}

{% highlight javascript %}
// fileOne.js
var myGlobalVar = 100;
function myFunctionSameName(a, b) {
    var div = document.getElementById("result");
    div.innerHTML = a * b;
    console.log("myGlobalVar " + myGlobalVar);
}
{% endhighlight %}

{% highlight javascript %}
// fileTwo.js
var myGlobalVar = 200;
function myFunctionSameName(a, b) {
    var div = document.getElementById("result");
    div.innerHTML = a + b;
     console.log("myGlobalVar " + myGlobalVar);
}
{% endhighlight %}

#### Escopos e declação de variáveis

Vamos relembrar o conceito do escopo de variáveis locais e globais em JavaScript. Nesse exemplo é fácil de entender a diferença:

{% highlight javascript %}
var iamVar = "global";
function check() {
    var iamVar = "local";      // variável local
    return iamVar;             // retorna valor da variável local
}
check();     // => "local"
{% endhighlight %}

Aproveite para ir se acostumando com esse outro estilo de declaração de variáveis:

{% highlight javascript %}
var firstName = "Joao",
    lastName = "Silva",
    age = 10;
{% endhighlight %}

#### Criação de objetos

JavaScript não tem o conceito de classes como em Java ou C#. Mas aplicando algumas técnicas podemos sim trabalhar de forma modularizada. Vamos começar com a criação básica de objetos. Existem basicamente duas maneiras para criar objetos em JS: usando `Constructor functions` ou `Literal notation.

{% highlight javascript %}
// Constructor notation
function SomeEmptyBasicObject = {
};

// Literal notation
var SomeEmptyLiteralObject = {
};
{% endhighlight %}

Agora, vamos adicionar propriedades e métodos em ambos os casos:

{% highlight javascript %}
// literal notation
var SomeThingLiteral = {
    // public properties
    someText: 'text literal',
    someValue: 777,

    // public methods
    fooMethod : function() {
     console.log("Literal method " + this.someText + " " + this.someValue);
    }
};
console.log(SomeThingLiteral.someText);
console.log(SomeThingLiteral.someValue);
SomeThingLiteral.fooMethod();
{% endhighlight %}


{% highlight javascript %}
// construction notation
function SomeThingBasic() {
    // public properties
    this.someText = 'text basic';
    this.someValue = 123;

    // public methods
    this.fooMethod = function() {
     console.log("Basic method " + this.someText + " " + this.someValue);
    };

};
var obj = new SomeThingBasic(); // create an object
console.log(obj.someText);
console.log(obj.someValue);
obj.fooMethod();
{% endhighlight %}

Você pode rodar esses exemplos e ver o resultado com o FireBug, por exemplo. Bom, note que se você usar uma `Constructor function` será necessário instanciar o objeto usando `new. Isso gera um objeto diferente em memória a cada nova chamada - isso lembra a forma tradicional de criação de objetos em outras linguagens.

Já se usar `Literal notation você não precisa instanciar o objeto e caso o estado do objeto seja alterado, todos os clientes que estejam apontando para esse objeto irão perceber essa mudança - afinal, é o mesmo objeto em memória. 

Note também que em ambos os casos as propriedades e métodos são públicos - em JavaScript, por default, tudo é público.

#### Prototype Pattern

Todo objeto em JavaScript tem uma propriedade chamada `prototype`. Ela serve para usarmos o conceito de prototipação de objetos. Prototipação permite que um objeto herde, sobrescreva ou extenda funcionalidades.

Por exemplo, vamos criar um objeto e definir seu estado:

{% highlight javascript %}
function Car(model, year) {
 // define o estado do objeto
  this.model = model;
  this.year = year;
}
{% endhighlight %}

Agora, vamos usar propriedade prototype para adicionar comportamento (um método) a esse objeto:

{% highlight javascript %}
Car.prototype.doSomething = function () {
    console.log("MODEL " + this.model + " YEAR " + this.year);
};
{% endhighlight %}

Agora podemos usar o objeto normalmente:

{% highlight javascript %}
var onix = new Car("Onix", 2014);
onix.doSomething();
{% endhighlight %}

Uma maneira alternativa de se usar o protype seria:

{% highlight javascript %}
Car.prototype = {
  doSomething: function() {
     console.log("MODEL " + this.model + " YEAR " + this.year);
  }
}
{% endhighlight %}

#### Namespaces

Como vimos no começo do post, é importante encontramos alguma forma de assegurarmos que objetos, funções ou variáveis existam de maneira única na nossa aplicação. O JavaScript tem o conceito de namespaces que ajuda nessa tarefa.

Nesse exemplo, iremos criar dois arquivos JavaScript contendo objetos com o mesmo nome, porém cada objeto terá seu próprio namespace, assegurando que sejam únicos na aplicação.

Vamos começar criando o namespace:

{% highlight javascript %}
// fileOne.js
var nsFileOne = nsFileOne || {}; // se já existir, use, senão crie um objeto vazio
{% endhighlight %}

Agora vamos criar um objeto - SomeObject - porém adicionando o namespace que criamos:

{% highlight javascript %}
//fileOne.js

var nsFileOne = nsFileOne || {};

nsFileOne.SomeObject = function() {
  this.someText = 'Hello, I'm from FILE ONE';
};

nsFileOne.SomeObject.prototype = {
  fooMethod: function() {
      var div = document.getElementById("result");
      div.innerHTML = this.someText;    
  }
}
{% endhighlight %}

Agora vamos criar o próximo objeto com nome idêntico, mas com um outro namespace.

{% highlight javascript %}
// fileTwo.js

var nsFileTwo = nsFileTwo || {};

nsFileTwo.SomeObject = function() {
  this.someText = 'Hello, I'm from FILE TWO';
};

nsFileTwo.SomeObject.prototype = {
  fooMethod: function() {
      var div = document.getElementById("result");
      div.innerHTML = this.someText;    
  }
}
{% endhighlight %}

Quando quisermos usar um dos objetos, basta especificar qual deles usar através do namespace:

{% highlight javascript %}
var xpto = new nsFileTwo.SomeObject();
xpto.fooMethod();
{% endhighlight %}

#### Módulos em JavaScript

Existem vários padrões que ajudam a modularizar nosso código JavaScript. Ou seja, temos como criar pedaços de código isolados com métodos e variáveis encapsulados. Aqui iremos abordar o Module Pattern e Revealing Module Pattern.

Vamos começar criando módulo usando usando o Pattern Module:

{% highlight javascript %}
/*
Module Pattern tem conceito de privacidade
*/
var ModulePatternExample = (function() {

function minhaFuncaoPrivada() {
  console.log("private");
};

// return é público
return {
         myPublicVar: "123456",

         somarDoisNumeros: function xpto(x, y) {
           return x + y;
         }
        };
})();

console.log("ModulePatternExample " + ModulePatternExample.myPublicVar);
console.log("ModulePatternExample " + ModulePatternExample.somarDoisNumeros(2, 2));

{% endhighlight %}

Como você pode notar, tudo que precisa ser público está no nosso return, sejam variáveis ou funções, deixando todo o resto privado. Vamos evoluir esse exemplo para o Revealing Prototype Pattern. A principal vantagem desse padrão é que teremos uma sintaxe mais consistente e clara. Criamos uma API pública onde exportamos apenas o que desejamos:

{% highlight javascript %}
/*
Revealing Module Pattern
*/
var ModulePatternRevealingExample = (function() {
 
// private
var algumValor = 1000;
 
// private
var calc = function minhaFuncao(x, y) {
return x * y * 2;
};
 
// public API
// cria e retorna um novo objeto
return {
calculadora : calc
};
 
})();
console.log("ModulePatternRevealingExample " + ModulePatternRevealingExample.calculadora(4, 4));
{% endhighlight %}

Bem, é isso. Espero que tenha sido claro. O código dos exemplos estão [no meu Github.][patterns-javascript]

[patterns-javascript]: https://github.com/acneto/codigo-modular-javascript.git