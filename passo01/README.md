# Passo 01: Input e Output

Nesta lição, você aprenderá a fazer:

- Ler de um arquivo
- "Imprimir" em uma saída padrão (standard output)
- Tratar com valores de múltiplos retornos
- Tratamento de erros
- Criar e adicionar um elemento a uma slice **
- Range loop over a slice
- Deslocar uma chamada de função
- Log de erros

** Slice em Go pode ser visto como um array elastico ou seja, é pensar em uma visão flexível e dinamicamente dimensionada para os elementos de um array. 

## Overview

Já temos o básico coberto, agora é hora de começar este jogo!

Primeiro, vamos ler os dados do labirinto. Temos um arquivo chamado `maze01.txt` (maze em tradução livre para pt-br significa labirinto ;)) que é basicamente uma representação ASCII do labirinto (você pode abri-lo em um editor de texto se quiser). Dito isso, podemos assumir que:

```
- # representa uma parede
- . representa um ponto
- P representa o jogador (pac-man)
- G representa os fantasmas (inimigos)
- X representa as pílulas de stamina
```

Nossa primeira tarefa consiste em inserir esta representação ASCII do labirinto para uma slice de strings e depois imprimi-la para na tela. Parece simples, certo? E é mesmo!


## Tarefa 01: Inserir o Labirinto

Let's start by reading the `maze01.txt` file.

Primeiro passo: começar lendo o arquivo `maze01.txt`.

Vamos utilizar a função `Open` do package `os` para abri-lo, e também um scanner object do buffered IO package (`bufio`) para lê-lo na memória (para uma variável global chamada `maze`). Por último, precisamos liberar o file handler chamando a função close do package `os` (`os.Close`).

Tudo isso se unem e formam o código abaixo:


```go
var maze []string

func loadMaze(file string) error {
    f, err := os.Open()
    if err != nil {
        return err
    }
    defer f.Close()

    scanner := bufio.NewScanner(f)
    for scanner.Scan() {
        line := scanner.Text()
        maze = append(maze, line)
    }

    return nil
}
```

Agora vamos debugar isso tudo e ver o que está acontecendo.

Por favor note que você precisa importar os pacotes `bufio` e `os` como mostrado abaixo:

```go
import "bufio"
import "os"
```

Alternativamente, como você já possui uma importação (`fmt`), você pode adicioná-la como uma lista:

```go
import (
    "bufio"
    "fmt"
    "os"
)
```

A função `os.Open()` retorna um par de valores: um arquivo e um erro. Retornar múltiplos valores de uma função é um padrão comum em Go, especialmente para retornar erros.

```go
f, err := os.Open(file)
```

O operador `:=` é um operador de atribuição, que possui a propriedade especial de definir automaticamente o tipo da(s) variável(s) com base no(s) que foi declarado do lado direito, ou seja, ele declara e atribui ao mesmo tempo.

Tenha em mente que Go é uma linguagem fortemente tipada, mas esse recurso legal nos poupa o trabalho de especificar o tipo quando é possível fazer inferências.

No caso acima, Go identifica automaticamente o tipo para ambas as variáveis `f` e `err`.

Quando uma função retorna um erro é um padrão comum verificar se o erro ocorre imediatamente após o fim da execução da função:

```go
    f, err := os.Open(file)
    if err != nil {
        // do something with err  //coloque aqui algo com err
        log.Print("...")
        return
    }
```

Nota: É uma boa prática manter o "caminho feliz" alinhado à esquerda, e o caminho triste à direita (ou seja, terminando a função antes).

`nil`em Go significa que nenhum valor é atribuído a uma variável.

A condição`if` executa uma instrução apenas se ela for verdadeira. Opcionalmente podemos ter uma cláusula de inicialização como a condição `for`, e uma cláusula `else` que executa caso a condição seja falsa. Por favor tenha em mente que o escopo da variável criada será apenas o corpo do comando if. Exemplo:

```go
// optional initialization clause
if foo := rand.Intn(2); foo == 0 {
    fmt.Print(foo) // foo is valid here // foo e aceito aqui
} else {
    fmt.Print(foo) // and here  // e aqui
}
// but you can't use foo here!  // mas nao pode ser colocado aqui
```

Outro aspecto interessante do código `loadMaze` é a utilização da palavra-chave `defer`. Ela basicamente determina que a função deve ser chamada após o `defer` ao final da função atual. É muito útil para fins de manutenção e clareza e neste caso estamos utilizando-a para fechar o arquivo que acabamos de abrir:

```go
func loadMaze(file) error {
    f, err := os.Open(file)
    // omitted error handling
    defer f.Close() // puts f.Close() in the call stack

    // rest of the code

    return nil
    // f.Close is called implicitly
}
```

A próxima parte do código apenas lê o arquivo, linha por linha, e o anexa à maze slice:

```go
    scanner := bufio.NewScanner(f)
    for scanner.Scan() {
        line := scanner.Text()
        maze = append(maze, line)
    }
```

Um scanner é uma maneira muito conveniente de ler um arquivo. O `scanner.Scan()` retornará sempre verdadeiro enquanto houver algo para ser lido do arquivo, e o `scanner.Text()` retornará a próxima linha de entrada.

A função `append` embutida é responsável por adicionar um novo elemento à slice `maze`.

## Tarefa 02: Imprimindo na Tela

Assim que tivermos o arquivo do maze todo carregado na memória, precisamos imprimi-lo na tela.

Uma maneira de fazer isso, é iterando sobre cada entrada na slice `maze` e por conseguinte, imprimindo-a. Isto pode ser feito de maneira conveniente com o operador do 'range':

```go
func printScreen() {
    for _, line := range maze {
        fmt.Println(line)
    }
}
```

Note que estamos utilizando o operador de atribuição `:=` para inicializar dois valores: o underscore (_) e a variável `line`. O underscore é apenas um espaço reservado para onde o compilador esperaria um nome de uma variável. Utilizando o underscore significa que estamos ignorando esse valor.

No caso do operador `range`, o primeiro valor de retorno é o índice do elemento, a partir de zero. O segundo valor de retorno é o próprio valor do elemento.

Se não escrevêssemos o caractere de underscore (_) para ignorar o primeiro valor, o operador `range` retornaria apenas o índice do elemento (e não retornadaria o valor). Por exemplo:

```go
for idx := range maze {
    fmt.Println(idx)
}
```

Como neste caso, precisamos apenas nos preocupamos com o conteúdo do elemento e não com o seu índice, podemos ignorar o índice com segurança, atribuindo-o ao underscore.

## Tarefa 03: Atualizando o loop do jogo

Agora que temos ambas as funções:`loadMaze` e `printScreen`, devemos atualizar a função `main` para inicializar o labirinto e imprimi-lo no loop do jogo. Veja como no código abaixo:

```go
func main() {
    // initialise game   // inicia o jogo

    // load resources    // carrega os recursos
    err := loadMaze("maze01.txt")
    if err != nil {
        log.Println("failed to load maze:", err)
        return
    }

    // game loop         // loop do jogo
    for {
        // update screen           //atualiza a tela
        printScreen()

        // process input           // processamento da entrada

        // process movement        // processamento do movimento

        // process collisions      // processamento das colisoes

        // check game over        // verifica o fim do jogo

        // Temp: break infinite loop  // quebra o loop infinito
        break

        // repeat                      // repeticao 
    }
}
```

Como sempre estamos mantendo o caminho feliz na esquerda, então se a função `loadMaze` falhar, usamos `log.Println` para loga-lo e depois `return` para encerrar a execução do programa. Como estamos utilizando um novo pacote, `log`, por favor certifique-se de que ele seja adicionado à seção/lista de importação:

```go
import (
    "bufio"
    "fmt"
    "log"
    "os"
)
```

Algumas IDEs, como a `vscode`, podem configurar isso automaticamente para você.

Nota: pode-se também utilizar o `log.Fatalln` para se obter o mesmo efeito, mas precisamos ter certeza que qualquer chamada diferida seja executada antes de sair da função `main`, e funções do conjunto `log.Fatal` ignoram chamadas de função diferidas chamando `os.Exit(1)` internamente. Por hora, ainda não temos nenhuma chamada diferida na função principal, mas iremos adicionar uma no próximo capítulo.

Agora que terminamos as modificações do loop do jogo, podemos rodar o programa com o comando 'go run' ou compilá-lo com 'go build' e executar como um programa standalone.

```sh
go run main.go
```

Você deverá ver o labirinto impresso no terminal.

[Leve-me ao passo 02!](.../passo02/README.md)
