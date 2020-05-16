# Passo 10: Fantasmas com power ups!

Nesta lição, você aprenderá a fazer: 

- Usar um timer 
- Quando e como usar uma Mutex lock

## Overview

Nesta lição iremos adicionar suporte para a pílula de power up. Vamos atualizar a configuração com as novas funcionalidades e adicionar o código para desenhar a pílula no labirinto. Também iremos gerenciar o processo após o pacman engolir uma pílula e colidir com um fantasma. Finalmente, vamos manipular casos em que o pacman tenta engolir uma pílula enquanto a anterior está ativa e como lidar com isso.

## Tarefa 01: Desenhando as Pílulas.

Antes mesmo de começarmos, devemos atualizar a configuração para suportar as pílulas de power up! Então para ambos `config_noemoji.json` e `config.json`, temos que adicionar as configurações `ghost_blue` (string) e `pill_duration_secs` (int).
Assim, nós atualizamos nossa struct `Config`:

```go
type config struct {
    ...
	GhostBlue        string        `json:"ghost_blue"`
	PillDurationSecs time.Duration `json:"pill_duration_secs"`
}
```

## Tarefa 02: Habilitar a ingestão da pílula

Para habilitar a ingestão da pílula pelo pacman devemos adicionar mais outra condição na func `movePlayer` para o caso da pílula

```go
case 'X':
	score += 10
	removeDot(player.row, player.col)
	go processPill()
```

Em que `X`  é o caractere de configuração da pílula.

Agora, antes de passarmos para a func `ProcessPill`, devemos adicionar mais algum código para os fantasmas suportarem o modo `Blue Ghosts`! Vamos acrescentar um novo `GhostStatus` do tipo string que irá manter o status de um fantasma. Os dois status que precisamos suportar são o `Normal` e o `Blue`.

```go
type GhostStatus string

const (
	GhostStatusNormal GhostStatus = "Normal"
	GhostStatusBlue   GhostStatus = "Blue"
)
```

Agora, cada fantasma deverá se enquadrar na sua posição atual, a `initialPosition`, onde será exibido após ter sido comido pelo pacman e seu status atual.

```go
type ghost struct {
	position sprite
	status   GhostStatus
}
```

Assim, a função `loadMaze` irá primeiramente desenhar os fantasmas com o status `Normal` e armazenar sua posição inicial.


```go
ghosts = append(ghosts, &ghost{sprite{row, col, row, col}, GhostStatusNormal})
```

A função `printScreen` também deve ser atualizada para suportar a impressão de fantasmas de ambos os tipos - fantasmas normais e azuis!

```go
for _, g := range ghosts {
		moveCursor(g.position.row, g.position.col)
		if g.status == Normal {
			fmt.Printf(cfg.Ghost)
		} else if g.status == Blue {
			fmt.Printf(cfg.GhostBlue)
		}
	}
```

A última coisa que falta é a função `processPill` que adicionamos um pouco antes. Esta função altera o status de todos os fantasmas para `Blue` durante o período definido pela configuração `PillDurationSecs`.
Para o processamento da pílula vamos usar um `Timer` retirado do pacote ['time' package](https://golang.org/pkg/time/). Vamos utilizar a função `NewTimer` para criar um novo Timer que enviará a hora atual em seu Channel após, pelo menos, ter a sua duração especificada.

O código "ProcessPill" muda todos os status dos fantasmas para `GhostStatusBlue`, depois ele bloqueia com `PillDurationSecs` e quando o tempo acaba, muda todos os status dos fantasmas de volta, para `GhostStatusNormal`.

```go
var pillTimer *time.Timer

func processPill() {
	for _, g := range ghosts {
		g.status = GhostStatusBlue
	}
	pillTimer = time.NewTimer(time.Second * cfg.PillDurationSecs)
	<-pillTimer.C
    for _, g := range ghosts {
		g.status = GhostStatusNormal
    }
}
```

## Tarefa 03: Suportar a ingestão simultânea de pílulas

A função `ProcessPill` que falamos anteriormente, tem um problema simples. Imagine o que pode acontecer se o pacman tentar engolir uma pílula de power up enquanto outra pílula ainda está ativa!? Atualmente, com a função proposta no código `processPill`, quando uma segunda pílula está sendo engolida pelo pacman, enquanto a primeira ainda está ativa, ao terminar o efeito da primeira pílula (após `PillDurationSecs`) todos os fantasmas voltarão ao normal. Para contornar isto, devemos verificar se existe uma pílula já ativa, fazemos isso verificando o temporizador, e depois se estiver, devemos pará-lo e reinicializá-lo.


```go
var pillTimer *time.Timer

func processPill() {
	updateGhosts(ghosts, GhostStatusBlue)
	if pillTimer != nil {
		pillTimer.Stop()
	}
	pillTimer = time.NewTimer(time.Second * cfg.PillDurationSecs)
	<-pillTimer.C
	pillTimer.Stop()
	updateGhosts(ghosts, GhostStatusNormal)
}
```

## Tarefa 04: Evitando as Condições da Corrida

Nos nossos cenários há duas condições de corrida possíveis. A primeira é a do cronômetro da pílula (pill timer) que mencionamos anteriormente. A função `processPill` é chamada de maneira assíncrona. Então, no caso a primeira função `processPill`  é executada logo após o `pillTimer.Stop()` enquanto a segunda está dentro do `if pillTimer != nil {` block. Neste caso raro, ocorre que enquanto uma pílula estiver ativa, consumindo a próxima no momento em que o código estiver neste ponto, podemos perder a segunda pílula, pois os fantasmas voltarão ao normal. 

Por esta razão, estamos inserindo uma trava, a "pillMx Mutex lock" que vamos adotar no início da função `ProcessPill` e liberar logo antes de começar a espera no timer channel. Também vamos adotá-lo logo após a função de bloqueio e liberá-lo no final da função.

```go
var pillTimer *time.Timer
var pillMx sync.Mutex

func processPill() {
	pillMx.Lock()
	updateGhosts(ghosts, GhostStatusBlue)
	if pillTimer != nil {
		pillTimer.Stop()
	}
	pillTimer = time.NewTimer(time.Second * cfg.PillDurationSecs)
	pillMx.Unlock()
	<-pillTimer.C
	pillMx.Lock()
	pillTimer.Stop()
	updateGhosts(ghosts, GhostStatusNormal)
	pillMx.Unlock()
}
```

Outra possível condição de corrida que pode surgir durante a execução é quando atualizamos o status dos fantasmas. Para este propósito, vamos usar um bloqueio RWMutex. Devemos implementar o cadeado sempre que lemos ou atualizamos o status de um fantasma. O RWMutex suporta o bloqueio mesmo para acesso de leitura ou escrita. Então estamos introduzindo o `var ghostsStatusMx sync.RWMutex` e também, uma função `updateGhosts`, que atualiza o status de um ou mais fantasmas.


```go 
var ghostsStatusMx sync.RWMutex

func updateGhosts(ghosts []*Ghost, ghostStatus GhostStatus) {
	ghostsStatusMx.Lock()
	defer ghostsStatusMx.Unlock()
	for _, g := range ghosts {
		g.status = ghostStatus
	}
}
```

Também temos que adotar um RLock sempre que lemos o status de um fantasma. Múltiplos cadeados de leitura podem ser obtidos simultaneamente, mas apenas um cadeado de escrita pode ser implementado. Vamos utilizar o `ghostsStatusMx.RLock()` e `ghostsStatusMx.RUnlock()` enquanto lemos o status dos fantasmas. Não podemos esquecer de desbloquear sempre o RLock antes de atualizar o status de um fantasma, caso contrário um deadlock ocorrerá.

Agora temos um pacman mais desafiador! Happy gaming/coding! :) 

## E isso é tudo pessoal!

Parabéns! Você completou todos os passos do tutorial.

Mas a sua jornada não deve terminar aqui. Se você está interessado em contribuir com um novo passo, dê uma olhada na [TODO list](../TODO.md) ou em qualquer assunto em aberto e envie um PR!
