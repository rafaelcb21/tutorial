# Introdução

O sistema de animação no Flutter é baseado na tipagem do objeto `Animation`. Widgets podem também incorporar essas animações em suas funções construtoras (build) diretamente pela leitura de seus valores correntes e ouvindo as mudanças em seus estados (state) ou eles podem usar as animações como base para animações mais elaboradas no qual eles passam por meio de outros widgets.

## Animação

O primeiro bloco construido no sistema de animação é a classe `Animation`. Uma animação representa o valor de um tipo específico que pode mudar ao longo da vida útil da animação. A maioria dos widgets que executam uma animação recebem um objeto `Animation` como parâmetro, no qual esses widgets lêem o valor atual da animação e escutam suas mudanças.

`addListener`

Sempre que o valor de uma animação alterar, a animação notificará para todos os ouvintes (listeners) adicionados com `addListener`. Normalmente, um objeto `State` que escuta uma animação chamará `setState` em si mesmo no retorno - `callback` - do ouvinte para notificar o sistema de widgets que ele precisa reconstruir com o novo valor da animação.

Esse padrão é tão comum que há dois widgets que ajudarão os widgets a se reconstruir quando a animação alterar seu valor. `AnimatedWidget` e `AnimateBuilder`. O primeiro, `AnimatedWidget` é mais útil para widgets animados sem estado (stateless). Para usar `AnimatedWidget`, basta estedê-lo na classe e implementar a função construtora (build).

```dart
class MyClass extends AnimatedWidget {
  @override
  Widget build(BuildContext context) {}
}
```

O segundo, `AnimateBuilder` é útil para widgets mais complexos que desejam incluir uma animação como parte de uma função de construção maior. Para usar `AnimatedBuilder`, basta construir o widget e passá-lo a uma função contrutora - `builder`.
<Exemplo>

`addStatusListener`

Animações também providenciam um `AnimationStatus`, no qual indica como a animação irá evoluir ao longo do tempo. Sempre que o estado da animação mudar, a animação notificará a todos os ouvintes adicionandos com o `addStatusListener`. Normalmente, as animações começam no status descartado, o que significa que eles estão no início de sua série. Por exemplo, as animações que progridem de 0,0 para 1,0 estarão descartadas quando seu valor for 0,0. Uma animação pode então avançar (por exemplo, de 0,0 à 1,0) ou talvez em sentido inverso (por exemplo, de 1,0 à 0,0). Eventualmente, se a animação atingir o fim de sua série (por exemplo, 1.0), a animação atinge o status de completado.
<Exemplo>

## AnimationController

Para criar uma animação, primeiro crie um `AnimationController`. Além de ser uma animação em si, um `AnimationController` permite que você controle a animação. Por exemplo, você pode dizer ao controlador para reproduzir a animação para `frente` ou `parar` a animação. Você também pode lançar animações, na qual usam simulações da física, como um salto, para dirigir a animação.
<Exemplo>

Uma vez criado um controlador de animação, você pode começar a criar outras animações baseados nele. Por exemplo, você pode criar uma `ReverseAnimation` que espelha a animação original, mas executa-lá em direção oposta (por exemplo, de 1,0 a 0,0). Da mesma forma, você pode criar uma `CurvedAnimation` cujo valor é ajustado por uma curva.
<Exemplo>

## Tweens (Extremidades 'inicio e fim' - oposto de betweens que significa 'entre, no meio')

Para animar além do intervalo de 0,0 a 1,0, você pode usar um `Tween<T>`, que interpola entre os valores `inicial` e `final`. Muitos tipos possuem subclasses `Tween` específicas que fornecem um tipo específico de interpolação. Por exemplo, `ColorTween` interpola entre cores e `RectTween` interpola entre retas. Você pode definir suas próprias interpolações criando sua própria subclasse de `Tween` e substituindo sua função `lerp`.

Por si só, uma interpolação apenas define como interpolar entre dois valores. Para obter um valor concreto para o frame atual de uma animação, você também precisa de uma animação para determinar o estado atual. Existem duas maneiras de combinar um tween com uma animação para obter o valor concreto:

1. Você pode calcular `evaluate` o tween como valor atual de uma animação. Esta abordagem é mais útil para widgets que já estão ouvindo a animação e consequentemente reconstruindo sempre que a animação muda de valor.

2. Você pode animar o tween com base na animação. Em vez de retornar um único valor, a função animação retorna uma nova Animação que incorpora o tween. Esta abordagem é mais útil quando você deseja dar a animação recém-criada a outro widget, que pode então ler o valor atual que incorpora o tween, assim como ouvir as alterações do valor.

# Arquitetura

As animações são realmente construídas a partir de varios núcleos de construção de blocos.

## Scheduler (Agendador)
<Exemplo>

O `SchedulerBinding` é uma classe única que expõe os agendamentos primários do Flutter.

Para essa discussão, a chave primária é um frame de retorno (callback). Cada vez que um quadro precisa ser exibido na tela, o mecanismo do Flutter desencadeia um retorno de chamada (callback) do "quadro inicial" que o agendador multiplexa para todos os ouvintes registrados usando `scheduleFrameCallback()`. Todos esses retornos (callbacks) recebem o carimbo de hora oficial do quadro, na forma de uma `Duration` de alguma época arbitrária. Já que todas as chamadas de retorno (callbacks) têm o mesmo tempo, todas as animações desencadeadas a partir dessas devoluções de chamada parecerão exatamente sincronizadas, mesmo que levem alguns milissegundos a serem executados.
<Exemplo>

## Tickers (Marcação de Relógio)
<Exemplo>

A classe `Ticker` se encaixa dentro do mecanismo agendador `scheduleFrameCallback()` para invocar um retorno de chamada para cada marca (tick).

Um `Ticker` pode ser iniciado e parado. Quando iniciado, ele retorna um futuro que será resolvido quando ele for parado.

Cada marca (tick), o `Ticker` fornece o retorno de chamada com a duração desde o primeiro tique-taque (tick) após o início.

Como os tique-taque do relógio (tickers) sempre dão o tempo decorrido em relação ao primeiro tique do relógio (tick) depois de serem iniciados, todos os tickers estão sincronizados. Se você começar três tickers em momentos diferentes entre dois quadros (frames), todos eles serão sincronizados com o mesmo horário de início e, posteriormente, seus tique-taques (marcações) estarão todos acertados, como se fossem três reloógios no qual seus ponteiros de segundos estivessem sincronizados, marcando todos o mesmo tempo.

## Simulações
<Exemplo>

A classe abstrata `Simulation` mapeia um valor de tempo relativo (um tempo decorrido) para um valor duplo,e tem uma noção de conclusão.

Em princípio as simulações não possuem estado, mas ná prática algumas simulações (por exemplo, `BouncingScrollSimulation` e `ClampingScrollSimulation`) alteram seus estados irreversivelmente quando requeridas.

Existem várias implementações concretas da classe `Simulation` para diferentes efeitos.

## Animatables (Habilidade de animar)

A classe abstrata `Animatables` mapeia um duplo valor para um tipo específico.

A classe `Animatables` não possui estado (stateless) e é imutável.

### Tweens (Extremidades 'inicio e fim' - oposto de betweens que significa 'entre, no meio')

A classe abstrata `Tween` mapeia um duplo valor nominalmente no intervalo de 0,0 à 1,0 para um valor de tipo específico (ex: o objeto `Color`). É um `Animatable`. Isso significa que o objeto Color possui a propriedade `Animatable`, ou seja, a cor pode ser animada.

Ele tem como saída um objeto do tipo (T), um valor de início `begin` e um valor final `end` desse tipo, e uma maneira de interpolar (`lerp` - interpolação linear, ou seja, gera os pontos entre o inicio e o fim de uma faixa) entre os valores inicial e final para um valor de entrada dado (o valor duplo, inicio e fim, geralmente está na faixa de 0,0 à 1,0).

A classe `Tween` não possui estado (stateless) e é imutável.

### Encadeamento de animatables (Encadeamento de objetos que estão animados)
<Exemplo>

Passando um `Animatable<double>` (pai) para o método `chain()` do Animatable cria uma nova subclasse `Animatable` que aplica o mapeamento do pai para as criaças mapeadas.

## Curvas
<Exemplo>

A classe abstrata `Curve` mapeia um duplo valor nominalmente no intervalo de 0,0 à 1,0 para dobrar nominalmente o intervalo de 0,0 à 1,0.

A classe `Curve` não possui estado (stateless) e é imutável.

## Animações

A classe abstrata `Animation` fornece um valor de um determinado tipo, um conceito de direção de animação e status de animação, e uma interface de ouvinte para registrar retornos de chamada (callbacks) que são invocados quando o valor ou o estado são alterados.

Algumas subclasses de Animação têm valores que nunca mudam (`kAlwaysCompleteAnimation`, `kAlwaysDismissedAnimation`, `AlwaysStoppedAnimation`); o registro de retornos de chamada (callbacks) nesses itens não tem efeito, já que as retornos de chamada (callbacks) nunca são chamados.

A variante `Animation<double>` é especial porque pode ser usada para representar um duplo nominalmente na faixa de 0,0 à 1.0, na qual a entrada esperada pelas classes `Curve` e `Tween`, bem como algumas subclasses adicionais da `Animation`.

Algumas subclasses de `Animation` não possuem estado, apenas enviando ouvintes aos pais. Algumas possuem estado.

### Encadeando animações
<Exemplos>

A maioria das subclasses de `Animation` levam "pai" explícito `Animation<double>`. Eles são conduzidos por esse pai.

A subclasse `CurvedAnimation` leva uma classe `Animation<double>` (pai) e um casal de classes `Curve` (as curvas para a frente e para trás - `forward` and `reverse`) como entrada e usa o valor do pai como entrada para as curvas para uma determinar saída. `CurvedAnimation` é imutável e sem estado.

A subclasse `ReverseAnimation` leva uma classe `Animation<double>` como pai e inverte todos os valores da animação. Ele assume que o pai está usando um valor nominalmente no intervalo de 0,0 à 1.0 e retorna um valor no intervalo 1,0 à 0,0. O status e a direção da animação pai também são revertidos. `ReverseAnimation` é imutável e sem estado.

A subclasse `ProxyAnimation` assume uma classe `Animation<double>` como pai e apenas encaminha o estado atual desse pai. No entanto, o pai é mutable.

A subclasse `TrainHoppingAnimation` leva dois pais, e alternam entre eles quando seus valores se cruzam.

### Controladores de animação
<Exemplos>

O `AnimationController` é um `Animation<double>` com estado que usa um `Ticker` para se dar vida. Ele pode ser iniciado e parado. Cada marcação (tick), leva o tempo decorrido desde que foi iniciado e passa para um `Simulation` para obter o valor. Esse é então o valor que relata. Se a `Simulation` relatar que, nesse momento, terminou, o controlador se parará.

O controlador de animação pode receber um limite inferior e superior para animar, e uma duração.

No caso simples (usando `forward()`, `reverse()`, `play()` ou `resume()`), o controlador de animação simplesmente faz uma interpolação linear do limite inferior para o limite superior (ou vice-versa, para a direção inversa) durante a tempo fornecido.

Ao usar `repeat()`, o controlador de animação usa uma interpolação linear entre os limites dados durante o tempo fornecido, mas não irá parar.

Ao usar `animateTo()`, o controlador de animação faz uma interpolação linear durante a duração dada do valor atual para o alvo especificado. Se nenhuma duração for dada ao método, a duração padrão do controlador e o intervalo descrito pelo limite inferior e limite superior do controlador são usados ​​para determinar a velocidade da animação.

Ao usar `fling()`, um `Force` é usado para criar uma simulação específica que é usada para conduzir o controlador.

Ao usar `animateWith()`, a simulação fornecida é usada para direcionar o controlador.

Todos esses métodos retornam o futuro que o `Ticker` fornece e que irá resolver quando o controlador parar ou alterar a simulação.

### Anexando animatables ​​para animações
<Exemplo>

Passando uma `Animation<double>` (o novo pai) para o método `animate()` de `Animatable` cria uma nova subclasse de `Animation` que age como `Animatable`, mas é conduzida pelo pai dado.

# Recursos

`Zero à Um com Flutter`: Ao utilizar `Tweens` para animar gráficos de barras.