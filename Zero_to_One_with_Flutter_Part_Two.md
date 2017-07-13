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

#Lerp entre valores compostos por componentes correspondentes.#

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