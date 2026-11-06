# 07 — Isolar e investigar: "onde o defeito mora?"

## Índice
1. Investigação como raciocínio: observação, hipóteses, refutação
2. TuTTu / TaTTa — testar *a* UI vs. *através* da UI; *a* API vs. *através* da API
3. Dimensões de isolamento
4. Reprodução mínima
5. Bugs intermitentes
6. Exemplo completo no SRD (moderno e legado)
7. O que registrar durante a investigação

Defeitos **[Ilustrativo]**.

---

## 1. Investigação como raciocínio

**[Fonte] Bolton:** como oráculos são falíveis, o testador relata o que *plausivelmente* parece um problema; para
decidir plausibilidade, aplica inferência abdutiva — ciclos de raciocínio para a melhor explicação: coletar
dados, formular hipóteses que expliquem os dados, avaliá-las e então escolher uma ou buscar mais dados.

**[Interpretação] Na prática:**
1. **Separe fato de interpretação.** "O toast verde apareceu" é fato; "salvou" é interpretação.
2. **Formule hipóteses concorrentes** — pelo menos duas. Uma só vira viés de confirmação.
3. **Desenhe o experimento que distingue as hipóteses** (o que eu veria se A fosse verdade e não B?).
4. **Mude uma variável por vez** ao isolar.
5. **Pare quando a explicação for suficiente para o objetivo:** o testador não precisa achar a linha de código;
   precisa dar ao time informação para agir.

## 2. TuTTu / TaTTa

**Identificação:** a ideia de distinguir *testar a UI* (Testing the UI) de *testar através da UI* (Testing
through the UI), e o mesmo para a API (Testing the API / Testing through the API), circula na comunidade com o
mnemônico **TuTTu/TaTTa**, atribuído a **Mark Winteringham** (autor de *Testing Web APIs*, 2022) em materiais
comunitários. **[Atribuição a confirmar]**: não localizamos a publicação original do autor; a distinção entre
testar "a" UI e "através da" UI também aparece em outros autores.

**Fundamentação [Interpretação]:**
- *Testar a UI*: o foco é a lógica e a apresentação do front-end — renderização, validações no cliente,
  componentes, serialização do que a tela envia.
- *Testar através da UI*: usar a interface para exercitar o sistema inteiro (regras, persistência, integrações).
- *Testar a API*: o contrato e o comportamento da API em si (status, validação, autorização, formato).
- *Testar através da API*: usar a API para exercitar regras e efeitos no back-end, sem a interface.

**Quando usar:** para isolar a camada de um defeito; para decidir onde um teste deve viver (e onde automatizar);
para descobrir validações que existem só no front-end.

**Como aplicar para isolar:** reproduza o mesmo comportamento pelos dois caminhos e compare:

| UI | API | Hipótese mais provável |
|---|---|---|
| Falha | Funciona | Defeito no front-end (envio, estado, renderização) |
| Funciona | Falha | API não usada pela UI dessa forma, ou validação só no front (risco de bypass) |
| Falha | Falha | Defeito no back-end, dados ou integração |
| Funciona | Funciona | Condição não reproduzida (tempo, sessão, dados, concorrência) — investigue as outras dimensões |

**Limitações:** exige acesso à API (ferramentas como o painel Rede do navegador, Postman, curl) e algum
conhecimento técnico; no legado sem API, o equivalente é comparar a tela com o banco, com o log ou com o arquivo
gerado (ver seção 6).

## 3. Dimensões de isolamento

[Interpretação] Ao isolar, varie uma dimensão por vez:
- **Dados:** outro registro, outro valor, outro tamanho, caracteres diferentes.
- **Usuário/perfil:** outro papel, outra equipe, usuário recém-criado vs. antigo.
- **Ambiente:** outro navegador, outra resolução, outro servidor, outra configuração.
- **Tempo:** outra hora (fuso!), outro dia (fim de semana, fim de mês), sessão nova vs. longa.
- **Sequência:** a mesma ação após caminhos diferentes; com e sem passos intermediários.
- **Concorrência:** sozinho vs. com outra sessão agindo sobre o mesmo dado.
- **Estado prévio:** cache limpo vs. cheio; primeira vez vs. repetida.

## 4. Reprodução mínima

[Interpretação] Uma reprodução mínima é o menor conjunto de passos e dados que ainda produz o problema. Ela:
acelera a correção, evita discussão, revela a causa e serve de base para um teste de regressão. Técnica:
remover passos um a um enquanto o problema persistir; simplificar dados (de 15 itens para 1).

## 5. Bugs intermitentes

[Interpretação] "Não reproduz" geralmente significa "não controlamos alguma variável". Suspeitos frequentes:
tempo (timeouts, relógio, fuso, expiração), concorrência (duas requisições, duas abas, eventos fora de ordem),
estado escondido (cache, sessão, dados de teste reutilizados), ambiente (um dos servidores atrás do balanceador
tem configuração diferente). Registre *todas* as ocorrências com horário, usuário e dados; padrões aparecem na
terceira ou quarta.

## 6. Exemplo completo no SRD [Ilustrativo]

**Observação:** na lista "Minhas solicitações", a solicitação #4812 mostra total de R$ 1.230,00; no detalhe,
R$ 1.180,00.

**Hipóteses concorrentes:**
- H1: a lista mostra um valor em cache, desatualizado após a edição de um item.
- H2: a lista soma valores brutos (com câmbio não arredondado) e o detalhe soma valores arredondados.
- H3: a lista inclui um item excluído logicamente.

**Experimentos:**
- Para H1: recarregar a lista com cache limpo; editar outra solicitação e ver se a lista atualiza.
- Para H2: verificar se a #4812 tem item em moeda estrangeira; criar uma solicitação só em BRL e comparar.
- Para H3: ver o histórico — houve exclusão de item de R$ 50? (1.230 − 1.180 = 50.) Criar solicitação, excluir
  um item e comparar lista vs. detalhe.

**Resultado:** a diferença é exatamente R$ 50,00 e há um item excluído de R$ 50,00. H3 se fortalece; o
experimento controlado reproduz.

**Isolamento de camada (moderno):** a API `GET /solicitacoes?usuario=...` (usada pela lista) devolve
`total: 1230.00`; a API `GET /solicitacoes/4812` devolve `total: 1180.00`. As duas são do back-end → o defeito
não está no front-end. Provável: a consulta da listagem não filtra itens com exclusão lógica.

**Isolamento de camada (legado):** sem API, compare a tela com uma consulta ao banco (se permitido) ou com
outra saída do sistema (exportação, relatório). Se a lista vem de uma *view* e o detalhe de uma *procedure*,
o defeito pode estar na view — informação útil para o desenvolvedor, obtida sem ler código.

**Oráculos:** Product (duas telas do mesmo produto divergem), Claims (o total deveria ser a soma dos itens
ativos), Purpose (o gestor decide pela lista).

**Por que importa:** a tela de aprovação do gestor usa a mesma consulta da lista? Se sim, o gestor aprova com
total errado — o risco sobe. *Essa pergunta vale outro experimento antes de relatar.*

## 7. O que registrar durante a investigação

[Interpretação] Fatos observados (com dados e horário), hipóteses e o resultado de cada experimento (inclusive
as refutadas — elas economizam tempo do desenvolvedor), evidências (capturas, requisições, logs), reprodução
mínima, impacto e oráculos. O relato de bug é a síntese disso (`10-comunicar.md`).
