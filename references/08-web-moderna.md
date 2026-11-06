# 08 — Aplicações web modernas

## Índice
1. Onde os defeitos costumam morar
2. Estado no cliente e navegação em SPA
3. Assincronia, condições de corrida e UI otimista
4. APIs e microsserviços
5. Autenticação, tokens e sessão
6. Autorização
7. Tempo real (WebSockets/SSE) e eventos
8. Processamento em segundo plano e integrações externas
9. Responsividade e componentes
10. Ferramentas de observação ao alcance do testador

[Interpretação] As heurísticas são as mesmas de qualquer sistema; o que muda é onde olhar e como observar. Todos
os exemplos do SRD moderno (React + API REST + microsserviços + fila + WebSocket) são **[Ilustrativo]**.

---

## 1. Onde os defeitos costumam morar

- **Fronteiras entre componentes:** front ↔ API, serviço ↔ serviço, serviço ↔ fila, sistema ↔ terceiro.
- **Estado distribuído:** cada serviço tem "sua" verdade; a consistência pode ser eventual.
- **Tempo e ordem:** requisições paralelas, respostas fora de ordem, retentativas.
- **Validação só no cliente:** o front bloqueia; a API aceita.
- **Configuração por ambiente:** flags, URLs, credenciais, limites diferentes em cada ambiente.

## 2. Estado no cliente e navegação em SPA

**O que testar:** botão voltar/avançar do navegador; atualizar (F5) no meio de um fluxo; abrir a mesma rota em
duas abas; copiar a URL de uma tela interna e abrir em outra sessão; estado que "vaza" entre telas (filtros,
formulários, modais).

**SRD [Ilustrativo]:** após enviar a solicitação, o usuário clica "voltar" e cai no formulário ainda preenchido
com o botão "Enviar" ativo — reenviar cria uma duplicata. Ao trocar de solicitação na lista, o modal de anexo
mostra os arquivos da solicitação anterior (estado do componente não reiniciado).

**Heurísticas ligadas:** Interrupções; ALTER FACE (Removable, Layers); transição de estados com eventos do
navegador.

## 3. Assincronia, condições de corrida e UI otimista

**Conceito [Interpretação]:** *UI otimista* exibe o resultado antes da confirmação do servidor, para parecer
rápida. Se a confirmação falhar, a UI precisa desfazer e avisar — e muitas vezes não faz.

**O que testar:**
- Respostas lentas e fora de ordem (filtro digitado rápido: a resposta da busca "ab" chega depois da de "abc").
- Falha após feedback de sucesso (toast "Salvo" seguido de erro 500 silencioso).
- Duplo clique e cliques durante o carregamento.
- Ações que dependem de outra ainda pendente (enviar enquanto o upload termina).

**Como provocar:** limitação de rede no DevTools, modo offline, bloqueio de uma requisição específica,
servidores de mock com atraso.

**SRD [Ilustrativo]:** o upload do 3º comprovante ainda está em andamento quando o usuário clica "Enviar"; a
solicitação vai sem o anexo, e a regra de comprovante obrigatório não é reavaliada no servidor.

**Raciocínio do sênior:** "Quando vejo um toast de sucesso, pergunto: ele aparece depois da resposta do
servidor, ou antes? Abro a aba Rede e olho a sequência. Se aparece antes, quero ver o que acontece quando a
resposta é erro."

## 4. APIs e microsserviços

**O que testar:** contrato (campos, tipos, obrigatoriedade, códigos de status); validações espelhadas da UI
(mesmas partições e limites de `04`); idempotência de operações críticas (reenviar o mesmo POST); paginação e
ordenação na API; mensagens de erro (não vazam detalhes internos?); compatibilidade quando um serviço muda
versão e o outro não.

**Consistência eventual:** após uma ação, outra tela (servida por outro serviço) pode demorar a refletir.
Pergunte ao time qual atraso é aceitável e teste nos dois lados desse limite. Um atraso aceitável sem feedback
ao usuário é um problema de Purpose/Users' expectations.

**Heurística ligada:** TuTTu/TaTTa (`07`) — compare UI e API.

## 5. Autenticação, tokens e sessão

**O que testar [Interpretação]:**
- Expiração do token de acesso no meio de um formulário longo: o refresh acontece? Os dados são preservados?
- Refresh token expirado ou revogado; logout em uma aba com outra aberta.
- Sessões simultâneas (dois dispositivos); troca de senha invalida outras sessões?
- Fluxos de recuperação de senha (ver `12-casos-por-funcionalidade.md`).
- Login via provedor externo (OIDC/SSO): usuário removido no provedor continua acessando?

**SRD [Ilustrativo]:** access token de 15 minutos; o gestor abre a tela de aprovação, vai almoçar, volta e
aprova 10 solicitações — as 10 chamadas falham com 401, mas a tela mostra "Aprovado" em todas (UI otimista
+ token expirado).

## 6. Autorização

[Interpretação] A pergunta central: **o servidor verifica, em cada requisição, se *este* usuário pode fazer
*esta* ação sobre *este* recurso?** Esconder um botão não é controle de acesso.

**Testes de autorização no próprio sistema sob teste (ambiente de teste):**
- Trocar o identificador na URL/requisição para um recurso de outra pessoa (ex.: `/solicitacoes/4813`).
- Usar um perfil sem permissão para chamar diretamente a API de uma ação (colaborador chamando `aprovar`).
- Aprovar a própria solicitação sendo gestor (regra de segregação de funções).
- Verificar se dados de outros usuários aparecem em listas, exportações ou mensagens de erro.

Combine com uma tabela de decisão perfil × ação × estado (`04`).

## 7. Tempo real e eventos

**O que testar:** perda e retomada de conexão; eventos duplicados ou fora de ordem; usuário em duas abas
recebendo o mesmo evento; tela aberta por horas; o que o usuário vê se a notificação em tempo real nunca chegar.

**SRD [Ilustrativo]:** o status atualiza por WebSocket. Após a rede cair e voltar, o colaborador continua vendo
"Enviado" embora a solicitação tenha sido devolvida — não há sincronização na reconexão.

## 8. Processamento em segundo plano e integrações externas

**O que testar:**
- Idempotência: a mesma mensagem processada duas vezes gera efeito duplo? (pagamento duplicado!)
- Retentativas e falhas definitivas: para onde vai a mensagem que falha sempre? Alguém é avisado?
- Ordem: o evento "cancelado" pode ser processado antes de "aprovado"?
- Indisponibilidade do terceiro: timeout, resposta inválida, resposta lenta; existe fallback? O usuário é
  informado?
- Observabilidade: dá para rastrear uma solicitação de ponta a ponta (ID de correlação nos logs)?

**SRD [Ilustrativo]:** o worker de OCR sugere valor e data a partir do comprovante. Com uma foto inclinada, o OCR
lê "1.250" como "1250,00" e preenche automaticamente sem destacar que foi sugestão — o colaborador não percebe.
Oráculos: Purpose, Users' expectations, Explainability.

## 9. Responsividade e componentes

ALTER FACE (`03`) em larguras diferentes (360, 768, 1024, 1440 px), com zoom de 200%, com teclado, com textos
longos e com conteúdo dinâmico. Verifique se ações críticas continuam alcançáveis (Achievable) e se elementos
fixos não cobrem conteúdo (Float, Layers).

## 10. Ferramentas de observação ao alcance do testador

[Interpretação] DevTools do navegador (Rede, Console, Aplicação/armazenamento, limitação de rede), clientes de
API (Postman, Insomnia, curl), logs e painéis de observabilidade (com permissão do time), ferramentas de proxy
para inspecionar tráfego no ambiente de teste. Observar mais do que a tela é o que separa "o botão não
funciona" de "a API devolve 403 porque o token não carrega a permissão".
