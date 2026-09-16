# ATV3 — RMI: controle remoto e Pedra, Papel, Tesoura, Lagarto e Spock

*Repositório temporário. A versão mantida está em [sistemas-distribuidos/atv3-rmi](https://github.com/Paulo-Henrique-RDL/sistemas-distribuidos/tree/main/atv3-rmi).*

Requisito: **JDK 17 ou superior**.

> **Enunciado:** criar aplicações com RMI em que um computador é o servidor e outro é o cliente. **(1)** Emular pelo menos 5 funções de um controle remoto. **(2)** Jogo Pedra, Papel, Tesoura, Lagarto e Spock para 2 a 5 clientes: o servidor controla as rodadas (10 s entre rodadas, 5 s para jogar, desclassificando quem não joga), indica o vencedor ou o empate e exibe o placar; o vencedor envia "Perdeu Loser...Tente na próxima." aos perdedores.

Com RMI o cliente chama métodos de um objeto que está em outra máquina como se ele fosse local. O servidor publica o objeto no *registry* com um nome; o cliente busca esse nome e recebe um *stub*, que transforma cada chamada de método em uma troca de mensagens pela rede.

## Aplicação 1 — Controle remoto

O servidor é a TV, que guarda o estado (ligada, canal, volume, mudo). O cliente é o controle, com 8 funções: ligar/desligar, volume +, volume −, canal +, canal −, ir para um canal, mudo e status. A TV registra cada comando com o IP de quem enviou.

```sh
cd controle-remoto
javac -d bin src/*.java
java -cp bin ServidorControle              # [porta] [ip-do-servidor]
java -cp bin ClienteControle localhost     # [host] [porta]
```

## Aplicação 2 — Pedra, Papel, Tesoura, Lagarto e Spock

RMI só vai do cliente para o servidor, mas o jogo precisa do caminho inverso: avisar que a rodada abriu, o resultado, o placar e a mensagem do vencedor. Para isso cada cliente publica seu próprio objeto remoto `Jogador` e o entrega ao entrar no jogo; o servidor passa a chamá-lo de volta (*callback*).

Regras que o enunciado deixa em aberto:

- **Vencedor:** cada jogador soma quantos adversários o seu gesto derrota, e a maior soma única vence. Se a maior soma se repetir, ou se todos jogarem o mesmo gesto, é empate e todos jogam de novo na rodada seguinte.
- **Sem jogada em 5 s:** o jogador é desclassificado da rodada e conta como perdedor. Se só um jogar, vence por W.O.; se ninguém jogar, a rodada é anulada.
- **Mensagem aos perdedores:** o cliente vencedor envia, e o servidor repassa, porque só ele conhece os demais clientes.

```sh
cd jokenpo-spock
javac -d bin src/*.java
java -cp bin ServidorJogo                          # [porta] [ip-do-servidor]
java -cp bin ClienteJogo localhost 1100 ana        # [host] [porta] [nome] [ip-deste-cliente]
```

As rodadas começam sozinhas quando há pelo menos 2 jogadores. Quando a rodada abrir, digite de 1 a 5 ou o nome do gesto; `sair` encerra.

## Entre dois computadores

O RMI grava o endereço do servidor dentro do *stub*. Se a máquina tiver mais de uma rede (Wi-Fi, cabo, VPN), ele pode escolher a errada, então informe o IP. No jogo o servidor também se conecta de volta ao cliente, por isso o cliente informa o próprio IP:

```sh
java -cp bin ServidorJogo 1100 192.168.0.10
java -cp bin ClienteJogo 192.168.0.10 1100 ana 192.168.0.20
```

O controle remoto usa a porta 1099 e o jogo, a 1100. Libere o Java no firewall das duas máquinas.
