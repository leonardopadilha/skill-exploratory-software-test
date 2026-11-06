# 05 — Teste exploratório: explorar com estrutura

## Índice
1. O que é (e o que não é)
2. Charters (Hendrickson)
3. Session-Based Test Management e debrief PROOF (Jonathan e James Bach)
4. Como conduzir uma sessão: o raciocínio passo a passo
5. Anotações durante a sessão
6. Exploração sem documentação
7. Quando a exploração precisa ir além do charter
8. Limitações e combinação com roteiros

Exemplos no **SRD**; defeitos **[Ilustrativo]**.

---

## 1. O que é

**[Fonte] Cem Kaner** cunhou o termo *exploratory testing* (anos 1980) e o publicou em *Testing Computer
Software*. A definição refinada em conferências entre colegas até 2007 descreve um estilo de teste que enfatiza
a liberdade e a responsabilidade pessoal do testador para otimizar continuamente o valor do seu trabalho,
tratando desenho, execução, interpretação de resultados e aprendizado como atividades que se apoiam e correm
em paralelo durante todo o projeto. Uma forma curta muito citada é "aprendizado, desenho e execução de testes
simultâneos".

**[Fonte] Bolton** ressalta que exploração não é desestruturada: não é estruturada por procedimento, mas é
*cognitivamente* estruturada.

**[Interpretação] O que não é:** não é "testar sem plano", nem "ad hoc" no sentido de aleatório, nem uma fase
que acontece no fim. Todo testador explora em algum grau — até para escrever um roteiro.

**[Fonte] ISTQB CTFL v4.0** classifica o teste exploratório entre as técnicas baseadas em experiência e
menciona sessões com tempo definido e charters. Ver a divergência de vocabulário em `00-fundamentos.md`.

## 2. Charters

**Identificação [Fonte]:** **Elisabeth Hendrickson**, *Explore It!* (2013), propõe um modelo simples de charter
em três partes, inspirado na carta de Jefferson a Lewis e Clark: **onde explorar** (alvo), **com quais recursos**
e **que informação descobrir**:

> *Explore* <alvo> *com* <recursos> *para descobrir* <informação>

Hendrickson observa que um bom charter funciona como um *prompt*: sugere fontes de inspiração sem ditar ações
ou resultados.

**Quando usar:** para dar foco a uma sessão; para planejar a exploração de uma funcionalidade; para comunicar a
gestores o que será investigado.

**Como escrever bons charters [Interpretação]:**
- O alvo deve caber numa sessão (60–120 min). "Explorar o SRD" é grande demais.
- Os recursos são o que torna o charter interessante: dados, ferramentas, perfis, técnicas, condições.
- A informação deve estar ligada a um risco, não a "ver se funciona".
- Gere charters a partir do SFDIPOT, da análise de risco e das perguntas em aberto.

**Exemplos para o SRD [Ilustrativo]:**
- *Explore* a submissão de despesas em moeda estrangeira *com* datas de fim de semana, feriados e despesas de
  90 dias atrás *para descobrir* se a cotação usada é a correta e se o valor convertido é estável entre telas,
  e-mail e exportação.
- *Explore* a tela de aprovação *com* dois gestores em sessões simultâneas *para descobrir* se é possível
  aprovar e devolver a mesma solicitação, ou deixá-la em estado inconsistente.
- *Explore* o upload de comprovantes *com* arquivos nos limites de tamanho, extensões trocadas e rede 3G
  simulada *para descobrir* como o sistema se comporta em falhas parciais.
- *Explore* a tela "Nova solicitação" *com* ALTER FACE e janelas de 360 px a 1920 px *para descobrir* problemas
  de componentes que impeçam ou induzam a erro no envio.

## 3. Session-Based Test Management (SBTM)

**Identificação [Fonte]:** desenvolvido em 2000 por **Jonathan Bach e James Bach** para tornar o teste
exploratório gerenciável e mensurável. Unidade de trabalho: a **sessão** — período sem interrupção, com um
charter, que gera um relatório (session sheet) revisado em um **debrief**.

**[Fonte] Debrief PROOF (Jonathan Bach):** *Past* (o que aconteceu na sessão), *Results* (o que foi obtido),
*Obstacles* (o que atrapalhou), *Outlook* (o que ainda precisa ser feito) e *Feelings* (como o testador se sente
sobre isso).

**[Fonte] Métricas de tempo:** o SBTM separa o tempo da sessão em desenho e execução de testes, investigação e
relato de bugs, e preparação (setup) — o que ajuda a mostrar, por exemplo, que um ambiente instável consome a
sessão.

**[Interpretação] Uso leve no dia a dia:** mesmo sem a ferramenta original, um time pode adotar: charter +
time-box + notas + debrief de 10 minutos com PROOF. O debrief é onde o aprendizado vira próximos charters.

## 4. Como conduzir uma sessão: o raciocínio passo a passo [Interpretação]

1. **Aquecimento (5–10 min):** releia o charter; defina os oráculos que vai usar; prepare dados e perfis.
2. **Reconhecimento:** percorra o alvo sem tentar quebrar; construa um modelo mental (o que existe, que estados
   há, que mensagens aparecem). Anote perguntas.
3. **Hipóteses:** a partir do modelo, formule 3–5 hipóteses de falha ligadas ao risco do charter.
   Ex.: "a cotação usa a data do lançamento"; "a segunda aprovação sobrescreve a devolução".
4. **Experimentos:** desenhe o experimento que *refutaria* cada hipótese e execute. Varie uma coisa por vez
   quando estiver isolando; varie muitas quando estiver procurando.
5. **Observação ampla:** olhe além do resultado esperado — rede, console do navegador, e-mail, histórico,
   outras telas. Defeitos frequentemente aparecem "ao lado".
6. **Julgamento:** para o que estranhar, aplique FEW HICCUPPS (`02`).
7. **Decisão contínua:** a cada achado, decida: anotar e seguir, aprofundar agora ou criar um novo charter.
8. **Fechamento:** escreva notas de cobertura (o que foi e o que não foi visto) e prepare o debrief.

**Raciocínio do sênior (trecho de sessão) [Ilustrativo]:**
> "Lancei USD 100 com data de sábado. Valor convertido: R$ 512,30. Espera — não existe cotação de sábado. Que
> cotação ele pegou? Sexta? Segunda? A do dia do lançamento? (Hipótese.) Lanço a mesma despesa com data de
> sexta: R$ 512,30 também. Lanço com data de quarta: R$ 512,30 de novo. Agora a hipótese 'usa a cotação do dia
> do lançamento, não da despesa' ficou forte. Confirmo pela API: o campo `dataCotacao` vem com a data de hoje.
> Isso viola Claims (a regra R6) e tem efeito financeiro. Aprofundo agora, porque é risco alto e o charter é
> exatamente esse."

## 5. Anotações durante a sessão

[Interpretação] Notas boas permitem que outra pessoa entenda o que foi testado. Registre: dados usados,
ambiente/versão, o que observou (fatos), hipóteses (marcadas como tal), bugs, perguntas, ideias para outros
charters e áreas não cobertas. Capturas de tela e gravações ajudam; requisições da aba Rede do navegador são
evidência valiosa no moderno.

## 6. Exploração sem documentação

[Interpretação]
- Comece com um **tour de reconhecimento** para construir o modelo (SFDIPOT) e um glossário do domínio.
- Use os oráculos disponíveis: History, Comparable products, Claims por conversa, World (`02`).
- Converse com quem usa e com quem suporta: os chamados de suporte são um mapa de riscos.
- Registre o que aprendeu como documentação mínima (regras descobertas, estados, dúvidas). Isso tem valor
  em si — especialmente em sistemas legados com conhecimento concentrado.

## 7. Quando a exploração precisa ir além do charter

[Interpretação] Sinais de que vale sair do charter (ou criar outro):
- Um comportamento não tem explicação (Explainability) e afeta dinheiro, dados ou segurança.
- Você encontrou um padrão (o mesmo erro em dois lugares) — talvez seja sistêmico.
- O modelo mental do sistema se mostrou errado — as hipóteses do charter se apoiavam nele.
- Há sinais de efeito colateral fora da área (outra tela mostra dado diferente).

Regra prática: se o desvio for curto (< 10 min), siga e anote; se for longo, crie um charter novo e volte ao
atual — ou converse com quem prioriza.

## 8. Limitações e combinação com roteiros

[Interpretação]
- Exploração depende de habilidade; testadores iniciantes precisam de charters mais detalhados e de pareamento.
- Cobertura é mais difícil de demonstrar — notas, debriefs e matrizes SFDIPOT ajudam.
- Para regras críticas e estáveis (cálculos regulatórios), complemente com casos derivados por técnica
  formal e, quando possível, com checagens automatizadas.
- Roteiros e exploração não competem: roteiros nascem de exploração, e exploração descobre o que os roteiros
  não previram.
