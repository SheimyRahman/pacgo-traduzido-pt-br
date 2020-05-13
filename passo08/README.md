# Passo 08: Parâmetros de Command line 

Nesta lição, você aprenderá a fazer: 

- Adicionar flags em uma aplicação de command line

## Overview

Na última lição adicionamos um arquivo de configuração `config.json` para lidar com nossa tradução de emoji. Também criamos um arquivo chamado `config_noemoji.json` que traduz para a representação original do jogo na tela.

Também utilizamos o arquivo `maze01.txt` para nossa representação do labirinto. Todos esses nomes são escritos diretamente no código fonte, mas lidar com esses arquivos de forma codificada não é o ideal, por isso vamos mudar essa situação.

## Tarefa 01: Criando flags para cada arquivo

O pacote `flag` da biblioteca padrão é o responsável pelo gerenciamento das flags das linha de comando. Por isso, nós vamos utilizá-lo para criar duas flags: `--config-file` e `--maze-file`.

No início do arquivo, logo após as importações, adicione as seguintes variáveis globais.

```go
var (
    configFile = flag.String("config-file", "config.json", "path to custom configuration file")
    mazeFile   = flag.String("maze-file", "maze01.txt", "path to a custom maze file")
)
```

A função `String` do pacote `flag` aceita três parâmetros: um nome para a flag, um valor padrão e uma descrição (a ser exibida quando `--help` for utilizado). Ele retorna um ponteiro para uma string que irá armazenar o valor da flag.

Observe que este valor só é preenchido após chamar a função `flag.Parse`, que deve ser chamada a partir da função `main`:

```go
func main() {
    flag.Parse()

    // initialise game
    initialise()
    defer cleanup()

    // rest of the function omitted...
}
```

Por favor, note que estamos chamando `flag.Parse()` como a primeira coisa no programa. Desejamos fazer isso pois queremos que as flags sejam analizadas **before** de mudar o console para o modo `cbreak`.

Quando é feito o parsed da flag, em caso de erro, ela chama a função`os.Exit`, o que significa que nossa outra função `cleanup` não seria chamada, deixando o terminal sem echo e ainda em modo cbreak, o que pode ser bastante inconveniente.

Com esta mudança, ao controlarmos a ordem em que as coisas são chamadas, estamos nos certificando de que só entraremos no modo cbreak quando o parsed das flags forem realizados com sucesso.

## Tarefa 02: Substituindo os arquivos hard coded com as flags

Já resolvemos o parsing, agora precisamos substituir o que esta em modo hard coded pelas suas flags equivalentes.

Isto é feito pela substituição do valor que está hard coded pelo valor da flag (note o de-reference operador (`*`) , pois as flags são ponteiros)

Em `main`:

```go
    // load resources
    err := loadMaze(*mazeFile)
    if err != nil {
        log.Println("failed to load maze:", err)
        return
    }

    err = loadConfig(*configFile)
    if err != nil {
        log.Println("failed to load configuration:", err)
        return
    }
```

Agora tente executar a seguinte command line:

```sh
go build
./step08 --help
```

Você deverá ver algo como:

```sh
$ ./step08 --help
Usage of ./step08:
  -config-file string
        path to custom configuration file (default "config.json")
  -maze-file string
        path to a custom maze file (default "maze01.txt")
```

Agora tente,  executar `passo08` com `--config-file config_noemoji.json` primeiro, e `--config-file config.json` depois para ver a diferença. Muito melhor com os emojis não é mesmo?!

Você também pode tentar copiar o arquivo `maze01.txt` para um novo arquivo e editá-lo para experimentar novas coisas.

Talvez você possa criar seus próprios temas agora... experimente visitar [Full Emoji List](https://unicode.org/emoji/charts/full-emoji-list.html) para se inspirar). :)

[Leve-me ao passo 09!](../passo09/README.md)
