# Changelog — juris-adv-os

## v0.2.0 — 05/07/2026 (auditoria tecnica completa)

### Correcoes criticas
- **F1** `validar-jurisprudencia`: criterios recalibrados para a realidade
  dos portais. STF e STJ/SCON verificados como JS-heavy (fetch retorna
  vazio) — agora ha 4 desfechos de fetch classificados por evidencia
  observavel; pagina vazia/JS vira ⚠️ com nota, nunca 🔴. Suporte a PDF
  (inteiro teor) documentado. Fallback com render JS elevado a
  "fortemente recomendado" para STF/STJ.
- **F2** Removido o criterio inverificavel "HTTP 200" (a ferramenta de
  fetch nao expoe status HTTP). Substituido por "conteudo substantivo
  contendo o numero do processo". Proibicao explicita de reportar codigo
  HTTP nao exibido.
- **F3** `commands/juris.md`: incluida a ferramenta `Skill` no
  `allowed-tools` (a cadeia inteira depende dela). Removidos `Bash`,
  `Glob` e `Grep` (nao usados — reducao de superficie).

### Correcoes medias
- **F4** Falha de acesso do ambiente nao e mais tratada como falha da
  citacao: 🔴 passa a exigir evidencia POSITIVA de inexistencia ou
  divergencia; bloqueio de ambiente = ⚠️ com nota propria e contagem
  separada no relatorio.
- **F5** `auditoria-pre-envio` reutiliza o conteudo dos fetches da
  validacao quando na mesma sessao (elimina redundancia de 2-3 fetches
  por citacao).
- **F6** Vereditos da auditoria recalibrados: criado ✅ APROVADA (base:
  ementa oficial) alem de ✅ APROVADA (inteiro teor); tratamento de PDF
  na Camada 1.
- **F7** Nova **Camada 4 — Vigencia**: checagem de superacao, afetacao a
  repetitivo/repercussao geral com sobrestamento e modulacao, com
  registro epistemicamente honesto ("sem indicio localizado" ≠
  "garantidamente vigente").
- **F8** CLAUDE.md interno EXCLUIDO do pacote distribuido (mantido apenas
  no repositorio-fonte privado; versao atualizada entregue a parte).

### Correcoes menores
- **F9** URL de busca do TRF4 atualizada para o endpoint eproc atual;
  tabela de fontes anotada com estado de verificacao e data.
- **F10** Datas hardcoded em exemplos substituidas por placeholders com
  instrucao de resolver pela data real da sessao (busca e ABNT).
- **F11** "Aspas portuguesas" corrigido para "aspas curvas duplas".
- **F12** `formatar-citacao`: removida instrucao inoperante de "default
  em 1 turno"; agora gera ambos os formatos por padrao.
- **F13** Terminologia "nexo causal" padronizada para "nexo de
  pertinencia" em plugin.json, comando e skills.
- **F14** Hook SessionStart REMOVIDO (injetava contexto em toda sessao,
  contrariando a propria doc; a descoberta das skills ja ocorre pela
  listagem nativa).

### Melhorias
- **M1** Cobertura de fontes ampliada: TNU/CJF, temas repetitivos do STJ,
  repercussao geral do STF, Pesquisa Pronta, processo.stj.jus.br.
- **M2** Tabela de fontes documentada com os dois usos distintos:
  filtro `site:` (principal) vs alvo de fetch (condicionado).
- **M3** Registro de diligencia no output da auditoria (bloco colavel no
  dossie do caso: citacao + URL + status + data/hora).
- **M6** Aviso legal agora presente no relatorio da validacao e como
  regra fixa do comando /juris (antes so aparecia na auditoria).

### Pendencias documentadas (nao resolvidas nesta versao)
- **M4** Teste de regressao periodico das URLs da tabela (10 dominios
  marcados como "conferir" — testar antes de cada release).
- **M5** Roadmap: gerar-quadro-comparativo e o proximo candidato.

## v0.1.0 — versao inicial
