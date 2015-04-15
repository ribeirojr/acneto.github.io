---
layout: post
title:  "SVG em JavaScript com Raphaël JS"
date:   2015-01-01 17:00
categories: javascript svg raphael module pattern
---

<center>*Nesse post vou apresentar uma versão simplificada de uma solução escrita em JavaScript com a biblioteca Raphaël e seguindo o Revealing Module Pattern. Basicamente precisei criar uma página web onde os usuários pudessem desenhar livremente com o mouse em cima de uma imagem JPG em background. Esses desenhos são salvos no banco de dados e o usuário pode carrega-los para a tela posteriomente.* </center>

#### Apresenando o Raphaël

O formato SVG (Scalable Vectorial Graphics) é uma especificação para gráficos vetoriais largamente usado na Web. Tem inúmeras vantagens sobre imagens convencionais, como por exemplo não perder qualidade quando se aplica zoom.

Um SVG pode ser desenhado no HTML com a tag `svg`, por exemplo:

{% highlight html %}
<svg>
<rect  width="90" height="60" x="50" y="0" fill="red"/>
</svg>
{% endhighlight %}

Ou também pode ser referenciada na tag `<img/>`, por exemplo:

{% highlight html %}
<img src="imagem.svg" alt="bla"/>
{% endhighlight %}

A biblioteca JavaScript Raphaël abstrai a complexidade de desenhar o SVG "na mão" com uma API uniforme, simplificando o nosso trabalho:

{% highlight html %}
<html>
<body>
  Hello Raphael.
</body>
<script src="raphael-min.js"></script>

<script type="text/javascript">
  var paper = Raphael(10, 50, 320, 200); // x, y, width, height
  var circle = paper.circle(50, 40, 10);
  circle.attr("fill", "#f00");
</script>
</html>
{% endhighlight %}

Aqui nós criamos uma área de desenho para trabalhar definindo sua posição na tela, largura e altura - referenciado por `paper` e em seguinda mandamos desenhar um círculo nessa mesma área.

#### O projeto

Precisamos criar uma página web onde o usuário faça desenhos em cima de uma imagem JPEG como background. Esses desenhos precisam ser salvos no banco de dados (as coordenadas propriamente ditas) para serem carregados posteriormente.

Vamos começar com os `divs` necessários:

{% highlight html %}
<div align="center">
  <input id="ativar" value="Ativar desenhos" type="button" />
  <input id="salvar" value="Salvar desenhos" type="button" />
</div>

<div class="imagem" id="tela"></div>
{% endhighlight %}

O usuário tem que pressionar o botão para começar a desenhar. Para salvar idem. Agora criamos uma classe para carregar a imagem:

{% highlight css %}
<style>
.imagem {
    background:url(torre-eiffel.jpg);
    min-height: 100%;
    min-width: 100%;
    position: fixed;
}
</style>
{% endhighlight %}

Vamos carregar nossas bibliotecas:

{% highlight html %}
<script src="jquery-1.8.3-min.js"></script>
<script src="raphael-min.js"></script>
<script src="app.js"></script>
{% endhighlight %}

Usamos o jQuery para fazer bind nos eventos do mouse e localizar elementos no DOM. Agora vamos criar o `app.js` usando o Revealing Module Pattern.

Começamos criando um objeto para representar nossa tela:

{% highlight js %}
var ImagemBackground = {
  config: {
    altura: $("#tela").height(),
    largura: $("#tela").width()
  }
};
{% endhighlight %}

Seguindo o Module Pattern, vamos criar uma interface pública com um método `init` e deixar o resto privado:

{% highlight js %}

var MinhaTelaSVG = (function() {

  criarAreaDesenho = function() {
    paper = Raphael("tela", ImagemBackground.config.largura, ImagemBackground.config.altura);
    listaElementosTela = paper.set();
  }

  init = function() {
    criarAreaDesenho();
    bindBotoesClicados();
    console.log('init');
  }

  // public API
  return {
    init: init,
  };

})();
{% endhighlight %}

Assim, o desenvolvedor tem apenas um ponto de acesso para inicializar o código:

{% highlight js %}
MinhaTelaSVG.init();
{% endhighlight %}

Vamos fazer os binds necessários nos botões:

{% highlight js %}

var MinhaTelaSVG = (function() {

    /*
    Métodos privados
  */
  bindBotoesClicados = function() {

    $("#ativar").click(function(e) {
      bindMouseDown();
      bindMouseUp();
    });

  /*
    Enviar por AJAX para o servidor. Cada item do array corresponde a um desenho ta tela.
    Para dá load e plotar os desenhos posteriormentes só precisamos armazenar o X, Y, WIDTH, HEIGHT.
   */
    $("#salvar").click(function(e) {
      listaElementosTela.forEach(function e(elemento) {
        console.log("ID ELEMENTO " + elemento.data('id'));
        // TODO: implementar post por Ajax
      });

    });
  }
})();

{% endhighlight %}

Quando o usuário manipula o mouse, pode criar retangulos na tela e armazena os mesmos em um array.

{% highlight js %}

var MinhaTelaSVG = (function() {
bindMouseUp = function() {
    $("#tela").mouseup(function(e) {
      $('#tela').unbind('mouseup');
      $('#tela').unbind('mousedown');
      $('#tela').unbind('mousemove');

      var BBox = objetoRect.getBBox();
      if ( BBox.width == 0 && BBox.height == 0 )
        objetoRect.remove();
    });
  }

  bindMouseDown = function() {
    $("#tela").mousedown(function(e) {
      e.originalEvent.preventDefault();

      var offset = $("#tela").offset();
      var mouseDownX = e.pageX - offset.left;
      var mouseDownY = e.pageY - offset.top;

      objetoRect = desenharRetanguloNaTela(mouseDownX, mouseDownY, 0, 0);

      $("#tela").mousemove(function(e) {
        var offset = $("#tela").offset();

        // calcula tamanho da área selecionada
        var upX = e.pageX - offset.left;
        var upY = e.pageY - offset.top;
        var width = upX - mouseDownX;
        var height = upY - mouseDownY;

        objetoRect.attr( { "width": width > 0 ? width : 0,
                  "height": height > 0 ? height : 0 } );
      });

      // temos um array com todos os nossos desenhos da tela
      listaElementosTela.push(objetoRect);
    });
  }
})();

{% endhighlight %}

Agora usamos a variável paper para criar o desenho. Cada objeto criado na tela é representado por um ID único.

{% highlight js %}

var MinhaTelaSVG = (function() {

  desenharRetanguloNaTela = function(x, y, w, h) {
    var element = paper.rect(x, y, w, h); // cria retangulo na posição e com tamanho indicado
    var idObjeto = 'rct' + x + y;

    element.attr({
      fill: "gray",
      opacity: .5,
      stroke: "gray"
    });

    element.data('posicaoX', x);
    element.data('posicaoY', y);
    element.data('id', idObjeto);

    $(element.node).attr('id', 'rct' + x + y); // garante ID do objeto no DOM para referencia futura

    return element;
  }
})();
{% endhighlight %}

É isso. Se tiver interesse pode [baixar o código do projeto][javascript-svg-raphael-tutorial] e roda-lo no navegador. Abaixo tem a imagem do arquivo `index.html`.

![]({{ site.url }}/assets/svg/svg.png)

[javascript-svg-raphael-tutorial]: https://github.com/acneto/javascript-svg-raphael-tutorial


