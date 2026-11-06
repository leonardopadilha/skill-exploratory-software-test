# 02 — Oráculos: "isso é um problema?"

## Índice
1. O que é um oráculo e por que ele é falível
2. FEW HICCUPPS — fonte primária e erros comuns de interpretação
3. Como aplicar FEW HICCUPPS durante o teste
4. Exemplo completo no SRD
5. Oráculos conflitantes
6. Oráculos sem documentação (legado)
7. Oráculos automatizáveis × humanos

Defeitos citados são **[Ilustrativo]**.

---

## 1. O que é um oráculo

**[Fonte] Michael Bolton:** oráculo é um princípio ou mecanismo heurístico pelo qual alguém reconhece um
problema. Se percebemos um problema, é porque um oráculo nos disse; se não percebemos, não significa que não
há problema — talvez não estejamos aplicando o oráculo certo. Oráculos são falíveis e dependentes de contexto.

**[Fonte] Bolton:** oráculos podem ser usados antes do teste (para desenhá-lo), durante (quando algo "salta aos
olhos") e depois (ao reinterpretar algo visto antes). A qualquer momento, você pode citar um oráculo para
explicar *por que* acredita que algo é um problema.

## 2. FEW HICCUPPS

**Identificação [Fonte]:** o núcleo HICCUPPS são heurísticas de consistência associadas a **James Bach**,
apresentadas e popularizadas por **Michael Bolton** (artigo "Testing Without a Map", 2005, e materiais do Rapid
Software Testing). Em 2012, Bolton publicou a extensão **FEW HICCUPPS**: com o acréscimo de *Explainability* e
*World*, o *Familiar problems* (que já existia como "F" ao final) foi movido para o início.

| Letra | Oráculo | Esperamos que o produto seja... [Fonte, parafraseado] |
|---|---|---|
| F | Familiar problems | **in**consistente com padrões de problemas conhecidos |
| E | Explainability | explicável — conseguimos explicar o comportamento a alguém |
| W | World | consistente com o que sabemos sobre o mundo real |
| H | History | consistente com versões anteriores dele mesmo |
| I | Image | consistente com a imagem que a organização quer projetar |
| C | Comparable products | consistente com produtos comparáveis (concorrentes, sistema antigo, planilha, formulário em papel) |
| C | Claims | consistente com o que pessoas importantes dizem sobre ele (documentos, conversas, marketing) |
| U | Users' desires/expectations | consistente com o que usuários razoavelmente querem |
| P | Product | consistente consigo mesmo (elementos comparáveis dentro do próprio produto) |
| P | Purpose | consistente com os propósitos explícitos e implícitos de uso |
| S | Standards and Statutes | consistente com normas e leis aplicáveis |

**Atenção a erros frequentes em fontes secundárias:**
- **Familiar** NÃO é "consistente com funcionalidades semelhantes" — isso é o oráculo *Product*. Na fonte
  primária, Familiar é a única heurística de *inconsistência*: queremos que o produto **não** se pareça com
  bugs conhecidos.
- **Claims** inclui não só documentos, mas também inferência (o que alguém importante diria) e conversa
  (o que alguém importante diz). A afirmação pode estar errada — o teste pode revelar um problema *na
  afirmação*, não no produto.
- **Comparable products** inclui o sistema que está sendo substituído, uma planilha, um formulário em papel,
  ou produtos de outra categoria que processam os mesmos dados.
- **History** é um caso particular de Comparable products (o produto comparado consigo mesmo no passado).
- O papel do testador diante de *claims* conflitantes **não** é decidir quem tem razão, e sim tornar a
  divergência visível.

## 3. Como aplicar FEW HICCUPPS

**Quando usar:** sempre que algo "parece estranho" e você precisa decidir se reporta; quando não há
especificação; quando há discussão "bug × feature"; ao revisar relatos de bug ("por que isso é um problema?");
ao desenhar testes (perguntando "o que violaria Purpose aqui?").

**Como aplicar [Interpretação]:**
1. **Prospectivamente:** antes de uma sessão, escolha 3–4 oráculos mais relevantes para a área (ex.: em
   cálculo monetário → Claims, World, Standards, Product).
2. **No momento:** ao notar algo, pergunte "com o que isso é inconsistente?" e percorra a lista mentalmente.
3. **No relato:** cite o oráculo e a pessoa afetada. Quanto mais oráculos concordarem, mais forte o argumento.
4. **Quando nenhum oráculo acusa nada:** registre como "curiosidade" ou pergunta; não descarte.

**Raciocínio do sênior:** "Eu não pergunto 'isso está certo?'. Pergunto 'em relação a quê isso estaria errado,
e quem se importaria?'. Essa troca transforma uma impressão vaga em argumento."

## 4. Exemplo completo no SRD [Ilustrativo]

**Observação:** colaborador lança uma despesa de alimentação de R$ 130,00 num dia em que o limite é R$ 120,00.
O sistema aceita sem aviso, e o gestor só vê "R$ 130,00" na tela de aprovação.

Percorrendo os oráculos:
- **Claims:** a política de viagens (documento do RH) diz que o limite diário é R$ 120. → inconsistente.
- **History:** na versão legada, o sistema bloqueava o valor e pedia justificativa. → inconsistente.
- **Product:** a categoria Hospedagem mostra alerta ao exceder limite; Alimentação não. → inconsistente.
- **Purpose:** o sistema existe para dar controle ao gestor; ele aprova sem saber do excesso. → inconsistente.
- **Users' expectations:** o colaborador esperava ser avisado para não ter o reembolso glosado depois.

**Como relatar:** "Despesas de alimentação acima do limite diário são aceitas sem alerta nem justificativa
(R$ 130 em dia com limite de R$ 120). Isso contradiz a política de viagens (Claims), a versão anterior
(History) e o comportamento da categoria Hospedagem (Product). Risco: gestor aprova excesso sem perceber e o
financeiro glosa depois — retrabalho e atrito com o colaborador."

Note como a mesma observação ficou muito mais forte com cinco oráculos do que com "não validou o limite".

## 5. Oráculos conflitantes

[Fonte] Bolton observa que oráculos podem se contradizer — por exemplo, um bug corrigido torna o produto
inconsistente com sua história, e usuários que tinham um contorno podem ficar insatisfeitos.

[Interpretação] Quando oráculos conflitam:
- Explicite o conflito no relato ("Claims diz X, mas History e Users' expectations dizem Y").
- Identifique quem decide (normalmente o dono do produto) e entregue a informação para decisão.
- Não "resolva" o conflito por conta própria ajustando o teste para passar.

## 6. Oráculos sem documentação (legado)

[Interpretação] Em sistemas sem documentação, os oráculos mais produtivos costumam ser:
- **History:** comportamento atual em produção (ou em um ambiente com a versão anterior) é a melhor
  referência. Testes de caracterização (Feathers) formalizam isso — ver `09-web-legada.md`.
- **Comparable products:** relatórios antigos, planilhas que o negócio usa, o sistema que vai substituir.
- **Claims por conversa:** o especialista que "sabe como funciona". Registre o que ele disse e quando.
- **World:** regras do mundo real (calendário, feriados, arredondamento bancário, legislação).
- **Product:** comparar telas e caminhos diferentes que produzem o mesmo dado.

## 7. Oráculos automatizáveis × humanos

[Interpretação]
- Facilmente automatizáveis: Claims bem especificadas (valores esperados), History (comparação com saída de
  referência — "golden master"), parte de Standards (validadores de HTML, acessibilidade básica), Product
  (comparar a mesma informação em duas telas ou na UI e na API).
- Difíceis de automatizar: Image, Purpose, Users' expectations, Explainability, World em sentido amplo.
  Esses dependem de julgamento — e por isso justificam tempo de teste humano.
