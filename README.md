# Pac Go - Traduzido para Pt-br

Traduzido por @Sheimy_Rahman | https://github.com/SheimyRahman

## Um clone do jogo do pac-man escrito em Go (com emojis!)

![screenshot](./screenshot.jpg)

## Introdução

Pessoas, sejam Bem Vindas ao Pac Go! Este projeto é um tutorial para apresentar a [linguagem de programação Go](https://golang.org).

### Por que um novo tutorial?

Nós temos vários tutoriais ótimos por aí, mas grande ideia deste tutorial é fazer algo que seja diferente e divertido enquanto se aprende Go. Sim, diariamente precisamos entender, ter como base e saber fazer APIs e CRUDs, mas quando queremos aprender algo novo, por que não aprender através do desenvolvimento de um jogo?

Go é uma das linguagens conhecidas por tornar a programação algo divertido novamente, e o que poderia ser mais divertido do que escrever um jogo? Achou interessante? Ótimo, vamos seguir em frente!

Vamos codificar um clone do Pac Man para o terminal. Ao escrever um jogo você será submetido a vários critérios interessantes que são um terreno fértil para que você possa explorar muitas características da linguagem, incluindo entrada, saída, funções, apontadores, concorrência e um pouco de matemática.

Você aprenderá também um pouco mais sobre o terminal e suas sequências mágicas de atalhos.

### Palestras em Conferências

Caso você queira dar uma olhada nesse tutorial em formato de palestra antes de tentar sozinho (cerca de 25 minutos), você pode acessar os links abaixo:

1. [Google Cloud Next UK '19](https://cloud.withgoogle.com/next/uk/sessions?session=DZ224), Londres, Reino Unido (21 de novembro de 2019)
1. [London Gophers](https://youtu.be/SM8LTMnB4x0), Londres, Reino Unido
1. [GoWayFest 3.0](https://youtu.be/0qvW4kIlS8I), Minsk, Belarus
1. [GothamGo 2019](https://youtu.be/GH0DlCKTppE), New York City, NY, EUA

### Contribuição

Este projeto é open source sob a licença do MIT, o que significa que você é basicamente livre para fazer o que quiser com ele, apenas me dê os devidos créditos Ok? :)

Se você quiser contribuir, basta sugerir/criar uma issue e/ou enviar um pull request.

Se você está procurando por inspiração você pode navegar pelas issues abertas ou então dar uma olhada na lista de coisas a serem feitas [TODO](TODO.md). 

Obs: Tudo na lista TODO deve ser planejado como um novo passo no tutorial, a menos que seja assinalado o contrário (exceção).

### License

See [LICENSE](LICENSE).

### Contatando a Autora

Se você tiver alguma dúvida, por favor entre em contato comigo em [daniela.petruzalek@gmail.com](mailto:daniela.petruzalek@gmail.com). Também estou no Twitter como [@danicat83](https://twitter.com/danicat83).

## Vamos Começar

### Pré-requisitos

É recomendado que se tenha:
- Conhecimento básico de como funcionam as linguagens de programação, pois não estaremos cobrindo o básico
- Conhecimento básico de terminais (saber utilizar aplicações de linha de comando)

-> Dica da tradutora: Você pode aprender um pouquinho dessas coisas assistindo aos vídeos (talks) desse tutorial.

Claro, se você não tem o acima mencionado, mas é um espírito curioso e quer tentar de qualquer forma, por favor sinta-se livre para fazê-lo.


### Aviso de Compatibilidade!!!

Este tutorial foi testado em ambos ambientes **Linux** e **Mac OS X***. Para ambientes Windows você talvez precise instalar um emulador de terminal, como [Git BASH](https://gitforwindows.org/).

Por favor, observe que como este código depende do terminal para renderizar o jogo, ele pode produzir resultados diferentes para configurações de diferentes de terminais.

Se você tiver algum problema, sinta-se à vontade para adicionar uma observação para que assim, possamos encontrar uma solução adequada, nomeando tanto o seu SO quanto os nomes e versões dos terminais.

**Note:** É um problema conhecido que a janela do terminal no VS Code não renderiza o jogo corretamente neste momento.


### Configuração

In order to start, make sure you have Go installed in your system.

If you don't, please follow the instructions in [golang.org](https://golang.org)

### How to use this tutorial

Para começar, verifique se você tem o Go instalado em seu sistema.

Para verificar, digite no terminal: 

```
$  go version

```
Caso não tenha, siga as instruções em [golang.org](https://golang.org)

### Como utilizar este tutorial

Em cada passo, incluindo o passo 0 (este), a tarefa estará descrita no arquivo README.md e logo em seguida o código que a executa e por fim, uma explicação de como ela funciona.

Cada passo está localizado em sua pasta separada, com o seu respectivo nome, exceto por este. Procure a pasta passoXX para encontrar qualquer passo dado.

Vamos sempre editar um arquivo chamado `main.go'. Todo o código deste tutorial residirá  neste arquivo. Um programa apropriado normalmente teria múltiplos arquivos de código fonte, mas por uma questão de simplicidade, vamos manter este programa limitado a um único código fonte.

O README.md para cada passo explicará a intenção e mostrará as modificações necessárias para prosseguir. Você deve fazê-las em seu próprio arquivo `main.go'.

Você também pode utilizar o arquivo `main.go` no passo 00 como ponto de partida e modificá-lo incrementalmente ao progredir pelos passos.

E não se preocupe! Se você se perder, cada passo tem seu próprio arquivo `main.go` com as alterações daquele passo já aplicadas. Isso também significa que se você quiser avançar para um determinado passo você pode começar com o arquivo `main.go` do passo anterior.

## Passo 00: Hello (Game) World

Vamos começar construindo desde a base um esqueleto de uma arquitetura mínima necessária para se parecer com um código de um jogo.

Escolha um diretório para ser o diretório do seu trabalho (por exemplo, `tutorial` dentro de sua pasta home) e crie um arquivo chamado `main.go` com o conteúdo abaixo.

Nota: Alternativamente você pode apenas clonar este repositório e editar o arquivo `main.go` em sua pasta raiz.

```go
package main

import "fmt"

func main() {
    // initialize game // inicia o jogo

    // load resources // carrega recursos

    // game loop      // looping do jogo
    for {
        // update screen       // atualiza a tela

        // process input       // processos de entrada

        // process movement    // processos de movimento

        // process collisions  // processos de colisao

        // check game over    // verifica o fim do jogo

        // Temp: break infinite loop  // finaliza o looping infinito
        fmt.Println("Hello, Pac Go!")
        break

        // repeat  // repete o processo
    }
}
```

### Executando seu primeiro programa Go

Agora que já temos um arquivo `main.go` (`.go` é a extensão de arquivo para o código fonte do Go), você pode executá-lo utilizando a ferramenta de linha de comando `go`. Para isto, basta digitar no terminal:

```sh
$ go run main.go
Hello, Pac Go!
```

É dessa maneira que nós executamos um único arquivo em Go. Você também pode compilar o arquivo como um programa executável por meio do comando `go build`.  Ao executar  o comando`go build` ele irá compilar os arquivos no diretório atual, gerando um binário com o nome do diretório. Então você poderá executá-lo como um programa normal, veja o exemplo:

```sh
$ go build
$ ./pacgo
Hello, Pac Go!
```

Para os propósitos deste tutorial, principalmente o de simplificar, estamos utilizando apenas um único arquivo de código (`main.go`), então você pode utilizar ou `go build` e executar o comando ou apenas `go run main.go` uma vez que ambos o fazem automaticamente.

#### Entendendo o programa

Agora vamos dar uma boa olhada no que o programa faz.

A primeira linha é o nome do `package`. Todo programa válido necessita ter um.. Além disso, se você quiser fazer um programa **runnable** (**executável**), você precisará de um pacote chamado `main` e também de uma função `main`, que será o ponto de entrada do programa.

Aqui, estamos criando um programa executável, então vamos utilizar o `package main` no topo do arquivo.

A seguir estão as instruções para que você consiga fazer o `import`. Os “imports” servem para tornar o código de outros pacotes acessíveis ao seu programa. 

Finalmente a função `main`. Em Go, você define uma função com a palavra-chave `func` seguida pelo seu nome, seus parâmetros devem estar entre um par de parênteses, seguido pelo valor de retorno e por fim, o corpo da função, que está contida em um par de chaves `{}`. Por exemplo:

```go
func main() {
    // I'm a function body  // Aqui temos o corpo de uma funcao
}
```

Esta é uma função chamada `main`, que não possui parâmetros vazio e não retorna nada. Esta é a declaração correta de uma função principal em Go, ao contrário das declarações em outras linguagens em que a função principal pode requisitar argumentos da linha de comando e/ou retornar um valor inteiro.

Temos maneiras diferentes de lidar com os argumentos da linha de comando e retornar valores em Go, que veremos em [Passo 08](passo08/README.md).

Na função principal do jogo temos alguns comentários (qualquer texto após `//` é um comentário) que estão atuando como placeholders (indicativos do local onde você deverá escrever) para o código do jogo atual. Vamos utilizá-los para conduzir cada modificação de forma ordenada.

O conceito mais importante em um jogo é o que é chamado de loop do jogo. Isso é basicamente um loop infinito onde toda a mecânica do jogo é processada.

Um loop em Go é definido com a palavra-chave `for`. O loop `for` pode ter um inicializador, uma condição e uma etapa de pós-processamento, mas todas elas são opcionais. Se você omitir tudo, você tem um loop infinito:


```go
for {
    // I never end  // loop infinito
}
```

Pode-se sair de um loop infinito através de uma declaração de `break`. Note que utilizamos o `break` no código de exemplo para finalizar o loop infinito após imprimir "Hello, Pac Go!" com a função `Println` do pacote `fmt` (comentários omitidos por concisão):

```go
func main() {
    for {
        fmt.Println("Hello, Pac Go!")
        break
    }
}
```

É claro que, neste caso, o loop infinito com uma quebra não condicional é inútil, mas fará sentido nas próximas etapas!

Parabéns, o passo 00 está completo!

[Leve-me para o passo 01!](passo01/README.md)
# pacgo-traduzido-pt-br
