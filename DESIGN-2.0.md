# Orbital 2.0 — Design Spec

> **Escrito por Fable 5 em 20/jul/2026** a partir de revisão visual da UI real (desktop/mobile/dark) + painel de lentes (Jobs, Godin, Lenny, Ferriss, neurociência 2e). **Implementação: Opus 4.8/Sonnet 5** — este doc existe para que nenhuma sessão futura precise re-derivar o raciocínio. Em dúvida de interpretação, a FILOSOFIA ganha do detalhe.

---

## 0. Filosofia (a sentença que governa tudo)

O Orbital 1.x é um **espelho** (responde "o que está acontecendo?"). O 2.0 é um **motor de arranque**: seu trabalho é transformar intenção em primeira ação em **menos de 60 segundos**, para um dono com perfil 2e (TDAH desatento + AH/SD) cujo gargalo documentado não é saber — é **começar** e **fechar**.

A convergência do painel, numa frase:

> A tela da manhã tem UM item, escolhido pela pergunta do dominó, formulado como desafio com tempo visceral, com o rascunho do Executor a um toque, protegida de toda distração antes das 9h — e o sistema mede uma única coisa: **fechamentos por semana**.

**North star: fechamentos/semana.** Métricas de suporte: ativação (segundos até 1ª ação do dia, meta <60s) · dias com brief aberto E ação (abertura sem ação = doomscrolling de produtividade) · streak de remessas.

### Princípios inegociáveis (herdados do 1.x — NÃO regredir)
1. **Confiança auditável**: banner de staleness, prova de cobertura (🛰️), write-back done:/push:/note: com ids idempotentes. Qualquer mudança preserva esses contratos.
2. **Arquivo único, zero framework, zero build.** A simplicidade É a feature.
3. **Dark mode com paridade total.**
4. **Nada é enviado a terceiros pela UI.** Copiar rascunho ≠ enviar; o envio é sempre ato humano no app de destino.

### Anti-princípios (o que matou os 4 protótipos de abr/mai — não repetir)
- Redundância como ênfase (item repetido 4× anestesia, não enfatiza).
- Ranquear por importância (P1) — o cérebro 2e ativa por **interesse/desafio/urgência-real/novidade**, não por dever. "P1" é etiqueta moralmente pesada e neurologicamente inerte.
- Vaidade métrica (contagem de páginas wiki não muda decisão nenhuma).
- Linguagem de débito/vergonha ("24 dias travado" como acusação). Vergonha rola feed; identidade age.

---

## 1. Modo Manhã (antes das 9h) — a mudança central

Antes das 9h (hora local), a página renderiza **apenas**:

1. **Linha editorial do CoS** (voz preservada, citação literal do brief — ex: *"item pronto que não foi enviado — teu padrão nº 1"*). Uma linha. É a razão de abrir a página.
2. **O Card Única Coisa** (§2).
3. **Streak de remessas** (§4), discreto.
4. **🧠 Faísca do dia** (§6), pequena, no rodapé.
5. Link discreto: **"ver o resto do dia →"** — expande a visão completa (seta `sessionStorage.orbital_full=1`; a visão completa é o 1.x atual). Depois das 9h, visão completa por padrão.

Caixa e Métricas **não são renderizadas** antes das 9h — nem colapsadas; ausentes (dieta de baixa informação do Ferriss). Se algo da caixa é urgente de verdade, é trabalho do CoS promovê-lo ao brief — não da UI exibir a caixa inteira.

## 2. Card Única Coisa

Anatomia (de cima para baixo):

- **Enquadramento como desafio, não dever**: `"Dá pra matar em 5 min?"` + título. (Neuro: desafio ativa; dever adia.)
- **Tempo visceral**: barra de envelhecimento colorida por dias parado (verde <3d → âmbar 3-10d → vermelho 10d+ com pulso sutil) + consequência nomeada em linguagem humana: *"Noemi trabalha há 24 dias sem contrato — cada semana fica mais constrangedor."* NUNCA só "(24d)".
- **Ações, nesta ordem**:
  - `[📋 copiar rascunho]` — se houver draft do Executor (§5). Um toque = clipboard. Este botão é acomodação neurológica (RSD): o custo emocional de ESCREVER a cobrança é maior que o de enviá-la.
  - `[feito ✓]` — dispara captura `done:` (contrato existente) + celebração sóbria (micro-animação ≤600ms; sem confete infantil — o dono é executivo sênior).
  - `[→ amanhã]` — **com custo progressivo** (§3).
- **Fear-setting inline** (só para itens marcados `confronto: true` pela rotina): link "dimensionar o medo" abre 3 perguntas no modal: *pior caso realista? · é reversível? · custo de NÃO fazer em 6 meses?* Sem persistência — o valor é o ato de responder.

**Seleção do item**: campo `briefing.one_thing` emitido pela rotina (ver §7 — pergunta do dominó). Fallback se ausente: `focus_today[0]`.

**Lote (batching)**: se a rotina emitir `briefing.batches` (ex: `{title:"5 cobranças prontas", minutes:10, items:[...]}`), o lote é apresentado como UM card com UM check — "10 min, 5 mensagens, zero produção". O 2e fecha lotes com prazer; abre 5 itens com dor.

## 3. Economia de fechamento (dopamina redesenhada)

- **Placar "Fechados esta semana"** — separado do "✓ hoje". Só conta atos de encerramento (contrato enviado, entrega feita, decisão comunicada) — a rotina marca itens `close: true`. Os fechamentos pendentes de junho aparecem como "chefes de fase" (lista curta, derrotável).
- **Custo progressivo do adiamento**: contador persistente de pushes por item (`localStorage.orbital_pushcount`, NÃO zerado por dia). 1º push: normal. 2º: botão amarelo. 3º+: o botão vira **"matar ou manter?"** — matar dispara captura `kill: <título>` (a rotina remove/arquiva a task e o Conselheiro registra o óbito declarado). Espelha o achado "óbito não declarado" — a UI força a honestidade que a lista esconde.
- **Envelhecimento visual** nas tasks da visão completa: mesmo gradiente de idade do Card Única Coisa. Cinco P1s idênticos = nenhum P1; a idade diferencia.

## 4. Copy — as regras de voz (Godin)

Toda string nova segue:
1. **Identidade, não débito**: "Você é o cara que fecha. Este leva 5 min." — nunca "você deve X há Y dias".
2. **Verbos de remessa**: enviar, fechar, despachar, matar — nunca "processar", "revisar", "ver".
3. **≤2 linhas por item.** O brief inteiro ≤300 palavras (regra já existente no CLAUDE.md do motor).
4. **Cada palavra paga a abertura de amanhã** — na dúvida, corta.
5. **Streak**: "3 dias seguidos entregando — não quebre a corrente." Quebrou? Sem drama: "corrente nova começa hoje."

## 5. Loop Executor → manhã (a integração de maior alavancagem)

Hoje os rascunhos do Executor (18h15) morrem numa notificação. Contrato novo:

1. **Executor** (task local): além do relatório, POSTa cada rascunho na fila de capturas — `POST /api/orbital/capture` com `{id:"draft-<data>-<slug>", text:"draft[<task-key>]: <corpo do rascunho>", source:"executor"}`. Idempotente pelo id (contrato existente do endpoint).
2. **Rotina noturna (ingest)**: ao montar o brief, tria capturas `draft[...]` e as embute no payload: `briefing.drafts = { "<task-key>": "<corpo>" }`, marcando as capturas como processadas. Drafts NÃO viram tasks — são anexos de tasks existentes.
3. **Dashboard**: item que tenha draft correspondente ganha `[📋 copiar rascunho]` (clipboard API; fallback: modal com o texto selecionado).

Resultado: às 7h, no pico, fechar = colar e enviar. Ataca o blocker nº 1 na raiz.

## 6. Proteções e alimento (neurociência)

- **Bloco sagrado 8h–9h**: se livre na agenda, renderizar como `🔒 deep work` (não "slot livre"). A rotina é instruída a nunca sugerir reunião ali e a **denunciar em 1 linha** quando alguém agendar por cima.
- **Slots úteis, não censo de slots**: eliminar "19 slots livres · 9h". Mostrar no máximo O próximo slot acionável: "30 min antes do 1:1 — dá pra matar a cobrança do Rohit."
- **🧠 Faísca do dia** (`briefing.spark = {title, link}`): uma ideia da semana (captura promovida, página nova do wiki) no rodapé do modo manhã. O lado AH/SD precisa de alimento intelectual visível — é o que faz o dono voltar por prazer, não por dever.
- **AGORA pós-19h**: o strip cede lugar ao ritual noturno (já existe). Nunca mostrar countdown de "em 9h58min" para o dia seguinte.

## 7. Contrato de payload (rotina → dashboard) — campos novos

Todos opcionais; o dashboard renderiza o que existir (padrão progressivo já usado por `vip_radar`/`coverage`/`fleet`):

```json
{
  "one_thing": {"key":"noemi", "title":"...", "minutes":5, "why":"consequência humana", "confronto":false, "close":true, "stalled_days":24},
  "editorial": "citação literal do CoS, 1 linha",
  "drafts": {"noemi":"corpo do rascunho..."},
  "batches": [{"title":"5 cobranças prontas","minutes":10,"items":["Rohit","João","Julliana","Kelly","Erick"]}],
  "spark": {"title":"Remotion: vídeo como código","link":"..."},
  "closes_week": {"count":2, "bosses":["Zenaide","GlobalCamp"]}
}
```

Lado da rotina (CLAUDE.md §10 do second-brain, regra 8 — já escrita): o `one_thing` é escolhido pela **pergunta do dominó**: *"qual item, feito hoje, torna os outros mais fáceis ou irrelevantes?"*

## 8. Instrumentação (Lenny) — o sistema aprende do uso

- Registrar localmente (`localStorage.orbital_metrics_<data>`): timestamp de abertura, timestamp da 1ª ação, ações totais, cards expandidos/ignorados, pushes.
- **Sexta à noite** (gatilho: primeira abertura após 18h de sexta, OU junto do ritual de fechar o dia): o dashboard POSTa um resumo semanal como captura `metrics: {json compacto}`.
- A rotina arquiva em `outputs/metrics/`; o **Conselheiro** lê na consolidação e propõe: matar cards ignorados 10+ dias, ajustar a seleção do one_thing se a ativação passar de 60s, etc. O produto passa a ter um growth analyst semanal de graça.

## 9. Cortes (a lista dos mil nãos)

- ❌ Card **Métricas** (open_p1/wiki_pages/decisions) — vaidade; o lugar disso é o relatório do Conselheiro. Manter apenas a linha de cobertura 🛰️ (é confiança, não vaidade) — movê-la para o rodapé.
- ❌ Censo de slots ("N slots livres · Xh").
- ❌ Countdown noturno no AGORA.
- ❌ Repetição do mesmo item em 4 cards: na visão completa, um item que é o one_thing aparece SÓ no card Única Coisa; Central/Tasks o omitem (dedupe por key).
- 🔇 Emojis de header: manter como âncora, mas hierarquia visual = AGORA/Única Coisa dominantes; demais headers -20% de peso (menor, mais claro).

## 10. Fases de implementação

**Onda 1 — dashboard puro (Opus 4.8; ~1 sessão).** Modo manhã + Card Única Coisa (fallback focus_today[0]) + custo progressivo do push + envelhecimento visual + copy pass (§4) + cortes (§9) + instrumentação local (§8, sem o POST semanal). Sem dependência de rotina.
**Onda 2 — rotina (prompt; já feita em 20/jul junto deste spec).** CLAUDE.md §10 regra 8: dominó, one_thing, editorial, drafts, batches, spark, closes_week, bloco sagrado. A rotina adota sozinha (lê o CLAUDE.md).
**Onda 3 — loops (Opus 4.8; ~1 sessão).** Executor POSTa drafts (§5.1) + dashboard consome `briefing.drafts` + POST semanal de métricas + Conselheiro instruído a ler `outputs/metrics/`.

### Critérios de aceite (por onda)
- O1: antes das 9h só modo manhã renderiza; 1ª ação possível em ≤2 toques; push 3× exige decisão; zero regressão no write-back (ids `done-`/`push-` idênticos); dark ok; teste com o harness de interceptação de fetch (ver §11).
- O2: brief do dia seguinte contém `one_thing` e `editorial`; bloco 8-9h respeitado.
- O3: rascunho do Executor de segunda aparece como botão na terça de manhã; captura `metrics:` chega na sexta.

## 11. Como testar (harness usado na revisão de 20/jul)

Servidor local: config `orbital-dash` no launch.json do Cowork (porta 8643). No console da página: monkeypatch de `window.fetch` devolvendo snapshot simulado + `#tok` fake + click em Entrar — exercita o pipeline real (render, história, cliques) sem token de produção. Validar sintaxe antes: extrair `<script>` e rodar `node --check`. Sempre testar: desktop + 375px + dark + o caminho de re-render (clicar um checkbox re-renderiza tudo — foi onde o bug do fleetHist apareceu).
