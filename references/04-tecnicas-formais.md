# 04 — Técnicas formais: derivar testes de forma sistemática

## Índice
1. Particionamento de equivalência (PE)
2. Análise de valor-limite (AVL)
3. Tabela de decisão
4. Transição de estados
5. Teste combinatório (pairwise)
6. Técnicas caixa-branca (quando o analista tem acesso ao código)
7. Escolhendo e combinando técnicas
8. Revisando casos de teste com estas técnicas

Base terminológica: **ISTQB CTFL v4.0** (cap. 4), que classifica PE, AVL, tabela de decisão e transição de
estados como técnicas caixa-preta. Exemplos no **SRD** (`11-cenario-integrado.md`); defeitos **[Ilustrativo]**.

[Interpretação] Técnicas formais são fortes onde há *regras explícitas* (faixas, condições, estados). Elas dão
cobertura sistemática e argumentável. São fracas onde a regra é desconhecida ou o problema está fora da regra
(layout, tempo, integração, uso real). Por isso, combine-as com modelagem (`01`), heurísticas (`03`) e
exploração (`05`).

---

## 1. Particionamento de equivalência

**Identificação [Fonte]:** técnica caixa-preta do ISTQB. Divide os dados em partições que o sistema deveria
tratar da mesma forma; testa-se ao menos um valor de cada partição (válidas e inválidas).

**Quando usar:** campos com faixas ou categorias; regras que tratam grupos de valores igualmente; reduzir um
domínio grande a poucos testes representativos.

**Como aplicar:**
1. Identifique a variável e a regra que a governa.
2. Liste partições válidas e inválidas — inclusive as que a especificação *não* menciona (vazio, tipo errado,
   formato regional).
3. Escolha um representante por partição. Partições inválidas: um por teste, para não mascarar erros.
4. Pergunte: "o sistema realmente trata todos estes valores igualmente?" — se houver dúvida, a partição está
   grossa demais.

**Exemplo — valor de um item (regra R1: R$ 0,01 a R$ 5.000,00) [Ilustrativo]:**

| Partição | Representante | Esperado |
|---|---|---|
| Válida: 0,01–5.000,00 | 250,00 | aceita |
| Inválida: ≤ 0 | -10,00 | rejeita com mensagem clara |
| Inválida: > 5.000,00 | 7.000,00 | rejeita |
| Inválida: não numérico | "abc" | rejeita |
| Inválida: mais de 2 casas | 10,999 | rejeita ou arredonda? (especificação omissa → pergunta ao PO) |
| Vazio | — | obrigatório |

**Raciocínio do sênior:** "A especificação fala em faixa, mas a regra de comprovante (R4) cria uma *subpartição*
dentro da faixa válida: até R$ 50 sem comprovante, acima com comprovante. E quilometragem é outra regra. Se eu
tratar 0,01–5.000 como uma partição só, perco isso. Partições vêm do *comportamento*, não só do intervalo."

**Problemas que pode revelar:** mensagens genéricas para inválidos; aceitação de negativos via API mesmo
bloqueados na tela; arredondamento silencioso de 3 casas.

**Limitações:** pressupõe que você conhece as partições; erra quando o sistema trata valores de uma partição
de forma diferente (por isso combine com AVL e exploração).

**Moderno × legado:** no moderno, teste a mesma partição na UI **e** na API — validações só no front-end são
um defeito clássico. No legado, a validação pode estar em JavaScript, no code-behind e numa procedure, com
regras diferentes em cada camada.

## 2. Análise de valor-limite

**Identificação [Fonte]:** técnica caixa-preta do ISTQB baseada nas bordas de partições ordenadas. O CTFL v4.0
apresenta a versão de 2 valores (o limite e o vizinho na partição adjacente) e a de 3 valores (o limite e os
vizinhos de ambos os lados).

**Quando usar:** sempre que houver partições ordenadas: valores, datas, tamanhos, quantidades, horários.

**Como aplicar:** identifique cada limite, determine a menor unidade significativa (centavo, dia, byte,
segundo) e teste o limite e seus vizinhos.

**Exemplo — prazo da despesa (R2: "não futura; até 90 dias atrás") [Ilustrativo]:**

| Caso | Data da despesa (lançada em 31/10) | Esperado |
|---|---|---|
| Futuro + 1 | 01/11 | rejeita |
| Hoje | 31/10 | aceita |
| 90 dias atrás | 02/08 | aceita? |
| 91 dias atrás | 01/08 | rejeita |

**Raciocínio do sênior:** "'Até 90 dias' é ambíguo: conta o dia de hoje? Conta em dias corridos ou úteis? É a
data da despesa vs. data do lançamento ou vs. data do envio? E se o rascunho for criado no dia 89 e enviado no
dia 92? A AVL me obrigou a escrever a data exata — e isso expôs a ambiguidade antes de qualquer execução. Levo
essas perguntas ao PO; a resposta vira oráculo (Claims)."

Outros limites do SRD: R$ 50,00/50,01 (comprovante), R$ 1.000,00/1.000,01 (alçada), 5 MB/5 MB + 1 byte,
10/11 anexos, quinta-feira 17:59:59/18:00:00 (corte de pagamento), 20/21 caracteres de justificativa.

**Problemas que pode revelar:** comparações `<` no lugar de `<=`; limite tratado em fuso UTC ("hoje" vira
"amanhã" após 21h em Brasília); limite de 5 MB calculado em 5.000.000 vs. 5.242.880 bytes.

**Limitações:** não encontra limites que você não sabe que existem (ex.: limite de tamanho do campo no banco
ou no arquivo do ERP). Exploração e "siga o dado" ajudam a descobri-los.

**Moderno × legado:** no legado, procure limites implícitos de infraestrutura (`maxRequestLength` em
configuração de servidor, tamanho de coluna VARCHAR, limite de sessão); no moderno, limites de gateway, de
payload de API e de storage, que podem diferir do limite exibido na tela.

## 3. Tabela de decisão

**Identificação [Fonte]:** técnica caixa-preta do ISTQB para regras com combinações de condições que geram
ações diferentes. Cada coluna é uma regra (combinação de condições → ações).

**Quando usar:** regras de negócio com várias condições; alçadas; elegibilidade; cálculo que muda por
combinação de fatores.

**Como aplicar:**
1. Liste condições (entradas) e ações (saídas).
2. Monte as combinações; elimine as impossíveis e *marque* as não especificadas (são perguntas).
3. Derive um teste por coluna relevante.

**Exemplo — exigência de comprovante e justificativa (R3/R4) [Ilustrativo]:**

| Condições | R1 | R2 | R3 | R4 | R5 | R6 |
|---|---|---|---|---|---|---|
| Categoria = Quilometragem | S | N | N | N | N | N |
| Categoria = Hospedagem | — | S | N | N | N | N |
| Categoria = Outros | — | N | S | S | N | N |
| Valor > R$ 50 | — | — | S | N | S | N |
| **Ações** | | | | | | |
| Exige comprovante | N | S | S | N | S | N |
| Exige justificativa | N | N | S | S | N | N |

**Raciocínio do sênior:** "A tabela me fez ver que Hospedagem exige comprovante *sempre*, mesmo abaixo de R$ 50
— regra fácil de esquecer na implementação. E levantei uma pergunta: quilometragem acima de R$ 50 dispensa
comprovante, mas precisa de origem/destino? A especificação não diz."

**Problemas que pode revelar:** combinações não tratadas; regras implementadas em ordem errada (um `if` que
captura antes do outro); ações conflitantes.

**Limitações:** explode com muitas condições (use combinatório ou agrupe); não revela regras que ninguém
documentou.

## 4. Transição de estados

**Identificação [Fonte]:** técnica caixa-preta do ISTQB que modela o sistema como estados, eventos e
transições; os testes cobrem estados, transições válidas e tentativas de transições inválidas.

**Quando usar:** entidades com status; fluxos de aprovação; sessões; assinaturas; pedidos.

**Exemplo — ciclo de vida da solicitação no SRD [Ilustrativo]:**

```
RASCUNHO --enviar--> ENVIADO --aprovar(gestor)--> APROVADO_GESTOR --aprovar(financeiro)--> APROVADO_FINANCEIRO --pagar--> PAGO
   ^  |                 |  \--devolver--> DEVOLVIDO --reenviar--> ENVIADO
   |  cancelar          |   \--reprovar--> REPROVADO
   |  v                 cancelar
 CANCELADO <------------+
(total ≤ R$ 1.000: APROVADO_GESTOR segue direto para pagamento)
```

**Como aplicar:**
1. Desenhe o diagrama (mesmo que o time não tenha um — desenhá-lo já revela lacunas).
2. Monte a tabela estado × evento, marcando cada célula como válida, inválida ou *não especificada*.
3. Teste: cada transição válida ao menos uma vez; transições inválidas relevantes; sequências longas.
4. Para cada transição, verifique efeitos: histórico, notificação, permissões de edição, integração.

**Transições inválidas que um sênior tentaria [Ilustrativo]:**
- Aprovar uma solicitação CANCELADA (via URL/API ou aba antiga aberta).
- Editar uma solicitação APROVADO_GESTOR pela tela de rascunho (botão "voltar" do navegador).
- Pagar duas vezes (evento duplicado da fila no moderno; job reexecutado no legado).
- Cancelar *enquanto* o gestor aprova (concorrência → ver `12-casos-por-funcionalidade.md`).

**Problemas que pode revelar:** estado inconsistente entre telas; transição permitida pela API e bloqueada só
na UI; histórico sem registro da transição; notificação enviada para transição revertida.

**Limitações:** o modelo é tão bom quanto o conhecimento de quem o desenhou; transições temporais (expiração,
agendamento) costumam ficar fora — inclua eventos de tempo explicitamente.

**Moderno × legado:** no moderno, estado pode estar distribuído (serviço de solicitações e serviço de
pagamentos podem divergir por consistência eventual). No legado, o estado pode ser alterado por jobs noturnos
ou procedures fora da aplicação — pergunte "quem mais escreve nessa coluna?".

## 5. Teste combinatório (pairwise)

**Identificação [Fonte]:** famílias de técnicas combinatórias, como *pairwise* (todas as combinações de pares
de valores), são tratadas em materiais de nível avançado (por exemplo, o syllabus ISTQB de Test Analyst) e na
literatura de projeto de testes. [Interpretação] Não estão no CTFL v4.0 Foundation.

**Quando usar:** muitos parâmetros independentes com poucos valores cada (navegador × perfil × categoria ×
moeda × tipo de anexo), em que testar todas as combinações é inviável.

**Como aplicar:** liste parâmetros e valores; gere um conjunto que cubra todos os pares (ferramentas como
PICT ajudam); adicione combinações de alto risco conhecidas manualmente.

**Limitação importante:** defeitos que dependem de três ou mais fatores podem escapar; pairwise reduz custo,
não substitui análise de risco.

## 6. Técnicas caixa-branca

**[Fonte] ISTQB CTFL v4.0:** teste de instrução e teste de ramo, com medidas de cobertura correspondentes.

**[Interpretação] Para o analista de testes:** mesmo sem escrever testes de unidade, ler um diff ou pedir ao
desenvolvedor para mostrar os `if`s de uma regra revela partições e limites que a especificação omite. Uma
pergunta útil em revisão: "quais ramos desta mudança não são exercitados por nenhum teste?".

## 7. Escolhendo e combinando técnicas

[Interpretação]
| Característica da regra | Técnica principal | Complemento |
|---|---|---|
| Faixa ou categoria de valores | PE + AVL | Goldilocks, ataques de tipo de dado |
| Várias condições combinadas | Tabela de decisão | Pairwise se muitas condições |
| Status, fluxo, ciclo de vida | Transição de estados | Interrupções, concorrência |
| Muitos parâmetros independentes | Pairwise | Risco para combinações críticas |
| Regra desconhecida | Exploração para descobrir a regra | Depois formalize com as técnicas acima |

## 8. Revisando casos de teste com estas técnicas

Checklist de revisão [Interpretação]:
- Cada regra com faixa tem os limites e vizinhos? Os limites estão em unidades exatas?
- Há partições inválidas, uma por caso?
- As regras compostas estão numa tabela, com combinações não especificadas sinalizadas?
- Há casos para transições inválidas e para eventos de tempo?
- O resultado esperado é observável e nomeia o oráculo (não "funciona corretamente")?
- Os casos verificam efeitos colaterais (histórico, notificação, integração), e não só a tela?
- Validações são testadas também fora da UI (API, outra tela, importação)?
