# 12 — Kits por funcionalidade

Cada kit traz riscos típicos, técnicas e heurísticas que costumam render, ideias de teste e diferenças entre
moderno e legado. São **pontos de partida [Interpretação]**, não checklists completos: adapte ao contexto e
priorize por risco (`06`). Exemplos são **[Ilustrativo]**.

## Índice
1. Autenticação e recuperação de senha
2. Cadastro e edição de registros
3. Formulários com validações
4. Consultas e filtros
5. Paginação e ordenação
6. Upload e download de arquivos
7. Transações e operações financeiras
8. Controle de permissões
9. Gestão de sessão
10. Integrações entre sistemas
11. Processamento assíncrono
12. Datas, horários e fusos
13. Valores monetários e cálculos
14. Fluxos com múltiplos estados
15. Concorrência e inconsistência de dados

---

## 1. Autenticação e recuperação de senha
**Riscos:** acesso indevido; usuário legítimo bloqueado; vazamento de quais e-mails existem; link de
recuperação reutilizável.
**Técnicas:** transição de estados (conta ativa/bloqueada/senha expirada; token emitido/usado/expirado), AVL
(tentativas até bloqueio, validade do token, tamanho da senha), tabela de decisão (política de senha).
**Ideias:** token de recuperação usado duas vezes; vários pedidos seguidos — só o último vale?; link aberto após
trocar a senha por outro caminho; mensagem igual para e-mail existente e inexistente; e-mail com maiúsculas e
espaços; troca de senha encerra outras sessões?; limite de tentativas por usuário e por origem; retorno à
página protegida após login.
**Oráculos:** Standards (boas práticas de autenticação), Users' expectations, Purpose.
**Moderno × legado:** moderno — SSO/OIDC, refresh tokens, usuário desativado no provedor; legado — senha em
formulário próprio, sessão criada antes do login (fixação de sessão), recuperação por pergunta secreta.

## 2. Cadastro e edição de registros
**Riscos:** perda de dados na edição; duplicidade; efeitos colaterais em registros relacionados.
**Técnicas:** CRUD + Zero-Um-Muitos; transição de estados quando houver status; "siga o dado".
**Ideias:** editar um campo e verificar se outros campos não exibidos se mantêm; editar o mesmo registro em
duas abas; criar duplicado (mesmos dados, com espaços, com maiúsculas); excluir registro com dependentes;
histórico/auditoria registra quem e quando.
**Moderno × legado:** legado costuma regravar a linha inteira (perde campos de outra tela); moderno pode fazer
PATCH parcial e divergir entre serviços.

## 3. Formulários com validações
**Riscos:** dado inválido aceito; dado válido rejeitado; mensagem inútil; perda do que foi digitado.
**Técnicas:** PE + AVL; ataques de tipo de dado; Goldilocks; tabela de decisão para campos dependentes;
ALTER FACE para mensagens e componentes.
**Ideias:** colar valores com espaços; formatos regionais (vírgula/ponto); campos dependentes alternados;
enviar com Enter; validar via API (moderno) ou desabilitando JavaScript (legado); mensagens de erro chegam ao
leitor de tela?

## 4. Consultas e filtros
**Riscos:** resultado incompleto ou com itens indevidos (inclusive de outros usuários); filtro ignorado na
exportação.
**Técnicas:** PE em cada filtro; pairwise nas combinações; AVL em faixas (período, valor); Zero-Um-Muitos.
**Ideias:** período com início = fim; datas invertidas; filtro por valor com centavos; acentos e caixa no texto;
combinação sem resultados; o CSV exporta o filtrado ou tudo?; digitar rápido (respostas fora de ordem no
moderno); filtros preservados ao voltar da tela de detalhe.

## 5. Paginação e ordenação
**Riscos:** itens repetidos ou perdidos entre páginas; ordenação inconsistente.
**Técnicas:** AVL (19, 20, 21, 40, 41 itens com 20 por página); Começo-Meio-Fim; Zero-Um-Muitos.
**Ideias:** inserir/excluir um item enquanto navega entre páginas; ordenar por coluna com valores iguais (ordem
estável?); ordenar valores monetários como texto ("1.000" antes de "200"); última página após excluir o único
item dela; ordenação + filtro + paginação juntos; URL da página 5 aberta direto.

## 6. Upload e download de arquivos
**Riscos:** arquivo malicioso aceito; arquivo válido rejeitado; arquivo trocado entre usuários; perda em falha
parcial.
**Técnicas:** PE (tipo, tamanho); AVL (tamanho, quantidade); Interrupções; ALTER FACE (progresso, lista).
**Ideias:** extensão trocada (executável renomeado para .pdf); arquivo de 0 byte; nome com acentos, espaços e
200 caracteres; dois arquivos com o mesmo nome; queda de rede no meio; download por URL direta de arquivo de
outro usuário (autorização); arquivo removido continua acessível por link?
**Moderno × legado:** moderno — URL pré-assinada (expira? é compartilhável?); legado — limite de requisição do
servidor diferente do limite exibido; arquivo salvo em pasta do servidor com nome previsível.

## 7. Transações e operações financeiras
**Riscos:** execução duplicada; execução parcial; valor errado; irreversibilidade.
**Técnicas:** transição de estados; error guessing (duplo clique, F5 após POST, voltar e reenviar);
tabela de decisão para regras de cálculo.
**Ideias:** repetir a mesma operação (idempotência); falha no meio (o que foi gravado?); confirmação de sucesso
só após o servidor confirmar; conciliação com o sistema de destino.

## 8. Controle de permissões
**Riscos:** acesso a dados ou ações de outros perfis; segregação de funções violada.
**Técnicas:** tabela de decisão perfil × ação × estado × propriedade do recurso.
**Ideias:** ver `08-web-moderna.md` seção 6. Testar também listas, exportações, mensagens de erro e e-mails —
canais por onde dados vazam sem que a "tela" principal mostre.
**Moderno × legado:** moderno — verificar a API diretamente; legado — verificar acesso direto por URL a páginas
"escondidas" do menu e parâmetros na query string.

## 9. Gestão de sessão
**Riscos:** perda de trabalho; sessão de um usuário vista por outro; sessão que nunca expira.
**Ideias:** expiração com formulário preenchido; logout em uma aba com outra aberta; troca de usuário no mesmo
navegador; botão voltar após logout mostra dados?; duas abas na mesma sessão (legado: variáveis de sessão
compartilhadas — ver `09` seção 4).

## 10. Integrações entre sistemas
**Riscos:** dado transformado errado na fronteira; indisponibilidade não tratada; duplicidade ou perda.
**Técnicas:** "siga o dado"; AVL em tamanhos de campo do destino; error guessing (codificação, separadores).
**Ideias:** terceiro lento, fora do ar, com resposta inválida; reprocessamento; layout do arquivo/contrato
com o maior e o menor valor possível; conciliação origem × destino.

## 11. Processamento assíncrono
**Riscos:** o usuário acha que terminou e não terminou; mensagem processada duas vezes ou nunca.
**Ideias:** ver `08-web-moderna.md` seção 8. Pergunte sempre: "como o usuário sabe que acabou? E que falhou?"

## 12. Datas, horários e fusos
**Riscos:** limite de dia deslocado pelo fuso; horário de verão; fim de mês; anos bissextos; feriados.
**Técnicas:** AVL em data e hora; tabela de decisão (dia útil × feriado × fim de semana).
**Ideias:** ações às 23h59 e 00h00 (horário de Brasília × UTC do servidor); 29/02; último dia do mês; datas
digitadas em formatos diferentes; prazos "em dias" — corridos ou úteis, inclusivos ou não?
**Oráculo World:** o calendário real, feriados oficiais, regras bancárias.

## 13. Valores monetários e cálculos
**Riscos:** arredondamento, truncamento, ponto flutuante, soma de valores arredondados × arredondamento da soma.
**Técnicas:** PE + AVL em centavos; "siga o dado"; tabela de decisão das regras de cálculo.
**Ideias:** 0,01; 0,005; valores que somados geram dízima (3 × 33,33); conversão de moeda e volta; totais em
telas diferentes; separador decimal em exportações; valores negativos via API.
**Oráculos:** World (regras de arredondamento usadas pelo negócio), Claims, Product (mesmo valor em telas
diferentes), Comparable products (planilha do financeiro).

## 14. Fluxos com múltiplos estados
**Técnica principal:** transição de estados (`04` seção 4) com eventos de tempo e de navegador.
**Ideias:** transições inválidas por URL/API; cancelamento durante transição; notificações e histórico para
cada transição; estado exibido igual em todas as telas.

## 15. Concorrência e inconsistência de dados
**Riscos:** última gravação sobrescreve a outra sem aviso; dupla aprovação; estado impossível.
**Como provocar:** duas sessões/navegadores (perfis diferentes ou iguais) agindo sobre o mesmo registro;
duplicar a requisição (reenvio pelo DevTools ou cliente de API no ambiente de teste); atrasar uma das
requisições com limitação de rede.
**Ideias no SRD:** gestor aprova enquanto colaborador cancela; dois gestores aprovam/devolvem ao mesmo tempo;
financeiro paga enquanto gestor devolve; job de pagamento roda duas vezes.
**O que observar:** estado final, histórico, notificações, integração — e se o segundo usuário é avisado de
que o registro mudou.
**Moderno × legado:** moderno — controle otimista (versão/ETag) existe e é respeitado na API?; legado — locks no
banco, transações longas, variáveis de sessão compartilhadas.
