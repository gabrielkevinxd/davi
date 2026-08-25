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

Os 5 cartões no topo (Total, Confirmados, Não atendeu, Por confirmar, Sem
nome) refletem **sempre o conjunto atualmente filtrado**, não a lista
completa. Isto aplica-se aos três filtros existentes e às suas combinações:

- Busca por texto (Ordem / Nome / Técnico)
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
