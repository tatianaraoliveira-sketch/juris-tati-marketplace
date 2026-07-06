---
name: buscar-jurisprudencia
description: >
  BUSCAR-JURISPRUDENCIA — Pesquisa jurisprudencia em portais oficiais (STF,
  STJ, TST, TNU, TRFs, TRTs, TJs) e agregadores (JusBrasil, Escavador) usando
  WebSearch + WebFetch nativos. Retorna cada citacao com URL da fonte e
  classificacao da origem (oficial vs agregador). Ao final, AUTO-INVOCA a
  skill validar-jurisprudencia para confirmar cada link. Use quando o
  usuario pedir "buscar jurisprudencia", "preciso de precedente sobre X",
  "tem julgado de Y", "pesquisar acordao", "encontrar sumula", "ver decisao
  do STJ sobre Z", ou disparar /juris.
---

# BUSCAR-JURISPRUDENCIA — Pesquisa de Precedentes com Rastreabilidade

## 1. REGRA DE OURO ANTI-HALUCINACAO

**Nao invente julgado.** Toda jurisprudencia retornada DEVE ter:
- URL real da fonte (obtida via WebSearch ou conhecida do tribunal)
- Classificacao da fonte: `OFICIAL` (dominio *.jus.br) ou `AGREGADOR` (JusBrasil/Escavador)
- Trecho da ementa transcrito (nao parafrase a primeira vez)

Se WebSearch retornar zero resultados confiaveis, voce DEVE dizer ao usuario:
> "Nao encontrei jurisprudencia consolidada sobre essa tese especifica nos portais oficiais consultados. Sugestoes: [...]"

Nao preencher o vazio com julgado inventado. **NUNCA.**

---

## 2. WORKFLOW

### Passo 1 — Capturar a tese

Pergunte ou extraia do prompt:
- **Tema/tese exata** (1-2 frases). Ex: "responsabilidade civil do banco por fraude em PIX nao autorizado"
- **Tribunal preferencial** (opcional): STF, STJ, TST, TNU, TRF1-6, TJ-X etc.
- **Esfera/area:** civel, penal, trabalhista, tributaria, previdenciaria, administrativa, etc.
- **Recencia:** ultimos 12 meses / 5 anos / qualquer

Se faltar info critica, pergunte UMA vez. Nao mais que isso.

### Passo 2 — Priorizar fontes oficiais

**IMPORTANTE — dois usos distintos da tabela abaixo:**

- **(a) Filtro `site:` no WebSearch** — uso PRINCIPAL e confiavel para
  TODOS os dominios. E assim que voce ACHA os julgados.
- **(b) Alvo de WebFetch direto** — uso CONDICIONADO. Varios portais de
  BUSCA sao renderizados por JavaScript e retornam conteudo vazio no
  fetch (verificado em STF e STJ/SCON). NAO tente "pesquisar" fazendo
  fetch da pagina de busca desses portais. O fetch e para as paginas de
  RESULTADO/documento que o WebSearch retornar.

| Fonte | Dominio para `site:` | Fetch da pagina de busca |
|-------|----------------------|--------------------------|
| STF | `jurisprudencia.stf.jus.br` | ⚠️ JS-heavy — nao fetchavel |
| STF Repercussao Geral (temas) | `portal.stf.jus.br` | conferir por tema |
| STJ (SCON) | `scon.stj.jus.br` | ⚠️ JS-heavy — nao fetchavel |
| STJ processos | `processo.stj.jus.br` | conferir |
| TST | `jurisprudencia.tst.jus.br` | conferir |
| TST sumulas/OJs | `www3.tst.jus.br` | conferir |
| TNU / Turmas Regionais (CJF) | `www.cjf.jus.br` | conferir |
| TRF1 | `jurisprudencia.trf1.jus.br` | conferir |
| TRF2 | `www10.trf2.jus.br` | conferir |
| TRF3 | `web.trf3.jus.br` | conferir |
| TRF4 | `jurisprudencia.trf4.jus.br` (busca atual: `/eproc2trf4/externo_controlador.php?acao=jurisprudencia@jurisprudencia/pesquisar`) | ✅ fetchavel (verificado) |
| TRF5 | `julia-pesquisa.trf5.jus.br` | conferir |
| TRF6 | `jurisprudencia.trf6.jus.br` | conferir |
| TJSP (e-SAJ) | `esaj.tjsp.jus.br` | ✅ fetchavel (verificado) |
| TJRJ | `www3.tjrj.jus.br` | conferir |
| TJMG | `www5.tjmg.jus.br` | conferir |
| TJRS | `www.tjrs.jus.br` | conferir |

(Lista nao exaustiva — consulte outros tribunais conforme o estado.
"Verificado" = testado por fetch em 07/2026; "conferir" = estado do
portal pode ter mudado, teste antes de confiar.)

### Passo 2-A — Teses vinculantes primeiro

Antes dos acordaos isolados, busque teses fixadas:
- Temas repetitivos do STJ: `site:scon.stj.jus.br "Tema [n]"` ou pagina
  de repetitivos do STJ via WebSearch
- Repercussao geral do STF: `site:portal.stf.jus.br repercussao geral "Tema [n]"`
- Sumulas (STF/STJ/TST/TNU), IAC e IRDR do tribunal local
- Pesquisa Pronta do STJ (compilacoes oficiais por tema)

### Passo 3 — Buscar via WebSearch

Use queries especificas combinando:
- Termos juridicos exatos (`"responsabilidade civil"` aspeado)
- Numero de lei/artigo (`art. 14 CDC`)
- Numero de tema repetitivo (`Tema 1051 STJ`)
- Site oficial (`site:scon.stj.jus.br`)
- Ano corrente ou faixa desejada — use SEMPRE a data real da sessao,
  nunca um ano fixo de exemplo

Exemplo (substitua [ANO] pelo ano corrente da sessao):
```
WebSearch: "responsabilidade banco fraude pix" site:scon.stj.jus.br
WebSearch: "fraude eletronica" "consumidor" julgado STJ [ANO]
```

**Limite: 6 buscas no maximo por tese.** Se nao achou, e porque o tema e
novo ou nao consolidado — informe ao usuario, nao force.

### Passo 4 — Buscar em agregadores APENAS como complemento

Se as fontes oficiais derem pouco resultado, complemente com:
- **JusBrasil** (`site:jusbrasil.com.br`)
- **Escavador** (`site:escavador.com`)

**MARQUE SEMPRE** essas citacoes como `AGREGADOR — conferir no tribunal`.

### Passo 5 — Estruturar cada citacao

Para cada julgado encontrado, monte:

```markdown
### [N] [Tribunal] — [Sumario em 1 linha]

- **Numero dos autos:** [completo conforme aparece]
- **Orgao julgador:** [Turma / Camara / Plenario / Secao]
- **Relator(a):** [nome]
- **Data de julgamento:** [DD/MM/YYYY]
- **Publicacao:** [DJe / data]
- **Fonte:** [URL completa]
- **Origem:** OFICIAL | AGREGADOR | VINCULANTE
- **Ementa (trecho relevante):**
  > [transcricao literal de 3-8 linhas, sem parafrase]
- **Tese central:** [1 frase que sintetiza o entendimento — sua sintese]
- **Relevancia para o caso do usuario:** [1-2 frases conectando a tese da busca]
```

Numere sequencialmente. Maximo 10 citacoes por busca — qualidade > quantidade.

### Passo 6 — AUTO-INVOCAR validacao

**OBRIGATORIO.** Apos montar a lista, IMEDIATAMENTE invoque a skill
`validar-jurisprudencia` passando a lista:

```
Acionando validar-jurisprudencia para confirmar cada citacao via fetch...
```

Use a Skill tool com `skill="validar-jurisprudencia"`. NAO PULE este
passo — e o coracao da promessa do plugin.

---

## 3. CASOS ESPECIAIS

### Sumula / Tese vinculante

Se o tema tiver sumula ou tese fixada (repetitivo / repercussao geral / IAC / IRDR):
- Cite a sumula/tese ANTES dos acordaos.
- Marque como `VINCULANTE` no campo Origem.
- Link para a publicacao oficial do tribunal.

### Tema novo / divergencia

Se a jurisprudencia ainda nao esta consolidada:
- Apresente as **duas correntes** com julgados representativos de cada.
- Sinalize: `⚠️ TEMA NAO CONSOLIDADO — duas correntes ativas`.

### Inteiro teor solicitado

Se o usuario pedir explicitamente "traz o inteiro teor de X", faca
WebFetch direto na URL do documento no tribunal e devolva o texto
integral relevante (10-30 paragrafos centrais), nao so a ementa. Se o
documento for PDF, faca o fetch do PDF e extraia o texto.

---

## 4. OUTPUT FINAL — TEMPLATE

```markdown
## Jurisprudencia encontrada — [Tese resumida]

**Busca realizada em:** [DATA E HORA REAIS DA SESSAO]
**Tribunais consultados:** [lista]
**Total de citacoes:** [N] · vinculantes: [N] · oficiais: [N] · agregadores: [N]

---

[Lista das citacoes estruturadas, numeradas 1, 2, 3...]

---

🔍 **Proximo passo automatico:** acionando `validar-jurisprudencia` para
confirmar cada URL via fetch real. Aguarde o relatorio de validacao
antes de citar em peca.
```

---

## 5. PROIBICOES ABSOLUTAS

1. **NAO inventar numero de processo.** Se nao tem certeza, omite e diz "numero indisponivel — confira no tribunal".
2. **NAO parafrasear ementa na primeira citacao.** Transcreva literal (uso de aspas).
3. **NAO atribuir a tribunal errado.** Em duvida, marca `⚠️ origem incerta`.
4. **NAO afirmar "validada" sem ter chamado a skill `validar-jurisprudencia`.**
5. **NAO retornar mais de 10 citacoes.** Foco em qualidade, nao quantidade.
6. **NAO usar so JusBrasil/Escavador.** Sempre tenta tribunais oficiais primeiro.
7. **NAO usar datas de exemplo como se fossem a data corrente.** Resolva
   sempre pela data real da sessao.
