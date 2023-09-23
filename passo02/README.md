# Passo 02: Gerenciamento da Entrada do Jogador

Nesse passo, você aprenderá:

- Trabalhar com diferentes modos de terminais
- Chamada de comandos externos a partir do código Go
- Enviar sequências de escape para o terminal
- Ler a partir de standards input (entradas padrões)
- Criar uma função que retorna múltiplos valores

## Overview

No último passo aprendemos como imprimir algo para uma saída padrão. Agora vamos aprender a ler a partir da entradas padrões.

Neste jogo iremos processar um conjunto restrito de movimentos, são eles: para cima, para baixo, para a esquerda e para a direita. Além desses, a única outra chave que estaremos usando é a de escape, para que o jogador possa sair do jogo com elegância. Os movimentos serão configurados para as setas do teclado.

Este passo irá mostrar como configurar a tecla Esc e a seguir, veremos como processar as teclas das setas no passo 03.

Mas, antes de entrarmos na implementação, precisamos saber um pouco sobre terminal modes.

## Introdução a terminal modes

Terminals podem ser executados em três modos: [modes](https://en.wikipedia.org/wiki/Terminal_mode):

1. Cooked Mode
2. Cbreak Mode
3. Raw Mode

O cooked mode é o que estamos acostumados a usar. Neste modo cada entrada que o terminal recebe é pré-processada, ou seja, o sistema intercepta caracteres especiais para dar-lhes um significado especial.

Nota: Os caracteres especiais incluem backspace, delete, Ctrl+D, Ctrl+C, teclas das setas e assim por diante...

O raw mode é o oposto: os dados são repassados como estão, sem qualquer tipo de pré-processamento.

O cbreak mode é o meio-termo. Alguns caracteres são pré-processados e outros não. Por exemplo, Ctrl+C tem como resultante uma interrupção do programa, mas as setas são repassadas para o programa como estão.

Usaremos o modo cbreak para nos permitir gerenciar as seqüências de escape correspondentes às teclas de saída e das setas.

## Tarefa 01: Habilitando o Modo Cbreak

Para habilitar o modo cbreak vamos chamar um comando externo que controla o comportamento do terminal, o comando 'stty'. Também vamos desabilitar o echo do terminal para não" poluir" a tela com a saída dos resultados dos pressionamentos de teclas.

Aqui está a definição do nosso `init':

```go
func initialise() {
    cbTerm := exec.Command("stty", "cbreak", "-echo")
    cbTerm.Stdin = os.Stdin

    err := cbTerm.Run()
    if err != nil {
        log.Fatalln("unable to activate cbreak mode:", err)
    }
}
```

Atenção! Você precisará adicionar a área de importação "os/exec" se sua IDE não estiver configurada para adicioná-la automaticamente.

A função `log.Fatalln' finalizará o programa após a impressão do log, em caso de erro. Esta ação é importante neste caso pois sem o modo cbreak o jogo fica inoperante. Como esta é a primeira função que chamaremos em nosso programa, não nos preocupamos em pular nenhuma deferred calls.

## Tarefa 02: Restaurando o Cooked Mode

Restaurar o cooked mode é um processo bastante simples. É o mesmo que ativar o modo cbreak, mas com as bandeiras invertidas:

```go
func cleanup() {
    cookedTerm := exec.Command("stty", "-cbreak", "echo")
    cookedTerm.Stdin = os.Stdin

    err := cookedTerm.Run()
    if err != nil {
        log.Fatalln("unable to restore cooked mode:", err)
    }
}
```

Agora precisamos chamar ambas as funções na função `main`:

```go
func main() {
    // initialise game
    initialise()
    defer cleanup()

    // load resources
    // ...
```

## Tarefa 03: Lendo a partir da Stdin

O processo de leitura da entrada padrão (standard input) consiste em chamar a função `os.Stdin.Read` com um determinado buffer de leitura.

Então, `os.Stdin.Read` retorna dois valores: o número de bytes lidos e um valor de erro. Veja abaixo como funciona o código da função `readInput`:


```go
func readInput() (string, error) {
    buffer := make([]byte, 100)

    cnt, err := os.Stdin.Read(buffer)
    if err != nil {
        return "", err
    }

    if cnt == 1 && buffer[0] == 0x1b {
        return "ESC", nil
    }

    return "", nil
}
```

A função `make` é uma [função embutida](https://golang.org/pkg/builtin/#make) que aloca e inicializa objetos. É utilizada apenas para slices, maps e channels. Neste caso estamos criando um array de bytes com tamanho 100 e retornando uma fatia que aponta para ela.

Após o procedimento de tratamento de erro usual (estamos apenas passando o erro para cima na stack de chamadas), vamos testar se nós lemos apenas um byte e se esse byte,  é a chave de escape. (0x1b é o código hexadecimal que representa Esc).

Retornamos "ESC" se a tecla Esc for pressionada ou se tivermos uma string vazia.

Agora você pode se perguntar por que alocar um buffer de 100 bytes, ou por que testar e contar uma quantidade exata de um byte...

E se o buffer de repente tivesse 5 elementos e um deles for a chave Esc? Não deveríamos nos preocupar em como processá-la? Será que essa tecla se perderá no processamento?

A resposta simples é que não devemos nos importar. Por favor, tenha em mente que isto é um jogo. Dependendo da velocidade de processamento e do tamanho do seu buffer de teclado, se processássemos eventos sequencialmente poderíamos introduzir defasagem de movimento, ou seja, tendo uma fila de teclas de setas que ainda não foram processadas.

Como estamos lendo a entrada em um loop, não há nenhum problema em colocar todos os comandos de teclas pressionadas em uma fila e apenas focar no último. Isso fará o jogo responder melhor do que se nos preocupássemos com cada uma das teclas pressionadas.

## Tarefa 04: Atualizando o Game Loop

Agora é hora de atualizar o loop do jogo para fazer a função `readInput` ser chamada a cada iteração. Por favor note que se ocorrer um erro, precisamos interromper o loop também.

```go
// process input
input, err := readInput()
if err != nil {
    log.Print("error reading input:", err)
    break
}
```

Finalmente, podemos nos livrar dessa declaração permanente de `break` e começar a testar para quando for pressionada a tecla "ESC".

```go
if input == "ESC" {
    break
}
```

## Tarefa 05: Limpando a Tela

Como agora temos um loop de jogo apropriado, precisamos limpar a tela após cada loop para que tenhamos uma tela em branco para desenhar na próxima iteração. Para isso, vamos utilizar algumas "escape sequences" especiais.

[Escape sequences](https://en.wikipedia.org/wiki/ANSI_escape_code#Escape_sequences) são chamados assim porque começam com o caractere ESC (0x1b) seguido por um ou mais caracteres. Esses caracteres funcionam como comandos para o emulador no terminal.

Na verdade você não precisará se preocupar com as seqüências dessas técnicas que vamos utilizar, pois vamos importar outro pacote chamado `simpleansi` que faz o trabalho para nós:

```go
import "github.com/danicat/simpleansi"
```

---
### Nota sobre pacotes externos

Desta vez não estamos importando um pacote da biblioteca padrão, mas sim um pacote externo. Se você olhar para a [implementação](https://github.com/danicat/simpleansi) do `simpleansi`, você perceberá que toda função começa com uma letra maiúscula, como `ClearScreen` ou `MoveCursor`.

Isso é importante em Go porque a capitalização de uma palavra define se essa função ou variável tem um escopo **público** ou **privado***.

Palavras que começam com um caracteres minúsculos são privadas para o pacote que o define, e palavras que começam com um caractere maiúsculo são públicas. Isso pode ser confuso para pessoas vindas de outros idiomas como java, mas se você seguir convenções de nomenclatura como "classes (structs) sempre começam com uma letra maiúscula" você pode acabar tornando inadvertidamente públicos todos os tipos no seu código, o que provavelmente não é o que você deseja.

---

Vamos atualizar a função printScreen para chamar o `simpleansi.ClearScreen` antes de fazer o printing, então teremos a certeza de estar utilizando uma tela em branco em cada frame:

```go
func printScreen() {
    simpleansi.ClearScreen()
    for _, line := range maze {
        fmt.Println(line)
    }
}
```

Agora, execute o jogo novamente e tente clicar na tecla `ESC`.

Por favor note que se você clicar em Ctrl+C sem querer o programa irá terminar sem chamar a função de limpeza, assim você não poderá ver o que você está digitando no terminal (por causa da bandeira `-echo`).

Se você entrar nessa situação, encerre o terminal e reabra-o ou simplesmente execute o jogo novamente e saia elegantemente usando a tecla `ESC`.

[Leve-me para o passo 03!](../passo03/README.md)