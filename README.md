# LogInsight AI

Análise forense de bundles de suporte **VMware ESXi, vCenter e switches de
rede/SAN**. Roda inteiramente na sua infraestrutura — nenhum log sai da
máquina.

> 779 linhas de log → 8 problemas distintos.
> É isso que a ferramenta faz: separa ruído de sinal num bundle real.

---

## O que ela responde

| Pergunta | Onde |
|---|---|
| *Por que só este nó quebra?* | **Divergências** — o que difere entre os hosts |
| *Quebrou depois do update?* | **Mudanças** — o que mudou no mesmo equipamento |
| *O que aconteceu antes?* | **Ciclo de Vida** — tudo em ordem cronológica |
| *O que tenho e o que sai de suporte?* | **Inventário** — parque, fim de suporte e garantia |

Além disso: PSOD e kernel panic com o contexto ao redor, saúde de hardware
(VOB, IPMI, watchdog), compatibilidade contra regras curadas, logs de switch
Dell/Cisco/HP/Brocade, e um terminal simulado sobre os arquivos do bundle.

**Switch Dell SmartFabric OS10** tem leitura própria: o sosreport é
reconhecido como switch, e não como Linux genérico, e a ferramenta lê o
estado dos comandos `show` — alarmes ativos, integridade do par VLT,
divergência entre a configuração de boot e a em uso, monitoramento óptico,
portas sem descrição e agregação sem redundância. Um log inundado vira **um**
achado dizendo quanto tempo de histórico o equipamento realmente guarda, em
vez de milhares de linhas repetidas.

## Instalação

Não precisa de internet no servidor — as imagens vão dentro do pacote.

```bash
tar -xzf loginsight-1.0.1-amd64.tar.gz
docker load -i imagens/loginsight.tar.gz
docker load -i imagens/postgres.tar.gz

cp .env.example .env      # defina POSTGRES_PASSWORD
docker compose up -d
```

Abra `http://localhost:5050` e crie a primeira conta — ela administra a
instalação.

**Manual completo:** [INSTALACAO.md](INSTALACAO.md) — requisitos, licença,
papéis de usuário, atualização, backup e os problemas mais comuns.

**Pelo Docker Hub**, se o servidor tiver saída para a internet: veja a aba
de releases para a tag da versão.

## Avaliação

Roda **7 dias** sem licença nenhuma. Depois disso a ferramenta trava e a tela
mostra como obter uma — **nenhum dado é apagado**, e instalar a licença
destrava na hora, sem reiniciar.

## O que sai da máquina

Nada, com uma exceção declarada: a **consulta de garantia** é opcional, só
roda quando você clica, e envia apenas o número de série ao fabricante.

Bundles de suporte trazem hostname, IP, serial, nome de VM e topologia. É
por isso que a análise acontece onde o dado já está.

## Requisitos

- Docker e Docker Compose
- 2 GB de RAM livres (padrão; ajustável no `.env`)
- Disco proporcional aos bundles que você guardar

## Conferindo o download

Cada release publica o SHA-256. Confira antes de rodar:

```bash
shasum -a 256 -c SHA256SUMS
```

## Suporte e licenças

O contato de quem fornece a ferramenta aparece dentro dela, na tela de
licença — inclusive quando a instalação está bloqueada.

---

Este repositório contém **apenas as entregas e esta documentação**. O código
não é distribuído.
