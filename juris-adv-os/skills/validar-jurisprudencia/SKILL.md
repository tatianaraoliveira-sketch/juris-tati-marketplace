---
name: validar-jurisprudencia
description: >
  VALIDAR-JURISPRUDENCIA — Confirma cada citacao retornada por
  buscar-jurisprudencia via WebFetch real na URL declarada. Verifica se a
  pagina retorna conteudo legivel, se contem o numero do processo, e se a
  ementa transcrita efetivamente aparece no texto. Emite status
  ✅ VALIDADA | ⚠️ PARCIAL | 🔴 NAO VALIDADA, distinguindo divergencia de
  conteudo de mera falha de acesso do ambiente. NUNCA emite ✅ sem fetch
  bem-sucedido. Auto-disparada apos buscar-jurisprudencia. Use tambem
  manualmente para validar citacoes que o usuario trouxe de outras fontes.
---

# VALIDAR-JURISPRUDENCIA — Confirmacao Anti-Halucinacao via Fetch Real

## 1. PRINCIPIO INVIOLAVEL

**Nenhuma citacao recebe ✅ sem que o WebFetch na URL retorne conteudo
substantivo E o trecho de ementa transcrito apareca de fato na pagina.**

Corolario igualmente inviolavel: **nenhuma falha do SEU ambiente de
fetch pode ser apresentada como prova de que a citacao nao existe.**
Nao afirmar o que nao se verificou — em NENHUMA das duas direcoes.

Este e o coracao do plugin. Nunca fingir validacao. Nunca "lembrar" um
status HTTP que voce nao viu — a ferramenta de fetch nao expoe codigo
HTTP confiavel; trabalhe apenas com o que e observavel: o CONTEUDO
retornado.

---

## 2. WORKFLOW DE VALIDACAO

Para CADA citacao recebida da lista da `buscar-jurisprudencia` (ou da
lista que o usuario pediu para validar):

### Passo 1 — WebFetch da URL declarada e classificacao do desfecho

```
WebFetch(url="<URL_DA_CITACAO>")
```

Classifique o desfecho em UMA de quatro categorias observaveis:

| Desfecho | Evidencia observavel | Tratamento |
|----------|----------------------|------------|
| **CONTEUDO LEGIVEL** | Texto substantivo retornado | Prossiga ao Passo 2 |
| **PAGINA VAZIA / JS-HEAVY** | Fetch "funcionou" mas retornou pagina vazia, casca de aplicacao, ou aviso de JavaScript (padrao conhecido de STF e STJ/SCON) | ⚠️ com nota `pagina exige render JS — citacao NAO desmentida; validacao manual no portal` |
| **BLOQUEIO DE AMBIENTE** | Fetch recusado pelo proprio ambiente, timeout de rede, dominio restrito | ⚠️ com nota `fonte inacessivel neste ambiente — citacao NAO desmentida; validacao manual obrigatoria` |
| **PAGINA INEXISTENTE / CONTEUDO DIVERGENTE** | Pagina de erro do tribunal, "documento nao encontrado", ou conteudo legivel que NAO contem o processo | Prossiga ao Passo 2 (tendera a 🔴) |

Se a URL apontar para **PDF** (inteiro teor costuma ser PDF), faca o
fetch do PDF e extraia o texto — PDF legivel conta como CONTEUDO LEGIVEL.

### Passo 2 — Checar 3 evidencias no conteudo (apenas se legivel)

| Evidencia | Verificacao |
|-----------|-------------|
| **(a) Numero do processo** | O numero declarado aparece literal no texto retornado? |
| **(b) Ementa transcrita** | O trecho de ementa (3-8 linhas) aparece literal ou com diferencas minimas (acentos/quebra de linha)? |
| **(c) Metadados** | Orgao julgador, relator e data batem com o que esta na pagina? |

### Passo 3 — Atribuir status

| Status | Quando emitir |
|--------|---------------|
| ✅ **VALIDADA** | Conteudo legivel + numero do processo presente + trecho de ementa presente (literal ou ~95%) + metadados batem. |
| ⚠️ **PARCIAL** | (i) pagina vazia/JS-heavy ou bloqueio de ambiente (citacao nao desmentida); OU (ii) conteudo legivel mas ementa parafraseada / metadados divergem em 1 campo / fonte e agregador. |
| 🔴 **NAO VALIDADA** | Conteudo legivel em que o numero do processo NAO aparece, OU pagina de erro "documento inexistente" do proprio tribunal, OU ementa flagrantemente divergente do transcrito. |

**Regras de bloqueio:**
- **NUNCA** emita ✅ se algum dos 3 itens (a/b/c) falhar. Em duvida, rebaixa pra ⚠️.
- **NUNCA** emita 🔴 por mera falha de acesso — 🔴 exige evidencia POSITIVA
  de inexistencia ou divergencia. Falha de acesso = ⚠️ com nota.

### Passo 4 — Caso especial: agregador com link pro tribunal

Se a fonte original era AGREGADOR (JusBrasil/Escavador) e o agregador
tem link "Ver no [tribunal]" ou link pra inteiro teor oficial:
1. Faca um segundo WebFetch nesse link oficial.
2. Se confirmar a citacao na fonte oficial, emita ✅ + observacao
   `confirmado tambem na fonte oficial: <URL>`.
3. Se nao confirmar, emita ⚠️ `agregador divergente do oficial`.

### Passo 5 — Saida estruturada

Para cada citacao, produza:

```markdown
### [N] [Citacao original — Tribunal · numero]

- **Status:** ✅ VALIDADA | ⚠️ PARCIAL | 🔴 NAO VALIDADA
- **Fonte declarada:** <URL>
- **Desfecho do fetch:** conteudo legivel | pagina vazia/JS | bloqueio de ambiente | pagina inexistente
- **Numero do processo presente na pagina:** sim | nao | nao verificavel
- **Ementa transcrita presente na pagina:** sim | parafraseada | nao | nao verificavel
- **Metadados batem:** sim | divergencia em [campos] | nao verificavel
- **Observacao:** [ex: "agregador, conferir no tribunal", "redirect para nova URL", "pagina exige render JS", etc]
- **URL alternativa (se descoberta):** [se aplicavel]
```

**Guarde o conteudo obtido nesta sessao** — a `auditoria-pre-envio`
pode reutiliza-lo sem refazer o fetch (ver secao 5).

### Passo 6 — Relatorio final

Apos validar TODAS as citacoes:

```markdown
## Relatorio de validacao — [N] citacoes

| # | Citacao | Status | Motivo | Acao recomendada |
|---|---------|--------|--------|------------------|
| 1 | [Tribunal · numero] | ✅ | conteudo confirmado | Pronta para citar |
| 2 | [Tribunal · numero] | ⚠️ | pagina JS-heavy | Conferir manualmente em <URL> |
| 3 | [Tribunal · numero] | 🔴 | processo nao consta na pagina | NAO CITAR sem validacao manual |

**Resumo:** ✅ N | ⚠️ N (dos quais [N] por falha de acesso, nao por divergencia) | 🔴 N

**Bloqueios criticos:** [lista de citacoes 🔴 que NAO devem ir pra peca sem validacao humana]

---

⚠️ **AVISO LEGAL:** Validacao assistida por IA. A conferencia humana
final, especialmente do inteiro teor e da pertinencia ao caso concreto,
e responsabilidade exclusiva do advogado antes de protocolar.
```

---

## 3. PROIBICOES ABSOLUTAS

1. **NUNCA** emitir ✅ sem fetch com conteudo legivel confirmando numero + ementa.
2. **NUNCA** emitir 🔴 sem evidencia positiva de inexistencia/divergencia — falha de acesso e ⚠️.
3. **NUNCA** "completar" uma citacao com informacoes que voce nao viu na pagina.
4. **NUNCA** silenciar um 🔴 — sinalize destacado no relatorio.
5. **NUNCA** afirmar "validada via google" ou "validada via memoria do modelo" — so via WebFetch real.
6. **NUNCA** reportar codigo HTTP que a ferramenta nao exibiu.

---

## 4. FALLBACK COM MCPs OPCIONAIS

Para STF e STJ (portais JS-heavy verificados), um MCP com renderizacao
e **fortemente recomendado** — sem ele, essas citacoes tenderao a ⚠️
por limitacao do ambiente, nao por defeito da citacao.

Se o usuario tiver **firecrawl** MCP (ou equivalente de scraping com
render JS) configurado, use como fallback quando o WebFetch nativo
retornar pagina vazia. Se o usuario tiver **perplexity** MCP, PODE usar
pra second-opinion com citations.

**Estes sao OPCIONAIS** — o plugin funciona stock com WebFetch nativo.
Se MCPs nao estiverem disponiveis, NAO falhe — emita ⚠️ com a nota de
ambiente e siga.

---

## 5. CONEXAO COM A PROXIMA SKILL

Apos emitir o relatorio, sinalize ao usuario:

> 🔍 Para auditoria completa antes de citar em peca (inclui pertinencia
> a sua tese e checagem de superacao do precedente), rode a skill
> `auditoria-pre-envio`.

Nao auto-dispara — auditoria-pre-envio precisa do contexto da peca/tese
do usuario. O conteudo ja obtido pelos fetches DESTA sessao pode ser
reutilizado pela auditoria (sem novo fetch), desde que na mesma sessao.
