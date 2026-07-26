# AULA Portas

Material de uma aula prática sobre **portas e conexões TCP**. Os alunos ficam
todos na mesma rede, se descobrem, escolhem uma porta e conversam.

Os scripts são propositalmente curtos e comentados: a ideia é que o aluno abra
e leia o que está rodando. `descobrir` não é mágica, é um laço de `ping`.

## A ideia da aula

Duas máquinas na mesma rede não conversam sozinhas. Para uma falar com a outra
precisa de três coisas, e o exercício introduz uma por vez:

1. **Um endereço** — saber para onde mandar. É o IP, e é o que `descobrir` acha.
2. **Uma porta** — saber para qual *serviço* daquela máquina falar. Uma máquina
   tem 65535 portas; é o número que separa "o site" de "o e-mail".
3. **Alguém escutando** — porta sem ninguém atrás recusa a conexão. É o
   significado exato de `connection refused`, e vale a pena o aluno provocar
   esse erro de propósito.

## Os comandos

| Comando | O que faz |
|---|---|
| `descobrir` | Varre a rede e lista quem está no ar, com IP e nome |
| `escutar PORTA` | Abre uma porta nesta máquina e espera alguém conectar |
| `conectar IP PORTA` | Liga na porta que um colega abriu |

## Roteiro sugerido

**1. Quem está aqui?**

```
$ descobrir
varrendo 10.99.0.0/24 a partir de 10.99.0.4 ...

IP             NOME
10.99.0.4      abc123-pc      <- você
10.99.0.7      def456-pc

2 máquinas no ar.
```

**2. Provocar o erro antes de acertar.** Peça para conectarem numa porta onde
ninguém está escutando. O `connection refused` que aparece é o conceito da aula,
e vale mais visto do que explicado.

```
$ conectar 10.99.0.7 4444
não consegui conectar em 10.99.0.7:4444.
```

**3. Agora com alguém do outro lado.** Um aluno abre a porta:

```
$ escutar 4444
abrindo a porta 4444 e esperando...
```

O outro conecta, e os dois digitam:

```
$ conectar 10.99.0.4 4444
conectado. O que você digitar aparece na tela do colega.
```

**4. Duas pessoas não cabem na mesma porta.** Peça para um terceiro tentar
`escutar 4444` na mesma máquina. O `Address already in use` mostra que a porta
é um recurso exclusivo — só um serviço por porta.

**5. Ver os pacotes.** Com a conversa aberta, num terceiro terminal:

```
$ tcpdump -i eth0 -A port 4444
```

O texto digitado aparece cru na tela. É o argumento mais convincente que existe
para "por que criptografia importa", e não precisa de slide.

## Além do `nc`

A imagem também traz `ncat`, `socat` e `nmap`, para quando a turma quiser ir
adiante:

```bash
nmap -p 1-10000 10.99.0.7        # quais portas o colega tem abertas
ncat -l 4444 -e /bin/bash        # o clássico: entregar um shell pela porta
socat TCP-LISTEN:4444 EXEC:/bin/bash
```

O `ncat -e` é o exemplo canônico de shell reverso. Vale mostrar **depois** do
`tcpdump`: fica evidente que quem está no meio da rede vê tudo.

## Como isso roda

Os scripts são instalados numa imagem Docker usada pela plataforma de labs da
turma. Cada aluno sobe o próprio container, e todos caem numa rede comum
(`10.99.0.0/24`) sem saída para a internet — o que mantém o exercício dentro da
sala.

Para construir a imagem, o `Dockerfile` clona este repositório numa tag:

```dockerfile
ARG AULA_REF=v1
RUN git clone --depth 1 --branch $AULA_REF \
      https://github.com/Doctorspeppers/aula-portas /opt/aula && \
    install -m755 /opt/aula/bin/* /usr/local/bin/
```

A referência é uma **tag**, não `main`: assim a imagem de uma turma não muda
sozinha quando alguém mexe aqui. Publicou script novo, crie uma tag nova e
reconstrua com `--build-arg AULA_REF=v2`.

## Rodar fora da plataforma

Os três scripts são bash puro e dependem só de `ping`, `nc` e `ip`. Funcionam em
qualquer Linux na mesma rede — inclusive em duas máquinas de verdade:

```bash
git clone https://github.com/Doctorspeppers/aula-portas
sudo install -m755 aula-portas/bin/* /usr/local/bin/
```
