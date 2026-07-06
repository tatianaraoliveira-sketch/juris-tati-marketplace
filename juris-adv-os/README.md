# Juris Adv-OS

Plugin Claude Code / Cowork para **buscar + validar + auditar
jurisprudencia** com rastreabilidade real.

> Anti-halucinacao por design: nenhuma citacao recebe selo de validada
> sem que o link da fonte seja conferido por fetch real — e nenhuma
> falha de acesso do ambiente e confundida com citacao inexistente.

---

## O que faz

- 🔎 **Busca** jurisprudencia em portais oficiais (STF, STJ, TST, TNU,
  TRFs, TJs) e agregadores (JusBrasil, Escavador), priorizando teses
  vinculantes (temas repetitivos, repercussao geral, sumulas) e fontes
  oficiais.
- ✅ **Valida** automaticamente cada citacao via fetch na URL declarada —
  confirma se o numero do processo e o trecho da ementa efetivamente
  aparecem na pagina. Status por citacao: ✅ VALIDADA / ⚠️ PARCIAL /
  🔴 NAO VALIDADA. Distingue divergencia de conteudo (🔴) de mera
  falha de acesso do ambiente (⚠️).
- 🎯 **Audita** em 4 camadas antes de citar em peca: conteudo analisado,
  link confirmado, nexo de pertinencia com a sua tese (PRO / CONTRA /
  NEUTRO / DEPENDE) e vigencia do entendimento (superacao, afetacao,
  sobrestamento).
- 📄 **Formata** citacoes aprovadas em ABNT (NBR 6023/10520) e em bloco
  pronto-pra-peca.

---

## Como usar

Apos instalar:

```
/juris responsabilidade civil do banco por fraude no PIX nao autorizado
```

O Claude vai:
1. Buscar em STJ, TJs, agregadores (teses vinculantes primeiro)
2. Validar cada link automaticamente
3. Oferecer auditoria pre-envio (pertinencia a sua tese + vigencia)
4. Formatar as aprovadas em ABNT + bloco pra peca

---

## Como instalar

### Via UI Cowork (recomendado)

1. Cowork -> Settings -> Plugins -> Pessoal -> "+" -> Adicionar marketplace
2. Cola a URL deste repositorio:
   ```
   https://github.com/tatianaraoliveira-sketch/juris-tati-marketplace
   ```
3. Sincronizar -> Instalar -> rodar `/juris` no Claude Code.

### Via CLI

```bash
claude plugin marketplace add https://github.com/tatianaraoliveira-sketch/juris-tati-marketplace
claude plugin install juris-adv-os@juris-tati-marketplace
```

---

## Skills disponiveis (4)

| Skill | Funcao |
|-------|--------|
| `buscar-jurisprudencia` | Busca em portais oficiais + agregadores |
| `validar-jurisprudencia` | Fetch real em cada URL declarada |
| `auditoria-pre-envio` | Pertinencia a tese + vigencia do precedente (4 camadas) |
| `formatar-citacao` | ABNT + bloco pronto-pra-peca |

---

## Limitacao conhecida (transparencia)

Os portais de busca do **STF** e do **STJ (SCON)** sao renderizados por
JavaScript e nao retornam conteudo em fetch simples. Citacoes desses
portais podem sair como ⚠️ "pagina exige render JS" — isso NAO significa
que a citacao e falsa; significa que a validacao automatica nao alcancou
a pagina e a conferencia manual e obrigatoria. Com um MCP de scraping
com renderizacao (ex.: firecrawl) configurado, o plugin o usa como
fallback automatico e reduz esses casos.

---

## Filosofia

Plugin enxuto, instalou-e-usou. Sem orquestrador, sem onboarding.

Funciona com **stock Claude Code** (`WebSearch` + `WebFetch` nativos).
MCPs de fallback (firecrawl, perplexity) sao **opcionais**, nunca
obrigatorios.

---

## ⚠️ Aviso legal

Validacao automatica e **assistida por IA**. A conferencia humana
final, especialmente do inteiro teor e da pertinencia ao caso concreto,
e responsabilidade exclusiva do advogado antes de protocolar.

---

## Licenca

MIT — ver [LICENSE](./LICENSE).

---

**Familia Adv-OS** — plugins juridicos modulares para Claude Code / Cowork.
