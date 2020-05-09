# Passo 04: Fantasmas!

Nesta lição, você aprenderá a fazer: 

- Criar um mapa (dicionário)
- Gerar números aleatórios
- Usar ponteiros

## Overview

Agora que podemos mover nosso jogador, é hora de fazer algo a respeito de nossos inimigos (fantasmas).

Aqui, utilizaremos a mesma mecânica de movimentação do jogador, a função `makeMove`, mas ao invés de ler a entrada do teclado, utilizaremos um algoritmo simples: gerar um número aleatório entre 0 e 3 e atribuir uma direção a cada um desses valores.

Se o Fantasma atingir uma parede não importa, ele apenas tentará novamente na próxima iteração.

## Tarefa 01: Criando os Fantasmas

Assim como criamos uma estrutura para guardar os dados dos nossos jogadores, vamos criar uma estrutura semelhante para os fantasmas.A única diferença é que ao invés de segurar uma variável global do player na memória, teremos uma fatia de ponteiros para sprites.  Dessa forma podemos atualizar a posição de cada fantasma de uma forma muito eficiente.

Primeiro, a declaração:

```go
var ghosts []*sprite
```

Note que o símbolo `*` que indica que `[]*sprite` é uma slice de **pointers*** (ponteiros) para os objetos sprite.

A seguir, faremos o loading. Na função `loadMaze`, vamos adicionar um novo caso à declaração ao  switch para fazer a manipulação dos símbolos `G` no mapa:

```go
for row, line := range maze {
    for col, char := range line {
        switch char {
        case 'P':
            player = sprite{row, col}
        case 'G':
            ghosts = append(ghosts, &sprite{row, col})
        }
    }
}
```

Por favor, observe o operador do "e comercial" (`&`). Isto significa que ao invés de adicionar um objeto do tipo sprite à slice estamos adicionando um ponteiro a ele.

Go é uma linguagem que possui garbage collector, o que significa que pode-se automaticamente desalocar um pedaço de memória quando ele não é mais usado. Por causa disso, podemos usar apontadores de uma forma muito mais segura do que, por exemplo, em C++. No entanto, também não estamos autorizados a fazer contas com ponteiros em Go. Temos que ter em mente que, um ponteiro em Go, funciona quase como uma referência.

Agora, como estamos manipulando `G`s na função `loadMaze`, precisamos também imprimi-los em `printScreen'. Para isso, basta adicionar o seguinte bloco após o "printing" do player/jogador:

```go
for _, g := range ghosts {
    simpleansi.MoveCursor(g.row, g.col)
    fmt.Print("G")
}
```

## Tarefa 02: Uma IA muito inteligente

Já mencionamos anteriormente que utilizaríamos um gerador de números aleatórios para controlar nossos fantasmas. Isso parece muito mais complexo do que realmente é. Dê uma olhada no código:

```go
func drawDirection() string {
    dir := rand.Intn(4)
    move := map[int]string{
        0: "UP",
        1: "DOWN",
        2: "RIGHT",
        3: "LEFT",
    }
    return move[dir]
}
```

A função `rand.Intn` do pacote `math/rand` gera um número aleatório entre o intervalo `[0, n)`, onde `n` é o parâmetro atribuído à função. (Note que o intervalo é aberto, portanto o `n` não está incluso).

Aqui, estamos utilizando um truque para mapear os números do tipo  inteiro para os movimentos reais utilizando um `mapa`. Um mapa é uma estrutura de dados que mapeia um valor para outro. Ou seja, no caso acima, o mapa `move` mapeia um número inteiro para uma string.

## Tarefa 03: Vamos adicionar algum movimento!

Finalmente, precisamos de uma função que processe o movimento dos fantasmas. A função `moveGhosts` é mostrada abaixo:

```go
func moveGhosts() {
    for _, g := range ghosts {
        dir := drawDirection()
        g.row, g.col = makeMove(g.row, g.col, dir)
    }
}
```

Agora atualize o loop do jogo para chamar a função `moveGhosts`:

```go
// game loop
for {
    // update screen
    printScreen()

    // process input
    input, err := readInput()
    if err != nil {
        log.Println("error reading input:", err)
        break
    }

    // process movement
    movePlayer(input)
    moveGhosts()

    // process collisions

    // check game over
    if input == "ESC" {
        break
    }

    // repeat
}
```

Acabamos! Agora temos fantasmas que se movem! Como é assustador huh?!-_-''''.

[Leve-me ao passo 05!](.../passo05/README.md)
