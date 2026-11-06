# 09 — Aplicações web legadas

## Índice
1. O que "legado" significa para o teste
2. Onde os defeitos costumam morar
3. Ciclo de requisição, postback e estado no servidor
4. Sessão no servidor
5. Lógica no banco e jobs agendados
6. Integrações por arquivo
7. Navegadores e componentes antigos
8. Pouca documentação e conhecimento concentrado
9. Testes de caracterização (Feathers) e o oráculo History
10. O que adaptar — e o que não forçar

Exemplos do SRD legado (ASP.NET Web Forms, monolito, banco único, job noturno para o ERP) são **[Ilustrativo]**.
O raciocínio vale para JSP/JSF, PHP tradicional e similares.

---

## 1. O que "legado" significa para o teste

**[Fonte] Michael Feathers**, *Working Effectively with Legacy Code*, define código legado como código sem
testes. **[Interpretação]** Para o testador, legado costuma significar também: tecnologia antiga, documentação
insuficiente, alto acoplamento, medo de mudar e conhecimento concentrado em poucas pessoas. Isso muda menos *as
técnicas* e mais *os oráculos disponíveis, os riscos de efeito colateral e o custo de observar*.

## 2. Onde os defeitos costumam morar

[Interpretação]
- **Efeitos colaterais:** uma alteração numa procedure usada por dez telas.
- **Regras espalhadas:** a mesma validação em JavaScript, no code-behind e numa trigger — com diferenças.
- **Estado no servidor:** sessão, ViewState, variáveis de aplicação.
- **Processos fora da tela:** jobs noturnos, arquivos de integração, relatórios agendados.
- **Ambiente:** configurações de servidor, versões de runtime, codificação de caracteres, navegadores modernos
  renderizando HTML antigo.

## 3. Ciclo de requisição e postback

**Conceito:** cada interação relevante envia o formulário ao servidor e recarrega a página inteira (postback em
Web Forms; submit de formulário em JSP/PHP). Em JSF e Web Forms, parte do estado da tela pode viajar num campo
oculto (ViewState) ou ficar na árvore de componentes no servidor.

**O que testar:**
- F5 após um POST (reenvio do formulário → duplicidade).
- Botão voltar após salvar e reenviar.
- Mensagens que desaparecem no recarregamento (*Timing* do ALTER FACE adaptado ao postback).
- Campos que perdem valor após um postback causado por outro campo (ex.: trocar a categoria limpa o valor).
- Postbacks em cascata (dropdown dependente) com rede lenta: o usuário clica de novo durante o recarregamento.
- Página muito pesada (ViewState grande) em conexões lentas.

**SRD [Ilustrativo]:** ao trocar a categoria, o postback recarrega a grade de itens e o anexo selecionado (ainda
não enviado) é perdido sem aviso — `<input type=file>` não sobrevive ao postback. Oráculos: Users'
expectations, Purpose.

## 4. Sessão no servidor

**O que testar:** expiração (timeout fixo, ex.: 20 min) com formulário preenchido; duas abas compartilhando a
mesma sessão (dados de uma aparecem na outra); dados da sessão anterior após logout/login com outro usuário no
mesmo navegador; balanceador sem afinidade de sessão (perde o usuário entre servidores).

**SRD [Ilustrativo]:** o gestor abre duas solicitações em abas diferentes; a sessão guarda "solicitação atual"
numa variável única; ao aprovar na primeira aba, aprova a segunda. Defeito grave, invisível em teste de uma aba
só. Heurística: Interrupções + concorrência; oráculo Purpose.

## 5. Lógica no banco e jobs agendados

**Perguntas a fazer [Interpretação]:** que procedures/triggers participam desta funcionalidade? Que outras telas
usam a mesma procedure? Que jobs rodam sobre estas tabelas, e quando? O que acontece se o job rodar duas vezes,
ou não rodar?

**SRD [Ilustrativo]:** o job noturno gera o arquivo para o ERP com as solicitações aprovadas até 23h59. Uma
aprovação às 23h59min58s, concluída às 00h00min01s no banco, fica fora do arquivo de hoje *e* do de amanhã
(o filtro usa a data de aprovação e o job do dia seguinte busca "aprovadas hoje"). AVL em horário + "siga o
dado" revela o caso.

## 6. Integrações por arquivo

**O que testar:** layout posicional (tamanhos de campo, preenchimento, alinhamento); codificação (acentos em
ISO-8859-1 vs. UTF-8); separador decimal; truncamento silencioso de campos longos; arquivo vazio; reprocessamento
do mesmo arquivo; nomes de arquivo com data/hora (colisão quando gerado duas vezes).

**SRD [Ilustrativo]:** a justificativa com "ç" vira "Ã§" no ERP; o nome do colaborador com 61 caracteres é
cortado em 60 e o ERP não encontra o fornecedor correspondente.

## 7. Navegadores e componentes antigos

**O que testar:** a aplicação foi homologada para navegadores que não existem mais; navegadores atuais mudaram
comportamentos (cookies de terceiros, pop-ups bloqueados, plugins removidos, modos de compatibilidade). Verifique
calendários, máscaras de campo, grades, upload, impressão e janelas pop-up com bibliotecas descontinuadas.

ALTER FACE se aplica: componentes jQuery UI, controles JSF/Web Forms com AJAX e janelas pop-up falham em
Layers, Removable, Achievable e Expansion — principalmente em telas menores do que as previstas na época.

## 8. Pouca documentação e conhecimento concentrado

[Interpretação] Estratégia de um sênior:
1. **Reconhecimento exploratório** para construir modelo e glossário (SFDIPOT, diagrama de estados).
2. **Entrevista curta e focada** com quem conhece: "o que quebra com frequência?", "o que você tem medo de
   mexer?", "que regra não está escrita em lugar nenhum?". Registre o que ouviu como *Claims* datadas.
3. **Chamados de suporte e logs de erro** como mapa de risco (Chronic do RCRCRC).
4. **Relatórios e planilhas do negócio** como oráculo Comparable products.
5. **Documentar o que descobrir** — regras, estados, dúvidas. É um resultado valioso do teste.

## 9. Testes de caracterização e o oráculo History

**Identificação [Fonte]:** termo cunhado por **Michael Feathers** para testes que descrevem (caracterizam) o
comportamento *atual* de um código existente, protegendo-o contra mudanças não intencionais. Feathers argumenta
que, em sistemas legados, o que o sistema faz costuma importar mais do que o que ele deveria fazer.

**[Interpretação] Adaptação para o analista de testes:** mesmo sem escrever código, aplique a ideia antes de
uma mudança:
1. Escolha entradas representativas (partições e limites) e registre as saídas atuais (tela, relatório,
   arquivo do ERP).
2. Após a mudança, compare. Diferenças são *perguntas*: intencional ou regressão?
3. Comportamentos estranhos encontrados na caracterização não são "o correto" — são "o atual". Leve-os ao dono
   do produto antes de decidir se a mudança deve preservá-los.

Isso operacionaliza o oráculo **History** e é especialmente útil em migrações (legado → moderno), onde o
sistema antigo vira o oráculo **Comparable products** do novo.

## 10. O que adaptar — e o que não forçar

[Interpretação]
- Não exija automação de UI em telas com IDs gerados dinamicamente e sem ambiente estável; prefira caracterização
  por saídas (arquivos, relatórios, banco) e exploração bem documentada.
- Não presuma API: muitas vezes não há. O equivalente a TuTTu/TaTTa é comparar a tela com o banco, o log ou o
  arquivo gerado.
- Não ignore técnicas modernas onde elas cabem: uma camada de serviço antiga pode ser testada diretamente; um
  proxy pode observar o tráfego; checagens de comparação de arquivos são simples de automatizar.
- Respeite as restrições do ambiente (janelas de manutenção, dados compartilhados, jobs que afetam outros
  times) — planeje o teste com quem opera o sistema.
