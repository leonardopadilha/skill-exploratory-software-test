# 11 — Cenário integrado: SRD, Sistema de Reembolso de Despesas

**Todo este cenário é fictício [Ilustrativo].** Ele existe para mostrar como as técnicas se combinam num mesmo
fluxo. Nenhum resultado aqui vem de execução real.

## Índice
1. Descrição do sistema (regras, perfis, estados)
2. As duas versões: moderna e legada
3. Percurso completo do analista sênior
   3.1 Contexto e risco · 3.2 Modelagem · 3.3 Ideias e técnicas · 3.4 Charters e sessão
   3.5 Julgamento · 3.6 Isolamento · 3.7 Comunicação · 3.8 Decisão de parada
4. Diferenças de abordagem: moderno × legado
5. Índice de defeitos ilustrativos usados na skill

---

## 1. Descrição do sistema

**Perfis:** Colaborador (cria e acompanha), Gestor (aprova a equipe), Financeiro (aprova acima da alçada e paga),
Auditor (consulta), Administrador (configura).

**Regras de negócio:**
- **R1 Valores:** item de R$ 0,01 a R$ 5.000,00; total por solicitação até R$ 20.000,00; duas casas decimais.
- **R2 Datas:** a data da despesa não pode ser futura e deve estar "até 90 dias atrás" (texto da especificação,
  propositalmente ambíguo).
- **R3 Categorias:** Alimentação (limite diário de R$ 120), Transporte, Hospedagem (comprovante sempre
  obrigatório), Quilometragem (valor = km × R$ 1,20; sem comprovante), Outros (justificativa ≥ 20 caracteres).
- **R4 Comprovantes:** obrigatórios para itens acima de R$ 50,00 (exceto Quilometragem); PDF, JPG ou PNG; até
  5 MB cada; até 10 por solicitação.
- **R5 Alçada:** total ≤ R$ 1.000,00 → aprovação do gestor basta; acima → gestor e financeiro. Ninguém aprova a
  própria solicitação.
- **R6 Moeda estrangeira:** USD e EUR convertidos pela cotação do dia da despesa, obtida de serviço externo.
- **R7 Pagamento:** aprovação final gera título no ERP; pagamento na sexta-feira seguinte; corte às quintas,
  18h (horário de Brasília).
- **R8 Edição:** o colaborador edita apenas em RASCUNHO e DEVOLVIDO; pode cancelar até a aprovação do gestor.
- **R9 Listas:** "Minhas solicitações" e "Aprovações pendentes", com filtros (status, período, valor),
  ordenação, paginação de 20 por página e exportação CSV.
- **R10 Notificações** por e-mail a cada mudança de status.

**Estados:** RASCUNHO, ENVIADO, DEVOLVIDO, REPROVADO, APROVADO_GESTOR, APROVADO_FINANCEIRO, PAGO, CANCELADO
(diagrama em `04-tecnicas-formais.md`, seção 4).

## 2. As duas versões

| Aspecto | SRD moderno | SRD legado |
|---|---|---|
| Front-end | SPA React | ASP.NET Web Forms (o raciocínio vale para JSP/JSF/PHP) |
| Back-end | API REST + microsserviços (solicitações, anexos, câmbio, notificações, pagamentos) | Monolito com code-behind e procedures |
| Estado | Distribuído; consistência eventual entre serviços | Banco único; sessão no servidor (20 min) |
| Assíncrono | Fila de eventos; worker de OCR; WebSocket para status | Job noturno gera arquivo para o ERP |
| Autenticação | OIDC/JWT (access token de 15 min + refresh) | Formulário próprio + sessão |
| Upload | URL pré-assinada para storage | `<input type=file>` com limite em configuração do servidor |
| Documentação | Histórias de usuário + contrato OpenAPI | Manual de 2014; regras "na cabeça" de um analista |

## 3. Percurso completo do analista sênior

### 3.1 Contexto e risco
Pedido do time: "Liberar a nova versão do envio de solicitações (com OCR e novo fornecedor de câmbio) em
duas semanas."

Perguntas que o sênior faz: quem são os usuários mais afetados (comercial viaja muito → moeda estrangeira)?
O que é irreversível (pagamento)? O que mudou (OCR, câmbio, novo formulário)? Onde houve bugs (exportação,
arredondamento)? O que já tem checagem automatizada (cálculos unitários do câmbio)?

Riscos priorizados (condição → consequência → quem):
1. Valor convertido errado → pagamento errado → colaborador, financeiro.
2. OCR preenche valor/data errados sem destaque → aprovação de valor incorreto → gestor, financeiro.
3. Envio sem comprovante obrigatório (corrida com upload) → glosa na auditoria → financeiro, auditor.
4. Dupla aprovação/estado inconsistente → pagamento indevido ou travado → financeiro.
5. Componentes da nova tela atrapalham o envio no celular → retrabalho, envios duplicados → colaborador.

### 3.2 Modelagem
SFDIPOT (resumo em `01`), diagrama de estados (`04`), "siga o dado" para o valor do reembolso: digitado ou
sugerido pelo OCR → convertido → somado → exibido (lista, detalhe, aprovação, e-mail, CSV) → ERP.

### 3.3 Ideias e técnicas escolhidas (com justificativa)
- **PE + AVL** em valor, data, tamanho de arquivo, quantidade de anexos e horário de corte — regras explícitas
  com faixas.
- **Tabela de decisão** para comprovante/justificativa por categoria e valor, e para alçada — combinações.
- **Transição de estados** para o ciclo de vida, incluindo eventos do navegador e concorrência.
- **ALTER FACE** na tela "Nova solicitação" — componentes novos.
- **Interrupções** no upload e no envio — risco 3.
- **TuTTu/TaTTa** para as validações — garantir que a API também valida.
- **Comparação com o fornecedor antigo** (Comparable products/History) para o câmbio.

### 3.4 Charters e trecho de sessão
Charters em `05-exploratorio.md` (seção 2). Trecho de sessão sobre câmbio em `05` (seção 4).

### 3.5 Julgamento
Exemplo do limite diário de alimentação, com cinco oráculos, em `02-oraculos.md` (seção 4).

### 3.6 Isolamento
Divergência de total lista × detalhe, isolada até a consulta da listagem, em `07-isolar-investigar.md`
(seção 6).

### 3.7 Comunicação
Status MCOASTER e relato orientado a risco em `10-comunicar.md`.

### 3.8 Decisão de parada
Ao fim da primeira semana, o sênior avalia: riscos 1, 3 e 4 têm informação suficiente (dois bugs graves
relatados, corrigidos e retestados); risco 2 (OCR) segue com incerteza, porque os comprovantes de teste são
"limpos demais". Decisão: uma sessão extra com 40 fotos reais anonimizadas de comprovantes (amassados,
inclinados, de outras moedas). Fica fora: desempenho sob carga (volume baixo; declarado como lacuna).

## 4. Diferenças de abordagem: moderno × legado

| Tema | Moderno | Legado |
|---|---|---|
| Validação de valor | Testar UI **e** API; validação só no front é defeito | Testar JavaScript, code-behind e procedure; regras podem divergir entre camadas |
| Corrida no envio | Upload assíncrono vs. botão "Enviar" | Postback limpa o `<input type=file>` |
| Concorrência na aprovação | Duas sessões chamando a API; eventos fora de ordem | Duas abas compartilhando a variável de sessão |
| Integração com ERP | Evento de pagamento processado duas vezes (idempotência) | Job noturno: horário de corte, codificação, layout posicional |
| Oráculos principais | Claims (histórias + OpenAPI), Product (UI × API) | History (produção atual), Comparable products (planilhas), Claims por conversa |
| Observação | DevTools, logs com ID de correlação | Banco (se permitido), arquivos gerados, logs do servidor |
| Regressão | Checagens automatizadas de API + exploração | Caracterização por saídas (arquivos/relatórios) + exploração |

## 5. Índice de defeitos ilustrativos usados na skill

Todos fictícios, escolhidos por serem padrões reais e recorrentes em aplicações web:
- Cotação pela data do lançamento em vez da data da despesa (`05`, `06`, `10`).
- Limite diário de alimentação sem alerta (`02`).
- Total da lista inclui item excluído logicamente (`07`).
- Upload em andamento + envio sem comprovante (`08`).
- Token expirado + UI otimista mostrando "Aprovado" (`08`).
- Variável de sessão única aprovando a solicitação da outra aba (`09`).
- Aprovação no limite da meia-noite fora do arquivo do ERP (`09`).
- Codificação de acentos no arquivo do ERP (`01`, `09`).
- Componentes: calendário atrás do modal, toast cobrindo o botão, anexos inalcançáveis (`03`).
