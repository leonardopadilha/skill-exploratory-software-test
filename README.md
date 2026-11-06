# skill-exploratory-software-test

Skill para o [Claude](https://claude.ai) que atua como **parceiro técnico de um Analista de Testes Sênior**,
com foco em **teste exploratório** e raciocínio baseado em risco, para aplicações web modernas (SPA, APIs,
microsserviços) e legadas (JSP, JSF, ASP.NET Web Forms, PHP, monolitos).

> **Versão:** v1.0.0 · **Idioma:** português (PT-BR) · **Licença:** MIT

*English summary: a Claude skill that acts as a senior test analyst's thinking partner. It covers exploratory
testing, test heuristics and oracles (SFDIPOT, FEW HICCUPPS, ALTER FACE, RCRCRC, MCOASTER), formal black-box
techniques, risk-based prioritization, bug isolation and risk-oriented reporting, for both modern and legacy
web apps. Content is in Brazilian Portuguese.*

## O que ela faz

A skill ajuda a **pensar melhor sobre teste**, em vez de despejar catálogos. Ela guia um ciclo de investigação:
contexto e risco → modelar o produto → gerar ideias → experimentar → julgar → isolar → comunicar → decidir.

Cobre, entre outros temas:

- Modelagem do produto (SFDIPOT/HTSM, CRUD, "siga o dado", modelos de estado)
- Oráculos (FEW HICCUPPS) para decidir "isso é um problema?"
- Heurísticas de ideias (ALTER FACE, Goldilocks, Zero-Um-Muitos, tours, error guessing)
- Técnicas formais (partição de equivalência, valor-limite, tabela de decisão, transição de estados, pairwise)
- Teste exploratório: charters, sessões (SBTM) e debrief PROOF
- Risco, priorização, regressão (RCRCRC) e critérios de parada
- Isolamento e investigação de bugs (incluindo intermitentes)
- Comunicação de risco (MCOASTER, relato de bug orientado a risco)
- Particularidades de web moderna e de sistemas legados, e kits por funcionalidade (login, upload, paginação, etc.)

### Rigor de fontes

Todo conteúdo é marcado como **[Fonte]**, **[Interpretação]** ou **[Ilustrativo]**, e o arquivo
[`references/referencias.md`](references/referencias.md) indica o status de verificação de cada referência
(Verificada, Confirmada ou A confirmar). A skill foi instruída a não inventar autores, técnicas ou resultados.

## Estrutura

```
SKILL.md                      # instruções principais e roteador de referências
references/
  00-fundamentos.md           # princípios, teste × checagem, vieses
  01-modelar-produto.md       # SFDIPOT, CRUD, siga o dado
  02-oraculos.md              # FEW HICCUPPS
  03-heuristicas-ideias.md    # ALTER FACE, Goldilocks, tours
  04-tecnicas-formais.md      # PE, AVL, tabela de decisão, estados, pairwise
  05-exploratorio.md          # charters, SBTM, sessões
  06-risco-e-parada.md        # risco, RCRCRC, quando parar
  07-isolar-investigar.md     # hipóteses, TuTTu/TaTTa, repro mínima
  08-web-moderna.md           # SPA, APIs, tokens, assíncrono
  09-web-legada.md            # postback, sessão, jobs, caracterização
  10-comunicar.md             # MCOASTER, relato de bug
  11-cenario-integrado.md     # exemplo fictício completo (SRD)
  12-casos-por-funcionalidade.md
  referencias.md
```

## Como usar

Copie `SKILL.md` e a pasta `references/` para uma pasta chamada `exploratory-software-test` dentro do
diretório de skills do seu ambiente (por exemplo, `~/.claude/skills/` no Claude Code):

```
~/.claude/skills/exploratory-software-test/
├── SKILL.md
└── references/
```

Depois, peça algo como:

- "Tenho uma tela de aprovação para testar amanhã e não há documentação. Por onde começo?"
- "Revise estes casos de teste e diga o que eles não cobrem."
- "Esse comportamento é bug ou feature? Ajude-me a argumentar."

## Sobre a autoria

Esta skill foi **gerada com o Claude a partir de um prompt meu** (Leonardo Padilha) e curada por mim. O conteúdo
sintetiza ideias publicadas por James Bach, Michael Bolton, Cem Kaner, Elisabeth Hendrickson, James Whittaker,
Karen N. Johnson, Mike Kelly, Júlio de Lima (ALTER FACE), Michael Feathers e o ISTQB, entre outros, sempre com
crédito e indicação do grau de verificação. Os exemplos são fictícios.

## Roadmap (próximas versões)

- [ ] Templates prontos (charter, session sheet, relato de bug)
- [ ] Mais cenários de exemplo além do SRD (e-commerce, saúde, bancário)
- [ ] Kits por funcionalidade mais aprofundados
- [ ] Cobertura de mobile, acessibilidade, desempenho e segurança
- [ ] Testes de sistemas com IA
- [ ] `description` mais enxuta e avaliações (evals) do comportamento da skill
- [ ] Versão em inglês

## Licença

[MIT](LICENSE)
