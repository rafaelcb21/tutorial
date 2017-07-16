(https://medium.com/dartlang/zero-to-one-with-flutter-part-two-5aa2f06655cb)

Mikkel RavnFollow

Software Engineer at Google

# **Zero à Um com Flutter, parte Dois**

Descobrindo como animar objetos gráficos compostos no contexto de um aplicativo móvel multiplataforma. Junte-se a um novato de conceito ávido ao aprender como aplicar o conceito de interpolação a animação de valores estruturados, exemplificados por gráficos de barras. Amostras de código completo, baterias incluídas.

Como você entra em um novo campo de programação? A experimentação é obviamente a chave, como por exemplo estudando e emulando programas escritos por colegas mais experientes. Eu pessoalmente gosto de complementar essas abordagens com o conceito de mineração: tentando trabalhar com os primeiros princípios, identificando conceitos, explorando sua força, buscando deliberadamente suas orientações. É uma abordagem racionalista que não pode suportar por conta própria, mas que é intelectualmente estimulante, e pode levar você a uma visão mais profunda e mais rápida.

Esta é a segunda e última parte de uma introdução ao Flutter e seus conceitos de widgets e interpolação. No final da primeira parte, chegamos a uma árvore de widgets que contém, entre vários widgets de layout e gerenciamento de estado,

* um widget para pintar uma única `Bar` usando código customizado de desenho e animação,

* um widget de botão de ação flutuante para iniciar uma mudança animada da altura da barra.

A animação foi implementada usando `BarTween`, e afirmei que o conceito de interpolação seria escalado para lidar com situações mais complexas. Aqui na segunda parte, cumprirei essa reivindicação, generalizando o design para barras com mais propriedades e para barras de gráficos contendo múltiplas barras em várias configurações.

Comecemos pela adição de cor à nossa barra única. Nós adicionamos um campo de cores `color` ao lado do campo de altura `height` da classe de `Bar` e atualizamos `Bar.lerp` para interpolar os dois. Esse padrão é típico:

*Lerp entre valores compostos por componentes correspondentes.*

Lembre-se da primeira parte que "lerp" é uma forma curta de "interpolar linearmente" ou "interpolação linear".

```dart
import 'dart:ui' show lerpDouble;

import 'package:flutter/animation.dart';
import 'package:flutter/material.dart';

class Bar {
  Bar(this.height, this.color);

  final double height;
  final Color color;

  static Bar lerp(Bar begin, Bar end, double t) {
    return new Bar(
      lerpDouble(begin.height, end.height, t),
      Color.lerp(begin.color, end.color, t),
    );
  }
}

class BarTween extends Tween<Bar> {
  BarTween(Bar begin, Bar end) : super(begin: begin, end: end);

  @override
  Bar lerp(double t) => Bar.lerp(begin, end, t);
}
```

Observe a utilidade do método estático `lerp` na linguagem aqui. Sem `Bar.lerp`, `lerpDouble` (moralmente `double.lerp`) e `Color.lerp`, teríamos que implementar o `BarTween` criando um `Tween<double>` para a altura e um `Tween<Color>` para a cor. Essas tweens seriam campos de instância do `BarTween`, inicializados pelo seu construtor e usados em seu método `lerp`. Estaríamos duplicando o conhecimento sobre as propriedades do `Bar` várias vezes, fora da classe do `Bar`. Os responsáveis pelo nosso código provavelmente acharão isso não muito ideal.

Para usar barras coloridas em nosso aplicativo, atualizaremos o `BarChartPainter` para obter a cor da barra do `Bar`. No `main.dart`, precisamos criar um `Bar` vazio e uma `Bar` aleatória. Usaremos uma cor totalmente transparente para o primeiro, e uma cor aleatória para o último. As cores serão tiradas de um `ColorPalette` simples que nós introduzimos rapidamente em um arquivo próprio. Faremos uma fábrica de construtores em `Bar.empty` e `Bar.random` no `Bar` (listagem de códigos).

Os gráficos de barras envolvem barras múltiplas em várias configurações. Para introduzir uma complexidade lentamente, nossa primeira implementação será adequada para gráficos de barras que exibam quantidades numéricas para um conjunto fixo de categorias. Exemplos incluem visitantes por dia da semana ou vendas por trimestre. Para esses gráficos, alterar o conjunto de dados para outra semana ou outro ano não altera as categorias usadas, apenas a barra mostrada para cada categoria.

Atualizaremos primeiro o `main.dart` desta vez, substituindo `Bar` por `BarChart` e `BarTween` pela `BarChartTween` (listagem de códigos).

Para tornar o analisador Dart feliz, criamos a classe `BarChart` em `bar.dart` e implementamos usando uma lista de tamanho fixa para a instância `Bar`. Usaremos cinco barras, uma para cada dia da semana de trabalho. Em seguida, precisamos mover a responsabilidade de criar instâncias vazias e aleatórias de `Bar` para `BarChart`. Com categorias fixas, um gráfico de barras vazia é razoavelmente considerado uma coleção de barras vazias. Por outro lado, deixar um gráfico de barras aleatório ser uma coleção de barras aleatórias faria nossos quadros um pouco caleidoscópicos. Em vez disso, vamos escolher uma cor aleatória para o gráfico e deixar cada barra, ainda de altura aleatória, herdar isso.

```dart
import 'dart:math';
import 'dart:ui' show lerpDouble;

import 'package:flutter/animation.dart';
import 'package:flutter/material.dart';

import 'color_palette.dart';

class BarChart {
  static const int barCount = 5;

  BarChart(this.bars) {
    assert(bars.length == barCount);
  }

  factory BarChart.empty() {
    return new BarChart(new List.filled(
      barCount,
      new Bar(0.0, Colors.transparent),
    ));
  }

  factory BarChart.random(Random random) {
    final Color color = ColorPalette.primary.random(random);
    return new BarChart(new List.generate(
      barCount,
      (i) => new Bar(random.nextDouble() * 100.0, color),
    ));
  }

  final List<Bar> bars;

  static BarChart lerp(BarChart begin, BarChart end, double t) {
    return new BarChart(new List.generate(
      barCount,
      (i) => Bar.lerp(begin.bars[i], end.bars[i], t),
    ));
  }
}

class BarChartTween extends Tween<BarChart> {
  BarChartTween(BarChart begin, BarChart end) : super(begin: begin, end: end);

  @override
  BarChart lerp(double t) => BarChart.lerp(begin, end, t);
}

class Bar {
  Bar(this.height, this.color);

  final double height;
  final Color color;

  static Bar lerp(Bar begin, Bar end, double t) {
    return new Bar(
      lerpDouble(begin.height, end.height, t),
      Color.lerp(begin.color, end.color, t),
    );
  }
}

class BarTween extends Tween<Bar> {
  BarTween(Bar begin, Bar end) : super(begin: begin, end: end);

  @override
  Bar lerp(double t) => Bar.lerp(begin, end, t);
}

class BarChartPainter extends CustomPainter {
  static const barWidthFraction = 0.75;

  BarChartPainter(Animation<BarChart> animation)
      : animation = animation,
        super(repaint: animation);

  final Animation<BarChart> animation;

  @override
  void paint(Canvas canvas, Size size) {
    void drawBar(Bar bar, double x, double width, Paint paint) {
      paint.color = bar.color;
      canvas.drawRect(
        new Rect.fromLTWH(x, size.height - bar.height, width, bar.height),
        paint,
      );
    }

    final paint = new Paint()..style = PaintingStyle.fill;
    final chart = animation.value;
    final barDistance = size.width / (1 + chart.bars.length);
    final barWidth = barDistance * barWidthFraction;
    var x = barDistance - barWidth / 2;
    for (final bar in chart.bars) {
      drawBar(bar, x, barWidth, paint);
      x += barDistance;
    }
  }

  @override
  bool shouldRepaint(BarChartPainter old) => false;
}
```

O `BarChartPainter` distribui a largura disponível uniformemente entre as barras e faz com que cada barra ocupe 75% da largura disponível.

Observe como `BarChart.lerp` é implementado em termos de `Bar.lerp`, regenerando a estrutura da lista em tempo real. Os gráficos de barras de categoria fixa são valores compostos para os quais o componente lerping faz todo o sentido, precisamente como para barras únicas com múltiplas propriedades.

Há um padrão em jogo aqui. Quando o construtor de uma classe Dart pega múltiplos parâmetros, você pode sempre ler cada parâmetro separadamente e a combinação também ficará boa. E você pode aninhar esse padrão de forma arbitrária: os painéis serão lerped por lerping constituidos por seus gráficos de barras, nas quais suas barras serão lerped por lerping, sua altura e cor serão lerped por lerping. E  seus componentes RGB e alfaas cores serão lerped por lerping. Com folhas desta recursão, nós teremos números lerp.

A inclinação matemática pode expressar isso dizendo que lerping comuta com a estrutura no sentido de valores compostos `C(x, y)`, que nós temos.

`lerp(C(x1, y1), C(x2, y2), t) == C(lerp(x1, x2, t), lerp(y1, y2, t))`

Como nós vemos, essa generalização agradável desses dois componentes (altura e cor de uma barra) para arbitrarirmos em muitos componentes (n barras de uma categoria fixa do gráfico de barras).

Há, no entanto, situações em que essa imagem bonita se rompe. Podemos desejar animar entre dois valores que não são compostos da mesma maneira. Como um exemplo simples, considere animar a partir de um gráfico de barras com dados para os cinco dias da semana de trabalho para um gráfico que inclui também o fim de semana.

Você pode prontamente encontrar várias soluções ad-hoc diferentes para este problema, e então pode pedir ao seu designer UX que escolha entre elas. Essa é uma abordagem válida, embora eu acredite que seja útil ter em mente durante sua discussão a estrutura fundamental comum a essas diferentes soluções: a interpolação. Lembre-se da primeira parte:

*Animar `T`s rastreando um caminho no espaço de todos os `T`s, pois o valor da animação é executado de zero a um. Modelar o caminho com um `Tween<T>`.*

A questão central para responder com o designer UX é esta: Com intermediará os valores entre um gráfico com cinco barras e um com sete? Uma escolha óbvia é ter seis barras, mas precisamos de mais valores intermediários do que isso para animar suavemente. Precisamos desenhar barras de forma diferente, pisando fora do reino de barras de largura igual, uniformemente espaçadas, ajustadas a 200 pixels. Em outras palavras, o espaço dos valores de T devem ser generalizados.

*Lerp entre valores com estruturas diferentes incorporando-os em um espaço de valores mais gerais, abrangendo como casos especiais tanto pontos finais de animação quanto todos os valores intermediários necessários.*

Podemos fazer isso em duas etapas. Primeiro, generalizamos o `Bar` para incluir a sua coordenada x e a largura como atributos:

```dart
class Bar {
  Bar(this.x, this.width, this.height, this.color);

  final double x;
  final double width;
  final double height;
  final Color color;

  static Bar lerp(Bar begin, Bar end, double t) {
    return new Bar(
      lerpDouble(begin.x, end.x, t),
      lerpDouble(begin.width, end.width, t),
      lerpDouble(begin.height, end.height, t),
      Color.lerp(begin.color, end.color, t),
    );
  }
}
```
Em segundo lugar, fazemos gráficos de suporte `BarChart` com diferentes contagens de barras. Nossos novos gráficos serão adequados para o conjuntos de dados onde bar `i` representa o `i` valor em algumas séries, como as vendas no dia `i` após o lançamento de um produto. Contando como programadores, qualquer gráfico desse tipo envolve uma barra para cada valor inteiro 0..n, mas a contagem de barras n pode ser diferente de um gráfico para o próximo.

Considere duas tabelas com cinco e sete barras, respectivamente. As barras para suas cinco categorias comuns, 0..5, podem ser animadas de forma composta como vimos acima. As barras com índice 5 e 6 não têm contrapartida no outro ponto final da animação, mas, como agora estamos livres para dar a cada barra sua própria posição e largura, podemos introduzir duas barras invisíveis para desempenhar esse papel. O efeito visual é que as barras 5 e 6 crescem em sua aparência final à medida que a animação prossegue. Animando na outra direção, as barras 5 e 6 diminuíram ou desapareceram na invisibilidade.

*Lerp entre os valores compostos por lerping correspondendo nos componentes. Onde um componente está faltando no ponto final, use um componente invisível em seu lugar.*

Muitas vezes, existem várias maneiras de escolher componentes invisíveis. Digamos que nosso amigável designer UX decidiu usar barras de zero-largura, zero-altura com coordenadas x e cor herdada de sua contraparte visível. Vamos adicionar um método à `Bar` para criar uma versão colapsada de uma determinada instância.

```dart
class BarChart {
  BarChart(this.bars);

  final List<Bar> bars;

  static BarChart lerp(BarChart begin, BarChart end, double t) {
    final barCount = max(begin.bars.length, end.bars.length);
    final bars = new List.generate(
      barCount,
      (i) => Bar.lerp(
            begin._barOrNull(i) ?? end.bars[i].collapsed,
            end._barOrNull(i) ?? begin.bars[i].collapsed,
            t,
          ),
    );
    return new BarChart(bars);
  }

  Bar _barOrNull(int index) => (index < bars.length ? bars[index] : null);
}

class BarChartTween extends Tween<BarChart> {
  BarChartTween(BarChart begin, BarChart end) : super(begin: begin, end: end);

  @override
  BarChart lerp(double t) => BarChart.lerp(begin, end, t);
}

class Bar {
  Bar(this.x, this.width, this.height, this.color);

  final double x;
  final double width;
  final double height;
  final Color color;

  Bar get collapsed => new Bar(x, 0.0, 0.0, color);
  
  static Bar lerp(Bar begin, Bar end, double t) {
    return new Bar(
      lerpDouble(begin.x, end.x, t),
      lerpDouble(begin.width, end.width, t),
      lerpDouble(begin.height, end.height, t),
      Color.lerp(begin.color, end.color, t),
    );
  }
}
```
A integração do código acima em nosso aplicativo envolve redefinir `BarChart.empty` e `BarChart.random` para esta nova configuração. Agora, um gráfico de barras vazio pode ser considerado razoável para conter zero barras, enquanto que um aleatório pode conter um número aleatório de barras da mesma cor escolhida aleatoriamente, e cada uma delas possui uma altura escolhida aleatoriamente. Mas, uma vez que a posição e a largura são agora parte da definição de `Bar`, precisamos do `BarChart.random` para especificar esses atributos também. Parece razoável fornecer `BarChart.random` com o parâmetro `Size` do gráfico e, em seguida, libere o `BarChartPainter.paint` da maioria dos seus cálculos (listagem de códigos).

![Image of Chart Image]
(https://cdn-images-1.medium.com/max/800/1*dN9og1kRYpRsL-cFIgO23w.gif)
###### Lerping de/para barras invisíveis.

O leitor astuto pode ter notado uma ineficiência potencial em nossa definição de `BarChart.lerp` acima. Estamos criando instâncias de `Bar` colapsadas apenas para serem dadas como argumentos para `Bar.lerp`, e isso acontece repetidamente, para cada valor do parâmetro de animação `t`. Em 60 quadros (frames) por segundo, isso poderia significar muitas instâncias do `Bar` alimentadas ao coletor de lixo, mesmo para uma animação relativamente curta. Existem alternativas:

* As instâncias da `Bar` colapsada podem ser reutilizadas sendo criadas apenas uma vez na classe `Bar`, em vez de em cada chamada para colapsar. Esta abordagem funciona aqui, mas não é geralmente aplicável.

* A reutilização pode ser tratada pelo `BarChartTween` em vez disso, o construtor cria uma lista `_tween` de instâncias `BarTween` usadas durante a criação do gráfico de barras lerped: `(i) => _tweens[i].lerp(t)`. Esta abordagem rompe com a convenção de usar métodos estáticos `lerp` em toda a parte. Não há nenhum objeto envolvido no estático `BarChart.lerp` no qual armazenar a lista de interpolação para a duração da animação. O objeto `BarChartTween`, ao contrário, é perfeitamente adequado para isso.

* Uma barra `null` pode ser usada para representar uma barra colapsada, assumindo lógica condicional adequada em `Bar.lerp`. Esta abordagem é lisa e eficiente, mas requer algum cuidado para evitar a desreferência ou a má interpretação `null`. É comumente usado no Flutter SDK onde os métodos estáticos `lerp` tendem a aceitar `null`  como um ponto final de animação, tipicamente interpretando-o como algum elemento invisível, como uma cor completamente transparente ou um elemento gráfico de tamanho zero. Como o exemplo mais básico, `lerpDouble` trata `null` como zero, a menos que ambos os pontos finais da animação sejam `null`.

O recorte abaixo mostra o código que escreveríamos seguindo a abordagem `null`:

```dart
class BarChart {
  BarChart(this.bars);

  final List<Bar> bars;

  static BarChart lerp(BarChart begin, BarChart end, double t) {
    final barCount = max(begin.bars.length, end.bars.length);
    final bars = new List.generate(
      barCount,
      (i) => Bar.lerp(begin._barOrNull(i), end._barOrNull(i), t),
    );
    return new BarChart(bars);
  }

  Bar _barOrNull(int index) => (index < bars.length ? bars[index] : null);
}

class BarChartTween extends Tween<BarChart> {
  BarChartTween(BarChart begin, BarChart end) : super(begin: begin, end: end);

  @override
  BarChart lerp(double t) => BarChart.lerp(begin, end, t);
}

class Bar {
  Bar(this.x, this.width, this.height, this.color);

  final double x;
  final double width;
  final double height;
  final Color color;

  static Bar lerp(Bar begin, Bar end, double t) {
    if (begin == null && end == null)
      return null;
    return new Bar(
      lerpDouble((begin ?? end).x, (end ?? begin).x, t),
      lerpDouble(begin?.width, end?.width, t),
      lerpDouble(begin?.height, end?.height, t),
      Color.lerp((begin ?? end).color, (end ?? begin).color, t),
    );
  }
}
```
Eu acho justo dizer que Dart `?` a sintaxe é adequada para a tarefa. Mas observe como a decisão de usar barras colapsadas (e não, digamos, transparentes) como elementos invisíveis agora está enterrada na lógica condicional em `Bar.lerp`. Essa é a principal razão pela qual escolhi a solução aparentemente menos eficiente anteriormente. Como sempre em questões de desempenho versus manutenção, sua escolha deve ser baseada em medidas.

Temos um passo a seguir antes de podermos abordar a animação do gráfico de barras em geral. Considere um aplicativo usando um gráfico de barras para mostrar as vendas por categoria de produto para um determinado ano. O usuário pode selecionar outro ano, e o aplicativo deve então se animar para o gráfico de barras desse ano. Se as categorias de produtos fossem as mesmas durante os dois anos, ou passassem a ser as mesmas, exceto para algumas categorias adicionais mostradas à direita em um dos gráficos, poderíamos usar nosso código existente acima. Mas e se a empresa tivesse categorias de produtos A, B, C e X em 2016, mas descontinuou B e introduziu D em 2017? Nosso código existente seria animado da seguinte maneira:

```
2016  2017
  A -> A
  B -> C
  C -> D
  X -> X
```
A animação pode ser bonita e sedosa, mas ainda é confusa para o usuário. Por quê? Porque não preserva a semântica. Ele transforma um elemento gráfico representando a categoria de produto B em um que representa a categoria C, enquanto o de C vai em outro lugar. Só porque 2016 B passa a ser desenhado na mesma posição em que 2017 C aparece depois não implica que o primeiro se transformasse no último. Em vez disso, 2016 B deve desaparecer, 2016 C deve se mover para a esquerda e se transformar em 2017 C e 2017 D deve aparecer à sua direita. Podemos implementar essa mistura usando um dos algoritmos mais antigos do livro: mesclando listas ordenadas.

*Lerp entre composição de valores por componentes lerping correspondendo semanticamente. Quando os componentes formam listas ordenadas, o algoritmo de mesclagem pode trazer esses componentes de forma par, usando componentes invisíveis conforme necessário para lidar com fusões unilaterais.*

Tudo o que precisamos é tornar as instâncias do `Bar` mutuamente comparáveis ​​em uma ordem linear. Então podemos juntá-los da seguinte maneira:

```dart
static BarChart lerp(BarChart begin, BarChart end, double t) {
  final bars = <Bar>[];
  final bMax = begin.bars.length;
  final eMax = end.bars.length;
  var b = 0;
  var e = 0;
  while (b + e < bMax + eMax) {
    if (b < bMax && (e == eMax || begin.bars[b] < end.bars[e])) {
      bars.add(Bar.lerp(begin.bars[b], begin.bars[b].collapsed, t));
      b++;
    } else if (e < eMax && (b == bMax || end.bars[e] < begin.bars[b])) {
      bars.add(Bar.lerp(end.bars[e].collapsed, end.bars[e], t));
      e++;
    } else {
      bars.add(Bar.lerp(begin.bars[b], end.bars[e], t));
      b++;
      e++;
    }
  }
  return new BarChart(bars);
}
```
Concretamente, nós atribuiremos a cada barra uma chave de ordenação na forma de um atributo `rank` inteiro. O ranking pode então ser convenientemente usado também para atribuir a cada barra uma cor da paleta, permitindo-nos acompanhar o movimento de barras individuais na demonstração da animação.

Um gráfico de barras aleatório agora será baseado em uma seleção aleatória de classificações para incluir (lista de códigos).

![Image of Chart Image]
(https://cdn-images-1.medium.com/max/800/1*MuSAOLktwY8bTJdPGuoNqA.gif)
###### Categorias arbitrárias. Lerping baseado em mesclagem.

Isso funciona bem, mas talvez não seja a solução mais eficiente. Estamos repetidamente executando o algoritmo de mesclagem em `BarChart.lerp`, uma vez para cada valor de `t`. Para corrigir isso, implementaremos a idéia mencionada anteriormente para armazenar informações reutilizáveis no `BarChartTween`.

```dart
class BarChartTween extends Tween<BarChart> {
  BarChartTween(BarChart begin, BarChart end) : super(begin: begin, end: end) {
    final bMax = begin.bars.length;
    final eMax = end.bars.length;
    var b = 0;
    var e = 0;
    while (b + e < bMax + eMax) {
      if (b < bMax && (e == eMax || begin.bars[b] < end.bars[e])) {
        _tweens.add(new BarTween(begin.bars[b], begin.bars[b].collapsed));
        b++;
      } else if (e < eMax && (b == bMax || end.bars[e] < begin.bars[b])) {
        _tweens.add(new BarTween(end.bars[e].collapsed, end.bars[e]));
        e++;
      } else {
        _tweens.add(new BarTween(begin.bars[b], end.bars[e]));
        b++;
        e++;
      }
    }
  }

  final _tweens = <BarTween>[];

  @override
  BarChart lerp(double t) => new BarChart(
        new List.generate(
          _tweens.length,
          (i) => _tweens[i].lerp(t),
        ),
      );
}
```
Agora podemos remover o método estático `BarChart.lerp`.

Vamos resumir o que aprendemos sobre o conceito de interpolação até agora:

*Animar `T`s rastreando um caminho no espaço de todos os `T`s, pois o valor da animação é executado de zero a um. Modelar o caminho com um `Tween<T>`.*

*Generalize o conceito de `T` conforme necessário até abranger todos os pontos finais da animação e valores intermediários.*

*Lerp entre os valores compostos pelo componentes correspondentes ao lerping.*

* *A correspondência deve ser baseada em semântica, e não em co-localização gráfica acidental.*

* *Onde um componente está faltando em um ponto final de animação, use um componente invisível em seu lugar, possivelmente derivado do outro ponto final.*

* *Onde os componentes formam listas ordenadas, use o algoritmo de mesclagem para trazer componentes semanticamente correspondentes em um par, apresentando componentes invisíveis conforme necessário para lidar com fusões unilaterais.*

*Considere implementar tweens usando métodos estáticos `Xxx.lerp` para facilitar a reutilização em implementações de interpolação composta. Quando uma recomputação significativa ocorre através de chamadas para `Xxx.lerp` para um único caminho de animação, considere mover a computação para o construtor da classe `XxxTween` e permitir que suas instâncias hospedem o resultado da computação.*

Armados com esses pensamentos, estamos finalmente em posição de animar gráficos mais complexos. Nós faremos barras empilhadas, barras agrupadas e barras agrupadas + agrupadas em sucessão rápida:

* As barras empilhadas são usadas para conjuntos de dados onde as categorias são bidimensionais e faz sentido adicionar a quantidade numérica representada pelas alturas da barra. Um exemplo pode ser receita por produto e região geográfica. O empilhamento por produto facilita a comparação do desempenho do produto no mercado global. A empilhamento por região mostra quais regiões são importantes.

![Image of Chart Image]
(https://cdn-images-1.medium.com/max/800/1*qKUFM56S-ZonH1amVDDXTw.gif)
###### Barras empilhadas

* As barras agrupadas também são usadas para conjuntos de dados com categorias bidimensionais, mas onde não é significativo ou desejável empilhar as barras. Por exemplo, se a quantidade numérica for de mercado em porcentagem por produto e região, o empilhamento por produto não faz sentido. Mesmo quando o empilhamento faz sentido, o agrupamento pode ser preferível, pois torna mais fácil fazer comparações quantitativas em ambas as dimensões da categoria ao mesmo tempo.

![Image of Chart Image]
(https://cdn-images-1.medium.com/max/800/1*YiojxPiaWY7lB5v9iZVgDg.gif)
###### Barras agrupadas

* As barras agrupadas + empilhadas suportam categorias tridimensionais, como receita por produto, região geográfica e canal de vendas.

![Image of Chart Image]
(https://cdn-images-1.medium.com/max/800/1*9ObVOKbos4DoQsmsqbMnRQ.gif)
###### Barras agrupadas + empilhadas

Em todas as três variantes, a animação pode ser usada para visualizar as mudanças de conjunto de dados, introduzindo assim uma dimensão adicional (geralmente tempo) sem desordenar os gráficos.

Para que a animação seja útil e não apenas bonita, precisamos certificar-se de que nosso lerp estja apenas entre componentes correspondentes semanticamente. Assim, o segmento de barras usado para representar a receita de um produto / região / canal específico em 2016 deve ser transformado em uma receita representativa para o mesmo produto / região / canal em 2017 (se presente).

O algoritmo de mesclagem pode ser usado para garantir isso. Como você pode ter adivinhado a partir da discussão anterior, a fusão será posta em prática em vários níveis, refletindo a dimensionalidade das categorias. Mesclaremos pilhas e barras em gráficos em gráficos pilhados, grupos e barras em um agrupamento de gráficos, e todos os três em gráficos agrupados + pilhados.

Para realizar isso sem muita duplicação de código, vamos resumir o algoritmo de mesclagem em um utilitário geral e colocá-lo em um arquivo próprio, `tween.dart`:

```dart
import 'package:flutter/animation.dart';
import 'package:flutter/material.dart';

abstract class MergeTweenable<T> {
  T get empty;

  Tween<T> tweenTo(T other);

  bool operator <(T other);
}

class MergeTween<T extends MergeTweenable<T>> extends Tween<List<T>> {
  MergeTween(List<T> begin, List<T> end) : super(begin: begin, end: end) {
    final bMax = begin.length;
    final eMax = end.length;
    var b = 0;
    var e = 0;
    while (b + e < bMax + eMax) {
      if (b < bMax && (e == eMax || begin[b] < end[e])) {
        _tweens.add(begin[b].tweenTo(begin[b].empty));
        b++;
      } else if (e < eMax && (b == bMax || end[e] < begin[b])) {
        _tweens.add(end[e].empty.tweenTo(end[e]));
        e++;
      } else {
        _tweens.add(begin[b].tweenTo(end[e]));
        b++;
        e++;
      }
    }
  }

  final _tweens = <Tween<T>>[];

  @override
  List<T> lerp(double t) => new List.generate(
        _tweens.length,
        (i) => _tweens[i].lerp(t),
      );
}
```
A interface `MergeTweenable<T>` captura precisamente o que é necessário para poder criar uma interpolação de duas listas ordenadas de `T`s por mesclagem. Vamos instanciar o parâmetro de tipo `T` com `Bar`, `BarStack` e `BarGroup` e fazer todos esses tipos implementar `MergeTweenable<T>`.

As implementações de empilhamento, agrupamento, e empilhamento + agrupamento foram escritas para serem diretamente comparáveis. Eu encorajo você a brincar com o código:

* Mude o número de grupos, pilhas e barras criados pelo `BarChart.random`.

* Mude as paletas de cores. Para barras empilhadas + agrupadas, eu usei uma paleta monocromática, porque acho que parece mais agradável. Você e seu designer UX podem discordar.

* Substitua `BarChart.random` e o botão de ação flutuante por um seletor de ano e crie instâncias de `BarChart` a partir de conjuntos de dados realistas.

* Implemente gráficos de barras horizontais.

* Implementar outros tipos de gráfico (torta, linha, área empilhada). Anime-os usando `MergeTweenable<T>` ou similar.

* Adicione legendas de gráfico e/ou rótulos e eixos, e então anime-os também.

As tarefas das duas últimas balas são bastante desafiadoras. Diverta-se.

A restauração por animação com `BarChartPainter` está configurada nesta linha (linha 28 do bar.dart)

```dart
BarChartPainter(Animation<Bar> animation)
  : animation = animation,
    super(repaint: animation);
```

que contém uma chamada para o construtor do `CustomPainter` com o `Animation<Bar>` como argumento. Veja [aqui](https://docs.flutter.io/flutter/rendering/CustomPainter-class.html) a documentação pertinente da Flutter API. Não há mágica Dart envolvida, apenas uma chamada para um construtor de superclasse.

Mais detalhes abaixo, caso você esteja interessado.

A história detalhada envolve o conceito `RenderObject` que não discuti no meu artigo, pois é um conceito de nível inferior na arquitetura Flutter. Muitos widgets são suportados por um `RenderObject` que é responsável por renderizar o widget, ou seja, para lidar com o ciclo de vida em torno do layout e da pintura na tela.

Assim, a restauração do `BarChartPainter` é realmente realizada na implementação do widget `CustomPaint` que configuramos com o `CustomPainter` (no [main.dart](https://gist.github.com/mravn-google/ec766e03aad1f4cca14cdae3e46170fa#file-main-dart-L53)). O `RenderObject` apoia que o widget anexa um ouvinte na `Animation`, e agende a repintura usando o método de ciclo de vida `markNeedsPaint` de `RenderObject` em cada tempo (tick) da animação. A linha relevante na atual implementação Flutter está [aqui](https://github.com/flutter/flutter/blob/master/packages/flutter/lib/src/rendering/proxy_box.dart#L1990).

Quando uma novo frame é devido, cada "sujo" `RenderObject` é solicitado pelo framework Flutter para ele mesmo pintar na tela, e nossa então delegará a subclasse `CustomPainter` o que nós escrevermos.

