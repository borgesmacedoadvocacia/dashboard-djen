# Dashboard — Publicações DJEN

Leitura **estratégica** dos resultados dos processos do escritório Borges Macedo Advocacia,
a partir das publicações do **Diário de Justiça Eletrônico Nacional**, lidas ao vivo da API
pública do DJEN direto do navegador.

**Acesso exclusivo das lideranças.** Só os perfis **Administração** e **Lideranças** têm
cofre neste painel.

> **Não é sistema de controle de prazos.** Esse controle é do **AdvBox**. Aqui não existe
> "prazo vencido": prazo aparece só como sinalização de carga, e apenas o que está em curso.
> A ferramenta também não substitui a consulta oficial ao diário.

## Para que serve

Responder às perguntas que orientam decisão, não à fila do dia:

- Quanto se ganha e quanto se perde, **por tipo de julgamento** — sentença, recurso, liminar.
- Dentro de recurso, **qual recurso**: apelação, agravo de instrumento, agravo interno,
  recurso inominado, embargos de declaração, especial/extraordinário.
- **Qual produto jurídico** ganha e qual para, e **em que fase** ele para.
- **Contra quem** se ganha e contra quem se perde.
- **Quanto há para executar**, e quanto disso já foi recebido.
- O que saiu no diário e **ainda não foi registrado** na planilha.

## As três fontes

| Fonte | O que vem de lá |
|---|---|
| **API do DJEN** (ao vivo) | As publicações, o texto integral e o dispositivo — o que acabou de ser decidido |
| **Planilha DataJud** (`Base geral`) | Resultado da sentença, resultado do recurso, situação do alvará, valor da causa, fase, dias sem movimentação |
| **Planilha Clientes e Processos** (`Todos os Processos`) | **Nome das partes e produto jurídico** — só isso; as demais colunas dessa base não são lidas |

O produto jurídico é **cadastrado**, não inventado: vem da mesma coluna que alimenta o painel
Processos Judiciais. Onde o cadastro ainda não existe, o painel deduz pelo assunto do CNJ,
devolve os mesmos nomes do cadastro e **marca a dedução na ficha**.

## Por que fala direto com a API

O coletor em Apps Script grava numa aba `_DJEN` que está **vazia**: a chamada sai dos IPs do
Google, que nem sempre são brasileiros, e a API tende a devolver 403 fora do Brasil. Falar
direto com a API a partir do navegador resolve isso — o DJEN libera CORS para a origem do
GitHub Pages — e ainda dá acesso a campos que o coletor descartava (`hash` da certidão,
status de cancelamento, classe processual). Consequência prática: **não há defasagem**.

## As abas

### Resultados
Cartões do período; **sentenças** separadas em procedente, parcialmente procedente,
improcedente, extinção do processo e acordo homologado, com **volume por mês empilhado por
resultado**; **acórdãos** com taxa própria por espécie de recurso; **liminares** com taxa de
deferimento; a matriz **julgamento × resultado**; desempenho **por produto** e **por
contraparte**; e a tabela de decisões, com filtro próprio e exportação em CSV e XLSX.

### Valores a executar
Quanto há identificado, **uma linha por processo** — o mesmo processo sai no diário várias
vezes com o mesmo valor, e somar publicação a publicação multiplicaria a carteira. Filtro por
julgamento (sentença, apelação, agravo…), por faixa de valor, por trânsito mencionado e pelo
**estágio de recebimento**; exportação direta do recorte.

### Carteira e movimento
Retrato da base inteira do DataJud (sem recorte de período), **divergências entre o diário e
o registro**, volume publicado, mapa de calor tribunal × ato, prazos em curso, processos mudos
e saúde da coleta.

### Relatórios
Extração livre: escolha a **unidade** (uma linha por publicação, por decisão, por processo ou
por valor citado), marque as **colunas** e exporte em CSV ou XLSX. Os filtros do topo valem
aqui — o arquivo sai com exatamente o recorte que está na tela.

### Visão CEO
Cockpit, matriz **produto × julgamento** (onde a tese ganha e onde ela para), leitura do
período em texto e as fontes ainda não conectadas.

## Como a leitura automática funciona — e onde ela para

A classificação é **heurística e assumida como tal**; toda decisão traz o grau de confiança
ao lado. Ela roda em camadas: sigilo (publicação sigilosa não traz inteiro teor), depois o
texto do dispositivo, depois o rótulo que o próprio tribunal anexou.

O inteiro teor chega ora em texto puro, ora em HTML completo com `<head>`, `<style>` e
entidades (`senten&ccedil;a`). O painel limpa a marcação e decodifica as entidades **antes**
de classificar.

**Resultado só quando há dispositivo.** Quando o dispositivo não aparece escrito, a publicação
fica em "sem dispositivo" e **sai do denominador**. Uma taxa calculada sobre tudo
transformaria "não deu para ler" em derrota.

### Valores: o que conta como condenação

Uma decisão cita muitos reais. Cada valor é classificado pelo que está escrito imediatamente
antes dele, e **só a natureza "condenação" entra na carteira a executar**:

- **Bloqueio/penhora** (SISBAJUD, BacenJud, RenaJud) fica de fora. Um único bloqueio de
  R$ 17,2 milhões num processo distorcia a carteira inteira quando era lido como condenação.
- **Custas, honorários, multa, valor da causa, depósito e alvará/RPV** têm cada um a sua
  natureza e não somam com a condenação.
- Valores repetidos na mesma decisão são **contados uma vez**: o mesmo número costuma aparecer
  três ou quatro vezes (dispositivo, correção, dispositivo final).

Ainda assim é **estimativa de leitura, não apuração contábil**. Cada linha traz o trecho de
onde o número saiu e o link da certidão. **Validação nos autos é obrigatória antes de executar.**

## Inscrições consultadas

| Inscrição | Situação observada |
|---|---|
| OAB/BA 41.438 | Carteira principal — concentra a quase totalidade do volume |
| OAB/SP 536.843 | Volume baixo, boa parte já vem pela inscrição da Bahia |
| OAB/RJ 271.081 | **Zero publicações** — ou é carteira parada, ou a inscrição não está cadastrada nos processos do RJ |

A API só responde ao número puro da inscrição; os sufixos `-O`, `-A`, `-N` e `-E` retornam vazio.

## As três datas do prazo (CPC, art. 224, §§ 2º e 3º)

Toda publicação traz as três, porque confundi-las é como se perde prazo: **disponibilização**
→ **publicação** (primeiro dia útil seguinte) → **início do prazo** (primeiro dia útil
seguinte). O calendário forense inclui feriados nacionais móveis e o recesso de 20/12 a 6/1.

## Limites conhecidos

- **Cobertura do DJEN é progressiva** — completa a partir de 2025, parcial em 2024, esparsa
  antes. Períodos anteriores exibem tarja de aviso.
- **Publicações órfãs** não têm processo correspondente na base: sem esse par o painel não
  sabe cliente nem produto, e elas ficam fora da análise por produto.
- **Estágio de recebimento** vem da coluna *Situação do Alvará* da planilha do DataJud. Ela
  diz o estágio, não o valor — o montante de cada alvará ainda não existe em lugar nenhum.

## Como funciona

- Publicações: `GET https://comunicaapi.pje.jus.br/api/v1/comunicacao` — API pública, sem
  autenticação, 50 itens por página, paginada com 450 ms de intervalo.
- Deduplicação por `id` do DJEN, acumulando quais inscrições trouxeram cada publicação.
- Cruzamento com as duas planilhas pelo número do processo (só dígitos).
- Credenciais e IDs num cofre **AES-256-GCM** com **PBKDF2-SHA256 (310.000 iterações)**.
- Exportação em XLSX via SheetJS.

## Publicação

GitHub Pages, branch `main`, raiz do repositório:
<https://borgesmacedoadvocacia.github.io/dashboard-djen/>

Também acessível pela [central de dashboards](https://borgesmacedoadvocacia.github.io/),
no grupo **Processual**.
