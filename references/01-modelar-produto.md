# 01 — Modelar o produto: "o que existe para eu olhar?"

## Índice
1. Por que modelar antes de testar
2. SFDIPOT (Elementos de Produto do HTSM)
3. Critérios de qualidade do HTSM (o "como pode ser bom")
4. CRUD
5. Siga o dado
6. Modelos de estado e de fluxo
7. Como combinar os modelos

Os exemplos usam o **SRD — Sistema de Reembolso de Despesas**, descrito em `11-cenario-integrado.md`.
Todos os defeitos citados são **[Ilustrativo]**.

---

## 1. Por que modelar antes de testar

[Interpretação] Software é invisível e complexo; sem um modelo, o testador testa o que lhe ocorre — em geral a
tela e o caminho feliz. Modelar é tornar visível o que pode ser testado para que a *escolha* do que testar seja
consciente. O modelo nunca é completo; o objetivo é reduzir o risco de ignorar uma área inteira.

## 2. SFDIPOT

**Identificação [Fonte]:** guia de *Elementos de Produto* do **Heuristic Test Strategy Model (HTSM)** de
**James Bach**. Originalmente SFDPO ("San Francisco DePOt"). **Michael Bolton** convenceu Bach a incluir
**Time**; depois, discutindo os vários tipos de interface, os dois acrescentaram **Interfaces**, e o mnemônico
virou SFDIPOT. Versões antigas de materiais (inclusive alguns resumos em português) ainda mostram SFDPOT.
A versão do HTSM consultada é a 6.0 (2024).

**Fundamentação:** lista de palavras-guia para lembrar as dimensões do produto. Ajuda a perceber áreas
"invisíveis" — jobs, integrações, dados persistidos, comportamento ao longo do tempo.

| Letra | Elemento | Pergunta-guia [Interpretação] |
|---|---|---|
| S | Structure | Do que o produto é feito? (código, componentes, serviços, banco, arquivos, configuração) |
| F | Function | O que ele faz? (regras, cálculos, validações, tratamento de erro, segurança) |
| D | Data | O que ele consome, transforma, armazena e produz? (entradas, saídas, estados, volume, ciclo de vida) |
| I | Interfaces | Por onde ele é acessado ou se expressa? (UI, API, arquivos, e-mail, integrações, impressão) |
| P | Platform | Do que ele depende fora de si? (navegador, SO, servidor, rede, serviços de terceiros) |
| O | Operations | Como é usado de verdade? (perfis, cenários, frequência, uso indevido, instalação, suporte) |
| T | Time | Como o tempo o afeta? (concorrência, timeouts, datas, fusos, sequência, envelhecimento dos dados) |

**Quando usar:** ao conhecer um produto novo; ao planejar teste de uma funcionalidade; ao revisar se casos de
teste ignoram dimensões; ao relatar cobertura ("cobri F e D, não cobri P e T").

**Como aplicar:**
1. Para cada letra, liste rapidamente tudo o que lhe ocorre sobre o item sob teste (5–10 min por letra no
   início; menos com prática).
2. Marque o que você não sabe — isso vira pergunta para o time ou tarefa de investigação.
3. Para cada item, pergunte "o que poderia dar errado aqui e quem se importaria?" — isso vira ideia de teste.
4. Priorize por risco (ver `06-risco-e-parada.md`).
5. Não se preocupe com sobreposições: os próprios autores tratam o modelo como uma abstração imperfeita.

**Exemplo prático (SRD — submissão de solicitação) [Ilustrativo]:**
- **S:** formulário React; serviço de solicitações; serviço de anexos; storage de arquivos; serviço de câmbio;
  fila de eventos; worker de OCR. No legado: página Web Forms, code-behind, procedures no banco.
- **F:** validar valor e data; exigir comprovante acima de R$ 50; calcular quilometragem (km × R$ 1,20);
  converter moeda; aplicar limite diário de alimentação; definir alçada de aprovação.
- **D:** valores com centavos; datas; moedas; arquivos (tipo, tamanho, nome com acentos); rascunhos salvos;
  histórico de status; dados que viajam ao ERP.
- **I:** tela; API REST; WebSocket de status; e-mail de notificação; exportação CSV; arquivo/integração com ERP.
- **P:** navegadores; storage; serviço externo de cotação; servidor de e-mail; ERP.
- **O:** colaborador lançando no celular no táxi; gestor aprovando 40 itens na sexta à tarde; financeiro
  fechando o mês; auditor consultando ano anterior.
- **T:** prazo de 90 dias; corte de pagamento quinta 18h; dois aprovadores simultâneos; sessão expirando no
  meio do preenchimento; cotação do dia da despesa vs. do dia do lançamento.

**Raciocínio do sênior:** "Quando listo Interfaces, percebo que a solicitação também *sai* do sistema — vai ao
ERP. Ninguém falou disso na refinação. Se o valor com centavos sai errado no arquivo do ERP, o colaborador
recebe o valor errado, e a tela vai mostrar tudo certo. Isso sobe na minha lista de risco."

**Problemas que pode revelar [Ilustrativo]:** campo "justificativa" com acentos corrompido no arquivo do ERP
(D+I); cotação buscada na data de lançamento, não na data da despesa (F+T); rascunho perdido quando a sessão
expira (T+D); notificação enviada para o gestor anterior após troca de equipe (O+D).

**Limitações:** não diz *como* testar cada item nem *se* um resultado é problema — combine com técnicas
(`04`), heurísticas de ideias (`03`) e oráculos (`02`). Pode gerar listas enormes; sem priorização por risco,
vira paralisia.

**Moderno × legado:** no moderno, Structure e Interfaces são distribuídos (vários serviços, contratos de API,
eventos) — investigue a arquitetura com o time de desenvolvimento. No legado, Structure costuma estar no banco
(procedures, triggers) e em jobs agendados — pergunte ao DBA e procure tarefas noturnas; Interfaces podem
incluir arquivos posicionais, FTP e relatórios impressos.

## 3. Critérios de qualidade do HTSM

**Identificação [Fonte]:** o HTSM de James Bach também traz *Quality Criteria Categories* — categorias de
"como o produto pode ser bom" (capacidade, confiabilidade, usabilidade, carisma, segurança, escalabilidade,
compatibilidade, desempenho, instalabilidade e critérios de desenvolvimento como suportabilidade, testabilidade,
manutenibilidade e portabilidade). A norma **ISO/IEC 25010** oferece uma taxonomia equivalente, mais formal.

**[Interpretação] Uso prático:** SFDIPOT diz *onde* olhar; os critérios de qualidade dizem *que tipo de
problema* procurar em cada lugar. Cruzar os dois em uma matriz curta (linhas = elementos de maior risco,
colunas = critérios relevantes) evita testar só "se funciona".

Exemplo [Ilustrativo]: Interface "upload de comprovante" × critérios → capacidade (aceita PDF/JPG/PNG?),
confiabilidade (o que ocorre se a rede cair no meio?), usabilidade (o usuário entende o erro de tamanho?),
segurança (aceita um executável renomeado para .pdf?), desempenho (10 arquivos de 5 MB no 4G?).

## 4. CRUD

**Identificação:** heurística amplamente difundida na comunidade (Create, Read, Update, Delete); aparece no
*Test Heuristics Cheat Sheet* de Hendrickson, Lyndsay e Emery associada a contagem, posição e seleção.
[Interpretação] Não tem um "autor" único — é um padrão de modelagem de dados herdado do desenvolvimento.

**Como aplicar:** para cada entidade, percorra o ciclo de vida completo e combine com Zero-Um-Muitos:
crie, leia em todos os lugares onde aparece, atualize cada campo, apague — e observe os *efeitos colaterais*
nas outras entidades e telas.

**Exemplo [Ilustrativo]:** criar solicitação com 0, 1 e 15 itens; editar um item já com comprovante;
excluir o último item de uma solicitação devolvida; excluir um comprovante e verificar se o item ainda
"passa" na regra de R$ 50; ler a solicitação na lista, no detalhe, na exportação CSV e no e-mail.

**Problemas típicos [Ilustrativo]:** total não recalculado após excluir item; exclusão lógica que continua
aparecendo na exportação; "Update" que perde campos não exibidos na tela (comum em formulários legados que
reescrevem a linha inteira).

## 5. Siga o dado

**Identificação:** heurística citada em listas comunitárias ("Follow the data"). [Interpretação] Não encontrei
autoria única confiável; trate como prática difundida.

**Como aplicar:** escolha um dado de alto valor (ex.: o *valor do reembolso*) e rastreie-o de ponta a ponta:
entrada → validação → armazenamento → cálculos → exibição → exportação → integração → relatório. Em cada
transição, pergunte se formato, precisão, codificação ou unidade podem mudar.

**Exemplo [Ilustrativo]:** R$ 1.234,565 digitado → front aceita 3 casas? → API recebe string ou número? →
banco grava DECIMAL(10,2) ou FLOAT? → arredonda ou trunca? → total da solicitação soma valores já arredondados
ou brutos? → arquivo do ERP usa ponto ou vírgula? → e-mail formata com separador correto?

**Por que é poderoso:** muitos defeitos graves moram nas *fronteiras* entre componentes, não dentro deles.

## 6. Modelos de estado e de fluxo

[Interpretação] Sempre que uma entidade tem status, desenhe o diagrama de estados — mesmo à mão. Ele revela
transições não especificadas (o que acontece se o colaborador cancelar *enquanto* o gestor aprova?) e alimenta
diretamente a técnica de transição de estados (`04-tecnicas-formais.md`).

## 7. Como combinar os modelos

[Interpretação] Sequência típica de um sênior em uma funcionalidade nova:
1. SFDIPOT rápido (20–30 min) para ter o mapa.
2. Diagrama de estados para as entidades com status.
3. "Siga o dado" para os 1–2 dados de maior valor.
4. Matriz curta elemento × critério de qualidade para as áreas de maior risco.
5. Converter em charters (`05-exploratorio.md`) e em casos derivados por técnica (`04`).
