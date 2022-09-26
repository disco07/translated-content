---
title: 'Tutorial Django Parte 3: Usando models'
slug: Learn/Server-side/Django/Models
translation_of: Learn/Server-side/Django/Models
---
<div>{{LearnSidebar}}</div>

<div>{{PreviousMenuNext("Learn/Server-side/Django/skeleton_website", "Learn/Server-side/Django/Admin_site", "Learn/Server-side/Django")}}</div>

<p class="summary">Este artigo mostra como definir os modelos para o website <a href="https://developer.mozilla.org/en-US/docs/Learn/Server-side/Django/Tutorial_local_library_website">LocalLibrary</a>. Ele explica o que é um modelo, como ele é declarado e mostra algum dos principais tipos de campo. Ele também mostra brevemente algumas das principais formas pelas quais você pode acessar dados do modelo.</p>

<table class="learn-box standard-table">
 <tbody>
  <tr>
   <th scope="row">Pré-requisitos:</th>
   <td><a href="/en-US/docs/Learn/Server-side/Django/skeleton_website">Django Tutorial Part 2: Criar um esqueleto de um site</a>.</td>
  </tr>
  <tr>
   <th scope="row">Objetivo:</th>
   <td>Ser capaz de projetar e criar seus próprios modelos, escolhendo os campos de forma apropriada.</td>
  </tr>
 </tbody>
</table>

<h2 id="Visão_Geral">Visão Geral</h2>

<p>Aplicativos Django acessam e gerenciam dados através de objetos Python chamados de modelos (models). Modelos definem a <em>estrutura</em> dos dados armazenados, incluindo os tipos de campos e possivelmente também o seu tamanho máximo, valores default, opções de listas de seleção, texto de ajuda para documentação, texto de labels para formulários, etc. A definição do modelo é independente do banco de dados - você pode escolher um tipo de banco como parte das configurações do seu projeto. Uma vez que você tenha escolhido qual banco será utilizado você precisa conversar diretamente com ele - você somente escreve a estrutura do seu modelo e outros códigos, e o Django faz todo o trabalho sujo de comunicação com o banco para você.</p>

<p>Este tutorial mostra como definir e acessar os modelos para o website <a href="https://developer.mozilla.org/en-US/docs/Learn/Server-side/Django/Tutorial_local_library_website">LocalLibrary website</a>.</p>

<h2 id="Projetando_os_modelos_para_o_website_LocalLibrary">Projetando os modelos para o website <a href="https://developer.mozilla.org/en-US/docs/Learn/Server-side/Django/Tutorial_local_library_website">LocalLibrary</a></h2>

<p>Antes de começarmos a criar nossos modelos, vale a pena perder alguns minutos pensando sobre os dados que iremos guardar e as relações entre os diferentes modelos que serão criados.</p>

<p>Sabemos que precisamos armazenar informação sobre os livros (título, resumo, autor, idioma, gênero, ISBN) e também que existem várias cópias do mesmo livro na biblioteca (com um id único, status de disponibilidade, etc.). Talvez queiramos armazenar mais informações sobre o autor além de somente seu nome, até porque existem vários autores com o mesmo nome, ou com nomes parecidos. Queremos ordernar a busca dos livros por título, autor, idioma e gênero.</p>

<p>Quando estamos projetando nossos modelos, faz sentido criar modelos separados para cada "objeto". Em nosso caso de estudo, os "objetos" são os livros (a informação de cada livro, não a cópia em si), as cópias dos livros (um livro pode ter mais de uma cópia) e os autores.</p>

<p>Você pode também utilizar modelos para representar opções em uma lista de seleção (por exemplo numa lista suspensa), o que é melhor do que trabalhar com opções predefinidas — isso é recomendado quando nem todas as opções são conhecidas ou podem mudar de acordo com um filtro. Obviamente, para nosso tutorial, modelos candidatos para esse caso são o gênero e o idioma.</p>

<p>Após decidirmos nossos modelos e campos, precisamos pensar no relacionamento dessas informações. Django permite que você defina relações que são um pra um (<code>OneToOneField</code>), um pra muitos (<code>ForeignKey</code>) e muitos pra muitos (<code>ManyToManyField</code>).</p>

<p>Com isso em mente, os diagramas UML de associação, mostram abaixo os modelos que definiremos nesse caso (como caixas).</p>

<p><img alt="LocalLibrary Model UML" src="https://mdn.mozillademos.org/files/15646/local_library_model_uml.png" style="height: 660px; width: 977px;"></p>

<p><span lang="pt">Como acima, criamos modelos para <code>Book </code>(que contém os detalhes genéricos do livro),<br>
 <code>BookInstance</code> (contém os status das cópias físicas e específicas dos livros disponíveis no sistema) e <code>Author.</code> Também decidimos ter um modelo para o gênero (<code>Genre</code>), para que os valores possam ser criados/selecionados através da interface administrativa. Decidimos não ter um modelo para o <code>BookInstance: status</code> - pois, codificamos os valores em (<code>LOAN_STATUS</code>) porque não esperamos que isso mude. Dentro de cada uma das caixas você pode ver o nome do modelo, os campos nomes e tipos e também os métodos e seus tipos de retorno.</span></p>

<p>O diagrama também mostra as relações entre os modelos, incluindo suas multiplicidades. As multiplicidades são os números no diagrama que mostram as quantidades (máxima e mínima) que cada modelo pode estar presente nos relacionamentos. Por exemplo, a linha que conecta as caixas mostra que <code>Book</code> e um <code>Genre</code> estão relacionados. Os números próximos ao modelo <code>Genre</code> mostram que um livro deve ter um ou mais gêneros (ou quantos você quiser), enquanto os números do outro lado da linha, ao lado do modelo <code>Book</code> mostram que um gênero pode ter zero ou muitos livros associados.</p>

<div class="note">
<p><strong>Nota</strong>: A próxima seção fornece uma explicação básica sobre como os modelos são definidos e usados. Ao ler sobre isso, considere como vamos construir cada um dos modelos conforme o diagrama acima.</p>
</div>

<h2 id="Model_primer">Model primer</h2>

<p>Esta seção fornece uma breve visão sobre como um modelo é definido e alguns dos mais importantes campos e argumentos dos campos.</p>

<h3 id="Definição_do_Modelo">Definição do Modelo</h3>

<p>Modelos são geralmente definidos no arquivo <strong>models.py</strong> em uma aplicação. Eles são implementados como subclasse de <code>django.db.models.Model</code>, e podem incluir campos, métodos e metadados. O fragmento de código abaixo, mostra um modelo "típico", chamado <code>MyModelName</code>:</p>

<pre class="notranslate">from django.db import models

class MyModelName(models.Model):
    """Uma típica classe definindo um modelo, derivada da classe Model."""

    # Campos
    my_field_name = models.CharField(max_length=20, help_text='Enter field documentation')
    ...

    # Metadados
    class Meta:
        ordering = ['-my_field_name']

    # Métodos
    def get_absolute_url(self):
        """Retorna a url para acessar uma instancia específica de MyModelName."""
        return reverse('model-detail-view', args=[str(self.id)])

    def __str__(self):
        """ String para representar o objeto MyModelName (no site Admin)."""
        return self.my_field_name</pre>

<p>Nas seções abaixa, exploraremos detalhadamente cada um dos recursos dentro do modelo:</p>

<h4 id="Campos_Fields">Campos (Fields)</h4>

<p>Um modelo pode ter um número árbitrário de campos, de qualquer tipo -- cada um representa uma coluna de dados que queremos armazenar em uma de nossas tabelas de banco de dados. Cada registro do banco de dados (row - linha) consitirá em um valor de cada campo. Vamos ver o exemplo visto acima:</p>

<pre class="brush: js notranslate">my_field_name = models.CharField(max_length=20, help_text='Enter field documentation')</pre>

<div>Nosso exemplo acima tem um único campo chamado <code>my_field_name</code>, do tipo <code>models.CharField</code> - o que significa que este campo conterá strings de caracteres alfanuméricos. Os tipos de cada campo são atribuídos usando classes específicas, que determinam o tipo de registro usado para armazenar os dados no banco de dados, juntamente com os critérios de validação a serem usados ​​quando os valores são recebidos de um formulário HTML (ou seja, o que constitui um valor válido). Os tipos de cada campo também podem receber argumentos que especifiquem como o campo é armazenado ou pode ser usado. Neste caso, estamos dando ao nosso campo dois argumentos:</div>

<ul>
 <li><code>max_length=20</code> — Afima que o valor máximo do comprimento desse campo é de 20 caracteres.</li>
 <li><code>help_text='Enter field documentation'</code> — fornece um rótulo de texto para exibir uma ajuda para os usuários saberem qual valor fornecer, quando esse valor é inserido por um usuário por meio de um formulário HTML.</li>
</ul>

<p>O nome do campo é usado para se referir a ele em consultas e modelos. Os campos também têm um rótulo, que é especificado como um argumento <code>(verbose_name)</code> ou inferido ao capitalizar a primeira letra do nome da variável do campo e substituindo quaisquer sublinhados por um espaço (por exemplo,<code> my_field_name</code> teria um rótulo padrão de <code>My field name</code>).</p>

<p>A ordem em que os campos são declarados afetará sua ordem padrão, se um modelo for representado em um formulário (por exemplo, no site Admin), embora isso possa ser substituído.</p>

<h5 id="Argumentos_comuns_de_um_campo">Argumentos comuns de um campo</h5>

<p>Os seguintes argumentos são comuns e podem ser usados quando declaramos muitos ou a maioria dos diferentes tipos de campos:</p>

<ul>
 <li><a href="https://docs.djangoproject.com/en/2.1/ref/models/fields/#help-text">help_text</a>: Fornece um rótulo de texto para formulários HTML (por exemplo, no site admin), conforme descrito acima.</li>
 <li><a href="https://docs.djangoproject.com/en/2.1/ref/models/fields/#verbose-name">verbose_name</a>: Um nome legível para o campo usado nos rótulos de campo. Se não for especificado, o Django irá inferir o nome detalhado do campo <code>name</code>.</li>
 <li><a href="https://docs.djangoproject.com/en/2.1/ref/models/fields/#default">default</a>: O valor padrão para o campo. Isso pode ser um valor ou um objeto que pode ser chamado. Cada vez que o objeto for chamado será criado um novo registro.</li>
 <li><a href="https://docs.djangoproject.com/en/2.1/ref/models/fields/#null">null</a>: Se for <code>True</code>, o Django armazenará valores em branco como <code>NULL</code> no banco de dados, para campos onde isso é apropriado (um <code>CharField </code>irá armazenar uma string vazia). O padrão é <code>False</code>.</li>
 <li><a href="https://docs.djangoproject.com/en/2.1/ref/models/fields/#blank">blank</a>:Se for <code>True</code>, o campo poderá ficar em branco nos seus formulários. O padrão é <code>False</code>, o que significa que a validação de formulário do Django forçará você a inserir um valor. Isso é frequentemente usado com <code>null = True</code>, porque se você permitir valores em branco, também desejará que o banco de dados possa representá-los adequadamente.</li>
 <li><a href="https://docs.djangoproject.com/en/2.1/ref/models/fields/#choices">choices</a>: Um grupo de escolhas para este campo. Se isso for fornecido, o padrão widget de formulário correspondente será uma caixa de seleção com essas opções, em vez do campo de texto padrão.</li>
 <li><a href="https://docs.djangoproject.com/en/2.1/ref/models/fields/#primary-key">primary_key</a>:Se <code>True,</code> define o campo atual como a chave primária do modelo (uma chave primária é uma coluna especial do banco de dados, designada para identificar exclusivamente as diferentes tabelas) . Se nenhum campo for especificado como a chave primária, o Django adicionará automaticamente um campo para essa finalidade.</li>
</ul>

<p>Existem muitas outras opções - você pode ver <a href="https://docs.djangoproject.com/en/2.1/ref/models/fields/#field-options">a lista completa de opções aqui</a>.</p>

<h5 id="Tipos_comuns_de_um_campo">Tipos comuns de um campo</h5>

<p>A lista a seguir descreve alguns dos tipos de campos mais usados.</p>

<ul>
 <li><a href="https://docs.djangoproject.com/en/2.1/ref/models/fields/#django.db.models.CharField">CharField</a> é usado para definir um tamanho fixo (médio a curto) para a string. Você deve especificar o <code>max_length (tamanho máximo) para o dado que será armazenado.</code></li>
 <li><a href="https://docs.djangoproject.com/en/2.1/ref/models/fields/#django.db.models.TextField">TextField</a> é usado para grandes strings de comprimento variado. Você pode especificar um <code>max_length</code> (tamanho máximo) para o campo, mas isso é usado somente quando o campo é exibido em formulários (forms) (ele não é imposto no nível do banco de dados).</li>
 <li><a href="https://docs.djangoproject.com/en/2.1/ref/models/fields/#django.db.models.IntegerField" title="django.db.models.IntegerField">IntegerField</a> é um campo para armazenar números inteiros e para validar valores inseridos como números inteiros em formulários.</li>
 <li><a href="https://docs.djangoproject.com/en/2.1/ref/models/fields/#datefield">DateField</a> e <a href="https://docs.djangoproject.com/en/2.1/ref/models/fields/#datetimefield">DateTimeField</a> são usados para armazenar / representar datas e informações de data / hora (como os objetos Python <code>datetime.date</code> in e <code>datetime.datetime</code>, respectivamente). Esses campos também podem declarar os parâmetros (mutuamente exclusivos) <code>auto_now = True</code> (para definir o campo para a data atual toda vez que o modelo é salvo), <code>auto_now_add</code> (para definir a data em que o primeiro modelo foi criado) e <code>default</code> (para definir uma data padrão que pode ser substituída pelo usuário).</li>
 <li><a href="https://docs.djangoproject.com/en/2.1/ref/models/fields/#emailfield">EmailField</a> é usada para armazenara e validar em endereço de email.</li>
 <li><a href="https://docs.djangoproject.com/en/2.1/ref/models/fields/#filefield">FileField</a> e <a href="https://docs.djangoproject.com/en/2.1/ref/models/fields/#imagefield">ImageField</a> são usados para carregar arquivos e imagens respectivamente, (o <code>ImageField</code> simplesmente valida de forma adicional que o arquivo enviado é uma imagem). Eles têm parâmetros para definir como e onde os arquivos enviados são armazenados.</li>
 <li><a href="https://docs.djangoproject.com/en/2.1/ref/models/fields/#autofield">AutoField</a> é um tipo especial de <code>IntegerField</code> que é incrementada automaticamente. Uma chave primária desse tipo é adicionada de forma automatica ao seu modelo, se você não especificar explicitamente um.</li>
 <li><a href="https://docs.djangoproject.com/en/2.1/ref/models/fields/#foreignkey">ForeignKey</a> é usado para especificar um relacionamento um-para-muitos com outro modelo do banco de dados (por exemplo, um carro tem um fabricante, mas um fabricante pode fazer muitos carros). O lado "um" do relacionamento é o modelo que contém a "chave" (os modelos que contêm uma "chave estrangeira" referem-se a essa "chave" e estão no lado "muitos" de tal relacionamento).</li>
 <li><a href="https://docs.djangoproject.com/en/2.1/ref/models/fields/#manytomanyfield">ManyToManyField</a> é usado para especificar um relacionamento muitos-para-muitos (por exemplo, um livro pode ter vários gêneros e cada gênero pode conter vários livros). Em nosso aplicativo de biblioteca, usaremos isso de maneira muito semelhante às <code>ForeignKeys</code>, mas elas podem ser usadas de maneiras mais complicadas para descrever as relações entre os grupos. Eles têm o parâmetro <code>on_delete</code> para definir o que acontece quando o registro associado é excluído (por exemplo, um valor de <code>models.SET_NULL</code> simplesmente definiria o valor como <code>NULL</code>).</li>
</ul>

<p>Existem muitos outros tipos de campos, incluindo campos para diferentes tipos de números (big integers, small integers, floats), booleanos, URLs, slugs, unique ids e outras informações "relacionadas ao tempo" (duração, tempo, etc.) . Você pode ver a <a href="https://docs.djangoproject.com/en/2.1/ref/models/fields/#field-types">lista completa AQUI</a>.</p>

<h4 id="Metadados_metadada">Metadados (metadada)</h4>

<p>Você pode declarar o nível de modelo para os metadados declarando <code>class Meta</code>, como mostrado.</p>

<pre class="brush: python notranslate">class Meta:
    ordering = ['-my_field_name']
</pre>

<p>Um dos recursos mais úteis desses metadados é controlar a ordem padrão dos registros retornados, quando você consulta o tipo de modelo. Você faz isso especificando a ordem de correspondência em uma lista de nomes para ordenar (<code>ordering</code>) o atributo , conforme mostrado acima. A ordem dependerá do tipo de campo (os campos de caractere são classificados em ordem alfabética, enquanto os campos de data são classificados em ordem cronológica). Como mostrado acima, você pode prefixar o nome do campo com um símbolo de menos (-) para inverter a ordem de classificação.</p>

<p>Então, como exemplo, se optássemos por ordenar livros como este por padrão:</p>

<pre class="brush: python notranslate">ordering = ['title', '-pubdate']</pre>

<p>Os livros seriam classificados em ordem alfabética por título, de A-Z e depois por data de publicação dentro de cada título, do mais recente ao mais antigo.</p>

<p>Outro atributo comum é <code>verbose_name</code>, um nome detalhado para a classe no singular e plural:</p>

<pre class="brush: python notranslate">verbose_name = 'BetterName'</pre>

<p>Outros atributos úteis permitem que você crie e aplique novas "permissões de acesso" para o modelo (as permissões padrão são aplicadas automaticamente), permitem a ordenação com base em outro campo ou declarar que a classe é "abstrata" (uma classe base que você não pode criar registros, e em vez disso, serão derivadas para criar outros modelos).</p>

<p>Muitas das outras opções de metadados controlam qual banco de dados deve ser usado para o modelo e como os dados são armazenados (eles são realmente úteis somente se você precisar mapear um modelo para um banco de dados existente).</p>

<p>A lista completa de opções de metadados pode ser encontrada aqui: <a href="https://docs.djangoproject.com/en/2.1/ref/models/options/">Opções de modelos de metadados</a> (Django docs).</p>

<h4 id="Métodos">Métodos</h4>

<p>Um modelo também pode ter métodos.</p>

<p><strong>Minimamente, em cada modelo você deve definir o método de classe padrão do Python <code>__str__()</code> para retornar uma string legível para cada objeto.</strong> Essa sequência é usada para representar registros individuais no site de administração (e em qualquer outro lugar que você precise se referir a uma instância de modelo). Muitas vezes isso retornará um campo de título ou nome do modelo.</p>

<pre class="brush: python notranslate">def __str__(self):
    return self.field_name</pre>

<p>Outro método comum a incluir nos modelos do Django é o <code>get_absolute_url()</code>, que retorna uma URL para exibir registros de modelos individuais no site (se você definir esse método, o Django adicionará automaticamente um botão "View on Site" às ​​telas de edição de registros do modelo o site Admin). Um padrão típico para <code>get_absolute_url ()</code> é mostrado abaixo.</p>

<pre class="brush: python notranslate">def get_absolute_url(self):
    """Retorna o URL para acessar uma instância específica do modelo."""
    return reverse('model-detail-view', args=[str(self.id)])
</pre>

<div class="note">
<p><strong>Nota: </strong>Supondo que você usará URLs como <code>/ myapplication / mymodelname / 2</code> para exibir registros individuais para seu modelo (onde "2" é o <code>id</code> para um registro específico), você precisará criar um mapeador de URL para passar a resposta e id para uma "vista detalhada do modelo" (que fará o trabalho necessário para exibir o registro). A função <code>reverse ()</code> acima é capaz de "inverter" seu mapeador de url (no caso acima chamado 'model-detail-view') para criar uma URL do formato correto.</p>

<p>Claro que para fazer este trabalho você ainda tem que escrever o mapeamento de URL, visão e modelo!</p>
</div>

<p>Você também pode definir outros métodos que desejar e chamá-los de seu código ou modelos (desde que eles não utilizem nenhum parâmetro).</p>

<h3 id="Gestão_de_modelos">Gestão de modelos</h3>

<p>Depois de definir suas classes de modelo, você pode usá-las para criar, atualizar ou excluir registros e executar consultas para obter todos os registros ou subconjuntos específicos de registros. Mostraremos como fazer isso no tutorial quando definirmos nossas visualizações, mas aqui está um breve resumo.</p>

<h4 id="Criando_e_modificando_registros">Criando e modificando registros</h4>

<p>Para criar um registro, você pode definir uma instância do modelo e, em seguida, chamar <code>save ()</code>.</p>

<pre class="brush: python notranslate"># Crie um novo registro usando o construtor do modelo.
record = MyModelName(my_field_name="Instance #1")

# Salve o objeto no banco de dados.
record.save()
</pre>

<div class="note">
<p><strong>Nota:</strong> Se você não tiver declarado qualquer campo como primary_key, o novo registro receberá um automaticamente, com o ID do nome do campo. Você poderia consultar este campo depois de salvar o registro acima e ele teria um valor de 1.</p>
</div>

<p>Você pode acessar os campos nesse novo registro usando a sintaxe de ponto e alterar os valores. Você precisa chamar <code>save ()</code> para armazenar valores modificados no banco de dados.</p>

<pre class="brush: python notranslate"># Acessar valores de campo de modelo usando atributos do Python.
print(record.id) # should return 1 for the first record.
print(record.my_field_name) # should print 'Instance #1'

# Altere o registro modificando os campos e, em seguida, chamando save ().
record.my_field_name = "New Instance Name"
record.save()</pre>

<h4 id="Procurando_por_registros">Procurando por registros</h4>

<p>Você pode procurar registros que correspondam a um determinado critério usando o atributo de objetos do modelo (fornecido pela classe base).</p>

<div class="note">
<p><strong>Nota: </strong>Explicar como procurar registros usando nomes de campos e modelos "abstratos" pode ser um pouco confuso. Na discussão abaixo, vamos nos referir a um modelo de livro com campos de título e gênero, onde o gênero também é um modelo com um único <code>nome</code> de campo.</p>
</div>

<p>Podemos obter todos os registros para um modelo como um <code>QuerySet</code>, usando<code> objects.all ()</code>. O <code>QuerySet</code> é um objeto iterável, o que significa que ele contém um número de objetos que podemos iterar / percorrer.</p>

<pre class="brush: python notranslate">all_books = Book.objects.all()
</pre>

<p>O método <code>filter ()</code> do Django nos permite filtrar o <code>QuerySet</code> retornado para corresponder a um campo especificado de <strong>texto</strong> ou <strong>numérico</strong> em relação a um critério específico. Por exemplo, para filtrar por livros que contenham "wild" no título e, em seguida, contá-los, poderíamos fazer o seguinte.</p>

<pre class="brush: python notranslate">wild_books = Book.objects.filter(title__contains='wild')
number_wild_books = Book.objects.filter(title__contains='wild').count()
</pre>

<p>Os campos a serem correspondidos e o tipo de correspondência são definidos no nome do parâmetro de filtro, usando o formato: field_name__match_type (observe o sublinhado duplo entre título e contém acima). Acima, estamos filtrando o título com uma correspondência de maiúsculas e minúsculas. Existem muitos outros tipos de correspondência que você pode fazer: icontains (insensitivo a maiúsculas e minúsculas), iexact (correspondência exata que não diferencia maiúsculas e minúsculas), exata (correspondência exata diferencia maiúsculas de minúsculas), gt (maior que), startswith, etc. é <a href="https://docs.djangoproject.com/en/2.1/ref/models/querysets/#field-lookups">aqui</a>.</p>

<p>Em alguns casos, você precisará filtrar um campo que defina um relacionamento um-para-muitos com outro modelo (por exemplo, uma <code>ForeignKey</code>). Nesse caso, você pode "indexar" campos no modelo relacionado com sublinhados duplos adicionais. Por exemplo, para filtrar livros com um padrão de gênero específico, você terá que indexar o <code>nome</code> por meio do campo de <code>gênero</code>, conforme mostrado abaixo:</p>

<pre class="brush: python notranslate"># Combinará em: ficção, ficção científica, não-ficção etc.
books_containing_genre = Book.objects.filter(genre<strong>__</strong>name<strong>__</strong>icontains='fiction')
</pre>

<div class="note">
<p><strong>Nota:</strong> Você pode usar sublinhados (__) para navegar quantos níveis de relacionamentos (<code>ForeignKey / ManyToManyField</code>) desejar. Por exemplo, um <code>Livro</code> que tinha tipos diferentes, definidos usando um relacionamento "cover" adicional, pode ter um nome de parâmetro: <code>type__cover__name__exact = 'hard'</code>.</p>
</div>

<p>Há muito mais que você pode fazer com as consultas, incluindo pesquisas para trás de modelos relacionados, encadeando filtros, retornando um conjunto menor de valores, etc. Para obter mais informações, consulte <a href="https://docs.djangoproject.com/en/2.1/topics/db/queries/">Fazendo consultas</a> (Django Docs).</p>

<h2 id="Definindo_os_modelos_LocalLibrary">Definindo os modelos LocalLibrary</h2>

<p>Nesta seção, começaremos a definir os modelos para a biblioteca. Abra models.py (em / locallibrary / catalog /). O código na parte superior da página importa o módulo de models, que contém os modelos da classe base do models. Modelo do qual nossos models herdarão.</p>

<pre class="brush: python notranslate">from django.db import models

# Create your models here.</pre>

<h3 id="Genre_model">Genre model</h3>

<p>Copie o código do model <code>Genre</code> , que aparece a baixo, e cole no seu arquivo <code>models.py</code> , abaixo do código de importação.  Esse model  guarda informaçoes sobre a categoria do livro — por exemplo se é ficção ou não ficção, romance ou história, etc. Como mencionado acima, criamos o genero como um model em vez de um campo de texto ou um lista de seleção para que os possíveis generos criados possam ser gerenciados pelo banco de dados, em vez de serem codificados.</p>

<pre class="syntaxbox notranslate">class Genre(models.Model):
    """Model representing a book genre."""
    name = models.CharField(max_length=200, help_text='Enter a book genre (e.g. Science Fiction)')

    def __str__(self):
        """String for representing the Model object."""
        return self.name</pre>

<p>O model tem apenas um campo do tipo <code>CharField</code> (<code>name</code>), que é usado para descrever o genero (esse campo é limitado a 200 caracteres e tem um <code>help_text</code>). No final da classe Genre declaramos o metodo  <code>__str__()</code> que retorna o nome do genero definido por um registro especifico. Não foi definido um argumento <code>verbose_name</code> , assim o campo será referenciado como  <code>Name</code> nos forms.</p>

<h3 id="Book_model">Book model</h3>

<p>Copie o model Book abaixo e cole novamente ao final do seu arquivo. O model Book representa todas as informações disponíveis sobre um livro de maneira geral, mas não incluí detalhes sobre o formato do encadernamento ou disponibilidade para empréstimo. O model usa um <code>CharField</code> para representar o  <code>title</code> (tílutlo) do livro e o <code>isbn</code> (veja que o <code>isbn</code> recebe um rótulo como "ISBN", usando o primeiro parâmetro não nomeado, porque, de outra forma, o rótulo ficaria "Isbn"). O model usa <code>TextField</code> para o campo <code>summary</code> porque ele precisa ser um tanto longo.</p>

<pre class="notranslate">from django.urls import reverse # Used to generate URLs by reversing the URL patterns

class Book(models.Model):
    """Model representing a book (but not a specific copy of a book)."""
    title = models.CharField(max_length=200)

    # Foreign Key used because book can only have one author, but authors can have multiple books
    # Author as a string rather than object because it hasn't been declared yet in the file
    author = models.ForeignKey('Author', on_delete=models.SET_NULL, null=True)

    summary = models.TextField(max_length=1000, help_text='Enter a brief description of the book')
    isbn = models.CharField('ISBN', max_length=13, help_text='13 Character &lt;a href="https://www.isbn-international.org/content/what-isbn"&gt;ISBN number&lt;/a&gt;')

    # ManyToManyField used because genre can contain many books. Books can cover many genres.
    # Genre class has already been defined so we can specify the object above.
    genre = models.ManyToManyField(Genre, help_text='Select a genre for this book')

    def __str__(self):
        """String for representing the Model object."""
        return self.title

    def get_absolute_url(self):
        """Returns the url to access a detail record for this book."""
        return reverse('book-detail', args=[str(self.id)])</pre>

<p>Genre é um campo <code>ManyToManyField</code>, de forma que um livro pode ter múltiplos gêneros e um gênero pode ter muitos livros. O campor Author é declarado como um <code>ForeignKey</code>, logo, cada livro terá somente um autor, mas um autor por der muitos livros (na prática, um livro pode ter múltiplos autores, mas não nesta implementação!)</p>

<p>Em ambos tipos de campo, a classe model relacionada é declarada como primeiro campo não nomeado usando a classe model ou contendo o nome do modelo relacionado. Você deve usar o nome do model como uma string se a classe associada ainda não tiver sido definida no arquivo, antes de referenciá-la. Outro parâmetro de interesse no campo <code>author</code> é <code>null=True</code>, o qual permite ao banco de dados armazer um valor <code>Null</code> se nenhum autor for selecionado, e, <code>on_delete=models.SET_NULL</code>, o qual definirá o valor <code>author</code> como <code>Null</code> se o registro do autor associado for excluído.</p>

<p>O mode também defini <code>__str__()</code>, usando o <code>title</code> do livro para representar o registro do <code>Book</code>. O método final, <code>get_absolute_url()</code>, retorna a URL que pode ser usada para acessar o registro detalhado deste modelo (para que isto funcione, teremos que definir um mapa de URL com o nome <code>book-detail</code> e associá-la a uma view e a um template.).</p>

<h3 id="BookInstance_model">BookInstance model</h3>

<p>Em seguida, copie o modelo <code>BookInstance</code> (exibido a abaixo) depois dos outros modelos. O <code>BookInstance</code> representa um exemplar específico do livro que pode ser emprestado, indica se ela está disponível ou a data programada de restituição, "imprint" ou detalhes da versão, e, um id único do livro na biblioteca.</p>

<p>Alguns dos campos e métodos serão familiares agora. O model usa</p>

<ul>
 <li><code>ForeignKey</code> para identificar o <code>Book</code> associado (cada livro pode ter muitos exemplares, porém uma cópia pode ter sometne um <code>Book</code>).</li>
 <li><code>CharField</code> para representar o imprint (edição específica) do livro.</li>
</ul>

<pre class="brush: python notranslate">import uuid # Required for unique book instances

class BookInstance(models.Model):
    """Model representing a specific copy of a book (i.e. that can be borrowed from the library)."""
    id = models.UUIDField(primary_key=True, default=uuid.uuid4, help_text='Unique ID for this particular book across whole library')
    book = models.ForeignKey('Book', on_delete=models.SET_NULL, null=True)
    imprint = models.CharField(max_length=200)
    due_back = models.DateField(null=True, blank=True)

    LOAN_STATUS = (
        ('m', 'Maintenance'),
        ('o', 'On loan'),
        ('a', 'Available'),
        ('r', 'Reserved'),
    )

    status = models.CharField(
        max_length=1,
        choices=LOAN_STATUS,
        blank=True,
        default='m',
        help_text='Book availability',
    )

    class Meta:
        ordering = ['due_back']

    def __str__(self):
        """String for representing the Model object."""
        return f'{self.id} ({self.book.title})'</pre>

<p>Nós declaramos adicionalmente alguns tipos novos de campos:</p>

<ul>
 <li><code>UUIDField</code> é usado no campo <code>id</code> para definí-lo como <code>primary_key</code> deste model. Este tipo de campo aloca um valor único global para cada instância (um para cada livro encontrado na biblioteca).</li>
 <li><code>DateField</code> é usado para a data <code>due_back</code> (na qual o livro deve retonar e ficar disponível após um empréstimo ou uma manutenção). O valor pode ser <code>blank</code> ou <code>null</code> (para quando o livro estiver disponível). O model metadata (<code>Class Meta</code>) usa este campo para ordenar os registros quando eles são retornados em uma query.</li>
 <li><code>status</code> é um <code>CharField</code> que define uma lista de opção/seleção. Como pode ser visto, nós definimos uma dupla contendo pares de valores-chave e a passamos no argumento da opção. O valor do par valor-chave é um valor que o usuário pode selecionar, enquanto que a chave é o valor que  realmetne será salvo se a opção for selecionada. Nós também definimos um valor default como "m" (maintenance), uma vez que os livros serão inicialmente criados como indisponíveis até que eles sejam disponibilizados nas prateleiras.</li>
</ul>

<p>O model <code>__str__()</code> representa o objeto <code>BookInstance</code> usando a combinação do seu id único e o título do <code>Book</code> associado.</p>

<div class="note">
<p><strong>Nota</strong>: Um pouco sobre Python:</p>

<ul>
 <li>Começando com Python 3.6, pode ser usada a sintaxe de interpolação  (também conhecida como f-strings): <code>f'{self.id} ({self.book.title})'</code>.</li>
 <li>Em versões anteriores deste tutorial, nós estávamos usando a sintaxe de  <a href="https://www.python.org/dev/peps/pep-3101/">formatted string</a>, a qual também é uma maneira válida de formatar  strings em Python (e.g. <code>'{0} ({1})'.format(self.id,self.book.title)</code>).</li>
</ul>
</div>

<h3 id="Author_model">Author model</h3>

<p>Copie o model <code>Author</code> (exibido abaixo) na sequência do código existente em  <strong>models.py</strong>.</p>

<p>Todos os campos/métodos já devem ser familiares agora. O model define um autor como tendo um nome, sobrenome, data de nascimento e data de morte (opcionalmente). Ele especifica que <code>__str__()</code> retorna por default sobrenome e nome, nesta ordem. O método <code>get_absolute_url()</code> reverte o mapaeamento da URL <code>author-detail</code> para obter a URL de display individual author.</p>

<pre class="brush: python notranslate">class Author(models.Model):
    """Model representing an author."""
    first_name = models.CharField(max_length=100)
    last_name = models.CharField(max_length=100)
    date_of_birth = models.DateField(null=True, blank=True)
    date_of_death = models.DateField('Died', null=True, blank=True)

    class Meta:
        ordering = ['last_name', 'first_name']

    def get_absolute_url(self):
        """Returns the url to access a particular author instance."""
        return reverse('author-detail', args=[str(self.id)])

    def __str__(self):
        """String for representing the Model object."""
        return f'{self.last_name}, {self.first_name}'

</pre>

<h2 id="Rode_novamente_a_migração_do_banco_de_dados">Rode novamente a migração do banco de dados</h2>

<p>Todos os modelos foram criados. Agora rode a migração do banco de dados para adicioná-los ao seu banco de dados.</p>

<pre class="notranslate"><code>python3 manage.py makemigrations
python3 manage.py migrate</code></pre>

<h2 id="Language_model_—_desafio">Language model — desafio</h2>

<p>Considere que um benfeitor local doe uma quantidade de livros escritos em outro idioma (digamos, Farsi). O desafio é desenvolver como eles seriam melhor representados no website da nossa biblioteca, e, então, adicioná-lo no models.</p>

<p>Algumas coisas a considerar:</p>

<ul>
 <li>O idioma seria associado ao  <code>Book</code>, <code>BookInstance</code>, ou algum outro objeto?</li>
 <li>Os diferentes idiomas seriam representados usando model, como campo de texto livre ou codificado como uma lista de seleção?</li>
</ul>

<p>Depois de decidido, adicione o campo. Pode ser visto o que decidimos no Github <a href="https://github.com/mdn/django-locallibrary-tutorial/blob/master/catalog/models.py">aqui</a>.</p>

<ul>
</ul>

<ul>
</ul>

<h2 id="Resumo">Resumo</h2>

<p>Neste artículo vimos como os  models são definidos, e, então, usamos esta informação para desenhar e implementar modelos apropriados para o website  <em>LocalLibrary</em>.</p>

<p>Neste ponto faremos um rápido desvio da criação do site para ver o <em>Django Administration site</em>. Este site nos permitirá adicionar alguns dados à biblioteca, os quais podemos então exibir usando nossos próprios views e templates (ainda não criados).</p>

<h2 id="Veja_também">Veja também</h2>

<ul>
 <li><a href="https://docs.djangoproject.com/en/2.1/intro/tutorial02/">Escrevendo nossa primeira app Django, parte 2</a> (Django docs)</li>
 <li><a href="https://docs.djangoproject.com/en/2.1/topics/db/queries/">Fazendo queries</a> (Django Docs)</li>
 <li><a href="https://docs.djangoproject.com/en/2.1/ref/models/querysets/">Referência a QuerySet API</a> (Django Docs)</li>
</ul>

<p>{{PreviousMenuNext("Learn/Server-side/Django/skeleton_website", "Learn/Server-side/Django/Admin_site", "Learn/Server-side/Django")}}</p>

<h2 id="Neste_módulo">Neste módulo</h2>

<ul>
 <li><a href="/en-US/docs/Learn/Server-side/Django/Introduction">Introdução ao Django</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Django/development_environment">Configurando um ambiente de desenvolvimento Django</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Django/Tutorial_local_library_website">Tutorial Django: Website de uma biblioteca local</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Django/skeleton_website">Tutorial Django Parte 2: Criando o escopo do website</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Django/Deployment">Tutorial </a><a href="/en-US/docs/Learn/Server-side/Django/Models">Django Parte 3: Utilizando models</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Django/Deployment">Tutorial </a><a href="/en-US/docs/Learn/Server-side/Django/Admin_site">Django Parte 4: Django admin site</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Django/Deployment">Tutorial </a><a href="/en-US/docs/Learn/Server-side/Django/Home_page">Django Parte 5: Criando nossa página principal</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Django/Deployment">Tutorial </a><a href="/en-US/docs/Learn/Server-side/Django/Generic_views">Django Parte 6: Lista genérica e detail views</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Django/Deployment">Tutorial </a><a href="/en-US/docs/Learn/Server-side/Django/Sessions">Django Parte 7: Framework de Sessões</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Django/Deployment">Tutorial </a><a href="/en-US/docs/Learn/Server-side/Django/Authentication">Django Parte 8: Autenticação de Usuário e permissões</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Django/Deployment">Tutorial </a><a href="/en-US/docs/Learn/Server-side/Django/Forms">Django Parte 9: Trabalhando com formulários</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Django/Deployment">Tutorial </a><a href="/en-US/docs/Learn/Server-side/Django/Testing">Django Parte 10: Testando uma aplicação web Django</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Django/Deployment">Tutorial Django Parte 11: Implantando Django em produção</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Django/web_application_security">Segurança de aplicações Django</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Django/django_assessment_blog">DIY Django mini blog</a></li>
</ul>