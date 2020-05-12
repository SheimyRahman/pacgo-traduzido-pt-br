# Passo 06: Tornando em (tempo)real as coisas

Nesta lição, você aprenderá a fazer:

- Usar goroutines
- Usar anonymous functions (lambdas)
- Usar canais (channels)
- Usar o comando select para ler os canais async
- Usar package time

## Overview

Neste ponto do tutorial nós temos um jogo completo: ele tem um objetivo claro, condições de ganhar e perder e a entrada do jogador funciona corretamente.

Mas tem um grande problema: os inimigos só se movem quando o jogador se move também. Isso não parece muito divertido no quesito de jogabilidade, então vamos fazer isso apropriadamente?!

Este problema acontece porque a leitura do input é lida como uma blocking operation (manobra síncrona). Para resolver esse problema, precisamos torná-la assíncrona de alguma forma... se ao menos tivéssemos alguma funcionalidade para executar as coisas assíncronas em movimento... 🧐
Oh, espere! Nós temos! :)

E para nos salvar, temos as fabulosas goroutines e os channels!

Goroutines são semelhantes às threads, mas com a vantagem que eles são muito mais performáticas (ligeiras). Por "debaixo" dos dos panos, o go runtime gerencia as threads que irão manipular as goroutines, e lembrando que uma única thread tem o poder de administrar várias goroutines, portanto sua relação é maior que 1:1.



Mas essa não é a melhor parte. O design da linguagem go torna muito fácil criar um goroutine: você só precisa adicionar a palavra-chave `go` antes da chamada da função e a função será executada como um Goroutine separada e de forma assíncrona

Dê uma olhada no código abaixo:

```go
func main() {
    go fmt.Println("hello")
    go fmt.Println("world")
}
```

Este código possui três goroutines: a primeira é a que executa a função `main`, a segunda é a que imprime o `hello` e a terceira é a que imprime a string `world`.

Uma coisa importante sobre as goroutines é lembrar de considerar que, como estamos fazendo coisas de maneira assíncronas, é seguro assumir que o programa anterior não produzirá nenhum resultado. Isso porque a função `main` tem uma alta probabilidade de terminar o programa antes de qualquer uma das outras duas goroutines serem executadas (pois temos algum overhead ao executar as goroutines).

Poderíamos introduzir um delay na função principal:

```go
func main() {
    go fmt.Println("hello")
    go fmt.Println("world")
    time.Sleep(100 * time.Millisecond)
}
```

Isso garantiria que as goroutines executariam, pois esperamos que sejam mais rápidas que 100ms, mas mesmo assim, a saída deste programa é imprevisível, pois não podemos contar com a ordem em que os goroutines são executados. 

Uma vez que uma declaração de `go` é executada, a responsabilidade pelo escalonamento da goroutine para sua respectiva execução é passada para o go runtime. Não temos controle sobre isso e nunca podemos assumir uma ordem específica de execução. Tenha isso em mente ao escrever um código async.

Além das goroutines, temos também as channel constructs. Os Channels nos possibilitam a comunicação com as goroutines, enviando ou recebendo valores. Ou ambos.

Para criar e declarar um Channel, utilizamos a função `make` embutida:

```go
ch := make(chan int)
```

Cada canal tem um tipo, e opcionalmente um buffer size (dimensão). Se nenhum tamanho for especificado, assume-se que seja igual a 1.

Reading and writing to a channel can be a blocking operation, unless the channel is empty.

Ler e escrever em um Channel pode ser uma blocking operation, a menos que o Channel esteja vazio.

Para escrever em um canal usa-se o operador seta (`<-`):

```go
// something is written to ch
ch <- something
```

No cenário acima, se o `ch` estiver vazio a operação não vai ser bloqueada, mas se estiver cheio a operação será pausada até o que o canal seja consumido do outro lado.

Da mesma forma, a leitura a partir de um canal também utiliza o operador de seta:

```go
// on a different goroutine
foo := <-ch
```

Ao projetar um processamento assíncrono devemos ter cuidado para que duas goroutino]es não dependam uma da outra de forma que ambos possam ser mantidos em estado de bloqueio ou produzir resultados inconsistentes. Para saber mais sobre bloqueios e race conditions, veja [this answer](https://stackoverflow.com/a/3130212/4893628) on StackOverflow.

## Tarefa 01: Refatorando o input code

Agora que sabemos o básico sobre goroutines e channels, vamos colocá-los em ação. Primeiro, vamos remover o código de manipulação de entrada do loop do jogo e inserir o código abaixo antes do início do loop.

```go
func main() {
    // init code omitted for brevity...

    // process input (async)
    input := make(chan string)
    go func(ch chan<- string) {
        for {
            input, err := readInput()
            if err != nil {
                log.Println("error reading input:", err)
                ch <- "ESC"
            }
            ch <- input
        }
    }(input)

    // game loop
    for {
        // loop code...
    }
}
```

Este código criará um canal (channel) chamado `input` e o passará como parâmetro para uma função anônima que é chamada com a instrução `go`. Esse é um padrão muito comum para processamento assíncrono em go.

A função anônima então cria um loop infinito onde espera pela entrada e a escreve no canal `ch` (dado pelo parâmetro da função). Em caso de erro, apenas retornamos o código "ESC", pois sabemos que isto irá terminar o programa

E no loop do jogo vamos substituir o código que processa o movimento do jogador pelo código abaixo:


```go
// process movement
select {
case inp := <-input:
    if inp == "ESC" {
        lives = 0
    }
    movePlayer(inp)
default:
}
```

Imagine que a declaração selecionada é como uma declaração de troca, mas para os channels. Este comando `select` tem uma característica de não-bloqueio, pois tem uma cláusula padrão. Isto significa que se o canal `input` tem algo para ser lido ele será lido, caso contrário o caso `default` é processado, que neste caso é um bloco vazio.

Finalmente, já que mudamos a lógica "ESC" para o bloco acima, vamos removê-la da condição de game over ( pois, `lives <= 0` já a satisfaz.

Também vamos precisar introduzir um delay de 200ms. Já que agora não estamos mais aguardando por input de informações, sendo assim, o jogo vai correr muito rápido sem ele. O snippet relevante está abaixo:

```go
    // update screen
    printScreen()

    // check game over
    if numDots == 0 || lives <= 0 {
        break
    }

    // repeat
    time.Sleep(200 * time.Millisecond)
```

Tente executar o jogo agora. Muito mais emocionante, não é? :)

[Leve-me para o passo 07!](.../passo07/README.md)
