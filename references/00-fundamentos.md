# 00 — Fundamentos e princípios

## Índice
1. Teste e checagem
2. Qualidade, valor e bug
3. Problema e oráculo
4. Heurística, mnemônico, modelo, técnica: uma taxonomia prática
5. Duas escolas, dois vocabulários: RST/Context-Driven e ISTQB
6. Implicações práticas para o dia a dia
7. Vieses que enganam o testador

---

## 1. Teste e checagem

**[Fonte] Bach & Bolton, "Testing and Checking Refined" (2013):**
- *Testar* é o processo de avaliar um produto aprendendo sobre ele por meio de exploração e experimentação, o
  que inclui, em algum grau, questionar, estudar, modelar, observar, inferir etc.
- *Checar* é o processo de fazer avaliações aplicando regras de decisão algorítmicas a observações específicas
  de um produto.

**[Fonte] Versão posterior (Rapid Software Testing):** a definição de teste passou a mencionar aprender sobre
o produto por meio de *experiência, exploração e experimentação* ("experiencing, exploring, and experimenting").
As duas formulações circulam; a mais recente acrescenta a experiência direta com o produto.

**[Fonte] Implicações declaradas pelos autores:** checar pode, em princípio, ser feito por uma ferramenta;
testar só pode ser *apoiado* por ferramentas. Toda checagem faz parte do teste, mas nem todo teste é checagem.

**[Interpretação] Por que isso importa na prática:**
- Uma suíte automatizada verde informa que as checagens programadas passaram — não que o produto não tem
  problemas importantes. Alguém precisou decidir *o que* checar e alguém precisa interpretar o resultado.
- O teste descobre o que importa; a checagem protege o que já foi aprendido. Ao encontrar um defeito
  explorando, pergunte: "isso merece virar uma checagem automatizada para não regredir?"
- Um caso de teste executado mecanicamente, sem atenção, é checagem feita por humano — o pior dos dois mundos:
  lenta como o humano e cega como o script.

## 2. Qualidade, valor e bug

**[Fonte] Gerald Weinberg:** "Qualidade é valor para alguma pessoa."
**[Fonte] Bach & Bolton** acrescentaram "que importa" para tornar explícito que avaliar qualidade começa com a
decisão sobre *de quem* são os valores que contam. Bolton menciona ainda uma variação que inclui o tempo
("em algum momento"), porque valores mudam.

**[Fonte] Rapid Software Testing:** bug é qualquer coisa que ameace o valor do produto. Bugs não são coisas
tangíveis e contáveis; são expressões de uma relação entre uma pessoa e um produto. E um *issue* é algo que
ameaça o valor do *teste* (ex.: ambiente instável, falta de dados).

**[Interpretação] Consequências:**
- "Está de acordo com o requisito" não encerra a discussão: o requisito pode estar errado ou incompleto.
- "Não está no requisito" não torna algo aceitável.
- Pessoas diferentes valorizam coisas diferentes (usuário final, financeiro, auditoria, suporte, operação).
  Ao avaliar um comportamento, pergunte *para quem* ele é um problema.
- Contagem de bugs é uma métrica pobre de qualidade; risco para pessoas que importam é melhor.

## 3. Problema e oráculo

**[Fonte] Gause & Weinberg (citados por Bolton):** um problema é uma diferença entre as coisas como são
percebidas e as coisas como são desejadas.

**[Fonte] Bolton:** oráculo é um princípio ou mecanismo heurístico pelo qual alguém reconhece um problema.
Oráculos são falíveis e dependentes de contexto; sugerem que pode haver um problema, não provam. Quando dois
profissionais discordam ("é bug" × "é feature"), em geral estão aplicando oráculos diferentes.

Ver `02-oraculos.md` para o uso prático (FEW HICCUPPS).

## 4. Heurística, mnemônico, modelo, técnica: uma taxonomia prática

**[Fonte] Bach & Kaner:** heurística é um método falível de resolver um problema ou tomar uma decisão. Bach
relata que os dois chegaram a essa definição após pesquisar décadas de literatura em psicologia e engenharia.
**[Fonte] Billy Vaughn Koen** (*Discussion of the Method*) é citado por Bach & Bolton: para Koen, o próprio
método de engenharia é o uso de heurísticas para causar a melhor mudança numa situação mal compreendida, com os
recursos disponíveis.

**[Fonte] Visão RST:** modelos, oráculos e técnicas de teste são todos heurísticas — nenhum garante que vai
funcionar. Um mnemônico é um tipo especial de heurística (para lembrar coisas); um checklist também.

**[Interpretação] Taxonomia usada nesta skill** (não é padrão de mercado; é uma forma de organizar):

| Tipo | Responde a | Exemplos |
|---|---|---|
| Princípio | No que acreditamos sobre teste? | Teste ≠ checagem; qualidade é valor para quem importa |
| Modelo | Como represento o produto? | SFDIPOT, diagrama de estados, mapa de fluxo de dados |
| Oráculo | Como reconheço um problema? | FEW HICCUPPS, especificação, produto comparável |
| Técnica | Como derivo testes sistematicamente? | Particionamento, valor-limite, tabela de decisão |
| Heurística de ideias | Que experimento tento? | ALTER FACE, Goldilocks, Zero-Um-Muitos, tours |
| Abordagem | Qual o estilo do trabalho? | Teste exploratório, teste roteirizado |
| Estratégia | Como combino tudo dado o risco e o contexto? | Plano de teste orientado a risco |
| Mnemônico | Como lembro de uma lista? | As próprias siglas (SFDIPOT, RCRCRC...) |

Note que a mesma coisa pode ocupar duas linhas: SFDIPOT é um modelo *e* um mnemônico; FEW HICCUPPS é uma lista
de oráculos *e* um mnemônico.

## 5. Duas escolas, dois vocabulários

**[Fonte] ISTQB CTFL v4.0:** classifica técnicas de teste em *caixa-preta* (particionamento de equivalência,
valor-limite, tabela de decisão, transição de estados), *caixa-branca* (teste de instrução e de ramo) e
*baseadas em experiência* (error guessing, teste exploratório, teste baseado em checklist), além de
*abordagens baseadas em colaboração* (escrita colaborativa de histórias, critérios de aceite, ATDD).

**[Fonte] RST / escola Context-Driven** (Kaner, Bach, Bolton e outros): trata o teste exploratório como uma
*abordagem* ou estilo, não como uma técnica entre outras; enfatiza habilidade, contexto e julgamento.

**[Interpretação] Como lidar com a divergência:**
- Ambas são úteis. O ISTQB oferece vocabulário comum e técnicas sistemáticas de cobertura; o RST oferece uma
  visão de teste como investigação e ferramentas de pensamento.
- Ao conversar com um time que segue ISTQB, use o vocabulário dele; ao defender exploração, explique o
  argumento do RST sem desqualificar o outro lado.
- Não diga "segundo o ISTQB" para ideias do RST nem o contrário.

## 6. Implicações práticas para o dia a dia

[Interpretação]
1. Antes de testar, descubra *para quem* o produto tem valor e o que ameaçaria esse valor.
2. Trate requisitos como uma fonte de oráculo (Claims), não como a verdade.
3. Use técnicas formais onde há regras, faixas e combinações; use exploração onde há incerteza.
4. Declare sempre cobertura e lacunas: "não vi problema" depende do que você olhou e de como olhou.
5. Prefira relatar risco a relatar contagem.
6. Transforme aprendizados estáveis em checagens; mantenha o teste humano para o que ainda é incerto.

## 7. Vieses que enganam o testador

[Interpretação, apoiada em literatura geral de vieses cognitivos — não específica de um autor de teste]
- **Viés de confirmação:** testar para provar que funciona. Antídoto: formular hipóteses de falha e tentar
  confirmá-las ou refutá-las.
- **Cegueira por desatenção:** seguir um roteiro e não perceber o que está fora dele. Kaner discute o risco de
  scripts induzirem esse efeito; antídoto: variar, alternar foco, parar para observar.
- **Ancoragem na especificação:** só procurar onde o documento aponta. Antídoto: múltiplos oráculos.
- **Efeito "já testei isso":** confiar em cobertura passada após mudanças. Antídoto: RCRCRC.
- **Normalização:** aceitar um comportamento estranho porque "sempre foi assim" (comum em legado). Antídoto:
  perguntar se é desejado ou apenas conhecido.
