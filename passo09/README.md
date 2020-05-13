# Passo 09: Buffer "The String Concatenation Slayer"

Nesta lição, você aprenderá a fazer

- Concatenar strings usando um Buffer

## Overview

Nesta lição iremos adicionar suporte para múltiplas vidas à nossa aplicação. Vamos atualizar o código de "collision tracking" para diminuir o número de vidas em detrimento de definir vidas como 0 para o caso de uma colisão ocorrer. Nós também iremos manter a localização da posição inicial do jogador para que o player reinicie o jogo no mesmo lugar, caso ele morra. Finalmente, adicionaremos emojis de jogador ao placar do jogo para monitorar o número de vidas restantes ao invés de mostrar vidas como um valor inteiro.

## Tarefa 01: Criar um Point type e uma struct de update Player para usar o Point type.

Precisamos rastrear a posição inicial do jogador para que possamos reiniciar a sua posição após a colisão com um fantasma. Faremos isso atualizando a estrutura do sprite para incluir as propriedades `startRow` e `startCol`.

```go
type sprite struct {
    row      int
    col      int
    startRow int
    startCol int
}
```

Assim, podemos preencher essas propriedades para o nosso jogador (e fantasmas) na função `loadMaze`:

```go
func loadMaze() error {
    //...omitted for brevity

    for row, line := range maze {
        for col, char := range line {
            switch char {
            case 'P':
                player = sprite{row, col, row, col}
            case 'G':
                ghosts = append(ghosts, &sprite{row, col, row, col})
            case '.':
                numDots++
            }
        }
    }

    return nil
}
```

Note que como temos as propriedades adicionais `startRow` e `startCol`, não podemos mais fazer uma simples comparação para detectar as colisões, pois o jogador nunca começa na mesma posição que um fantasma. Vamos fazer com que essa mudança seja feita na próxima sessão.

## Tarefa 02: Atualizar o número inicial de `vidas`(lives) para ser maior que 1 e reduzir as vidas apósuma colisão com um dos fantasmas.

Como ponto de partida, vamos definir nosso número inicial de vidas para 3

```go
var lives = 3
```

Em seguida, atualizaremos o código que processa as colisões para diminuir o número de vidas em 1 unidade cada vez que ocorrer uma colisão. Finalmente vamos verificar se extrapolamos o número de vidas e reinicializar o emoji do nosso jogador para a posição inicial para assim, reiniciar o jogo.

```go
    // process collisions
    for _, g := range ghosts {
        if player.row == g.row && player.col == g.col {
            lives = lives - 1
            if lives != 0 {
                moveCursor(player.row, player.col)
                fmt.Print(cfg.Death)
                moveCursor(len(maze)+2, 0)
                time.Sleep(1000 * time.Millisecond) //dramatic pause before resetting player position
                player.row, player.col = player.startRow, player.startCol
            }
        }
    }
```

## Tarefa 03: Atualização do placar para mostrar emojis do jogador correspondente ao número de vidas


Anteriormente o número de vidas estava sendo exibido como um número inteiro no placar do jogo. Agora vamos atualizar o placar para exibir o número de vidas com os emojis do jogador. Vamos adicionar uma função `getLivesAsEmoji` para concatenar o número correto de emojis do jogador com base nas vidas restantes no jogo. Esta função cria um buffer e então escreve a string emoji do jogador para o buffer baseado no número de vidas e então retorna esse valor como uma string. Esta função será chamada na última linha da função `printScreen` para atualizar o placar.


```go
func printScreen() {
    //...omitted for brevity

    moveCursor(len(maze)+1, 0)

    livesRemaining := strconv.Itoa(lives) //converts lives int to a string
    if cfg.UseEmoji {
        livesRemaining = getLivesAsEmoji()
    }

    fmt.Println("Score:", score, "\tLives:", livesRemaining)
}

//concatenate the correct number of player emojis based on lives
func getLivesAsEmoji() string{
    buf := bytes.Buffer{}
    for i := lives; i > 0; i-- {
        buf.WriteString(cfg.Player)
    }
    return buf.String()
}
```

Então por que usar um buffer? Acontece que existem outras maneiras de concatenar strings em Go. A opção mais simples seria utilizar apenas o operador `+` para concatenar duas strings:

```go
string1 := "pac"
string2 := "go"
pacgo := string1 + string2 //"pacgo"
```

Para comparação, isto é como a função `getLivesAsEmoji` ficaria se utilizássemos a abordagem de operador `+`.

```go
func getLivesAsEmoji() string {
    emojiString := ""
    for i := lives; i > 0; i-- {
        emojiString = emojiString + cfg.Player
    }
    return emojiString
}
```

Esta versão do `getLivesAsEmoji` seria menos eficiente do que a versão da função que utiliza um buffer. Parte da razão para esta diferença de desempenho deve-se à alocação de memória necessária para a concatenação de strings, como já vimos antes, as strings em Go são imutáveis.

Na versão da função utilizando o operador `+`, há uma operação de alocação de memória acontecendo para cada iteração do loop for. Enquanto para a versão de buffer da função há apenas uma única alocação de memória ocorrendo quando o buffer é inicializado. Um exemplo mais detalhado desta diferença de desempenho é discutido aqui -> [here](https://billglover.me/2019/03/13/learn-go-by-concatenating-strings/)


[Leve-me ao passo 10!](../passo10/README.md)
