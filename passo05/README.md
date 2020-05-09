# Passo 05: Game over man, game over!

Nesta lição, você aprenderá a fazer:

- Usar fallthrough statement em switch blocks
- Trabalhar com slices

## Overview

Estamos quase lá. Nós temos tanto o movimento do jogador quanto o movimento dos fantasmas. Mas os nossos fantasmas ainda são inofensivos.

É hora de acrescentar algum perigo a este jogo. Além disso, com grandes riscos devem vir grandes recompensas, então também estaremos enfrentando a condição de vitória do jogo, limpando o tabuleiro de todos os seus pontos.

## Tarefa 01: Preparação

Para a condição "game win" (vitória do jogo), precisamos acompanhar quantos pontos temos no tabuleiro, e declarar vitória quando este número for igual a zero.

Vamos precisar de um mecanismo para retirar os pontos do tabuleiro uma vez que o jogador fique em cima deles. Também faremos o acompanhamento de um placar para exibir para o jogador.

Para o jogo sobre o cenário, vamos dar ao jogador uma vida e, quando um fantasma o atingir, essa vida é zerada. Então testamos para zero vidas no loop do jogo para terminar o jogo. (Deverá ser bem simples adicionar suporte para várias vidas, mas faremos isso em um passo posterior).

Adicione as seguintes variáveis globais para monitorar todos os itens mencionados acima:

```go
var score int
var numDots int
var lives = 1
```
A seguir, precisamos de inicializar a variável `numDots` em `loadMaze`. Apenas precisamos de um novo caso no switch que trate do `.` caracter':

```go
for row, line := range maze {
    for col, char := range line {
        switch char {
        case 'P':
            player = sprite{row, col}
        case 'G':
            ghosts = append(ghosts, &sprite{row, col})
        case '.':
            numDots++
        }
    }
}
```

Agora precisamos atualizar a função `printScreen` para imprimir os pontos novamente. Este é um caso interessante para a declaração `fallthrough`:

```go
func printScreen() {
    simpleansi.ClearScreen()
    for _, line := range maze {
        for _, chr := range line {
            switch chr {
            case '#':
                fallthrough
            case '.':
                fmt.Printf("%c", chr)
            default:
                fmt.Print(" ")
            }
        }
        fmt.Println()
    }
    // rest of the function omitted for brevity...
}
```

Finalmente, ao final da função `printScreen` vamos adicionar nossa pontuação e nosso painel de vidas:

```go
func printScreen() {
    // code omitted...

    // print score
    simpleansi.MoveCursor(len(maze)+1, 0)
    fmt.Println("Score:", score, "\tLives:", lives)
}
```

## Task 02: Game over

Processar o game over é muito simples. A cada vez que o jogador e o fantasma estiverem na mesma posição, o jogador/player deve morrer. Vamos adicionar o código que detecta isso ao loop do jogo. Também modificaremos a condição de abandono do jogo adicionando `lives <= 0` and `numDots == 0`:

```go
// game loop
for {
    // code ommited...

    // process collisions
    for _, g := range ghosts {
        if player == *g {
            lives = 0
        }
    }

    // check game over
    if input == "ESC" || numDots == 0 || lives <= 0 {
        break
    }

    // repeat
}
```

Observe que a maneira mais verbosa de verificar a posição do jogador é `player.row == g.row && player.col == g.col`, mas como tanto o jogador quanto o fantasma são sprites pode-se usar uma simples comparação `player == *g`. Ainda precisamos desreferenciar o `g` porque não podemos comparar os tipos ponteiro e não ponteiro.

## Tarefa 03:  Game win (Jogo Ganho)

Agora só nos falta o código para remover os pontos do jogo e incrementar a pontuação.

Vamos adicionar este código à função `movePlayer`:

```go
func movePlayer(dir string) {
    player.row, player.col = makeMove(player.row, player.col, dir)
    switch maze[player.row][player.col] {
    case '.':
        numDots--
        score++
        // Remove dot from the maze
        maze[player.row] = maze[player.row][0:player.col] + " " + maze[player.row][player.col+1:]
    }
}
```

O código acima funciona da seguinte forma: primeiro fazemos o movimento. Depois verificamos qual personagem está no mesmo lugar que o jogador. Se for um ponto, diminuímos o número total de pontos (`numDots`), incrementamos a pontuação e removemos o ponto do tabuleiro.

Vale a pena mencionar que as strings em Go são imutáveis. Não poderíamos simplesmente atribuir um espaço a uma determinada posição em uma string. Isso não funcionaria.

Assim, estamos usando aqui um truque ao criar uma nova string composta por duas slices da string original. As duas slices juntas fazem exatamente a mesma que a string original, exceto por uma posição, que nós substituímos por um espaço.

Para mais informações sobre as slices, por favor veja [here](https://blog.golang.org/go-slices-usage-and-internals).

Agora temos tanto o "game win" ( vitória do jogo) quanto o "game over"  (final do jogo) sobre as condições. Tente construir um mapa com apenas alguns pontos e teste a vitória no jogo. Colida com um fantasma para testar o game over. 

Estamos fazendo progresso!

(Dica: o maze01.txt na pasta passo05 possui apenas 3 pontos).

[Leve-me ao passo 06!](.../passo06/README.md)
