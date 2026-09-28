# A Factory

> **TL;DR (EN):** A fleet of AI agents that operates a real e-commerce business end to end — not a coding assistant, an engine. 24 days old, 545 merged PRs, ~R$1.590/month of infrastructure, running an actual store (MyBagCenter) toward a real revenue target. This repo is the doctrine, the architecture, and the numbers behind it.

---

## Isso não é "usar IA pra programar"

A maior parte do que se chama de "agente de IA" hoje é um gerador de texto com acesso a um terminal. Ele propõe, alguém revisa, alguém decide, alguém aplica. Quando o humano sai da frente, o sistema para.

A Factory é outra coisa: um **sistema de controle** — objetivo, ação, medição, correção, repete — onde sistema, IA e humano trabalham em três papéis fixos, e nenhum dos três é o gargalo:

- **o sistema** guarda contexto, estado, regra e histórico;
- **a IA** explora, propõe, implementa e compara;
- **o humano** define intenção, resolve ambiguidade e assume responsabilidade.

O objetivo declarado, por escrito, desde o primeiro dia:

> *Transformar trabalho subjetivo, ambíguo e dependente de julgamento humano em algo explícito, mensurável, testável, automatizado — e barato.*

Isso não é uma frase de efeito. É um critério de engenharia que a Factory aplica sobre si mesma tanto quanto aplica sobre o trabalho de fora.

## A prova: uma loja de verdade, não uma demo

Regra da casa: *"não se remenda pano velho com tecido novo"* e *"descoberta sem entrega é entropia adiada"* — o que importa não é o sistema existir, é o sistema **entregar**.

A prova viva é a **MyBagCenter**, uma operação de e-commerce real (múltiplas contas, múltiplos marketplaces) que a Factory constrói e opera:

| O que é medido | Número |
|---|---|
| Faturamento de base (30 dias, todas as contas) | R$ 100.000 |
| Meta declarada para a próxima campanha | R$ 1.000.000 |
| Repositório do produto: idade | 24 dias |
| Pull requests mergeados nesse período | 545 |
| Recorde de PRs mergeados numa única madrugada | 13, com o pipeline "dia e noite" |
| Recorde de PRs recebidos pela branch principal numa noite | 31 |
| Custo mensal de toda a infraestrutura (CI + o "cérebro" de IA local) | ≈ R$ 1.590 |

Esses números não são projeção. São contagem direta do histórico de commits e da fatura de nuvem no momento em que este texto foi escrito.

O ponto não é o volume — é que **cada PR passou pelo mesmo funil de controle de qualidade** que qualquer engenheiro sênior exigiria: tipagem, testes automatizados, revisão adversarial de código, CI que trava em vermelho de verdade, e um humano decidindo o que entra em produção. Volume sem esse funil é ruído. Com ele, é escala.

## Como funciona: uma engine, não um conversador

A referência de design não é um chatbot. É um motor de xadrez.

Stockfish opera perto de 3600 de Elo. O recorde humano é 2882. A diferença não é vantagem — é outra categoria. E a parte "criativa" da engine, o gerador de lances, é relativamente burra. A força vem de duas coisas que um generalista não tem: **ela nunca avalia errado, e nunca cansa.**

Uma engine tem três peças:

```
gerador   → propõe candidatos (pode ser um modelo de linguagem)
avaliador → pontua cada candidato — precisa ser BARATO e EXATO
busca     → propõe, pontua, guarda o melhor, repete
```

A peça que decide é o avaliador. Um LLM sozinho é só o gerador — é por isso que "colocar um agente pra trabalhar" sem essa estrutura ao redor não escala: sem avaliador, o modelo está adivinhando, e adivinhar bem não escala, enquanto buscar escala com CPU barata.

**Onde cabe uma engine:** em todo domínio onde verificar é mais barato que gerar. Rodar o teste é mais barato que reler o código. Medir contraste é mais barato que julgar "essa cor parece boa". Abrir o link e ver se ele renderiza é mais barato que perguntar "será que publicou". Essa pergunta — *qual seria o avaliador barato aqui?* — é a primeira coisa que a Factory faz diante de qualquer tarefa dita "subjetiva". Quase sempre existe uma resposta; ela só não tinha sido instrumentada ainda.

### O ciclo, na prática

```
demanda real
   ↓
regra + contexto (o sistema)
   ↓
a IA executa, numa cópia isolada do código
   ↓
teste automatizado + revisão adversarial (o avaliador)
   ↓
evidência suficiente? → não → volta pra regra + contexto
                       → sim → merge, com humano no controle final
   ↓
produto em operação
   ↓
o resultado vira estado, e o estado vira contexto pro próximo ciclo
```

Isso roda de dia e de noite. Uma fila recebe trabalho; um executor pega o próximo item, cria uma cópia isolada do repositório, propõe a mudança, valida (checagem de tipos, testes, CI); um humano funciona como o merge final — decide o que realmente vai para produção. Nenhuma etapa pula a validação para "ir mais rápido".

## A doutrina: dez regras, e cada uma custou caro

A parte que não aparece em nenhum framework de agentes: um conjunto de princípios operacionais, escritos depois de erros reais, que existem porque já custaram tempo, dinheiro ou confiança quando foram violados.

Algumas amostras:

> **"Pediram pão? Faça a padaria."** — mas construir a capacidade que não entrega o pedido que a originou é decoração, não engenharia. O teste é sempre: *o pedido foi atendido, hoje?*

> **"Reverter é a condição da limpeza."** — nada é destruído sem que exista, primeiro, a prova de que dá para reconstruir. Só então a limpeza (apagar o que sobrou) é segura. Sem essa ordem, "prudência" vira acúmulo, e acúmulo vira entulho que ninguém sabe mais o que é.

> **"O erro é a medida da entropia local, não a dívida de alguém."** — onde um agente ou um humano erra, o sistema deixou uma decisão em aberto. A pergunta correta nunca é "quem errou", é "o que aqui tornou esse erro provável" — porque só essa pergunta produz uma regra nova.

> **"Enviado não é entregue."** `exit 0` não prova nada. Todo resultado se prova pelo estado depois, nunca pela ausência de erro.

Nenhuma dessas regras é decoração de README. Cada uma delas existe porque, sem ela, algo real quebrou uma vez — e a Factory aplica sobre si mesma o mesmo processo que aplica sobre qualquer outro problema: **erro vira regra determinística, sempre.**

## O que isso custa, de verdade

Toda a infraestrutura que roda essa operação — o pipeline de integração contínua, a frota de execução, o modelo de linguagem que faz o trabalho pesado repetitivo — custa hoje algo da ordem de **R$ 1.590 por mês**. Parte dela liga sozinha quando há trabalho na fila e desliga sozinha quando fica ociosa; parte roda um modelo aberto, auto-hospedado, sem depender de API paga para o volume do dia a dia.

545 PRs em 24 dias, por menos de R$ 2.000/mês de infraestrutura, é a única linha deste documento que realmente precisa de contexto pra ser acreditada — e é por isso que ela é verificável: está no histórico de commits, não numa promessa.

## Pra quem isso importa

- **Você investe ou é sócio de operação:** o produto não é uma tese, é uma loja com faturamento real sendo escalada por um sistema que já entrega ~23 mudanças por dia em produção, com controle de qualidade real.
- **Você constrói sistemas de IA/engenharia:** a parte interessante não é "GPT escreve código" — é a arquitetura de controle ao redor (avaliador barato, malha fechada, memória coletiva, reversibilidade) que faz um LLM genérico virar uma engine confiável.
- **Você tem um negócio que depende de trabalho repetitivo e caro:** esse é exatamente o tipo de problema que a Factory foi desenhada pra resolver — e a MyBagCenter é a prova de que funciona fora do laboratório.
- **Você só quer acompanhar:** o resto deste repositório continua sendo escrito, pelo mesmo sistema, sobre si mesmo.

---

*Este documento descreve arquitetura e resultados; não expõe código proprietário, credenciais ou infraestrutura operacional do produto real (MyBagCenter). Contato: [a definir].*
