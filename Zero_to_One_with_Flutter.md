Mikkel RavnFollow

Software Engineer at Google

# **Zero to One with Flutter**

Era final do verão de 2016, e minha primeira tarefa como nova contratação no escritório do Google em Aarhus, na Dinamarca, era implementar gráficos animados em um aplicativo Android/iOS usando Flutter e Dart. Além de ser um "Noogler", eu era novo no Flutter, novo no Dart e novo em animações. Na verdade, eu nunca tinha feito um aplicativo para dispositivos móveis antes. Meu primeiro smartphone tinha apenas alguns meses de idade - comprei em um ataque de pânico que eu poderia falhar na entrevista por telefone respondendo a chamada no meu antigo Nokia...

Eu tive alguma experiência anterior com gráficos no desktop com Java, mas isso não foi animado. Eu me senti... estranho. Em parte um dinossauro, em parte renascido.

Descobrindo a força do widget Flutter e os conceitos de interpolação, escrevendo gráficos animados no Dart para um aplicativo Android/iOS.

A mudança para uma nova pilha de desenvolvimento faz você conhecer suas prioridades. Perto do topo da minha lista estão estes três:

* **Conceitos fortes** lidam eficazmente com a complexidade ao fornecer formas simples e relevantes de estruturar pensamentos, lógica ou dados.

* **Código limpo** nos permite expressar esses conceitos de forma clara, sem sermos distraídos por armadilhas de linguagem, excesso de calibração ou detalhes auxiliares.

* **Interação rápida** é a chave para experimentação e aprendizagem - e as equipes de desenvolvimento de software aprendem a viver: quais são realmente os requisitos e a melhor maneira de cumpri-los com conceitos expressos em código.

Flutter é uma nova plataforma para o desenvolvimento de aplicativos Android e iOS a partir de uma única base de código, escrita em Dart. Uma vez que nossos requisitos falavam de uma UI bastante complexa incluindo gráficos animados, a idéia de construí-la apenas uma vez parecia muito atraente. Minhas tarefas envolveram exercitar as ferramentas CLI do Flutter, alguns widgets pré-construídos, e seu mecanismo de renderização 2D - além de escrever um monte de código Dart simples para modelar e animar gráficos. Vou compartilhar abaixo alguns destaques conceituais da minha experiência de aprendizado e fornecer um ponto de partida para sua própria avaliação da pilha Flutter/Dart.

![Image of Chart Image]
(https://cdn-images-1.medium.com/max/800/1*OKV3RzTg89W3VxXnpAH3Eg.gif)
###### Um gráfico de barras animado simples, capturado a partir de um simulador iOS durante o desenvolvimento

Esta é parte um de duas partes, Introdução ao Flutter e seus conceitos de "widget" e "interpolação". Eu vou ilustrar a força desses conceitos, usando-os para exibir e animar gráficos como o mostrado acima. Nos exemplos, o código completo deve fornecer um nível de clareza ao Dart. E incluirei detalhes suficientes para que você possa acompanhar em seu próprio laptop (e emulador ou dispositivo) e experimente o comprimento do ciclo de desenvolvimento Flutter.

O ponto de partida é a [instalação do Flutter](https://flutter.io/setup/). Rode

```
$ flutter doctor
```

para checar a configuração:

```
$ flutter doctor
[✓] Flutter (on Mac OS, channel master)
  • Flutter at /Users/mravn/flutter
  • Framework revision 64bae978f1 (7 hours ago), 2017-02-18 21:00:27
  • Engine revision ab09530927
  • Tools Dart version 1.23.0-dev.0.0
[✓] Android toolchain - develop for Android devices
    (Android SDK 24.0.2)
  • Android SDK at /Users/mravn/Library/Android/sdk
  • Platform android-25, build-tools 24.0.2
  • Java(TM) SE Runtime Environment (build 1.8.0_101-b13)
[✓] iOS toolchain - develop for iOS devices (Xcode 8.2.1)
  • Xcode at /Applications/Xcode.app/Contents/Developer
  • Xcode 8.2.1, Build version 8C1002
  • ios-deploy 1.9.1
[✓] IntelliJ IDEA Community Edition (version 2016.3.4)
  • Dart plugin version 163.13137
  • Flutter plugin version 0.1.10
[✓] Connected devices
  • iPhone SE • 664A33B0-A060-4839-A933-7589EF46809B • ios •
    iOS 10.2 (simulator)
```

Com marcas de verificação suficientes, você pode criar um aplicativo Flutter. Vamos chama-lo de `charts`:

```
$ flutter create charts
```

Isso deve dar-lhe um diretório do mesmo nome:

```
charts
  android
  ios
  lib
    main.dart
```

Cerca de cinquenta arquivos foram gerados, formando um aplicativo de exemplo completo que pode ser instalado tanto no Android como no iOS. Faremos toda a nossa codificação no `main.dart` e nos arquivos irmãos, sem necessidade de pressionar qualquer outro arquivo ou diretório.

Você deve verificar o que você pode iniciar com o aplicativo de exemplo. Inicie um emulador ou conecte um dispositivo e execute dentro do diretório `charts`

```
$ flutter run
```
Você deve então ver um aplicativo de contagem simples em seu emulador ou dispositivo. Ele usa widgets do Material Design, o que é bom, mas opcional. O widget Material Design esta na camada mais alta da arquitetura Flutter, contudo ele é completamente substituível.

Comecemos por substituir o conteúdo do `main.dart` pelo código abaixo, um ponto de partida simples para jogar com animações de gráfico.

```dart
import 'dart:math';

import 'package:flutter/material.dart';

void main() {
  runApp(new MaterialApp(home: new ChartPage()));
}

class ChartPage extends StatefulWidget {
  @override
  ChartPageState createState() => new ChartPageState();
}

class ChartPageState extends State<ChartPage> {
  final random = new Random();
  int dataSet;
  
  void changeData() {
    setState(() {
      dataSet = random.nextInt(100);
    });
  }
  
  @override
  Widget build(BuildContext context) {
    return new Scaffold(
      body: new Center(
        child: new Text('Data set: $dataSet'),
      ),
      floatingActionButton: new FloatingActionButton(
        child: new Icon(Icons.refresh),
        onPressed: changeData,
      ),
    );
  }
}
```

Salve as alterações e reinicie o aplicativo. Você pode fazer isso a partir do terminal, pressionando `R`. Esta operação de "reiniciar completamente" tira o estado do aplicativo e, em seguida, reconstrói a interface do usuário. Para situações em que o estado do aplicativo existente ainda faz sentido após a mudança do código, pode-se pressionar `r` para fazer uma "recarga quente", que só reconstrói a IU. Há também um plugin Flutter para o IntelliJ IDEA, fornecendo a mesma funcionalidade integrada com um editor Dart.

Uma vez reiniciado, o aplicativo mostra um rótulo de texto centrado dizendo `Data set: null` e um botão de ação flutuante para atualizar os dados. Sim, começos humildes.

Para ter uma idéia da diferença entre o recarregamento quente e o reinício completo, tente o seguinte: Depois de pressionar o botão de ação flutuante algumas vezes, anote o número atual do conjunto de dados e em seguida, substitua `Icons.refresh` por `Icons.add` no código, salve e faça uma recarga quente. Observe que o botão muda, mas que o estado do aplicativo é mantido; Ainda estamos no mesmo lugar no fluxo aleatório de números. Agora desfaça a mudança de ícone, salve e faça um reinício completo. O estado do aplicativo foi reiniciado e voltamos ao `Data set: null`.

Nosso simples aplicativo mostra dois aspectos centrais a respeito do conceito de widget no Flutter:

* A interface do usuário é definida por uma árvore de **widgets imutáveis**, que é construída através de um dança de chamadas de construtores (onde você consegue configurar widgets) e métodos de construção `build` (onde as implementações de widgets conseguem decidir como suas sub-árvores se parecem). A estrutura de árvore resultante para o nosso aplicativo é mostrada abaixo, com o papel principal de cada widget entre parênteses. Como você pode ver, enquanto o conceito do widget é bastante amplo, cada tipo de widget concreto geralmente tem uma responsabilidade muito focada.

```
MaterialApp                    (navigation)
  ChartPage                    (state management)
    Scaffold                   (layout)
      Center                   (layout)
        Text                   (text)
      FloatingActionButton     (user interaction)
        Icon                   (graphics) 
```

* Com uma árvore imutável de widgets imutáveis ​​que definem a interface do usuário, a única maneira de mudar essa interface é reconstruir a árvore. Flutter cuida disso, quando o próximo quadro `frame` é devido. Tudo o que temos a fazer é dizer ao Flutter que algum estado na qual uma sub-eárvore depende foi alterado. A raiz de uma sub-árvore dependente desse estado deve ser um `StatefulWidget`. Como qualquer widget decente, um `StatefulWidget` não é mutable, mas sua sub-árvore é construída por um objeto `State` que é. Flutter mantém os objetos `State` por meio da reconstrução das árvores e anexa cada respectivo widget na nova árvore durante a construção. Eles então determinam como a sub-árvore de widget foi construída. No nosso aplicativo, `ChartPage` é um `StatefulWidget` com `ChartPageState` como seu `State`. Sempre que o usuário pressiona o botão, executamos algum código para alterar o `ChartPageState`. Nós demarcamos a mudança com `setState` para que Flutter possa fazer seu arrumação e agendar a árvore de widget para reconstrui-lá. Quando isso acontece, o `ChartPageState` irá criar uma sub-árvore ligeiramente diferente enraizada na nova instância do `ChartPage`.

Os widgets imutáveis e as árvores dependentes de estado são as principais ferramentas que o Flutter coloca à nossa disposição para abordar as complexidades do gerenciamento do estado em UI elaboradas que respondem a eventos assíncronos, como pressões de botões, tiques temporizados ou dados recebidos. Da experiência da minha área de trabalho eu diria que essa complexidade é muito real. Avaliar a força da abordagem Flutter é - e deve ser - um exercício para o leitor: experimente-o em algo não trivial.






