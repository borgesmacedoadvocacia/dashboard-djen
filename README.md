# Dashboard — Publicações DJEN

Leitura e triagem das publicações do **Diário de Justiça Eletrônico Nacional** do escritório
Borges Macedo Advocacia, lidas **ao vivo da API pública do DJEN** direto do navegador.

**Acesso exclusivo das lideranças.** Só os perfis **Administração** e **Lideranças** têm
cofre neste painel.

> **Ferramenta de apoio. Não substitui a consulta oficial ao diário.**
> O painel indica; ele não lança prazo em agenda. Toda sugestão de prazo traz o grau de
> confiança ao lado, e nenhuma entra em agenda sem validação humana.

## Por que ele fala direto com a API

O coletor em Apps Script grava numa aba `_DJEN` que está **vazia**: a chamada sai dos IPs do
Google, que nem sempre são brasileiros, e a API tende a devolver 403 fora do Brasil. Falar
direto com a API a partir do navegador resolve isso — o DJEN libera CORS para a origem do
GitHub Pages — e ainda dá acesso a campos que o coletor descartava (`hash` da certidão,
status de cancelamento, classe processual).

Consequência prática: **não há defasagem**. O que está no diário de hoje está no painel hoje,
sem depender de gatilho agendado.

## Inscrições consultadas

| Inscrição | Situação observada |
|---|---|
| OAB/BA 41.438 | Carteira principal — concentra a quase totalidade do volume |
| OAB/SP 536.843 | Volume baixo, boa parte já vem pela inscrição da Bahia |
| OAB/RJ 271.081 | **Zero publicações** — ou é carteira parada, ou a inscrição não está cadastrada nos processos do RJ |

A API só responde ao número puro da inscrição. Os sufixos `-O`, `-A`, `-N` e `-E` retornam
vazio, e por isso não são usados.

## As três datas do prazo (CPC, art. 224, §§ 2º e 3º)

O painel nunca mostra uma data só. Toda publicação traz as três, porque confundi-las é
como se perde prazo:

1. **Disponibilização** — o dia em que o ato entrou no diário eletrônico.
2. **Publicação** — o primeiro dia útil seguinte à disponibilização.
3. **Início do prazo** — o primeiro dia útil seguinte à publicação.

O prazo fatal sugerido conta a partir da terceira data, em dias úteis, sobre um calendário
forense que inclui feriados nacionais móveis (Carnaval, Sexta-Feira Santa, Corpus Christi) e
o recesso de 20/12 a 6/1.

## O que é exibido

### Operação diária
| Seção | Conteúdo |
|---|---|
| Saúde da coleta | Situação por inscrição, HTTP, páginas lidas e tempo. **Se esta faixa não estiver verde, nenhum número abaixo é confiável** |
| Cartões do dia | Novas hoje, fila de triagem, prazos em 72 h, órfãs, desfavoráveis na semana, valor citado, confiança baixa, canceladas |
| Semáforo de prazos | Vencido · Hoje · Amanhã · D-2 e D-3 · D+4 a D+10, com a confiança à vista em cada cartão |
| Fila de triagem | Publicações dos últimos 30 dias, da mais urgente para a menos |
| Alertas ativos | Só o que exige decisão, com o limiar ao lado do número |

### Análise
| Seção | Conteúdo |
|---|---|
| Termômetro de resultados | Favorável, parcial e desfavorável **sobre as que têm dispositivo legível** — as demais ficam em "não aplicável" em vez de entrarem num lado |
| Mapa de calor | Tribunal × tipo de ato; a célula abre as publicações do cruzamento |
| Tendência | Volume por dia com média móvel de 7 dias — queda abrupta costuma ser falha de coleta, não calmaria |
| Cobertura por inscrição | Se uma inscrição some por semanas, ou é carteira parada ou é a chave de consulta que quebrou |
| Radar de processos mudos | Processos ativos na base do DataJud sem publicação e com mais de 90 dias sem movimento |

### Visão CEO
| Seção | Conteúdo |
|---|---|
| Cockpit | Oito números de abertura de dia, cada um com a lista que o sustenta a um clique |
| Dinheiro identificado | Alvará, honorários, custas, depósito judicial, RPV e precatório citados no texto |
| Painel de risco | Indicador, valor de agora, limiar, situação e a ação correspondente |
| Fontes não conectadas | O que falta para o cockpit financeiro completo, e qual planilha alimentaria cada bloco |

## Como a leitura automática funciona — e onde ela para

A classificação é **heurística e assumida como tal**. Ela roda em três camadas, nesta ordem:

1. **Sigilo** — publicação sigilosa não traz inteiro teor; o DJEN devolve só o aviso.
   Vira categoria própria (`Sigiloso`, confiança 40%) em vez de "não classificado", porque
   o problema real não é a falta de classificação, é a impossibilidade de ler.
2. **Texto** — expressões do dispositivo (`julgo procedente`, `nego provimento`, `defiro a
   liminar`…). É a camada de maior confiança, 60% a 90%.
3. **Rótulo do tribunal** — quando o texto não decide, o `tipoDocumento` / `tipoComunicacao`
   que o próprio tribunal anexou ainda informa. Entra com confiança menor (45% a 75%).

O inteiro teor chega ora em texto puro, ora em HTML completo com `<head>`, `<style>` e
entidades (`senten&ccedil;a`). O painel limpa a marcação e decodifica as entidades **antes**
de classificar — sem isso o classificador procurava "sentença" onde estava escrito
`senten&ccedil;a` e errava.

**O que sobra sem classificação** é honesto: publicações que o tribunal rotulou apenas como
"Intimação" e cujo texto não traz palavra decisiva. Elas aparecem com confiança 45% e caem
na lista de leitura humana obrigatória, que é onde devem estar.

### Resultado só quando há dispositivo

`Favorável` / `Desfavorável` / `Parcial` só são afirmados quando o dispositivo aparece
escrito. Quando não aparece, a publicação fica em **"não aplicável"** e **sai do
denominador** da taxa de êxito. Uma taxa calculada sobre tudo transformaria "não deu para
ler" em derrota.

## Limites conhecidos

- **Cobertura do DJEN é progressiva** — completa a partir de 2025, parcial em 2024, esparsa
  antes. Períodos anteriores exibem tarja de aviso: volume baixo ali pode ser a data de
  adesão do tribunal, não escritório parado.
- **Valor citado** captura o primeiro `R$` do texto. Serve para localizar dinheiro que passou
  pelo diário, não como apuração contábil.
- **Publicações órfãs** são as que não têm processo correspondente na base do DataJud. Ou a
  carteira está desatualizada, ou é processo de terceiro — as duas exigem conferência.

## Como funciona

- Publicações: `GET https://comunicaapi.pje.jus.br/api/v1/comunicacao` — API pública, sem
  autenticação, máximo de 50 itens por página, paginada com 450 ms de intervalo.
- Deduplicação por `id` do DJEN, acumulando quais inscrições trouxeram cada publicação.
- Cliente, área e fase vêm da aba **Base geral** da planilha do DataJud, cruzadas pelo número
  do processo (só dígitos).
- Credenciais e IDs ficam num cofre **AES-256-GCM** com **PBKDF2-SHA256 (310.000 iterações)**,
  aberto pela combinação usuário + senha. Nada sensível trafega em texto claro no repositório.
- Exportação em CSV com 24 colunas, incluindo as três datas, a confiança e o link da certidão.

## Publicação

GitHub Pages, branch `main`, raiz do repositório:
<https://borgesmacedoadvocacia.github.io/dashboard-djen/>

Também acessível pela [central de dashboards](https://borgesmacedoadvocacia.github.io/),
no grupo **Processual**.
