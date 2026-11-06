# 06 — Risco, priorização e quando parar

## Índice
1. Por onde começar: perguntas de contexto
2. Identificando riscos
3. Priorizando: probabilidade × impacto, com julgamento
4. RCRCRC — regressão após mudanças (Karen N. Johnson)
5. Decidir a profundidade: aprofundar ou seguir
6. Quando parar
7. Exemplo: planejando o teste de uma mudança no SRD

Defeitos e números do SRD são **[Ilustrativo]**.

---

## 1. Por onde começar: perguntas de contexto

[Interpretação] Antes de escolher técnicas, um sênior descobre o suficiente para saber o que importa. Se não
houver respostas, ele assume hipóteses explícitas e segue, revisando-as à medida que aprende.

**Sobre valor e pessoas:** quem usa? Quem paga? Quem sofre se falhar (usuário, financeiro, auditoria, suporte,
reputação)? Há obrigação legal ou regulatória?
**Sobre a mudança:** o que mudou exatamente (código, configuração, dados, dependência)? Por quê? O que *não*
deveria ter mudado?
**Sobre a história:** onde o sistema já falhou? Que chamados de suporte se repetem? Que área "ninguém gosta de
mexer"?
**Sobre a arquitetura:** que componentes e integrações participam? Onde está o estado? O que é síncrono e o que
é assíncrono? Há jobs agendados?
**Sobre o projeto:** quanto tempo há? Que ambientes e dados estão disponíveis? Quem pode responder perguntas?
O que já é coberto por checagens automatizadas?

## 2. Identificando riscos

[Interpretação] Duas direções complementares:
- **De dentro para fora:** a partir do produto (SFDIPOT), pergunte em cada elemento "o que pode dar errado?".
- **De fora para dentro:** a partir de ameaças conhecidas (pessoas que importam, critérios de qualidade,
  bugs históricos, falhas típicas da tecnologia), pergunte "onde isso pode acontecer no produto?".

Escreva cada risco como **condição → consequência → quem é afetado**:
"Se a cotação usada for a do dia do lançamento, o colaborador recebe valor diferente do gasto, e o financeiro
tem divergência na conciliação." Isso evita riscos vagos como "câmbio".

[Fonte, para aprofundar] James Bach escreveu sobre *teste baseado em risco heurístico* — identificação de
riscos por listas de perguntas e categorias, sem pretensão de precisão numérica (ver `referencias.md`).

## 3. Priorizando

[Interpretação] Uma matriz simples de probabilidade × impacto ajuda, mas não substitui argumento. Fatores que
aumentam **probabilidade**: código novo ou muito alterado, complexidade, integrações, concorrência, histórico de
bugs, tecnologia nova para o time, pressão de prazo. Fatores que aumentam **impacto**: dinheiro, dados
pessoais, segurança, obrigações legais, volume de usuários, irreversibilidade, visibilidade.

**Irreversibilidade merece peso especial:** um erro de layout é corrigido no próximo deploy; um pagamento
duplicado enviado ao banco, não.

## 4. RCRCRC

**Identificação [Fonte]:** heurística de **Karen N. Johnson** (publicada em 2009) para decidir o que testar em
regressão quando o software muda.

| Letra | Pergunta [Fonte, parafraseado] | SRD [Ilustrativo] |
|---|---|---|
| Recent | O que é novo ou foi alterado recentemente? | Novo serviço de câmbio |
| Core | Que funções essenciais precisam continuar funcionando? | Enviar, aprovar, pagar |
| Risky | Que áreas são inerentemente mais arriscadas? | Cálculo monetário, integração com ERP |
| Configuration sensitive | Que código depende de configuração de ambiente? | URL do serviço de cotação, fuso do servidor, limites de upload |
| Repaired | Que correções podem ter introduzido novos problemas? | Correção recente do arredondamento |
| Chronic | Que áreas quebram com frequência? | Exportação CSV |

**Quando usar:** antes de cada liberação; para escolher o subconjunto de testes de regressão; para revisar se a
suíte automatizada cobre o que importa.

**Raciocínio do sênior:** "Configuration sensitive é a letra que mais esquecem. O bug só aparece em produção
porque o fuso do servidor é diferente do de homologação."

**Limitações:** indica *onde* olhar, não *como* testar; combine com técnicas e oráculos.

## 5. Decidir a profundidade

[Interpretação] Sinais para **aprofundar**: o achado envolve dinheiro, dados, segurança ou estado inconsistente;
o comportamento é inexplicável; há indícios de padrão sistêmico; o risco está no topo da lista.
Sinais para **anotar e seguir**: impacto baixo e isolado; já há informação suficiente para quem decide; o
aprofundamento cabe melhor em outra sessão ou exige ambiente/dado indisponível agora.

Pergunta-chave: "O que eu sei agora é suficiente para alguém tomar uma boa decisão? Se não, o que falta?"

## 6. Quando parar

[Interpretação] Testes nunca são completos; parar é uma decisão de risco, não uma constatação de que "não há mais
bugs". Sinais razoáveis para encerrar (ou encerrar uma linha de investigação):
- A missão foi cumprida: a informação que motivou o teste foi obtida.
- O tempo acabou e quem decide prefere a informação atual a mais teste.
- Novos testes deixaram de revelar informação nova relevante (rendimento decrescente).
- A missão mudou ou perdeu sentido (a funcionalidade foi cortada ou será reescrita).
- O teste está bloqueado por obstáculos que precisam ser escalados.

[Fonte, para aprofundar] Michael Bolton publicou uma lista de *heurísticas de parada* (por exemplo: tempo
esgotado, missão cumprida, rendimento decrescente) — ver `referencias.md`.

**Ao parar, comunique:** o que foi coberto, o que não foi, que riscos permanecem e qual a sua avaliação (ver
`10-comunicar.md`). Parar sem declarar lacunas transforma "não testei" em "funciona" na cabeça de quem lê.

## 7. Exemplo: planejando o teste de uma mudança no SRD [Ilustrativo]

**Mudança:** o serviço de câmbio foi substituído por outro fornecedor de cotações. Prazo: 2 dias.

**Raciocínio:**
1. *Contexto:* afeta despesas em moeda estrangeira (≈ 15% das solicitações, mas concentradas na diretoria
   comercial, que viaja e aprova valores altos). Impacto financeiro direto.
2. *RCRCRC:* Recent (novo fornecedor), Core (conversão no envio), Risky (dinheiro), Configuration sensitive
   (credenciais e URL por ambiente, fuso), Repaired (arredondamento corrigido mês passado), Chronic (exportação).
3. *Riscos priorizados:*
   1. Cotação errada (data, moeda, sentido compra/venda) → valor pago errado → colaborador e financeiro.
   2. Indisponibilidade do fornecedor → envio bloqueado ou valor zerado → todos com despesa internacional.
   3. Arredondamento diferente do anterior → divergências de centavos na conciliação → financeiro.
   4. Feriados/fins de semana sem cotação → valor indefinido.
4. *Estratégia:* AVL nas datas (sexta/sábado/domingo/feriado, 90 dias); tabela de decisão para moeda × data
   disponível; comparação com o fornecedor antigo (Comparable products/History) para 30 despesas reais
   anonimizadas; teste de falha do fornecedor (simular timeout); "siga o dado" até o arquivo do ERP.
5. *Fora do escopo declarado:* desempenho sob carga (risco baixo para este volume); telas não relacionadas.
6. *Critério de parada:* os 4 riscos com informação suficiente para decisão; divergências explicadas ou
   reportadas.
