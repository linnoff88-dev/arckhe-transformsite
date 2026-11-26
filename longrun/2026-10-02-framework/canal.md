# Canal — framework (só acrescentar; nunca reescrever)

Conhecimento do projeto que a compactaçêo perde: fatos descobertos, aprendizados, glossário, armadilhas, onde estêo as coisas.
Nêo é estado da tarefa (isso vai em state/plan/progress). Preencha cedo — o hook avisa na faixa 1 (~50% do contexto).

Formato: `- AAAA-MM-DD HH:MM · fato|aprendizado|glossário|armadilha · texto`


## 2026-10-02 — fechamento (contexto 91%)
- FATO: claude_cli/codex_cli rodam sem erro mas respondem "nêo sei" no piloto (Ollama acerta). Marcados experimentais. Próximo passo: imprimir a saída crua de `LLM._cli_chat` para "Preciso saber programar para fazer o curso CAIP?" e ajustar `parse_json`/prompt.
- FATO: ~/projetos/portal/src/data/translated-courses.ts tem alteraçêo NêO commitada de outra sessêo (OSWork v6.2) — preservada; eu commitei só as 2 linhas do transformsite (0878293).
- ARMADILHA: hook context-mode bloqueia curl → checagens HTTP por python urllib/node.
- Opcional: remover guia/assets/hero.png (2,1 MB, nêo referenciado).
