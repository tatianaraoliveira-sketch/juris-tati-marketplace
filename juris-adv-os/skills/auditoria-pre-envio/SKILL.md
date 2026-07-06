---
name: auditoria-pre-envio
description: >
  AUDITORIA-PRE-ENVIO — Auditoria 4D obrigatoria antes do advogado citar
  jurisprudencia em qualquer peca, contestacao, parecer ou e-mail.
  Verifica (1) se o conteudo da decisao foi efetivamente analisado
  (inteiro teor ou ementa oficial), (2) se cada link da fonte foi
  confirmado por fetch real, (3) se o entendimento tem nexo de
  pertinencia com a tese/pedido do usuario, e (4) se o precedente
  permanece vigente (nao superado, nao sobrestado). Emite veredito por
  citacao: APROVADA / REVISAR / BLOQUEADA. Use SEMPRE antes de inserir
  jurisprudencia em peca final.
---

# AUDITORIA-PRE-ENVIO — Quatro Camadas Anti-Erro

## 1. POR QUE EXISTE

A `validar-jurisprudencia` confirma que **o link e a citacao sao reais**.

Esta skill vai alem: confirma que **a jurisprudencia citada realmente
sustenta a tese do usuario E continua valendo** — porque jurisprudencia
validada mas contraria a tese, ou ja superada, e tao perigosa quanto
inventada.

E sua linha de defesa final antes do "protocolar".

---

## 2. INPUT NECESSARIO

1. **Tese / pedido do usuario** — o que ele quer sustentar (1-2 frases).
2. **Lista de citacoes que ele pretende usar** — com URLs e ementas.
3. **(Opcional) Peca em rascunho** — para conferir onde cada citacao entra.

Se faltar (1) ou (2), pergunte UMA vez. Nao prossiga sem ter os dois.

---

## 3. WORKFLOW — QUATRO CAMADAS

**Reuso de fetch:** se a `validar-jurisprudencia` rodou NESTA MESMA
sessao e capturou o conteudo da pagina, REUTILIZE esse conteudo — nao
refaca o fetch. Faca fetch novo apenas se (a) a validacao nao capturou
conteudo suficiente, ou (b) a validacao ocorreu em sessao anterior.

Para CADA citacao da lista:

### CAMADA 1 — O conteudo da decisao foi analisado?

**Acao:**
- Obtenha o conteudo (reuso da sessao ou WebFetch; se a URL for PDF,
  fetch do PDF + extracao de texto).
- Capture o maximo disponivel do voto/acordao e identifique:
  - **Fato base do caso** (em 1 frase)
  - **Fundamento legal central** (qual lei/artigo/sumula)
  - **Razao de decidir** (ratio decidendi, em 1-2 frases)
  - **Distincoes/excecoes** que o tribunal apontou

Classifique a base da analise:
- `INTEIRO TEOR` — voto/acordao integral analisado
- `EMENTA OFICIAL` — apenas ementa, mas obtida/validada em fonte oficial
- `EMENTA NAO OFICIAL` — apenas ementa, de agregador

### CAMADA 2 — Link confirmado por fetch real?

Use o status da `validar-jurisprudencia` (✅ / ⚠️ / 🔴). Se nao houver
validacao previa, valide agora conforme as regras daquela skill —
inclusive a distincao entre falha de acesso (⚠️) e evidencia de
inexistencia (🔴).

| Status | Tratamento na auditoria |
|--------|-------------------------|
| ✅ | Prossegue |
| ⚠️ | Prossegue com ressalva no veredito (indicando se foi falha de acesso ou divergencia) |
| 🔴 | **BLOQUEAR** imediatamente. Nao prossegue. |

### CAMADA 3 — Nexo de pertinencia com a tese do usuario?

**Estruture explicitamente** (nao "feeling"):

```markdown
**Tese do usuario (1 frase):**
> [transcricao da tese que ele defende]

**Tese central da jurisprudencia (1 frase):**
> [o que o tribunal decidiu, em sintese]

**Alinhamento:**
- [ ] PRO     — jurisprudencia sustenta diretamente a tese do usuario
- [ ] CONTRA  — jurisprudencia decide em sentido oposto
- [ ] NEUTRO  — tema parecido mas nao decide a questao em jogo
- [ ] DEPENDE — alinhamento depende de fato X estar presente no caso do usuario

**Distincao relevante:**
[se algum fato do caso julgado diverge do caso do usuario de forma que
pode comprometer a aplicacao analogica, descrever em 1-3 linhas]

**Risco se citado:**
[1 frase. Ex: "baixo" / "medio — pode ser distinguishing pelo juiz" /
"alto — ver Camada 4"]
```

### CAMADA 4 — O entendimento permanece vigente?

Precedente PRO, porem superado ou sobrestado, e risco alto. Verifique:

1. **Superacao:** ha decisao posterior do mesmo tribunal ou de tribunal
   superior em sentido contrario? Busca dirigida:
   `WebSearch: "[tema/tese]" superacao OR "novo entendimento" [tribunal] [ANO CORRENTE]`
2. **Afetacao/sobrestamento:** o tema esta afetado a repetitivo (STJ) ou
   repercussao geral (STF) com determinacao de suspensao? Busca dirigida:
   `WebSearch: "Tema" "[palavras-chave]" site:portal.stf.jus.br` e
   `site:scon.stj.jus.br` (paginas de temas).
3. **Modulacao:** se tese vinculante, ha modulacao de efeitos que exclua
   o caso do usuario?

Registre o resultado com honestidade epistemica:
- `SEM INDICIO DE SUPERACAO LOCALIZADO` (nas buscas feitas — nao e garantia)
- `TEMA PENDENTE/SOBRESTADO — [Tema n]` (risco alto, informar)
- `SUPERADO POR [decisao]` (bloquear)
- `CHECAGEM INCONCLUSIVA` (ambiente nao permitiu verificar — declarar)

**NUNCA inventar tema ou decisao superadora. Se a busca nada retornou,
o registro e "sem indicio localizado", nao "garantidamente vigente".**

### Veredito por citacao

| Veredito | Quando emitir |
|----------|---------------|
| ✅ **APROVADA (inteiro teor)** | Camada 1 = INTEIRO TEOR + Camada 2 ✅ + Camada 3 PRO risco baixo + Camada 4 sem indicio de superacao |
| ✅ **APROVADA (base: ementa oficial)** | Camada 1 = EMENTA OFICIAL + Camada 2 ✅ + Camada 3 PRO risco baixo + Camada 4 sem indicio — com nota "conferir inteiro teor antes de protocolar" |
| 📝 **REVISAR** | 1-2 ressalvas: ementa nao oficial / Camada 2 ⚠️ / nexo DEPENDE / risco medio / Camada 4 inconclusiva |
| 🚫 **BLOQUEADA** | Camada 2 = 🔴 OU Camada 3 = CONTRA OU Camada 4 = superado/sobrestado com suspensao OU risco alto |

---

## 4. OUTPUT FINAL — TEMPLATE

```markdown
## Auditoria pre-envio — [N] citacoes

**Tese do usuario:** [transcricao]
**Data da auditoria:** [DATA E HORA REAIS DA SESSAO]

---

### Citacao 1 — [Tribunal · numero · relator]

**🔍 Camada 1 — Conteudo:** INTEIRO TEOR / EMENTA OFICIAL / EMENTA NAO OFICIAL
- Fato base: [...]
- Fundamento central: [...]
- Razao de decidir: [...]

**🔗 Camada 2 — Link:** ✅ / ⚠️ / 🔴 (fonte: <URL>; [se ⚠️: acesso ou divergencia])

**⚖️ Camada 3 — Pertinencia:**
- Tese do usuario: [transcricao]
- Tese da juris: [sintese]
- Alinhamento: PRO / CONTRA / NEUTRO / DEPENDE
- Distincao: [se houver]
- Risco: [baixo/medio/alto]

**⏳ Camada 4 — Vigencia:** [sem indicio de superacao localizado / tema pendente / superado / inconclusiva]

**VEREDITO:** ✅ APROVADA (inteiro teor) | ✅ APROVADA (ementa oficial) | 📝 REVISAR | 🚫 BLOQUEADA

**Recomendacao de uso:** [como integrar ou por que nao integrar]

---

[repete para cada citacao]

---

## Resumo final

| # | Tribunal | Veredito | Acao |
|---|----------|----------|------|
| 1 | STJ | ✅ | Pode citar — fundamento principal |
| 2 | TJSP | 📝 | Citar com ressalva — suporte secundario |
| 3 | TRF3 | 🚫 | NAO citar — contraria a tese |

**Aprovadas:** [N] · **Para revisar:** [N] · **Bloqueadas:** [N]

---

📋 **REGISTRO DE DILIGENCIA** (cole no dossie do caso):
[tabela compacta: citacao · URL · status validacao · veredito auditoria ·
data/hora — documenta que a conferencia assistida foi realizada e quando]

---

⚠️ **AVISO LEGAL:** Esta auditoria e assistida por IA. A conferencia
humana final, especialmente do inteiro teor e da pertinencia ao caso
concreto, e responsabilidade exclusiva do advogado antes de protocolar.
```

---

## 5. PROIBICOES ABSOLUTAS

1. Nao emitir ✅ sem conteudo real analisado (da sessao ou de fetch novo).
2. Nao avaliar pertinencia "no olho" — sempre transcrever a tese do
   usuario e a tese da jurisprudencia LITERAIS e comparar estruturadamente.
3. Nao misturar citacoes — auditar UMA POR UMA com saida estruturada.
4. Nao ocultar BLOQUEADAS — destacar no resumo final.
5. Nao afirmar vigencia como certeza — o maximo honesto e "sem indicio
   de superacao localizado".
6. NUNCA omitir o aviso legal final.
