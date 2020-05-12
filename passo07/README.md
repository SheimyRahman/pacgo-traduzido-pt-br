# Passo 07: Finalmente, emojis!

Nesta lição, você aprenderá a fazer: 

- Carregar um arquivo json
- Imprimir emojis (na tela)!!!

## Overview

Então, nós conseguimos criar um jogo apropriado no terminal. Mas eu prometi os emojis, onde eles estão? Bem, a hora deles chegou, finalmente!

Neste passo vamos criar um arquivo chamado `config.json`. Neste arquivo nós armazenaremos os mapeamentos para cada símbolo que utilizarmos em nosso jogo. Nos jogos 2D, normalmente chamamos as peças que se movem de "sprites".

Como a maioria dos terminais atualmente suporta unicode, podemos usar emojis como nossos sprites sem a necessidade de recorrer a qualquer biblioteca gráfica.

O arquivo `config.json' criado deve ser parecido com este:

```json
{
    "player": "😋",
    "ghost": "👻",
    "wall": "🌵",
    "dot": "🧀",
    "pill": "🍹",
    "death": "💀",
    "space": "  ",
    "use_emoji": true

    // optional player - "😷" adding a mask because of COVID-19
}
```

Este é o mapeamento padrão, mas sinta-se à vontade para brincar com toda a paleta de emojis. Nós temos infinitas possibilidades!

Um aspecto importante sobre o arquivo de configuração é a configuração `use_emoji`. Nós utilizamos esta flag para sinalizar para o jogo quando estivermos utilizando emojis. Isto é necessário pois os emojis geralmente utilizam mais de um caractere  na tela (a maioria deles utiliza 2).

Usando essa flag podemos ter paths de código alternativos que fazem ajustes para compensar essa diferença. Caso contrário o labirinto ficaria distorcido.

## Tarefa 01: Carregar um arquivo json

Go tem suporte para carregamento de arquivos json na sua biblioteca padrão.

Primeiro precisamos definir uma estrutura para guardar os dados no json. O texto entre os backticks (\`) é chamado de `struct tag`. Ela é utilizada pelo decodificador json para saber qual campo da estrutura corresponde a cada campo no arquivo json.

```go
// Config holds the emoji configuration
type Config struct {
    Player   string `json:"player"`
    Ghost    string `json:"ghost"`
    Wall     string `json:"wall"`
    Dot      string `json:"dot"`
    Pill     string `json:"pill"`
    Death    string `json:"death"`
    Space    string `json:"space"`
    UseEmoji bool   `json:"use_emoji"`
}

var cfg Config
```

Note que utilizamos elementos públicos para a struct`Config`. Isso é necessário para que o decodificador json funcione corretamente.

O código abaixo faz o parse do json e o depois o armazena na variável global `cfg`.

```go
func loadConfig(file string) error {
    f, err := os.Open(file)
    if err != nil {
        return err
    }
    defer f.Close()

    decoder := json.NewDecoder(f)
    err = decoder.Decode(&cfg)
    if err != nil {
        return err
    }

    return nil
}
```

Agora adicione a chamada `loadConfig` na parte de inicialização da função main, após o `loadMaze`:

```go
err = loadConfig("config.json")
if err != nil {
    log.Println("failed to load configuration:", err)
    return
}
```

## Tarefa 02: Ajustando o deslocamento horizontal

Precisamos criar uma função personalizada que chamaremos de `moveCursor` para corrigir o deslocamento horizontal quando a flag emoji estiver ativada:

```go
func moveCursor(row, col int) {
    if cfg.UseEmoji {
        simpleansi.MoveCursor(row, col*2)
    } else {
        simpleansi.MoveCursor(row, col)
    }
}
```

Certifique-se de substituir todas as chamadas para a`simpleansi.MoveCursor` por chamadas para a nova função `moveCursor'`(exceto as que estão dentro da nova função).

O redimensionamento do valor `col` por 2,  irá garantir o posicionamento correto de cada caractere. E também nos trará o efeito colateral positivo de fazer o labirinto parecer maior.

## Task 03: Replace hardcoded characters with configuration

A parte final é substituir os caracteres que estão hardcoded pelos seus equivalentes na função `printScreen` de configuração. Também vamos utilizar a função `simpleansi.WithBlueBackground` para mudar a cor das paredes para torná-la mais parecidas com a do jogo original.

```go
func printScreen() {
    simpleansi.ClearScreen()
    for _, line := range maze {
        for _, chr := range line {
            switch chr {
            case '#':
                fmt.Print(simpleansi.WithBlueBackground(cfg.Wall))
            case '.':
                fmt.Print(cfg.Dot)
            default:
                fmt.Print(cfg.Space)
            }
        }
        fmt.Println()
    }

    moveCursor(player.row, player.col)
    fmt.Print(cfg.Player)

    for _, g := range ghosts {
        moveCursor(g.row, g.col)
        fmt.Print(cfg.Ghost)
    }

    moveCursor(len(maze)+1, 0)
    fmt.Println("Score:", score, "\tLives:", lives)
}
```

## Tarefa 04: Game over and pills

Como um bônus adicional, vamos adicionar um sprite de game over dentro da condição de game over.

```go
// check game over
if numDots == 0 || lives == 0 {
    if lives == 0 {
        moveCursor(player.row, player.col)
        fmt.Print(cfg.Death)
        moveCursor(len(maze)+2, 0)
    }
    break
}
```

Além disso, vamos adicionar o código para tratar a "power up pill" como um item que vale mais pontos, ou seja, como um espaço reservado para a atual mecânica de power up. Estamos fazendo dessa maneira agora apenas para ter uma sensação de um jogo completo, mas vamos implementar a mecânica de power up mais apropriada em um passo mais adiante.

```go
func movePlayer(dir string) {
    player.row, player.col = makeMove(player.row, player.col, dir)

    removeDot := func(row, col int) {
        maze[row] = maze[row][0:col] + " " + maze[row][col+1:]
    }

    switch maze[player.row][player.col] {
    case '.':
        numDots--
        score++
        removeDot(player.row, player.col)
    case 'X':
        score += 10
        removeDot(player.row, player.col)
    }
}
```

Uma coisa interessante sobre o código acima é que estamos definindo uma função inline para fazer a remoção tanto do ponto como do X do jogo quando temos uma colisão. Poderíamos também ter repetido o código, mas isso o torna mais legível e sustentável.

E agora, temos emojis! Quão maravilhoso isso é?! :)

[Leve-me ao passo 08!](.../passo08/README.md)