# LogInsight AI — Manual de instalação

Vale para o pacote `loginsight-<versão>.tar.gz` e para a imagem do Docker
Hub. Se você só quer subir e usar, os passos 1 a 4 bastam.

---

## Antes de começar

| | |
|---|---|
| **Sistema** | Linux, Windows ou macOS com Docker e Docker Compose |
| **Arquitetura** | x86_64 ou ARM64 — há um pacote para cada |
| **Memória** | 2 GB livres |
| **Disco** | 5 GB + o espaço dos bundles que você retiver |
| **Rede** | **Nenhuma.** O pacote instala sem internet |

A ferramenta sobe uma porta local (5050 por padrão). Ela **não deve ficar
exposta à internet** — o padrão já publica só em `127.0.0.1`.

---

## 1. Extrair

```bash
tar -xzf loginsight-1.0.3-amd64.tar.gz
cd loginsight-1.0.3-amd64   # ou a pasta onde extraiu
```

Confira o que baixou, se veio pela internet:

```bash
shasum -a 256 -c SHA256SUMS
```

## 2. Carregar as imagens

```bash
docker load -i imagens/loginsight.tar.gz
docker load -i imagens/postgres.tar.gz
```

Demora um ou dois minutos. Não precisa de internet.

## 3. Configurar

```bash
cp .env.example .env
```

Abra o `.env` e defina **`POSTGRES_PASSWORD`** — é a única coisa
obrigatória. Se a porta 5050 já estiver em uso, mude `WEB_PORT`.

## 4. Subir

```bash
docker compose up -d
```

Acesse **http://localhost:5050**.

A primeira tela pede a criação de uma conta. **Essa conta administra a
instalação** — ela cria as demais e é a única que lê a auditoria. Anote a
senha; não há recuperação automática por e-mail.

---

## Licença

A ferramenta roda **7 dias** sem licença nenhuma, com tudo liberado.

Depois disso ela trava e mostra uma tela explicando. **Nenhum dado é
apagado** — banco, bundles e usuários continuam onde estavam.

Para licenciar, mande ao fornecedor o **identificador da instalação**, que
aparece nessa tela (e em *Administração → Licença*). Você recebe um arquivo
de licença e o instala colando o conteúdo dele na própria tela. **Vale na
hora, sem reiniciar.**

Se a instalação estiver travada, a caixa para colar aparece sem precisar
fazer login — que seria impossível, já que a licença vencida bloqueia
também a entrada.

---

## Usuários

Três papéis:

| Papel | O que faz |
|---|---|
| **Administrador** | Cria e exclui contas, exclui clientes e arquivos, lê a auditoria |
| **Técnico** | Cadastra clientes, analisa bundles, usa todas as telas de análise |
| **Cliente (só leitura)** | Vê apenas o inventário e os relatórios da **própria empresa** |

O papel *Cliente* existe para entregar acesso ao dono dos servidores sem
lhe dar a ferramenta inteira.

Esqueceu a senha? Um administrador redefine em *Administração → Técnicos*.
Se **você é** o administrador e ficou de fora, veja a tela *Esqueci minha
senha* — ela mostra o comando a rodar no servidor.

---

## Atualização

Substitua os arquivos na **mesma pasta** — cada pasta é uma instalação
separada, com seu próprio banco.

```bash
docker load -i imagens/loginsight.tar.gz
docker compose up -d
```

Os volumes são preservados: banco de clientes e casos, bundles originais,
licença e identificador da instalação.

---

## Backup

Tudo o que importa está em dois volumes Docker:

```bash
docker run --rm -v loginsight_pgdata:/dados -v "$PWD":/saida alpine \
    tar -czf /saida/backup-banco.tar.gz -C /dados .
docker run --rm -v loginsight_bundle_storage:/dados -v "$PWD":/saida alpine \
    tar -czf /saida/backup-bundles.tar.gz -C /dados .
```

Os nomes exatos aparecem em `docker volume ls`. O primeiro contém clientes,
casos, análises, usuários e a trilha de auditoria; o segundo, os bundles
originais.

---

## Problemas comuns

**A porta 5050 já está em uso**
Mude `WEB_PORT` no `.env` e rode `docker compose up -d` de novo. No macOS a
5000 costuma ser tomada pelo AirPlay — por isso o padrão é 5050.

**Subiu, mas o navegador não abre nada**
`docker compose ps` deve mostrar os dois containers. Se o `web` estiver
reiniciando, veja `docker compose logs web` — quase sempre é
`POSTGRES_PASSWORD` em branco no `.env`.

**"A ferramenta está bloqueada"**
A avaliação terminou ou a licença venceu. A própria tela traz o contato e a
caixa para instalar a licença nova.

**Análise falha em bundle muito grande**
Aumente `WEB_MEM_LIMIT` no `.env` (padrão 2 GB). Um bundle de 3 GB costuma
pedir 4 GB.

**"Assinatura inválida — esta licença não foi emitida para este produto"**
O texto da licença foi alterado no caminho até você. As causas comuns são um
espaço no fim de uma linha e as aspas trocadas por “curvas” — os dois
acontecem quando a licença viaja no corpo de um e-mail ou passa por Word ou
WordPad. Peça o arquivo `.license` **como anexo**, abra em editor simples
(Bloco de Notas no Windows, `cat` no Linux e macOS) e copie de lá.

**"Esta licença pertence a outra instalação"**
A licença foi emitida para outro servidor. Cada instalação tem um
identificador próprio, e a licença é presa a ele. Se você recriou os volumes
(`docker compose down -v`) ou instalou do zero, o identificador mudou: mande o
novo ao fornecedor, que reemite.

**Quero começar do zero**
`docker compose down -v` apaga **tudo**, inclusive os dados. Sem `-v`,
apenas para os containers.

---

## Alternativa: Docker Hub

Se o servidor tem saída para a internet:

```bash
docker pull franklinmoura/loginsightai:1.0.3
```

Para uso individual, sem PostgreSQL:

```bash
docker run -d --name loginsight -p 5050:5000 \
  -v loginsight-data:/data \
  -e DATABASE_URL=sqlite:////data/loginsight.db \
  -e SECRET_KEY="uma-frase-longa-e-secreta" \
  franklinmoura/loginsightai:1.0.3
```

SQLite não aguarda escrita concorrente — esse modo é para **um técnico**.
Em equipe, use o pacote com PostgreSQL.

---

## O que sai da máquina

**Nada**, com uma exceção declarada: a consulta de garantia é opcional, só
roda quando você clica, e envia apenas o número de série ao fabricante.

Bundles de suporte carregam hostname, IP, serial, nome de VM e topologia da
rede. É por isso que a análise acontece onde o dado já está.

---

**Suporte:** contact@loginsightai.com.br
