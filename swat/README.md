# SWAT — Planeamento ETS

**Ferramenta interna DST** · Uso exclusivo da equipa RC  
Ficheiro único `index.html` — funciona offline, dados guardados no browser (localStorage).

---

## O que é

O SWAT substitui os 5+ Excel abertos em simultâneo usados no planeamento diário das ETS.  
Centraliza numa só interface:

| Tab | Função |
|-----|--------|
| 📋 **Painel do Dia** | Vista diária da Agenda RC por Região e Técnico, com status de cada OT |
| 🔍 **Lista de Espera** | Pesquisa e filtragem das OTs disponíveis (regra +3 dias) |
| 👷 **Equipas** | Diretório de equipas com contactos e zonas de atuação |
| 📊 **Gantt Semanal** | Timeline visual das equipas por hora (9h–19h), importado do Planeamento FF DROP |
| 🗺️ **Mapa** | Mapa dark das instalações e posição das equipas em tempo real por hora |

---

## Ficheiros necessários

| Ficheiro | Onde importar |
|----------|---------------|
| `Piloto Agenda Diária RC.xlsx` | Tab Painel do Dia |
| `Lista Espera CGO.xlsx` | Tab Lista de Espera |
| `Dados das equipas RC.xlsx` | Tab Equipas |
| `Planeamento_FF_DROP_W*.xlsb` ou `.xlsx` | Tab Gantt Semanal |
| `norte.kmz` / `centro.kmz` / `sul.kmz` (rede FO) | Tab Mapa (opcional — desbloqueia coordenadas) |

---

## Como usar — fluxo diário

1. Abre `swat/index.html` (ou o link **⚡ SWAT** na Central de Confirmações)
2. **Painel do Dia** → importa a Agenda Diária RC → vês todas as OTs do dia por técnico
3. **Lista de Espera** → importa a lista CGO → filtras por zona, data (+3 dias), tipo
4. **Gantt** → importas o Planeamento da semana → escolhes o dia → vês todas as equipas hora a hora
5. **Mapa** → importas o KMZ da rede → os POPs do Gantt ganham coordenadas → moves o slider de hora e vês onde cada equipa está
6. Clica num slot do Gantt → vês cliente + telemóvel da Lista de Espera correlacionado automaticamente

---

## Funcionalidades do Gantt

- **4 linhas por equipa**: OT / Tipo de serviço / POP / Observações (estrutura real do Excel)
- **Cores por operador**: MEO=azul · VDF=vermelho · NPR=verde · NOS=roxo · NOWO=laranja
- **Pesquisa rápida**: pesquisa por POP, OT, equipa, gestor, VDF/MEO, cliente, telemóvel — filtra em tempo real com highlight amarelo nos slots correspondentes
- **Correlação Lista Espera**: hover num slot → tooltip com nome do cliente + tlm; clique → modal com dados completos
- **Edição de slots**: clica → edita tipo, OT, POP, obs → guardado em localStorage
- **Exportar XLSX**: exporta o dia ativo com todas as edições aplicadas

## Funcionalidades do Mapa

- Mapa dark sem API key (OSM invertido)
- **Instalações**: círculos coloridos por operador (da Lista de Espera), popup com cliente + OT + tlm
- **Equipas**: marcadores no mapa à hora do slider, posição extraída do POP do Gantt → coordenadas via KMZ
- **Rede KMZ**: overlay da rede de fibra (norte/centro/sul)
- **Slider 9h–19h**: scrub pelo dia, equipas movem-se em tempo real

---

## Dados e privacidade

- Todos os dados ficam **apenas neste browser** (localStorage)
- Nada é enviado a servidores
- Cada browser/dispositivo tem o seu próprio estado
- Limpar o cache do browser apaga os dados importados

---

## Roadmap

- [ ] Planeamento automático de rotas por zona (otimização de sequência de visitas)
- [ ] Integração com disponibilidade de clientes (reagendamentos automáticos)
- [ ] Notificações de equipas com slots vazios disponíveis
- [ ] Exportar rota para GPS diretamente do mapa

---

*Desenvolvido para uso interno DST · Projeto Davi 3.0*
