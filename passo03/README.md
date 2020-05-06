# Passo 03: Adicionando Movimento

Nesta lição, você aprenderá a fazer: 

- Criar uma estrutura
- Usar um switch statement
- Manipular as setas do teclado
- Usar named return values

## Overview

Neste ponto, nós temos um labirinto, e tambem ja podemos sair do jogo graciosamente... mas nada muito excitante esta acontecendo, certo? Então vamos apimentar um pouco essa coisa e adicionar algum movimento!

Neste passo vamos inserir o personagem (player) e viabilizar seus movimento através das setas do teclado.

## Tarefa 01: Rastreando a posição do jogador

O primeiro passo em nossa jornada é criar uma variável para armazenar os dados do jogador. Como vamos rastrear as coordenadas em 2D (linha e coluna), precisamos definir uma estrutura para guardar essas informações:

```go
type sprite struct {
    row int
    col int
}

var player sprite
```

Estamos também definindo o jogador como uma variável global, apenas por uma questão de simplicidade.

Em seguida, precisamos capturar a posição do jogador assim que carregarmos o labirinto, na função `loadMaze`:

```go
// traverse each character of the maze and create a new player when it locates a `P`
for row, line := range maze {
    for col, char := range line {
        switch char {
        case 'P':
            player = sprite{row, col}
        }
    }
}
```

Note que desta vez estamos utilizando a forma completa do operador `range`, pois estamos interessados em qual linha e coluna localizaremos o jogador.

Aqui está o completo `loadMaze` apenas como referência:

```go
func loadMaze(file string) error {
    f, err := os.Open(file)
    if err != nil {
        return err
    }
    defer f.Close()

    scanner := bufio.NewScanner(f)
    for scanner.Scan() {
        line := scanner.Text()
        maze = append(maze, line)
    }

    for row, line := range maze {
        for col, char := range line {
            switch char {
            case 'P':
                player = sprite{row, col}
            }
        }
    }

    return nil
}
```

---

### Opcional: Uma nota sobre visibilidade

Nós estamos mantendo as coisas simples aqui, para o propósitoe simplificar do tutorial. Como tudo é um único arquivo, não estamos levando em conta a visibilidade das variáveis, ou seja, se elas são públicas ou privadas.

No entanto, Go tem uma mecânica interessante no que diz respeito à definição de visibilidade. Ao invés de ter uma palavra-chave pública, ele considera público todo símbolo cujo nome começa com uma letra maiúscula. Por outro lado, se um nome começa com uma letra minúscula, é um símbolo privado.

É por isso que todo nome de função de biblioteca que usamos até agora começa com uma letra maiúscula. É também por este motivo que sua IDE pode reclamar da falta de comentários se você definir qualquer variável, função ou tipo com um caractere inicial em maiúsculas. No idioma Go, símbolos públicos devem ser sempre comentados, pois são extraídos posteriormente para se tornarem a documentação do pacote.

Neste caso em particular, estamos utilizando símbolos em letras minúsculas para todas as nossas variáveis, tipos e funções, uma vez que não faz sentido exportar um símbolo do pacote `main`.

---

## Tarefa 02: Manuseio de teclas do tipo seta

A seguir, precisamos modificar o `readInput` para lidar com as setas do teclado:

```go
if cnt == 1 && buffer[0] == 0x1b {
    return "ESC", nil
} else if cnt >= 3 {
    if buffer[0] == 0x1b && buffer[1] == '[' {
        switch buffer[2] {
        case 'A':
            return "UP", nil
        case 'B':
            return "DOWN", nil
        case 'C':
            return "RIGHT", nil
        case 'D':
            return "LEFT", nil
        }
    }
}
```

A seqüência de saída para as setas é composta por 3 bytes de comprimento, começando com `ESC+[` e depois uma letra de A a D.

Agora precisamos de uma função para controlar o movimento:

```go
func makeMove(oldRow, oldCol int, dir string) (newRow, newCol int) {
    newRow, newCol = oldRow, oldCol

    switch dir {
    case "UP":
        newRow = newRow - 1
        if newRow < 0 {
            newRow = len(maze) - 1
        }
    case "DOWN":
        newRow = newRow + 1
        if newRow == len(maze) {
            newRow = 0
        }
    case "RIGHT":
        newCol = newCol + 1
        if newCol == len(maze[0]) {
            newCol = 0
        }
    case "LEFT":
        newCol = newCol - 1
        if newCol < 0 {
            newCol = len(maze[0]) - 1
        }
    }

    if maze[newRow][newCol] == '#' {
        newRow = oldRow
        newCol = oldCol
    }

    return
}
```

Nota: Se você está acostumado com o switch statement em outros idiomas, por favor, tenha cuidado, pois em Go isto é um implícito`break` após cada condição de `case`. Portanto, não precisamos interromper explicitamente após cada bloco. ISe quisermos entrar no próximo `case`, podemos usar a palavra-chave `fallthrough`.

A função acima utiliza o `named return values` para retornar a nova posição (`newRow` e `newCol`) (linha e coluna novas respectivamente) após a mudança. Basicamente a função "tenta"  primeiro, o movimento, e se por acaso a nova posição atingir uma parede (`#`), o movimento é cancelado.

Ela também trata da situação em que a personagem move-se para fora do intervalo do labirinto, tendo como consequência, o seu reaparecimento do lado oposto.

E, a última peça desse "quebra-cabeça" sobre o movimento é definir a função que irá fazer o jogador se mov:

```go
func movePlayer(dir string) {
    player.row, player.col = makeMove(player.row, player.col, dir)
}er
```

## Tarefa 03: Atualizando o labirinto

Temos toda a lógica de movimento no lugar, mas precisamos fazer com que a tela mostre corretamente essa lógica. Iremos então, refatorar a função `printScreen` para imprimir apenas as coisas que queremos imprimir, ao invés de todo o mapa.

Isso nos dará mais controle, permitindo-nos colocar o jogador em uma posição arbitrária com a função de `moveCursor`. Veja como, com o código abaixo:

```go
func printScreen() {
    simpleansi.ClearScreen()
    for _, line := range maze {
        for _, chr := range line {
            switch chr {
            case '#':
                fmt.Printf("%c", chr)
            default:
                fmt.Print(" ")
            }
        }
        fmt.Println()
    }

    simpleansi.MoveCursor(player.row, player.col)
    fmt.Print("P")


    // Move cursor outside of maze drawing area
    simpleansi.MoveCursor(len(maze)+1, 0)
}
```

Observação: Por enquanto, estamos ignorando qualquer coisa que não seja uma parede ou o jogador.

## Tarefa 04: Animação!

Finalmente, precisamos chamar a função `movePlayer` para loop do jogo:

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

    // process collisions

    // check game over
    if input == "ESC" {
        break
    }

    // repeat
}
```

Vamos continuar?!

[Leve-me ao passo 04!](.../passo04/README.md)
