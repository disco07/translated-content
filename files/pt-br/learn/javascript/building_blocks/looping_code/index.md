---
title: Código em loop
slug: Learn/JavaScript/Building_blocks/Looping_code
tags:
  - Artigo
  - CodingScripting
  - DO
  - Guide
  - Iniciante
  - JavaScript
  - Loop
  - break
  - continue
  - for
  - while
translation_of: Learn/JavaScript/Building_blocks/Looping_code
original_slug: Aprender/JavaScript/Elementos_construtivos/Código_em_loop
---
<div>{{LearnSidebar}}</div>

<div>{{PreviousMenuNext("Learn/JavaScript/Building_blocks/conditionals","Learn/JavaScript/Building_blocks/Functions", "Learn/JavaScript/Building_blocks")}}</div>

<p class="summary">As linguagens de programação são muito úteis para concluir rapidamente tarefas repetitivas, desde vários cálculos básicos até praticamente qualquer outra situação em que você tenha muitos itens semelhantes para manipular. Aqui, veremos as estruturas de loop disponíveis no JavaScript que ajudam com essas necessidades..</p>

<table class="learn-box standard-table">
 <tbody>
  <tr>
   <th scope="row">Prerequisitos:</th>
   <td>Conhecimento básico em informática, um básico entendimento  de HTML e CSS, <a href="/en-US/docs/Learn/JavaScript/First_steps">JavaScript first steps</a>.</td>
  </tr>
  <tr>
   <th scope="row">Objectivo:</th>
   <td>Entender como usar loops em JavaScript.</td>
  </tr>
 </tbody>
</table>

<h2 id="Mantenha-me_no_looplaço">Mantenha-me no loop(laço)</h2>

<p>Loops, loops, loops. Além de estarem associados a <a href="https://en.wikipedia.org/wiki/Froot_Loops">populares cereais matinais</a>, <a href="https://pt.wikipedia.org/wiki/Montanha-russa">montanhas-russas</a> e <a href="https://pt.wikipedia.org/wiki/Loop_(m%C3%BAsica)">produção musical</a>, eles também são um conceito importante na programação. O loop de programação é como fazer a mesma coisa repetidas vezes - o que é chamado de <strong>iteração</strong> na linguagem de programação.</p>

<p>Vamos considerar o caso de um agricultor que quer se certificar de que ele terá comida suficiente para alimentar sua família durante a semana. Ele pode usar o seguinte loop para conseguir isso:</p>

<p><br>
 <img alt="" src="https://mdn.mozillademos.org/files/13755/loop_js-02-farm.png" style="display: block; margin: 0 auto;"></p>

<p>Um loop geralmente possui um ou mais dos seguintes itens:</p>

<ul>
 <li> O <strong>contador</strong>, que é inicializado com um certo valor - este é o ponto inicial do loop ("Iniciar: não tenho comida", figura acima).</li>
 <li>A <strong>condição de saída</strong>, que é o critério no qual o loop pára - geralmente o contador atinge um certo valor. Isso é ilustrado por "Tenho comida suficiente?",  na figura acima. Vamos dizer que ele precisa de 10 porções de comida para alimentar sua família.</li>
 <li>Um <strong>iterador</strong>, que geralmente incrementa o contador em uma pequena quantidade a cada loop, sucessivamente, até atingir a condição de saída. Nós não ilustramos explicitamente isso acima, mas poderíamos pensar sobre o agricultor ser capaz de coletar 2 porções de comida por hora. Depois de cada hora, a quantidade de comida que ele coletou é incrementada em dois, e ele verifica se ele tem comida suficiente. Se ele atingiu 10 porções (a condição de saída), ele pode parar de coletar e ir para casa.</li>
</ul>

<p>No seu {{glossary("pseudocode")}}, isso seria algo como o seguinte:</p>

<pre>loop(food = 0; foodNeeded = 10) {
  if (food = foodNeeded) {
    exit loop;
    // Nós temos comida o suficiente, Vamos para casa
  } else {
    food += 2; // Passe uma hora coletando mais 2 alimentos(food)
    // loop será executado novamente
  }
}</pre>

<p>Assim, a quantidade de comida necessária é fixada em 10, e o montante que o agricultor tem atualmente é fixado em 0. Em cada iteração do ciclo, verificamos se a quantidade de alimento que o agricultor tem é igual à quantidade que ele precisa. Se assim for, podemos sair do loop. Se não, o agricultor gasta mais uma hora coletando duas porções de comida, e o laço é executado novamente.</p>

<h3 id="Porque_se_importar">Porque se importar?</h3>

<p>Neste ponto, você provavelmente já entendeu o conceito de alto nível por trás dos loops, mas provavelmente está pensando "OK, ótimo, mas como isso me ajuda a escrever um código JavaScript melhor?" Como dissemos anteriormente, <strong>os loops têm tudo a ver com fazer a mesma coisa repetidas vezes, o que é ótimo para concluir rapidamente tarefas repetitivas</strong>.</p>

<p>Muitas vezes, o código será um pouco diferente em cada iteração sucessiva do loop, o que significa que você pode completar toda uma carga de tarefas que são semelhantes, mas não são totalmente iguais — se você tem muitos cálculos diferentes para fazer, e você quer fazer um diferente do outro, e não o mesmo repetidamente!</p>

<p>Vejamos um exemplo para ilustrar perfeitamente por que os loops são uma coisa tão boa. Digamos que quiséssemos desenhar 100 círculos aleatórios em um elemento  {{htmlelement("canvas")}}  (pressione o botão Atualizar para executar o exemplo várias vezes para ver os conjuntos aleatórios diferentes):</p>

<div class="hidden">
<h6 id="Hidden_code">Hidden code</h6>

<pre class="brush: html">&lt;!DOCTYPE html&gt;
&lt;html&gt;
  &lt;head&gt;
    &lt;meta charset="utf-8"&gt;
    &lt;title&gt;Random canvas circles&lt;/title&gt;
    &lt;style&gt;
      html {
        width: 100%;
        height: inherit;
        background: #ddd;
      }

      canvas {
        display: block;
      }

      body {
        margin: 0;
      }

      button {
        position: absolute;
        top: 5px;
        left: 5px;
      }
    &lt;/style&gt;
  &lt;/head&gt;
  &lt;body&gt;

  &lt;button&gt;Update&lt;/button&gt;

  &lt;canvas&gt;&lt;/canvas&gt;


    &lt;script&gt;
    var btn = document.querySelector('button');
    var canvas = document.querySelector('canvas');
    var ctx = canvas.getContext('2d');

    var WIDTH = document.documentElement.clientWidth;
    var HEIGHT = document.documentElement.clientHeight;

    canvas.width = WIDTH;
    canvas.height = HEIGHT;

    function random(number) {
      return Math.floor(Math.random()*number);
    }

    function draw() {
      ctx.clearRect(0,0,WIDTH,HEIGHT);
      for (var i = 0; i &lt; 100; i++) {
        ctx.beginPath();
        ctx.fillStyle = 'rgba(255,0,0,0.5)';
        ctx.arc(random(WIDTH), random(HEIGHT), random(50), 0, 2 * Math.PI);
        ctx.fill();
      }
    }

    btn.addEventListener('click',draw);

    &lt;/script&gt;

  &lt;/body&gt;
&lt;/html&gt;</pre>
</div>

<p>{{ EmbedLiveSample('Hidden_code', '100%', 400, "", "", "hide-codepen-jsfiddle") }}</p>

<p>Você não precisa entender todo esse código por enquanto, mas vamos ver a parte do código que realmente desenha os 100 círculos:</p>

<pre class="brush: js">for (var i = 0; i &lt; 100; i++) {
  ctx.beginPath();
  ctx.fillStyle = 'rgba(255,0,0,0.5)';
  ctx.arc(random(WIDTH), random(HEIGHT), random(50), 0, 2 * Math.PI);
  ctx.fill();
}</pre>

<ul>
 <li><code>random()</code>, definido anteriormente no código, retorna um número inteiro entre <code>0</code> e <code>x-1</code>.</li>
 <li><code>WIDTH</code> e <code>HEIGHT</code> são a largura e a altura da janela interna do navegador. </li>
</ul>

<p>Você deve ter notado - estamos usando um loop para executar 100 iterações desse código, cada uma delas desenhando um círculo em uma posição aleatória na página. A quantidade de código necessária seria a mesma se estivéssemos desenhando 100 círculos, 1.000 ou 10.000. Apenas um número tem que mudar.</p>

<p>Se não estivéssemos usando um loop aqui, teríamos que repetir o seguinte código para cada círculo que queríamos desenhar:</p>

<pre class="brush: js">ctx.beginPath();
ctx.fillStyle = 'rgba(255,0,0,0.5)';
ctx.arc(random(WIDTH), random(HEIGHT), random(50), 0, 2 * Math.PI);
ctx.fill();</pre>

<p>Isso ficaria muito chato, difícil e lento de manter. Loops são realmente os melhores.</p>

<h2 id="O_for_loop_padrão">Loop padrão for</h2>

<p>Vamos começar a explorar alguns exemplos específicos de construções de loop. O primeiro e que você usará na maior parte do tempo, é o loop <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for">for</a> - ele tem a seguinte sintaxe:</p>

<pre>for (inicializador; condição-saída; expressão-final) {
  // código para executar
}</pre>

<p>Aqui nós temos:</p>

<ol>
 <li>A palavra-chave <code>for</code>, seguido por parênteses.</li>
 <li>Dentro do parênteses temos três itens, separados por ponto e vírgula:
  <ol>
   <li>O<strong> inicializador</strong>— geralmente é uma variável configurada para um número, que é incrementado para contar o número de vezes que o loop foi executado. É também por vezes referido como uma <strong>variável de contador</strong>.</li>
   <li>A <strong>condição-saída</strong> — como mencionado anteriormente, aqui é definido quando o loop deve parar de executar. Geralmente, essa é uma expressão que apresenta um operador de comparação, um teste para verificar se a condição de saída foi atendida.</li>
   <li>A <strong>expressão-final </strong>— isso sempre é avaliado (ou executado) cada vez que o loop passou por uma iteração completa. Geralmente serve para incrementar (ou, em alguns casos, decrementar) a variável do contador, aproximando-a do valor da condição de saída.</li>
  </ol>
 </li>
 <li>Algumas chaves contêm um bloco de código - esse código será executado toda vez que o loop for iterado.</li>
</ol>

<p>Vejamos um exemplo real para podermos visualizar o que isso faz com mais clareza.</p>

<pre class="brush: js">var cats = ['Bill', 'Jeff', 'Pete', 'Biggles', 'Jasmin'];
var info = 'My cats are called ';
var para = document.querySelector('p');

for (var i = 0; i &lt; cats.length; i++) {
  info += cats[i] + ', ';
}

para.textContent = info;</pre>

<p>Isso nos dá a seguinte saída:</p>

<div class="hidden">
<h6 id="Hidden_code_2">Hidden code 2</h6>

<pre class="brush: html">&lt;!DOCTYPE html&gt;
&lt;html&gt;
  &lt;head&gt;
    &lt;meta charset="utf-8"&gt;
    &lt;title&gt;Basic for loop example&lt;/title&gt;
    &lt;style&gt;

    &lt;/style&gt;
  &lt;/head&gt;
  &lt;body&gt;

  &lt;p&gt;&lt;/p&gt;


    &lt;script&gt;
    var cats = ['Bill', 'Jeff', 'Pete', 'Biggles', 'Jasmin'];
    var info = 'My cats are called ';
    var para = document.querySelector('p');

    for (var i = 0; i &lt; cats.length; i++) {
      info += cats[i] + ', ';
    }

    para.textContent = info;

    &lt;/script&gt;

  &lt;/body&gt;
&lt;/html&gt;</pre>
</div>

<p>{{ EmbedLiveSample('Hidden_code_2', '100%', 60, "", "", "hide-codepen-jsfiddle") }}</p>

<div class="note">
<p><strong>Nota: </strong>Você pode encontrar este <a href="https://github.com/mdn/learning-area/blob/master/javascript/building-blocks/loops/basic-for.html">código de exemplo no GitHub</a> (também <a href="https://mdn.github.io/learning-area/javascript/building-blocks/loops/basic-for.html">veja em execução </a>).</p>
</div>

<p>Aqui mostra um loop sendo usado para iterar os itens em uma matriz e fazer algo com cada um deles - um padrão muito comum em JavaScript. Aqui:</p>

<ol>
 <li>O iterador, <code>i</code>, começa em <code>0</code> (<code>var i = 0</code>).</li>
 <li>Foi dito para executar até que não seja menor que o comprimento do array dos gatos. Isso é importante - a condição de saída mostra a condição sob a qual o loop ainda será executado. No caso, enquanto <code>i &lt; cats.length</code>  for verdadeiro, o loop continuará executando.</li>
 <li>Dentro do loop, nós concatenamos o item de loop atual (<code>cats[i]</code> é <code>cats[o nome do item que está iterado no momento]</code>) junto com uma vírgula e um espaço, no final da variável <code>info</code> . Assim:
  <ol>
   <li>Durante a primeira execução, <code>i = 0</code>, então <code>cats[0] + ', '</code> será concatenado na variável info ("Bill").</li>
   <li>Durante a segunda execução, <code>i = 1</code>, so <code>cats[1] + ', '</code> será concatenado na variável info ("Jeff, ")</li>
   <li>E assim por diante. Após cada execução do loop, 1 será adicionado à <code>i</code> (<code>i++</code>), então o processo será iniciado novamente.</li>
  </ol>
 </li>
 <li>Quando <code>i</code> torna-se igual a <code>cats.length</code>, o loop é interrompido e o navegador passará para o próximo trecho de código abaixo do loop.</li>
</ol>

<div class="note">
<p><strong>Nota</strong>:Nós fizemos a condição de saída <code>i &lt; cats.length</code>, e não <code>i &lt;= cats.length</code>, porque os computadores contam a partir de 0, não 1 - estamos começando <code>i</code> em <code>0</code>, e indo até <code>i = 4</code> (o index do último item do array). <code>cats.length</code> retorna 5, pois há 5 itens no array, mas não queremos chegar até  <code>i = 5</code>, pois isso retornaria  <code>undefined</code> para o último item (não há nenhum item  no índice 5 do array). Então, portanto, queremos ir até 1 a menos de  <code>cats.length</code> (<code>i &lt;</code>), não é o mesmo que <code>cats.length</code> (<code>i &lt;=</code>).</p>
</div>

<div class="note">
<p><strong>Nota</strong>: Um erro comum nas condições de saída é usá-las "igual a" (<code>===</code>) em vez de dizer "menor ou igual a" (<code>&lt;=</code>). Se quiséssemos executar nosso loop até  <code>i = 5</code>, a condição de saída precisaria ser  <code>i &lt;= cats.length</code>. Se nós setarmos para  <code>i === cats.length</code>, o loop não seria executado em todos, porque  <code>i</code> não é igual a <code>5</code> na primeira iteração do loop, a execução pararia imediatamente.</p>
</div>

<p>Um pequeno problema que nos resta é que a sentença de saída final não é muito bem formada:</p>

<blockquote>
<p>Meus gatos se chamam: Bill, Jeff, Pete, Biggles, Jasmin,</p>
</blockquote>

<p>Neste caso, queremos alterar a concatenação na iteração final do loop, para que não tenhamos uma vírgula no final da frase. Bem, não há problema - podemos muito bem inserir uma condicional dentro do nosso loop for para lidar com este caso especial:</p>

<pre class="brush: js">for (var i = 0; i &lt; cats.length; i++) {
  if (i === cats.length - 1) {
    info += 'and ' + cats[i] + '.';
  } else {
    info += cats[i] + ', ';
  }
}</pre>

<div class="note">
<p><strong>Nota: </strong>Você pode encontrar este <a href="https://github.com/mdn/learning-area/blob/master/javascript/building-blocks/loops/basic-for-improved.html">código de exemplo no GitHub</a> (também <a href="https://mdn.github.io/learning-area/javascript/building-blocks/loops/basic-for-improved.html">veja em execução </a>).</p>
</div>

<div class="warning">
<p><strong>Importante</strong>: Com <strong>for</strong> - como acontece com todos os loops - você deve certificar-se de que o inicializador está iterado(configurado) para que ele atinja a condição de saída. Caso contrário, o loop continuará indefinidamente executando e o navegador irá forçá-lo a parar ou falhará. Isso é chamado de<strong> loop infinito</strong>.</p>
</div>

<h2 id="Saindo_do_loop_com_o_break">Saindo do loop com o break</h2>

<p>Se você quiser sair de um loop antes que todas as iterações sejam concluídas, você poderá usar a instrução <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/break">break</a>. Nós já encontramos isso em um artigo anterior, quando observamos as instruções switch - quando um determinado caso é atendido em uma <a href="https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Building_blocks/conditionals#switch_statements">condição do switch</a> e que corresponde à expressão de entrada informada, a instrução break sai imediatamente da instrução switch e passa para o trecho de código seguinte.</p>

<p>É o mesmo com loops  — um comando  <code>break</code>  irá imediatamente sair do loop e fazer o navegador passar para o código seguinte.</p>

<p>Digamos que quiséssemos pesquisar por uma variedade de contatos e números de telefone e retornar apenas o número que queríamos encontrar? Primeiro, algum HTML simples  — um texto {{htmlelement("input")}} permitindo-nos iserir um nome para pesquisar, {{htmlelement("button")}} elemento para submeter a pesquisa e um {{htmlelement("p")}} elemento para mostrar o resultado em:</p>

<pre class="brush: html">&lt;label for="search"&gt;Search by contact name: &lt;/label&gt;
&lt;input id="search" type="text"&gt;
&lt;button&gt;Search&lt;/button&gt;

&lt;p&gt;&lt;/p&gt;</pre>

<p>Agora para no JavaScript:</p>

<pre class="brush: js">var contacts = ['Chris:2232322', 'Sarah:3453456', 'Bill:7654322', 'Mary:9998769', 'Dianne:9384975'];
var para = document.querySelector('p');
var input = document.querySelector('input');
var btn = document.querySelector('button');

btn.addEventListener('click', function() {
  var searchName = input.value;
  input.value = '';
  input.focus();
  for (var i = 0; i &lt; contacts.length; i++) {
    var splitContact = contacts[i].split(':');
    if (splitContact[0] === searchName) {
      para.textContent = splitContact[0] + '\'s number is ' + splitContact[1] + '.';
      break;
    } else {
      para.textContent = 'Contact not found.';
    }
  }
});</pre>

<div class="hidden">
<h6 id="Hidden_code_3">Hidden code 3</h6>

<pre class="brush: html">&lt;!DOCTYPE html&gt;
&lt;html&gt;
  &lt;head&gt;
    &lt;meta charset="utf-8"&gt;
    &lt;title&gt;Simple contact search example&lt;/title&gt;
    &lt;style&gt;

    &lt;/style&gt;
  &lt;/head&gt;
  &lt;body&gt;

  &lt;label for="search"&gt;Search by contact name: &lt;/label&gt;
  &lt;input id="search" type="text"&gt;
  &lt;button&gt;Search&lt;/button&gt;

  &lt;p&gt;&lt;/p&gt;


    &lt;script&gt;
    var contacts = ['Chris:2232322', 'Sarah:3453456', 'Bill:7654322', 'Mary:9998769', 'Dianne:9384975'];
    var para = document.querySelector('p');
    var input = document.querySelector('input');
    var btn = document.querySelector('button');

    btn.addEventListener('click', function() {
      var searchName = input.value;
      input.value = '';
      input.focus();
      for (var i = 0; i &lt; contacts.length; i++) {
        var splitContact = contacts[i].split(':');
        if (splitContact[0] === searchName) {
          para.textContent = splitContact[0] + '\'s number is ' + splitContact[1] + '.';
          break;
        } else if (i === contacts.length-1)
              para.textContent = 'Contact not found.';
        }
    });
    &lt;/script&gt;

  &lt;/body&gt;
&lt;/html&gt;</pre>
</div>

<p>{{ EmbedLiveSample('Hidden_code_3', '100%', 100, "", "", "hide-codepen-jsfiddle") }}</p>

<ol>
 <li>Primeiramente, temos algumas definições de variáveis —  temos um array com as informações dos contatos, cada item é uma string e contém um nome e um número de telefone separados por dois pontos.</li>
 <li>Em seguida, anexamos um ouvinte de evento ao botão (<code>btn</code>),  de modo que quando ele é pressionado, algum código é invocado para executar a pesquisa e retornar os resultados.</li>
 <li>Armazenamos o valor que foi inserido no input de texto em uma variável chamada  <code>searchName</code>, antes de limpar a entrada de texto e focar novamente, deixando o campo pronto para a próxima pesquisa.</li>
 <li>Agora, na parte interessante, o loop for:
  <ol>
   <li>Iniciamos o contador em <code>0</code>, executamos o loop até que o contador não seja menor que  <code>contacts.length</code>, e incrementamos <code>i</code> com 1 depois e cada iteração do loop.</li>
   <li>Dentro do loop, primeiro dividimos o contato atual (<code>contacts[i]</code>) no caractere " : " e armazenamos os dois valores resultantes em uma matriz chamada  <code>splitContact</code>.</li>
   <li>Em seguida, usamos uma instrução condicional para testar se o  <code>splitContact[0]</code> (nome do contato) é igual ao  <code>searchName</code>. Se for igual, inserimos uma string no parágrafo para mostrar em tela qual é o número do contato e usamos o <code>break</code> para encerrar o loop.</li>
  </ol>
 </li>
 <li>
  <p>Após <code>(contacts.length-1)</code> iterações, se o nome do contato não corresponder à pesquisa inserida, o texto do parágrafo será definido como "Contato não encontrado" e o loop continuará a iterar.</p>
 </li>
</ol>

<div class="note">
<p><strong>Nota</strong>: Você pode encontrar este <a href="https://github.com/mdn/learning-area/blob/master/javascript/building-blocks/loops/contact-search.html">código de exemplo no GitHub</a> (também <a href="https://mdn.github.io/learning-area/javascript/building-blocks/loops/contact-search.html">veja em execução</a> ).</p>
</div>

<h2 id="Ignorando_iterações_com_continue">Ignorando iterações com continue</h2>

<p>A instrução <a href="/en-US/docs/Web/JavaScript/Reference/Statements/continue">continue</a> funciona de maneira semelhante ao  <code>break</code>, mas ao invés de sair inteiramente do loop, ele pula para a próxima iteração do loop. Vejamos outro exemplo que usa um número como entrada e retorna apenas os números que são quadrados de inteiros (números inteiros).</p>

<p>O HTML é basicamente o mesmo do último exemplo - uma entrada de texto simples e um parágrafo para saída. O JavaScript é basicamente o mesmo, embora o próprio loop seja um pouco diferente:</p>

<pre class="brush: js">var num = input.value;

for (var i = 1; i &lt;= num; i++) {
  var sqRoot = Math.sqrt(i);
  if (Math.floor(sqRoot) !== sqRoot) {
    continue;
  }

  para.textContent += i + ' ';
}</pre>

<p>Aqui está a saída:</p>

<div class="hidden">
<h6 id="Hidden_code_4">Hidden code 4</h6>

<pre class="brush: html">&lt;!DOCTYPE html&gt;
&lt;html&gt;
  &lt;head&gt;
    &lt;meta charset="utf-8"&gt;
    &lt;title&gt;Integer squares generator&lt;/title&gt;
    &lt;style&gt;

    &lt;/style&gt;
  &lt;/head&gt;
  &lt;body&gt;

  &lt;label for="number"&gt;Enter number: &lt;/label&gt;
  &lt;input id="number" type="text"&gt;
  &lt;button&gt;Generate integer squares&lt;/button&gt;

  &lt;p&gt;Output: &lt;/p&gt;


    &lt;script&gt;
    var para = document.querySelector('p');
    var input = document.querySelector('input');
    var btn = document.querySelector('button');

    btn.addEventListener('click', function() {
      para.textContent = 'Output: ';
      var num = input.value;
      input.value = '';
      input.focus();
      for (var i = 1; i &lt;= num; i++) {
        var sqRoot = Math.sqrt(i);
        if (Math.floor(sqRoot) !== sqRoot) {
          continue;
        }

        para.textContent += i + ' ';
      }
    });
    &lt;/script&gt;

  &lt;/body&gt;
&lt;/html&gt;</pre>
</div>

<p>{{ EmbedLiveSample('Hidden_code_4', '100%', 100, "", "", "hide-codepen-jsfiddle") }}</p>

<ol>
 <li>Nesse caso, a entrada deve ser um número (<code>num</code>). O loop <code>for</code> recebe um contador começando em 1 (como não estamos interessados em 0 neste caso), uma condição de saída que diz que o loop irá parar quando o contador se tornar maior que o input <code>num</code>, e um iterador que adiciona 1 ao contador a cada vez.</li>
 <li>Dentro do loop, encontramos a raiz quadrada de cada número usando <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math/sqrt">Math.sqrt(i)</a>, e, em seguida, verificamos se a raiz quadrada é um inteiro, testando se é igual a ela mesma quando foi arredondada para o inteiro mais próximo é o que <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math/floor">Math.floor()</a> faz com o número que é passado).</li>
 <li>Se a raiz quadrada e a raiz quadrada arredondada não forem iguais (<code>! ==</code>), isso significa que a raiz quadrada não é um número inteiro, portanto, não estamos interessados nela. Nesse caso, usamos a instrução <code>continue</code> para pular para a próxima iteração de loop sem registrar o número em nenhum lugar.</li>
 <li>Se a raiz quadrada é um inteiro, nós pulamos o bloco if inteiramente para que a instrução <code>continue</code> não seja executada; em vez disso, concatenamos o valor <code>i</code> atual mais um espaço até o final do conteúdo do parágrafo.</li>
</ol>

<div class="note">
<p><strong>Note</strong>: Você pode encontrar este <a href="https://github.com/mdn/learning-area/blob/master/javascript/building-blocks/loops/integer-squares.html">código de exemplo no GitHub</a> (também <a href="https://mdn.github.io/learning-area/javascript/building-blocks/loops/integer-squares.html">veja em execução</a> ).</p>
</div>

<h2 id="while_e_o_do_..._while">while e o do ... while</h2>

<p><code>for</code> não é o único tipo de loop disponível em JavaScript. Na verdade, existem muitos outros, embora você não precise entender tudo isso agora, vale a pena dar uma olhada na estrutura dos outros tipos de loops para que você possa reconhecer a mesma lógica na funcionalidade porém empregada de uma forma diferente.</p>

<p>Primeiro, vamos dar uma olhada no <a href="/en-US/docs/Web/JavaScript/Reference/Statements/while">while</a> loop. A sintaxe deste loop é assim:</p>

<pre>inicializador
while (condição-saída) {
  // code to run

  expressão-final
}</pre>

<p>While funciona de maneira muito semelhante ao loop for, exceto que a variável inicializadora é definida antes do loop, e a expressão final é incluída dentro do loop após o código a ser executado - em vez de esses dois itens serem incluídos dentro dos parênteses. A condição de saída está incluída dentro dos parênteses, que são precedidos pela palavra-chave <code>while</code> e não por <code>for</code>.</p>

<p>Os mesmos três itens ainda estão presentes, e eles ainda são definidos na mesma ordem do loop for - isso faz sentido, já que você ainda precisa ter um inicializador definido antes de poder verificar se ele atingiu a condição de saída ; a condição final é então executada após o código dentro do loop ser executado (uma iteração foi concluída), o que só acontecerá se a condição de saída ainda não tiver sido atingida.</p>

<p>Vamos dar uma olhada novamente no nosso exemplo de lista de gatos, que reescrevemos para usar um loop while:</p>

<pre class="brush: js">var i = 0;

while (i &lt; cats.length) {
  if (i === cats.length - 1) {
    info += 'and ' + cats[i] + '.';
  } else {
    info += cats[i] + ', ';
  }

  i++;
}</pre>

<div class="note">
<p><strong>Nota: </strong>Isso ainda funciona da mesma forma esperada — dê uma olhada no <a href="http://mdn.github.io/learning-area/javascript/building-blocks/loops/while.html">código em execução</a>  (também veja o <a href="https://github.com/mdn/learning-area/blob/master/javascript/building-blocks/loops/while.html">código fonte completo</a>).</p>
</div>

<p>O <a href="/en-US/docs/Web/JavaScript/Reference/Statements/do...while">do...while</a> loop é muito semelhante, mas fornece uma variação na estrutura while:</p>

<pre>initializer
do {
  // code to run

  final-expression
} while (exit-condition)</pre>

<p>Nesse caso, o inicializador novamente vem em primeiro lugar, antes que o loop seja iniciado. A palavra-chave precede diretamente as chaves que contêm o código a ser executado e a expressão final.</p>

<p>O diferenciador aqui é que a condição de saída vem depois de todo o resto, envolvida em parênteses e precedida por uma palavra-chave while. Em um loop do ... while, o código dentro das chaves é sempre executado uma vez antes da verificação ser feita para ver se deve ser executada novamente (no while e para, a verificação vem primeiro, então o código pode nunca ser executado ).</p>

<p>Vamos reescrever nosso exemplo de listagem de gato novamente para usar um loop do ... while:</p>

<pre class="brush: js">var i = 0;

do {
  if (i === cats.length - 1) {
    info += 'and ' + cats[i] + '.';
  } else {
    info += cats[i] + ', ';
  }

  i++;
} while (i &lt; cats.length);</pre>

<div class="note">
<p><strong>Nota: </strong>Novamente, isso funciona exatamente como esperado - dê uma olhada nele <a href="https://mdn.github.io/learning-area/javascript/building-blocks/loops/do-while.html">executando no GitHub</a> (veja também o<a href="https://github.com/mdn/learning-area/blob/master/javascript/building-blocks/loops/do-while.html"> código fonte completo</a>).</p>
</div>

<div class="warning">
<p><strong>Importante:</strong> Com while e do ... while - como em todos os loops - você deve certificar-se de que o inicializador esteja iterado para que ele atinja a condição de saída. Caso contrário, o loop continuará indefinidamente e o navegador irá forçá-lo a parar ou falhará. Isso é chamado de loop infinito.</p>
</div>

<h2 id="Aprendizado_ativo_contagem_regressiva!">Aprendizado ativo: contagem regressiva!</h2>

<p>Nesse exercício, nós queremos que você imprima uma simples contagem regressiva na caixa de saída, de 10 até terminar. Especificamente, queremos que você:</p>

<ul>
 <li>Faça um loop de 10 até 0. Fornecemos à você um inicializador  — <code>var i = 10;</code>.</li>
 <li>Para cada iteração, crie um novo parágrafo e o anexe à saída  <code>&lt;div&gt;</code>, que selecionamos usando <code>var output = document.querySelector('.output');</code>. Nos comentários, nós providenciamos a você três linhas de código que precisam ser usadas em algum lugar dentro do loop:
  <ul>
   <li><code>var para = document.createElement('p');</code> — cria um novo parágrafo.</li>
   <li><code>output.appendChild(para);</code> — anexa o parágrafo à saída <code>&lt;div&gt;</code>.</li>
   <li><code>para.textContent =</code> — faz o texto dentro do parágrafo ser igual ao que você coloca do lado direito, depois do sinal de igual.</li>
  </ul>
 </li>
 <li>Números de iteração diferentes exigem que texto diferente seja inserido no parágrafo para essa iteração (você precisará de uma declaração condicional e várias linhas <code>para.textContent =</code> ):
  <ul>
   <li>Se o número é 10, imprima "Contagem regressiva 10" no parágrafo.</li>
   <li>Se o número é 0, imprima "Lançar!" no parágrafo.</li>
   <li>Para qualquer outro número, apenas o imprima no parágrafo.</li>
  </ul>
 </li>
 <li>Lembre-se de incluir um iterador! No entanto, neste exemplo, estamos em contagem regressiva após cada iteração, e não progressiva, então você não vai querer usar <code>i++</code> — como você itera para baixo?</li>
</ul>

<p>Se você cometer um erro, sempre poderá redefinir o exemplo com o botão "Reset". Se você realmente ficar preso, pressione "Show solution" para ver uma solução.</p>

<div class="hidden">

<pre class="brush: html">&lt;h2&gt;Live output&lt;/h2&gt;
&lt;div class="output" style="height: 410px;overflow: auto;"&gt;

&lt;/div&gt;

&lt;h2&gt;Editable code&lt;/h2&gt;
&lt;p class="a11y-label"&gt;Press Esc to move focus away from the code area (Tab inserts a tab character).&lt;/p&gt;
&lt;textarea id="code" class="playable-code" style="height: 300px;width: 95%"&gt;
var output = document.querySelector('.output');
output.innerHTML = '';

// var i = 10;

// var para = document.createElement('p');
// para.textContent = ;
// output.appendChild(para);
&lt;/textarea&gt;

&lt;div class="playable-buttons"&gt;
  &lt;input id="reset" type="button" value="Reset"&gt;
  &lt;input id="solution" type="button" value="Show solution"&gt;
&lt;/div&gt;
</pre>


<pre class="brush: css">html {
  font-family: sans-serif;
}

h2 {
  font-size: 16px;
}

.a11y-label {
  margin: 0;
  text-align: right;
  font-size: 0.7rem;
  width: 98%;
}

body {
  margin: 10px;
  background: #f5f9fa;
}</pre>

<pre class="brush: js">var textarea = document.getElementById('code');
var reset = document.getElementById('reset');
var solution = document.getElementById('solution');
var code = textarea.value;
var userEntry = textarea.value;

function updateCode() {
  eval(textarea.value);
}

reset.addEventListener('click', function() {
  textarea.value = code;
  userEntry = textarea.value;
  solutionEntry = jsSolution;
  solution.value = 'Show solution';
  updateCode();
});

solution.addEventListener('click', function() {
  if(solution.value === 'Show solution') {
    textarea.value = solutionEntry;
    solution.value = 'Hide solution';
  } else {
    textarea.value = userEntry;
    solution.value = 'Show solution';
  }
  updateCode();
});

var jsSolution = 'var output = document.querySelector(\'.output\');\noutput.innerHTML = \'\';\n\nvar i = 10;\n\nwhile(i &gt;= 0) {\n var para = document.createElement(\'p\');\n if(i === 10) {\n para.textContent = \'Contagem regressiva \' + i;\n } else if(i === 0) {\n  para.textContent = \'Lançar!\';\n } else {\n para.textContent = i;\n }\n\n output.appendChild(para);\n\n i--;\n}';
var solutionEntry = jsSolution;

textarea.addEventListener('input', updateCode);
window.addEventListener('load', updateCode);

// stop tab key tabbing out of textarea and
// make it write a tab at the caret position instead

textarea.onkeydown = function(e){
  if (e.keyCode === 9) {
    e.preventDefault();
    insertAtCaret('\t');
  }

  if (e.keyCode === 27) {
    textarea.blur();
  }
};

function insertAtCaret(text) {
  var scrollPos = textarea.scrollTop;
  var caretPos = textarea.selectionStart;

  var front = (textarea.value).substring(0, caretPos);
  var back = (textarea.value).substring(textarea.selectionEnd, textarea.value.length);
  textarea.value = front + text + back;
  caretPos = caretPos + text.length;
  textarea.selectionStart = caretPos;
  textarea.selectionEnd = caretPos;
  textarea.focus();
  textarea.scrollTop = scrollPos;
}

// Update the saved userCode every time the user updates the text area code

textarea.onkeyup = function(){
  // We only want to save the state when the user code is being shown,
  // not the solution, so that solution is not saved over the user code
  if(solution.value === 'Show solution') {
    userEntry = textarea.value;
  } else {
    solutionEntry = textarea.value;
  }

  updateCode();
};</pre>

</div>

<p>{{ EmbedLiveSample('Active_learning', '100%', 880, "", "", "hide-codepen-jsfiddle") }}</p>

<h2 id="Aprendizado_ativo_preenchendo_uma_lista_de_convidados">Aprendizado ativo: preenchendo uma lista de convidados</h2>

<p>Nesse exercício, nós queremos que você pegue uma lista de nomes armazenados em um array, e os coloque em uma lista de convidados. Mas não tão fácil — nós não queremos que Phil e Lola estejam nela porque eles são gananciosos e rudes, e sempre comem a comida toda! Nós temos duas listas, uma para convidados aceitos, e uma para convidados recusados.</p>

<p>Especificamente, nós queremos que você:</p>

<ul>
 <li>Escreva um loop que iterará de 0 até o comprimento do array <code>people</code>. Você precisará começar com um inicializador de  <code>var i = 0;</code>, Mas qual condição de saída você precisa?</li>
 <li>Durante cada iteração de loop, verifique se o item atual do array é igual a "Phil" ou "Lola" usando uma declaração condicional:
  <ul>
   <li>Se for, concatene o item do array no final do <code>textContent</code> do paragrafo <code>refused</code>, seguido por uma vírgula e um espaço.</li>
   <li>Se não for, concatene o item do array no final do <code>textContent</code> do paragrafo <code>admitted</code>, seguido por uma vírgula e um espaço.</li>
  </ul>
 </li>
</ul>

<p>Nós já fornecemos a você:</p>

<ul>
 <li><code>var i = 0;</code> — Seu inicializador.</li>
 <li><code>refused.textContent +=</code> — o início de uma linha que concatenará algo no final de <code>refused.textContent</code>.</li>
 <li><code>admitted.textContent +=</code> — o início de uma linha que concatenará algo no final de <code>admitted.textContent</code>.</li>
</ul>

<p>Questão bônus extra  — depois de completar as tarefas acima com sucesso, você terá duas listas de nomes, separados por vírgulas, mas eles estarão desarrumados — haverá uma vírgula no final decada um. Você pode descobrir como escrever linhas que que cortam a última vírgula em cada caso, e adicionar um ponto final? Dê uma olhada no artigo <a href="/en-US/docs/Learn/JavaScript/First_steps/Useful_string_methods">Métodos úteis de string</a> para ajuda.</p>

<p>Se você cometer um erro, sempre poderá redefinir o exemplo com o botão "Reset". Se você realmente ficar preso, pressione "Show solution" para ver uma solução.</p>

<div class="hidden">
<h6 id="Active_learning_2">Active learning 2</h6>

<pre class="brush: html">&lt;h2&gt;Live output&lt;/h2&gt;
&lt;div class="output" style="height: 100px;overflow: auto;"&gt;
  &lt;p class="admitted"&gt;Admit: &lt;/p&gt;
  &lt;p class="refused"&gt;Refuse: &lt;/p&gt;
&lt;/div&gt;

&lt;h2&gt;Editable code&lt;/h2&gt;
&lt;p class="a11y-label"&gt;Press Esc to move focus away from the code area (Tab inserts a tab character).&lt;/p&gt;
&lt;textarea id="code" class="playable-code" style="height: 400px;width: 95%"&gt;
var people = ['Chris', 'Anne', 'Colin', 'Terri', 'Phil', 'Lola', 'Sam', 'Kay', 'Bruce'];

var admitted = document.querySelector('.admitted');
var refused = document.querySelector('.refused');
admitted.textContent = 'Admit: ';
refused.textContent = 'Refuse: '

// var i = 0;

// refused.textContent += ;
// admitted.textContent += ;

&lt;/textarea&gt;

&lt;div class="playable-buttons"&gt;
  &lt;input id="reset" type="button" value="Reset"&gt;
  &lt;input id="solution" type="button" value="Show solution"&gt;
&lt;/div&gt;
</pre>

<pre class="brush: css">html {
  font-family: sans-serif;
}

h2 {
  font-size: 16px;
}

.a11y-label {
  margin: 0;
  text-align: right;
  font-size: 0.7rem;
  width: 98%;
}

body {
  margin: 10px;
  background: #f5f9fa;
}</pre>

<pre class="brush: js">var textarea = document.getElementById('code');
var reset = document.getElementById('reset');
var solution = document.getElementById('solution');
var code = textarea.value;
var userEntry = textarea.value;

function updateCode() {
  eval(textarea.value);
}

reset.addEventListener('click', function() {
  textarea.value = code;
  userEntry = textarea.value;
  solutionEntry = jsSolution;
  solution.value = 'Show solution';
  updateCode();
});

solution.addEventListener('click', function() {
  if(solution.value === 'Show solution') {
    textarea.value = solutionEntry;
    solution.value = 'Hide solution';
  } else {
    textarea.value = userEntry;
    solution.value = 'Show solution';
  }
  updateCode();
});

var jsSolution = 'var people = [\'Chris\', \'Anne\', \'Colin\', \'Terri\', \'Phil\', \'Lola\', \'Sam\', \'Kay\', \'Bruce\'];\n\nvar admitted = document.querySelector(\'.admitted\');\nvar refused = document.querySelector(\'.refused\');\n\nadmitted.textContent = \'Admit: \';\nrefused.textContent = \'Refuse: \'\nvar i = 0;\n\ndo {\n if(people[i] === \'Phil\' || people[i] === \'Lola\') {\n refused.textContent += people[i] + \', \';\n } else {\n admitted.textContent += people[i] + \', \';\n }\n i++;\n} while(i &lt; people.length);\n\nrefused.textContent = refused.textContent.slice(0,refused.textContent.length-2) + \'.\';\nadmitted.textContent = admitted.textContent.slice(0,admitted.textContent.length-2) + \'.\';';
var solutionEntry = jsSolution;

textarea.addEventListener('input', updateCode);
window.addEventListener('load', updateCode);

// stop tab key tabbing out of textarea and
// make it write a tab at the caret position instead

textarea.onkeydown = function(e){
  if (e.keyCode === 9) {
    e.preventDefault();
    insertAtCaret('\t');
  }

  if (e.keyCode === 27) {
    textarea.blur();
  }
};

function insertAtCaret(text) {
  var scrollPos = textarea.scrollTop;
  var caretPos = textarea.selectionStart;

  var front = (textarea.value).substring(0, caretPos);
  var back = (textarea.value).substring(textarea.selectionEnd, textarea.value.length);
  textarea.value = front + text + back;
  caretPos = caretPos + text.length;
  textarea.selectionStart = caretPos;
  textarea.selectionEnd = caretPos;
  textarea.focus();
  textarea.scrollTop = scrollPos;
}

// Update the saved userCode every time the user updates the text area code

textarea.onkeyup = function(){
  // We only want to save the state when the user code is being shown,
  // not the solution, so that solution is not saved over the user code
  if(solution.value === 'Show solution') {
    userEntry = textarea.value;
  } else {
    solutionEntry = textarea.value;
  }

  updateCode();
};</pre>
</div>

<p>{{ EmbedLiveSample('Active_learning_2', '100%', 680, "", "", "hide-codepen-jsfiddle") }}</p>

<h2 id="Which_loop_type_should_you_use">Which loop type should you use?</h2>

<p>For basic uses, <code>for</code>, <code>while</code>, and <code>do...while</code> loops are largely interchangeable. They can all be used to solve the same problems, and which one you use will largely depend on your personal preference — which one you find easiest to remember or most intuitive. Let's have a look at them again.</p>

<p>First <code>for</code>:</p>

<pre>for (initializer; exit-condition; final-expression) {
  // code to run
}</pre>

<p><code>while</code>:</p>

<pre>initializer
while (exit-condition) {
  // code to run

  final-expression
}</pre>

<p>and finally <code>do...while</code>:</p>

<pre>initializer
do {
  // code to run

  final-expression
} while (exit-condition)</pre>

<p>Nós recomendamos o uso do <code>for</code>, pelo menos no começo, já que ele é provavelmente a forma mais fácil de lembrar de tudo  — o inicializador, a condição de saída, e a expressão final final tudo fica ordenadamente dentro dos parênteses, então é fácil de ver onde eles estão e para verifcar se você não os esqueceu.</p>

<div class="note">
<p><strong>Note</strong>: There are other loop types/features too, which are useful in advanced/specialized situations and beyond the scope of this article. If you want to go further with your loop learning, read our advanced <a href="/en-US/docs/Web/JavaScript/Guide/Loops_and_iteration">Loops and iteration guide</a>.</p>
</div>

<h2 id="Conclusion">Conclusion</h2>

<p>This article has revealed to you the basic concepts behind, and different options available when, looping code in JavaScript. You should now be clear on why loops are a good mechanism for dealing with repetitive code, and be raring to use them in your own examples!</p>

<p>If there is anything you didn't understand, feel free to read through the article again, or <a href="/en-US/Learn#Contact_us">contact us</a> to ask for help.</p>

<h2 id="See_also">See also</h2>

<ul>
 <li><a href="/en-US/docs/Web/JavaScript/Guide/Loops_and_iteration">Loops and iteration in detail</a></li>
 <li><a href="/en-US/docs/Web/JavaScript/Reference/Statements/for">for statement reference</a></li>
 <li><a href="/en-US/docs/Web/JavaScript/Reference/Statements/while">while</a> and <a href="/en-US/docs/Web/JavaScript/Reference/Statements/do...while">do...while</a> references</li>
 <li><a href="/en-US/docs/Web/JavaScript/Reference/Statements/break">break</a> and <a href="/en-US/docs/Web/JavaScript/Reference/Statements/continue">continue</a> references</li>
 <li><a href="https://www.impressivewebs.com/javascript-for-loop/">What’s the Best Way to Write a JavaScript For Loop?</a> — some advanced loop best practices</li>
</ul>

<p>{{PreviousMenuNext("Learn/JavaScript/Building_blocks/conditionals","Learn/JavaScript/Building_blocks/Functions", "Learn/JavaScript/Building_blocks")}}</p>

<h2 id="In_this_module">In this module</h2>

<ul>
 <li><a href="/en-US/docs/Learn/JavaScript/Building_blocks/conditionals">Making decisions in your code — conditionals</a></li>
 <li><a href="/en-US/docs/Learn/JavaScript/Building_blocks/Looping_code">Looping code</a></li>
 <li><a href="/en-US/docs/Learn/JavaScript/Building_blocks/Functions">Functions — reusable blocks of code</a></li>
 <li><a href="/en-US/docs/Learn/JavaScript/Building_blocks/Build_your_own_function">Build your own function</a></li>
 <li><a href="/en-US/docs/Learn/JavaScript/Building_blocks/Return_values">Function return values</a></li>
 <li><a href="/en-US/docs/Learn/JavaScript/Building_blocks/Events">Introduction to events</a></li>
 <li><a href="/en-US/docs/Learn/JavaScript/Building_blocks/Image_gallery">Image gallery</a></li>
</ul>