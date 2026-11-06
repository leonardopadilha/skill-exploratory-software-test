# 03 — Heurísticas de ideias: "que experimento faço aqui?"

## Índice
1. ALTER FACE (Júlio de Lima) — componentes de UI web
2. Heurísticas do Test Heuristics Cheat Sheet (Hendrickson, Lyndsay, Emery)
   - Goldilocks · Zero-Um-Muitos · Começo-Meio-Fim · Interrupções · Ataques de tipo de dado
3. Tours exploratórios (James Whittaker)
4. Error guessing e ataques
5. Como escolher entre elas

Exemplos no **SRD** (`11-cenario-integrado.md`). Defeitos são **[Ilustrativo]**.

---

## 1. ALTER FACE

**Identificação [Fonte]:** heurística criada por **Júlio de Lima** (especialista brasileiro em teste de
software), publicada em slides em 2020 ("ALTER FACE — A Test Heuristic for UI Web Applications") e apresentada
em palestras. O autor descreve que a criou a partir da sua experiência testando aplicações Web modernas, para
oferecer atalhos a quem faz teste exploratório.

**Fundamentação [Fonte, parafraseado]:** aplicações Web modernas são construídas com componentes feitos à mão,
e isso faz surgir uma série de falhas ligadas à *forma como os componentes se comportam* — não à regra de
negócio. ALTER FACE propõe testes que revelam essas falhas.

| Letra | Princípio | Enunciado [Fonte, parafraseado] | Pergunta para o teste [Interpretação] |
|---|---|---|---|
| A | Activation | Objetos ativados por evento devem mudar de estado quando o evento ocorre | O componente reage a clique, hover, foco, seleção, tecla? Reage *só* quando deveria? |
| L | Layers | O objeto deve ocupar apenas a camada a que se destina | Está à frente/atrás do que deveria? (toast acima de tudo, dropdown acima do modal) |
| T | Timing | O objeto deve ficar visível exatamente pelo tempo pretendido | Aparece, some ou ganha foco no tempo certo? |
| E | Exclusion principle | Dois objetos não podem ocupar o mesmo espaço ao mesmo tempo | Dois popups, mensagens ou elementos se sobrepõem? |
| R | Removable | Objetos removíveis devem poder ser removidos ou escondidos | Fecha pelo X, por tecla (Esc), ao perder o foco? |
| F | Float | Objetos flutuantes devem flutuar durante a rolagem | Cabeçalho/menu/botão fixo continua visível e no lugar ao rolar? |
| A | Achievable | Objetos fora da área visível devem ser alcançáveis | Itens escondidos têm rolagem ou controles para serem vistos? |
| C | Collision | O objeto não deve colidir com outros enquanto se move | Um elemento em movimento empurra ou desloca outros? |
| E | Expansion | Objetos internos não devem sobrepor nem expandir o tamanho normal do container | Um item em foco/hover altera o tamanho do menu ou empurra vizinhos? |

**Quando usar:** telas com componentes dinâmicos (modais, toasts, dropdowns, menus, carrosséis, abas,
acordeões, tooltips, cabeçalhos fixos); após mudança de biblioteca de componentes ou de CSS; em telas
responsivas; quando o time de front-end é grande e cada um constrói componentes "à mão".

**Como aplicar:**
1. Inventarie os componentes dinâmicos da tela (isso já é modelagem: SFDIPOT → Structure/Interfaces).
2. Para cada componente, percorra as nove letras perguntando "isso se aplica aqui?". Nem toda letra se aplica
   a todo componente.
3. Varie as condições: tamanho de janela, zoom do navegador, conteúdo longo, muitos itens, conexão lenta,
   teclado em vez de mouse, dois componentes abertos ao mesmo tempo.
4. Use oráculos (`02`) para julgar: Product (os outros modais se comportam assim?), Users' expectations,
   Image.

**Exemplo prático — tela "Nova solicitação" do SRD moderno [Ilustrativo]:**
- **Activation:** ao selecionar categoria "Quilometragem", o campo "km" deveria aparecer e o campo
  "comprovante" deixar de ser obrigatório. Teste: alternar várias vezes entre categorias; selecionar por
  teclado; colar valor antes de trocar de categoria.
- **Layers:** o calendário do campo "data da despesa" abre *atrás* do modal de item em telas menores.
- **Timing:** o toast "Rascunho salvo" some em 1 s — antes que o usuário leia; ou nunca some e cobre o botão
  "Enviar".
- **Exclusion:** ao salvar com erro de validação, o toast de erro e o balão de ajuda do campo aparecem no mesmo
  lugar, um sobre o outro.
- **Removable:** o modal "Adicionar item" não fecha com Esc; ou fecha ao clicar fora *e perde os dados digitados*
  sem confirmação.
- **Float:** o rodapé fixo com "Total: R$ ..." e o botão "Enviar" somem ao rolar uma solicitação com 15 itens.
- **Achievable:** a lista de anexos mostra 3 de 10 arquivos, sem rolagem — os outros 7 não podem ser removidos.
- **Collision:** a barra de progresso do upload empurra o botão "Enviar" para baixo enquanto avança, e o usuário
  clica no botão errado.
- **Expansion:** o nome de arquivo longo ("comprovante_hotel_sao_paulo_marco_2026_reuniao_cliente.pdf") estica o
  card do item e quebra o layout da tabela.

**Raciocínio do sênior:** "Esses defeitos raramente estão em casos de teste, porque ninguém os especifica — mas
o usuário os sente na primeira hora de uso. Uso ALTER FACE em 20–30 minutos por tela, depois de entender o
fluxo, e não antes: preciso saber quais componentes importam mais. O de Collision perto do botão 'Enviar' me
preocupa mais que o de Expansion: clicar no lugar errado tem consequência."

**Possíveis problemas:** ver lista acima; em geral, defeitos de usabilidade que viram defeitos funcionais quando
o usuário não consegue concluir uma ação ou executa a ação errada.

**Limitações:** foca em comportamento visual de componentes; não cobre regras de negócio, dados, integrações
nem segurança. Não substitui teste de acessibilidade (embora ajude: Removable por teclado, Achievable). Combine
com SFDIPOT para o produto inteiro e com FEW HICCUPPS para julgar.

**Moderno × legado [Interpretação]:**
- Moderno (SPA): os princípios se aplicam diretamente; atenção a estado do componente após navegação sem
  recarregar a página (o modal some, mas o *overlay* fica — Layers + Removable).
- Legado: muitas telas têm menos componentes dinâmicos, mas bibliotecas antigas (jQuery UI, componentes
  JSF/PrimeFaces antigos, controles Web Forms com AJAX) também falham assim, com o agravante de navegadores
  modernos renderizando HTML antigo. No legado, *Timing* inclui o postback: a mensagem de sucesso desaparece
  porque a página recarrega. *Achievable* inclui tabelas largas em `<iframe>` ou `frameset` sem rolagem.

## 2. Heurísticas do Test Heuristics Cheat Sheet

**Identificação [Fonte]:** o *Test Heuristics Cheat Sheet* reúne ideias originais de **Elisabeth Hendrickson,
James Lyndsay e Dale Emery**; uma versão atualizada foi publicada pela Ministry of Testing com contribuições
da comunidade. Hendrickson também aprofunda várias dessas ideias no livro *Explore It!* (2013).

### Goldilocks (muito grande, muito pequeno, na medida)
- **Quando:** campos com tamanho, quantidade ou valor; arquivos; listas.
- **Como:** para cada entrada, teste o "na medida", o "pequeno demais" e o "grande demais" — inclusive o vazio.
- **SRD [Ilustrativo]:** justificativa com 19, 20 e 5.000 caracteres; arquivo de 0 byte, 5 MB e 5 MB + 1 byte;
  solicitação com 0 itens.
- **Relação:** é a intuição por trás da análise de valor-limite (`04`), sem exigir partições formais.

### Zero, Um, Muitos
- **Quando:** coleções, listas, relacionamentos, paginação, textos com plural.
- **SRD [Ilustrativo]:** gestor com 0, 1 e 300 aprovações pendentes; "1 itens" no resumo; paginação que aparece
  com 0 resultados; exportação CSV vazia sem cabeçalho.

### Começo, Meio, Fim
- **Quando:** posições em listas, sequências, textos, arquivos.
- **SRD [Ilustrativo]:** excluir o primeiro, um do meio e o último item; espaço no início/fim do valor colado;
  último dia do prazo de 90 dias.

### Interrupções
- **Quando:** fluxos com várias etapas, uploads, transações, sessões.
- **Como:** interrompa no meio: feche a aba, volte no navegador, perca a rede, deixe a sessão expirar, clique
  duas vezes, atualize a página.
- **SRD [Ilustrativo]:** fechar a aba durante o upload do 4º comprovante; voltar do navegador após "Enviar" e
  reenviar; sessão expirar com o formulário preenchido.

### Ataques de tipo de dado
- **Como:** valores fora do tipo esperado: texto em campo numérico, separadores decimais diferentes (vírgula ×
  ponto), notação científica, números negativos, Unicode, emojis, espaços, HTML/script em campos de texto.
- **SRD [Ilustrativo]:** "1.234,56" × "1,234.56" × "1234.56"; "-50"; "1e3"; justificativa com emoji indo para o
  arquivo do ERP.

[Interpretação] Outras ideias da comunidade aparecem na versão atualizada da cheat sheet; antes de citar uma
como "de Hendrickson", confira se ela está nas ideias originais.

## 3. Tours exploratórios

**Identificação [Fonte]:** **James Whittaker**, *Exploratory Software Testing* (2009), propõe a *metáfora do
turista*: o software é uma cidade dividida em "distritos", e cada distrito tem tours com um objetivo de teste.
O livro organiza tours em distritos como o de Negócios, o Histórico, o Turístico, o de Entretenimento, o
Hoteleiro e o "Decadente" (Seedy). Whittaker também é autor de *How to Break Software*, um catálogo de ataques.

**Exemplos de tours úteis [Fonte para os nomes; aplicação Ilustrativa]:**
- **Landmark (Negócios):** percorrer as funcionalidades principais em ordens diferentes. SRD: criar → enviar →
  aprovar → pagar, depois aprovar antes de anexar, cancelar depois de devolver.
- **Intellectual (Negócios):** submeter o software às entradas mais exigentes. SRD: solicitação com 50 itens, 10
  moedas, 10 anexos de 5 MB.
- **Bad-Neighborhood (Histórico):** voltar às áreas que historicamente concentram bugs. SRD: conversão de moeda,
  que já teve três correções.

**Quando usar:** para dar forma a uma sessão exploratória sem roteiro; para variar a perspectiva de quem já
conhece muito o sistema.

**Limitações:** os nomes podem virar jargão sem conteúdo; o valor está no objetivo de cada tour, não no nome.

## 4. Error guessing e ataques

**Identificação [Fonte]:** o ISTQB (CTFL v4.0) descreve *error guessing* como técnica baseada em experiência:
antecipar erros, defeitos e falhas a partir do conhecimento do testador (histórico da aplicação, erros comuns
de desenvolvedores, falhas em aplicações semelhantes). Uma forma metódica é manter uma lista de falhas
possíveis e desenhar testes para expô-las.

**[Interpretação] Lista de partida para aplicações web** (expanda com o histórico do seu produto):
duplo clique/duplo envio; voltar do navegador após ação; F5 após POST; sessão expirada no meio do fluxo;
mesma entidade aberta em duas abas; arredondamento monetário; fuso horário e horário de verão; caracteres
especiais e acentos em integrações; arquivos com extensão trocada; IDs sequenciais acessíveis por URL (falha
de autorização); listas vazias; paginação no último item; cache exibindo dado antigo.

## 5. Como escolher entre elas

[Interpretação]
| Situação | Comece com |
|---|---|
| Tela rica em componentes dinâmicos | ALTER FACE |
| Campos e dados de entrada | Goldilocks, ataques de tipo de dado, depois técnicas formais |
| Listas, coleções, relacionamentos | Zero-Um-Muitos, Começo-Meio-Fim, CRUD |
| Fluxos longos e transações | Interrupções, tours Landmark |
| Área com histórico de bugs | Bad-Neighborhood, error guessing |
| Sem tempo e sem documentação | Tours + FEW HICCUPPS como oráculo |

Heurísticas de ideias geram *experimentos*; elas não dizem se o resultado é um problema — isso é papel dos
oráculos (`02`) — nem garantem cobertura sistemática — isso é papel das técnicas formais (`04`).
