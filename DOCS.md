# Central de Confirmações — Documentação

App estática de página única (`index.html`) para confirmar telefonicamente
agendamentos (Ordens de Trabalho). Sem backend, sem conta, sem servidor:
tudo corre no browser e os dados ficam em `localStorage` do dispositivo.

## Ficheiros

- `index.html` — a app inteira (HTML + CSS + JS inline).
- `vercel.json` — config mínima para deploy estático na Vercel.

## Privacidade / armazenamento

- Os registos vivem em `localStorage` sob a chave `davi-central-confirmacoes-v2`.
- O tema (claro/escuro) fica em `davi-central-theme`.
- Nada é enviado para nenhum servidor — a Vercel só serve os ficheiros
  estáticos, não há API nem base de dados. Se abrires a app noutro
  computador ou noutro browser, começas com os dados vazios (é local a
  cada instalação de browser).

## Dashboard dinâmico

Os 6 cartões no topo (Total, Confirmados, Não atendeu, Reagendar,
Cancelados, Por confirmar) refletem **sempre o conjunto atualmente
filtrado**, não a lista completa. Isto aplica-se aos três filtros
existentes e às suas combinações:

- Busca por texto (ver "Busca abrangente" abaixo)
- Data
- Status (incluindo "— Por confirmar —")

Exemplo: se filtrares pela data `01/09/2026`, os cartões mostram quantas
Ordens **desse dia** estão confirmadas, quantas não atenderam, etc. — não
o total geral do dia todo.

Quando há um filtro ativo, o cartão "Total" passa a chamar-se
"Total filtrado" e mostra `filtrado / geral` (ex.: `34 / 703`) para dar
contexto de quanto do universo total aquele filtro representa. Sem filtro
nenhum ativo, volta ao comportamento normal (`703`, rótulo "Total").

A implementação está em `renderStats(lista, filtroAtivo)` — chamada a
partir de `renderizar()` depois de calcular `filtrados`, para que os
cartões e a tabela usem sempre exatamente o mesmo conjunto de dados.

### Correção: os 6 cartões agora são uma partição exata do Total

**Bug reportado:** faltava um cartão para o status `Reagendar` — existia
nos dados (visível no gráfico e na tabela) mas não tinha lugar no
dashboard, então `Confirmados + Não atendeu + Cancelados + Por confirmar`
nunca batia certo com o Total (ficava sempre a faltar exatamente o nº de
Ordens em `Reagendar`). Além disso, "Sem nome" estava misturado nos
mesmos 6 cartões como se fosse mais um status — mas não é: qualquer
registo, com qualquer status, pode não ter nome. Misturar as duas coisas
tornava a soma ainda mais confusa de verificar.

Correção:
- Adicionado o cartão **Reagendar**. Agora os 6 cartões somam sempre
  exatamente o Total: `Confirmados + Não atendeu + Reagendar + Cancelados
  + Por confirmar = Total` (todo registo tem exatamente um destes status,
  incluindo o "vazio" = Por confirmar).
- **Sem nome** e **Sem data** saíram dos cartões de status e passaram a
  uma linha secundária de "pills" (`#statsQualidade`), visualmente mais
  discreta, deixando claro que são métricas de qualidade de dados, não
  parte da contagem de status.

Validado com os 704 registos reais: `69 Confirmados + 48 Não atendeu + 1
Reagendar + 6 Cancelados + 580 Por confirmar = 704` — bate certo.

## Busca abrangente (inclui telefone)

A busca já não olha só para Ordem/Nome/Técnico — agora cobre também
Contacto (telefone), Morada, Cidade, Observação de origem e Observação de
validação (`bateBusca()`). Escreveres um número de telefone, um pedaço de
morada, ou uma palavra de uma nota antiga também encontra o registo.

## Gráfico de tendência — barras, Dia ou Mês

Aparece automaticamente por cima da tabela, mas **só na vista geral**
(campo de data em "Todas as datas") — esconde-se assim que filtras por um
dia específico, porque o próprio eixo X do gráfico é feito de datas.

### Redesenho (barras em vez de área/linha sobreposta)

A primeira versão usava uma área "montanha" (Confirmados) sobreposta a
uma linha fina (Total), sem eixo numérico. Ficava ilegível quando os
valores diários eram pequenos (1–3 OTs) — não dava para saber se um ponto
valia 1 ou 30 só de olhar. Passou a ser um **gráfico de barras agrupadas**:

- **Barra cinza** — total de OTs do dia/mês. **Barra verde** — quantas
  dessas estão Confirmadas. Lado a lado, com o número escrito por cima de
  cada uma (até 16 colunas; com mais, só fica o tooltip ao passar o rato).
- **Eixo Y com grelha e valores** (0 / metade / máximo), para leres a
  escala em vez de adivinhar pela altura relativa.
- **Linha tracejada laranja** — média do período mostrado, para veres se
  o dia/mês está acima ou abaixo da tendência recente.
- Reage aos filtros de busca e status: filtrar por `Cancelado` faz o
  gráfico passar a mostrar a tendência de cancelamentos, não de
  confirmações. Só o filtro de Data é ignorado nesse cálculo
  (propositadamente — é o próprio eixo do gráfico).
- Com menos de 2 colunas no conjunto filtrado, o gráfico esconde-se (não
  há tendência para comparar com 1 ponto só).
- A área do gráfico tem scroll horizontal próprio quando há muitas
  colunas, para as barras nunca ficarem espremidas a ponto de perderem a
  legenda.

### Toggle Dia / Mês

Botão no canto superior do painel alterna entre `agruparPorDia()` e
`agruparPorMes()`:

- **Dia** — até 14 colunas, mas (ver correção abaixo) escolhidas por
  proximidade a hoje, não pelas últimas cronologicamente.
- **Mês** — agrupa **todos** os meses presentes no conjunto filtrado
  (não há corte de 14), útil para ver o panorama quando os dados cobrem
  várias semanas ou meses.

### Correção: janela de dias "mais próxima de hoje", não "últimos cronologicamente"

**Bug encontrado:** a versão anterior pegava nos últimos 14 *dias
distintos* ordenados cronologicamente (`slice(-14)`). Numa OT reagendada
para daqui a 2 meses (um caso isolado), esse dia distante entrava na
janela e **empurrava para fora os dias com o volume real de trabalho**,
próximos de hoje — resultando no gráfico "achatado" com só um pico
isolado, sem nexo, que motivou este pedido de correção.

Agora `agruparPorDia()` ordena os dias disponíveis por distância a
**hoje** (`new Date()` do browser do utilizador) e escolhe os 14 mais
próximos — só depois volta a ordená-los cronologicamente para desenhar o
gráfico da esquerda para a direita. Isto garante que a janela mostrada é
sempre relevante ao trabalho atual, e não é distorcida por reagendamentos
isolados no futuro distante. Validado com um conjunto sintético: 20 dias
com volume real (3–8 OTs/dia) à volta de hoje + 1 dia isolado 60 dias no
futuro — o outlier fica corretamente de fora da vista "Dia" e só aparece
agregado no seu mês, na vista "Mês".

Implementado em `agruparPorDia()` / `agruparPorMes()` + `renderChart()`,
com um `<svg>` gerado diretamente em JS (sem biblioteca externa de
gráficos).

## Deteção de nome — Title Case além de MAIÚSCULAS

A função `detectarNome()` só reconhecia nomes quando a observação de
origem trazia o nome em MAIÚSCULAS no início (formato mais comum do
export bruto da plataforma). Ficheiros já processados por esta ou outras
ferramentas por vezes trazem o nome já em Title Case (`Joaquim Gil de
Carvalho`), que essa regra rejeitava — ficando "Sem nome" mesmo havendo
nome disponível.

Adicionado `pareceNome()`: aceita um segmento de 2 a 6 palavras, todas
capitalizadas (ou conectores `de/da/do/dos/das/e` em minúscula), sem
dígitos nem pontuação de frase (`:`, `;`, `,`, `.`, `-`, `/`, `#`, `[`,
`]`, `|`). Textos como observações de diagnóstico (`Diagnóstico: ONT OFF
com luzes`), notas com números de telefone, ou relatos de incidente
continuam corretamente a **não** ser tratados como nome — a pontuação e os
dígitos que normalmente acompanham esse tipo de texto excluem-nos.

**Validado** com os 703 registos reais fornecidos: dos 451 registos "Sem
nome", a maioria (211) não tem texto nenhum na observação de origem (nada
a recuperar), e a maior parte do restante é texto de diagnóstico/sistema
— corretamente ainda sem nome. Cerca de 23 nomes genuínos em Title Case
foram recuperados com a correção, sem introduzir falsos positivos nas
amostras verificadas manualmente.

## Fluxo de trabalho

1. **Importar** — CSV ou Excel exportado da plataforma (ou uma planilha
   "mestre" já trabalhada anteriormente).
2. **Confirmar** — por telefone, marcando o Status (Confirmado / Não
   atendeu / Reagendar / Cancelado), corrigindo o Nome se necessário, e
   escrevendo uma nota em "Observação (validação)".
3. **Exportar** — Excel (`.xlsx`, formatado) ou CSV, para partilhar ou
   arquivar.
4. **Reimportar mais tarde** — quando chega um novo export da plataforma
   (mesmas Ordens, dados atualizados), importa por cima: o que já foi
   confirmado **não se perde** (ver secção seguinte).

## Merge / cache — como o "por cima" funciona

Cada importação (CSV ou Excel) é indexada pela coluna de Ordem (Nº OT /
ORDEM). Antes de substituir a lista em memória, a app guarda os registos
atuais num mapa `ordem → registo`. Para cada linha do ficheiro novo:

| Campo           | Regra                                                                 |
|-----------------|------------------------------------------------------------------------|
| `status`        | Mantém o valor já guardado localmente. Só usa o do ficheiro se **não havia** registo local e o ficheiro trouxer um status reconhecido. |
| `nome`          | Mantém o nome já guardado localmente (podes tê-lo corrigido à mão). Caso contrário, usa a coluna Nome do ficheiro (se existir) ou tenta detetar automaticamente a partir da observação de origem. |
| `obsValidacao`  | Mantém a nota de validação já escrita localmente. Caso contrário, usa a do ficheiro (se existir essa coluna). |
| `obsOrigem`, `data`, `horario`, `contacto`, `estado`, `tecnico`, `morada`, `cidade`, `tipo` | **Sempre atualizados** com o valor mais recente do ficheiro importado (a plataforma pode ter mudado a morada, o técnico, reagendado a hora, etc.). |

Ou seja: o teu trabalho de confirmação nunca é apagado por uma reimportação,
mas os dados operacionais (data/hora/técnico/morada) ficam sempre
sincronizados com o último export.

### Correção crítica: importar nunca mais apaga Ordens que faltem no ficheiro

**Bug reportado:** ao importar um ficheiro "cru" (export da plataforma que
só trazia as Ordens novas do dia, não o universo completo), a app
**substituía** a lista inteira pelo conteúdo desse ficheiro — todas as
Ordens que já estavam a ser trabalhadas e não constavam desse export em
particular **desapareciam silenciosamente** do dashboard, da tabela e do
gráfico. Era a "alucinação" reportada: números do dashboard mudavam
drasticamente ao importar um ficheiro que devia só ter *acrescentado*
trabalho novo.

Causa: a última linha de `processarLinhas()` fazia `registos = novos`,
onde `novos` continha só as linhas encontradas *nesse* ficheiro. Um
export parcial da plataforma nunca foi pensado para representar o
universo inteiro — mas o código tratava-o sempre como se fosse.

Correção: a importação agora faz uma **união**, nunca uma substituição.
```js
const mapaFinal = new Map(registos.map(r => [r.ordem, r]));
novos.forEach(n => mapaFinal.set(n.ordem, n));
registos = [...mapaFinal.values()];
```
- Uma Ordem que já existia e **não** aparece no ficheiro importado
  continua exatamente como estava (não é tocada).
- Uma Ordem que já existia e **aparece** no ficheiro é atualizada (dados
  operacionais frescos, status/nome/validação preservados — regra da
  tabela acima).
- Uma Ordem que não existia é adicionada.

O toast pós-importação passou a refletir isto com números explícitos:
`"N Ordens novas adicionadas, M atualizadas. (P anteriores mantidas sem
alteração)"` — para nunca mais haver dúvida sobre o que uma importação
fez.

Se um dia for mesmo necessário substituir tudo (ex.: recomeçar do zero
com um ficheiro totalmente diferente), usa "Limpar tudo" antes de
importar — a importação em si nunca remove nada por conta própria.

**Validado:** com os 704 registos reais do utilizador como base +
simulação de um export parcial (3 Ordens novas, formato bruto da
plataforma) — resultado: 707 no total, as 704 antigas intactas
(confirmações incluídas), as 3 novas adicionadas. Reimportar a própria
base 704 depois disso mantém as 3 extra intocadas (`"704 anteriores
mantidas sem alteração"`).

### Correção: "Pendente" deixa de contar como "status não reconhecido"

Efeito colateral do bug acima: como cada reimportação da própria app
escreve `"Pendente"` na coluna Status de toda Ordem sem confirmação (ver
`exportXlsxBtn`), reimportar o próprio ficheiro gerava sempre um aviso
alarmante tipo `"529 status não reconhecidos"` — mas eram só `"Pendente"`
voltando, o que é normal e esperado, não um problema. Adicionado
`statusEhVazioConhecido()` para tratar `"Pendente"`/`"Por confirmar"`/`"-"`
como vazios conhecidos, não como erro de normalização.

## Formatos de ficheiro aceites na importação

A app reconhece automaticamente três variantes de cabeçalho (case- e
acento-insensível), para não obrigar a reformatar nada:

1. **Export bruto da plataforma** (CSV) — colunas como `Nº OT`,
   `Início Agendamento`, `Fim Agendamento`, `Técnico`, `Contacto`,
   `Observações`, `Estado`, `Morada`, `Cidade`, `Tipo`.
2. **Export da própria app** (CSV ou Excel) — `Ordem`, `Status`, `Data`,
   `Horário`, `Nome`, `Contacto`, `Observação origem`,
   `Observação (validação)`, `Estado`, `Técnico`, `Morada`, `Cidade`, `Tipo`.
3. **Planilhas "mestre" antigas** — variações como `ORDEM`, `STATUS`,
   `DATA`, `HORARIO`, `NOME`, `Observações origem`, etc.

Se a coluna de Ordem não for encontrada em nenhuma variante, a importação
é recusada com um aviso (nada é apagado).

### Normalização de status

Ficheiros antigos podem ter o status em maiúsculas/minúsculas
inconsistentes (`confirmado`, `Confirmado`, `NÃO ATENDEU`, `cancelado`…).
A app normaliza tudo para os 4 valores canónicos usados na UI:
`Confirmado`, `Não atendeu`, `Reagendar`, `Cancelado`. Valores não
reconhecidos (ex.: `A confirmar`) ficam como "Pendente" e são contabilizados
no aviso pós-importação, para nunca passarem despercebidos.

### Excel — leitura e escrita

- **Importar** `.xlsx`/`.xls`: lido com [ExcelJS](https://github.com/exceljs/exceljs)
  no browser. Datas em células Excel são convertidas para o formato
  `dd/mm/aaaa hh:mm` usado internamente. Só a **primeira folha** do
  workbook é lida (planilhas com abas extra, como um resumo por dia, são
  ignoradas sem erro).
- **Exportar** `.xlsx`: gerado com ExcelJS — cabeçalho a negrito com fundo
  verde, colunas com largura ajustada, primeira linha fixa (freeze),
  autofiltro, e células de Status coloridas por valor.
- **Grade (bordas)**: todas as células — cabeçalho e dados, com ou sem
  conteúdo — recebem borda fina preta (`FF000000`), formando um grid
  completo e bem definido, como uma folha de cálculo tradicional.
  (Versão anterior deixava células vazias sem contorno; o pedido seguinte
  do utilizador pediu o grid completo em preto, revertendo esse
  comportamento — ver histórico de commits se precisares da versão
  seletiva.) Validado nas 685 linhas reais do utilizador: zero células
  sem borda, zero com cor diferente de preto.

## Prioridade de contacto — a coluna "Estado" da plataforma

A coluna `Estado` (vem do export bruto da plataforma — não confundir com
`Status`, que é o campo de confirmação editável pela app) diz em que fase
do fluxo interno a Ordem está. Regra de negócio definida pelo utilizador:

| Estado da plataforma | O que significa | O que a app faz |
|---|---|---|
| **Atribuída** | Já tem técnico — é o único caso onde vale mesmo a pena ligar ao cliente. | Fica como "Por confirmar" normalmente; aparece com badge verde "vale a pena ligar" e conta no indicador **"X a ligar (Atribuída)"** do dashboard. |
| **Recebida** | O próprio operador está a criar a OT a pedido do cliente — o cliente já está à espera, não é preciso confirmar por telefone. | **Confirmado automaticamente** ao importar (só se ainda não havia nenhum status definido, para nunca sobrepor uma decisão manual). A observação de validação recebe a nota `"Confirmado automaticamente — Estado "Recebida""` para ficar rastreável. |
| Qualquer outro valor (`Não atribuída`, `Despachada`, etc.) | Ainda sem técnico atribuído — não há ninguém para visitar o cliente ainda, não vale a pena ligar já. | Fica "Por confirmar", mas a linha fica visualmente esmaecida (opacidade reduzida) e conta em **"X à espera de técnico"**. Volta ao normal assim que o Estado mudar para Atribuída ou Recebida num próximo import. |

Implementado em `prioridadeEstado(estado)` (retorna `'ligar' | 'auto' |
'espera'`) e `badgeEstado(estado)` (gera o badge colorido). A app não
tenta adivinhar "Não agendada" — essa categoria nem chega a aparecer no
export bruto da plataforma (OTs sem técnico simplesmente não têm uma
linha ali), por isso não há nada a tratar por código para esse caso.

### Filtro por Estado e indicadores no dashboard

- Novo dropdown de filtro **"Todos os estados"**, ao lado dos filtros
  existentes — selecionar "Atribuída" isola exactamente a lista de
  quem precisa de uma chamada agora.
- Duas pills novas na linha de qualidade de dados: **"X a ligar
  (Atribuída)"** (destacada, cor de acento) e **"X à espera de
  técnico"** — ambas contam só entre os registos **ainda sem status**
  (`!r.status`), para não incluir Ordens já tratadas.
- A coluna "Estado" passou a aparecer também na tabela em ecrã (antes só
  existia no export Excel), com o mesmo badge colorido.

**Validado** com o ficheiro real do utilizador (685 Ordens, 4 estados:
Atribuída 426, Não atribuída 142, Recebida 59, Despachada 58): as 59
"Recebida" ficaram todas com status "Confirmado" automaticamente, zero
"Atribuída" foi tocada, indicador "a ligar" mostrou exatamente 426,
"à espera de técnico" mostrou exatamente 200 (142+58).

## Validação feita (dados reais, 2026-08-25)

Testado com os ficheiros fornecidos pelo utilizador:

- `Ordem de Trabalho (2).csv` — export bruto da plataforma, 704 Ordens.
  Todas as colunas esperadas (`Nº OT`, `Início Agendamento`, etc.)
  reconhecidas sem alterações.
- `folha_mestre_confirmacao_1.xlsx` — planilha mestre com 703 Ordens e uma
  segunda aba de resumo (`Resumo_por_Dia`, ignorada corretamente). Continha
  59 registos já confirmados com status em capitalizações mistas
  (`Confirmado`/`confirmado`, `cancelado`, etc.) — todos normalizados
  corretamente para os 4 status canónicos.

Cenário validado ponta-a-ponta:
1. Importar o `.xlsx` mestre → 703 registos, 59 com status preservável.
2. Importar por cima o `.csv` bruto da plataforma (dados mais recentes) →
   continuam 703 registos, os mesmos 59 com status e nome mantidos, e a
   observação de origem atualizada com o texto mais recente da plataforma.
3. Exportar para `.xlsx` e reimportar esse mesmo ficheiro → nenhuma perda
   de dados (round-trip completo).

Sem erros de consola em nenhum dos passos.

## Deploy

```bash
cd davi-confirmacoes
npx vercel --prod
```

Site 100% estático — não precisa de variáveis de ambiente nem de build step.

## Limitações conhecidas / próximos passos

- Dados são por-browser/por-dispositivo (sem sincronização entre
  computadores). Planeado para uma versão futura com base de dados.
- A deteção automática de nome (`detectarNome`) assume que o nome do
  cliente aparece em maiúsculas no início da observação, separado por `|`.
  Pode falhar em formatos de observação muito diferentes dos vistos até
  agora — nesses casos, o campo Nome fica vazio para preenchimento manual.
