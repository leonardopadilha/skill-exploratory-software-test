# Referências

Status: **Verificada** = fonte primária ou do próprio autor consultada durante a criação da skill;
**Confirmada** = confirmada por fontes secundárias confiáveis e convergentes; **A confirmar** = atribuição ou
detalhe sem fonte primária localizada — use com cautela e diga isso ao usuário.

## Fundamentos e Rapid Software Testing
| Referência | Uso na skill | Status |
|---|---|---|
| Bach, J.; Bolton, M. "Testing and Checking Refined" (2013), satisfice.com/blog/archives/856 | Definições de teste e checagem | Verificada (via apresentações de Bolton) |
| Bolton, M. "Exploratory Testing 3.0" (EuroSTAR 2015), developsense.com/presentations | Evolução das definições de ET e de teste | Verificada |
| Bolton, M. "Oracles" (Test Framing, 2010), developsense.com/resource/Oracles.pdf | Oráculo, HICCUPPS(F), inferência abdutiva | Verificada |
| Bolton, M. "FEW HICCUPPS" (2012), developsense.com/blog/2012/07/few-hiccupps | Extensão FEW | Verificada |
| Bolton, M. "Testing Without a Map", *Better Software* (jan. 2005) | Apresentação original de HICCUPPS | Confirmada |
| Bach, J. Heuristic Test Strategy Model, v6.0 (2024), developsense.com/htsm | SFDIPOT, critérios de qualidade | Verificada (conferir a lista de critérios na versão vigente) |
| Bolton, M. post sobre a inclusão de Interfaces (SFDIPOT), developsense.com/?p=1773 | Histórico do mnemônico | Verificada |
| Bach, J. "Heuristics" (2010), satisfice.com/blog/archives/462 | Definição de heurística (Bach & Kaner) | Verificada |
| Koen, B. V. *Discussion of the Method* | Engenharia como uso de heurísticas | Confirmada (citado por Bach & Bolton) |
| Weinberg, G. *Quality Software Management, Vol. 1: Systems Thinking* | "Qualidade é valor para alguma pessoa" | Confirmada (Bolton, entrevista a1qa) |
| Gause, D.; Weinberg, G. *Are Your Lights On?* (1990) | Definição de problema | Verificada (citada por Bolton) |
| Bolton, M. "Issues About Metrics About Bugs", *Better Software* (2009) | Bug como ameaça ao valor; issue | Verificada |
| Bolton, M. "Braiding the Stories (Test Reporting Part 2)" | História do teste em três partes | Confirmada (citada pelo autor) |
| Bach, J. "Heuristic Risk-Based Testing" (1999) | Teste baseado em risco heurístico | A confirmar (não consultada nesta versão) |
| Bolton, M. post sobre heurísticas de parada ("When Do We Stop a Test?") | Critérios de parada | A confirmar (não consultada nesta versão) |

## Teste exploratório
| Referência | Uso | Status |
|---|---|---|
| Kaner, C. *Testing Computer Software* (1988) | Origem do termo exploratory testing | Confirmada |
| Kaner, C. "Exploratory Testing After 23 Years" (2007) | Definição refinada | Confirmada (citada por Bolton) |
| Kaner, C.; Bach, J.; Pettichord, B. *Lessons Learned in Software Testing* (2001) | Escola Context-Driven | Confirmada |
| Bach, Jonathan. "Session-Based Test Management" (2000), satisfice.com | SBTM, PROOF | Confirmada |
| Hendrickson, E. *Explore It!* (Pragmatic Bookshelf, 2013) | Charters, heurísticas | Confirmada |
| Hendrickson, E.; Lyndsay, J.; Emery, D. *Test Heuristics Cheat Sheet* (e versão atualizada da Ministry of Testing) | Goldilocks, ZOM, CRUD etc. | Confirmada |
| Whittaker, J. *Exploratory Software Testing* (Addison-Wesley, 2009) | Tours e distritos | Confirmada |
| Whittaker, J. *How to Break Software* | Ataques | Confirmada |

## Heurísticas específicas
| Referência | Uso | Status |
|---|---|---|
| Lima, J. de. "ALTER FACE — A Test Heuristic for UI Web Applications" (slides, 2020), slideshare.net/juliodelimas | ALTER FACE | Verificada |
| Lima, J. de. Palestra "ALTER FACE: A Software Test Heuristic to Test Web Applications" (Geekle) | Contexto de criação | Verificada |
| Johnson, K. N. "A Heuristic for Regression Testing" (2009) | RCRCRC | Confirmada (Ministry of Testing) |
| Kelly, M. D. artigo na InformIT sobre relatório de status; post "Dimensions of a good test report" (2005) | MCOASTER | Confirmada |
| TuTTu/TaTTa atribuída a Mark Winteringham (repositório comunitário) | Isolamento UI × API | **A confirmar** |
| Winteringham, M. *Testing Web APIs* (Manning, 2022) | Teste de APIs | Confirmada |
| PEOPLE WORKing (Bolton, EuroSTAR 2014; coautores Bach e Barantsev) | Relato de bug | **A confirmar** (fonte secundária) |

## Técnicas formais, legado e normas
| Referência | Uso | Status |
|---|---|---|
| ISTQB *Certified Tester Foundation Level Syllabus* v4.0 (cap. 4) | PE, AVL, tabela de decisão, transição de estados, técnicas baseadas em experiência | Confirmada |
| Syllabus ISTQB de nível avançado (Test Analyst) | Técnicas combinatórias | A confirmar (versão vigente) |
| Feathers, M. *Working Effectively with Legacy Code* (2004) | Código legado; testes de caracterização | Confirmada |
| ISO/IEC 25010 | Modelo de qualidade do produto | Confirmada |

## Interpretações próprias desta skill
Sem fonte externa, marcadas no texto como [Interpretação]: o ciclo de investigação; a taxonomia de tipos de
conhecimento; o enquadramento "cada heurística responde a uma pergunta"; os kits por funcionalidade; todo o
cenário SRD e seus defeitos. A sequência "observar → isolar → comunicar" foi proposta numa publicação do
LinkedIn de terceiros, cujo autor declara que a conexão entre as heurísticas é leitura pessoal dele.
