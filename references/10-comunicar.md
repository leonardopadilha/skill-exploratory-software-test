# 10 — Comunicar: "o que quem decide precisa saber?"

## Índice
1. Princípio: o produto do teste é informação
2. A história do teste em três partes (RST)
3. MCOASTER (Mike Kelly)
4. Relato de bug orientado a risco
5. Exemplos: de observação a risco
6. Adaptando ao público

---

## 1. Princípio

[Fonte] Bolton descreve o papel do testador como identificar problemas que ameaçam o valor do produto para que o
dono do produto — quem decide sobre qualidade — tome decisões informadas; histórias de teste são, na maior
parte, sobre problemas que causariam perda, dano ou incômodo a alguém que importa.

[Interpretação] Um bom relato permite a alguém decidir. Um relato que só conta casos executados e bugs abertos
não permite.

## 2. A história do teste em três partes

**[Fonte] Rapid Software Testing (Bach & Bolton)** propõe relatar o teste como histórias entrelaçadas (Bolton,
"Braiding the Stories"): (1) a história do **produto** — como ele é, que problemas tem; (2) a história do
**teste** — como foi testado, o que foi e o que não foi coberto; (3) a história da **qualidade do teste** — o que
tornou o teste mais fácil ou mais difícil, quão confiáveis são as conclusões.

[Interpretação] A terceira parte é a mais esquecida e a que mais protege o testador: "não encontrei problemas
na integração com o ERP, mas o ambiente de homologação usa um simulador que aceita qualquer layout".

## 3. MCOASTER

**Identificação [Fonte]:** **Mike (Michael D.) Kelly** desenvolveu uma heurística para relatar o status do teste
com confiança e ajustar o relatório ao público. A origem está num conselho de **James Bach** a Kelly em 2005
sobre as dimensões de um bom relatório. O mnemônico MCOASTER aparece em materiais do Rapid Software Testing
atribuído a Kelly.

| Letra | Dimensão | Pergunta |
|---|---|---|
| M | Mission | O que eu estava tentando descobrir? |
| C | Coverage | O que eu olhei? (SFDIPOT ajuda a descrever) |
| O | Obstacles | O que me impediu ou atrasou? Preciso de ajuda? |
| A | Audience | Para quem é este relato e que nível de detalhe ele quer? |
| S | Status | Onde estou? O que encontrei? Quanto falta? |
| T | Techniques | Como eu procurei? |
| E | Environment | Onde, quando, com que dados, com quem? |
| R | Risk | Que riscos eu estava procurando — e quais permanecem? |

**Quando usar:** status diário, fim de sessão, reunião de go/no-go, pergunta de corredor "como está o teste?".

**Exemplo — status verbal de 30 segundos [Ilustrativo]:**
> "Estou testando a troca do fornecedor de câmbio (M) para o financeiro (A). Cobri conversão no envio e o
> arquivo do ERP; ainda não cobri a exportação CSV (C). Encontrei um problema sério: a cotação usa a data do
> lançamento, não a da despesa — diferenças de até 3% (S, R). Estou comparando com o fornecedor antigo em 30
> despesas reais anonimizadas (T) em homologação (E). Obstáculo: o simulador do fornecedor não tem feriados;
> preciso de dados de teste deles (O)."

## 4. Relato de bug orientado a risco

[Interpretação] Estrutura recomendada:
1. **Título:** comportamento + condição + consequência. "Cotação usa a data do lançamento em vez da data da
   despesa — valores reembolsados divergem do gasto."
2. **Impacto e quem é afetado:** a frase que faz alguém priorizar.
3. **Passos mínimos para reproduzir** (ver `07`), com dados exatos.
4. **Observado × esperado,** com o **oráculo** que sustenta o esperado (Claims: regra R6; History: versão
   anterior).
5. **Evidências:** capturas, requisições, logs, horário, ambiente/versão.
6. **Isolamento já feito:** hipóteses descartadas, camada provável.
7. **Extrapolação:** onde mais isso pode acontecer? (outras telas que usam a cotação?)
8. **Contorno,** se existir.

**[Fonte, via apresentação de terceiros]** Bolton apresentou na EuroSTAR 2014 uma heurística para relatos de
bug chamada *PEOPLE WORKing* (Problem, Example, Oracle, Polite, Literate, Extrapolation, Workaround), com
coautoria de Bach e Alexei Barantsev. [Atribuição a confirmar na fonte original.]

**Tom:** descreva o comportamento do produto, não a pessoa que o construiu. Fatos primeiro, interpretação
marcada como tal.

## 5. Exemplos: de observação a risco [Ilustrativo]

| Observação (fraca) | Informação de risco (forte) |
|---|---|
| "Toast fica atrás do modal." | "A confirmação de envio fica atrás do modal; o colaborador não vê que enviou e envia de novo, gerando solicitações duplicadas para o gestor." |
| "Lista e detalhe mostram totais diferentes." | "A lista (e possivelmente a tela de aprovação) soma itens excluídos; gestores podem aprovar valores maiores que os devidos." |
| "Sessão expira e perde dados." | "Colaboradores com solicitações longas (viagens) perdem 15–20 min de digitação quando a sessão expira; tende a gerar lançamentos incompletos e chamados." |

## 6. Adaptando ao público

[Interpretação]
- **Desenvolvedor:** reprodução mínima, isolamento, evidências técnicas.
- **Dono do produto:** impacto em usuários e negócio, frequência estimada, opções (corrigir, contornar, aceitar).
- **Gestão/go-no-go:** riscos restantes, cobertura e lacunas, confiança no teste, recomendação clara e quem
  decide.
- **Auditoria/conformidade:** rastreabilidade, evidências, padrões e normas envolvidos.
