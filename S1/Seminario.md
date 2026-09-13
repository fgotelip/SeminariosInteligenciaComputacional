# Perguntas a Serem Respondidas
- Qual é o problema e seu objetivo? Exemplo de aplicação.
- Como a solução inicial é construída?
Como funciona a busca local?
- Qual é a vizinhança/movimentos utilizados?
Como as soluções são avaliadas?
- Quais são os principais resultados apresentados no trabalho?

# 1. Introdução
- Problema já definido: `timetabling` $\rightarrow$ organizar `x` eventos para `y` recursos garantindo as restrições `z`
- Objetivo principal: facilitar o trabalho da coordenação utilizando uma abordagem metaheurística para a otimização da alocação de disciplinas para professores nos cursos da Unicamp: Tecnologia em Análise e Desenvolvimento de Sistemas (TADS) e Bacharelado em Sistemas de Informação (BSI) 
- Fatores envolvidos: interesses individuais, distribuição de carga horária equilibrada entre os professores
- Complexidade: muitas disciplinas e muitos docentes $\rightarrow$ inúmeras combinações

# 3. Metodologia
## 3.1. Modelagem do problema
### Problema de Otimização Combinatória

Requisitos Principais: 
- garantir carga horária balanceada
- considerar aptidão dos professores para ministrar determinada disciplina

### Planilha anônima para a construção:
| ID | Disciplinas | Créditos em Sala | Oferecimentos | Ana | Beatriz | Carlos | Daniela |
| :--- | :--- | :---: | :--- | :---: | :---: | :---: | :---: |
| TT001 | Administração I | 4 | Serviço | -1 | -1 | -1 | -1 |
| TT002 | Administração II | 4 | Serviço | -1 | -1 | -1 | -1 |
| TT003 | Algoritmos I | 6 | BSI,TADS | -1 | -1 | -1 | 0 |
| TT004 | Algoritmos II | 6 | BSI, TADS | -1 | -1 | -1 | 3 |
| TT005 | Análise de Sistemas | 4 | Serviço | -1 | -1 | -1 | -1 |

O programa foi rodado com a planilha original.


## 3.2. Implementação
### Oferecimentos
Uma disciplina precisa ser oferecida mais de uma vez (uma vez por curso). Criação de par composto 
(Disciplina,número da oferta) $\rightarrow$ disciplinas podem aparecer mais de uma vez, exemplo:

(TT010, 0) -> Representa a 1ª oferta (ex: turma de BSI).

(TT010, 1) -> Representa a 2ª oferta (ex: turma de TADS).

(TT010, 2) -> Representa a 3ª oferta (ex: turma de Serviço).

## 3.2.1. Criaçao da Solução Inicial
Dada uma certa disciplina e sua quantidade de oferecimentos, para cada oferecimento será escolhido aleatoriamente um professor cuja Aptidão é > -1.

### Algoritmo 1: Geração de uma solução inicial aleatória
---
Entrada: Lista de professores P, lista de oferecimentos D e tabela de aptidões

Saída: Uma solução
```py
para cada oferecimento ∈ D faça
    aptos ← ∅
    para cada professor ∈ P faça
        se aptidao(professor, disciplina) > -1 então
            aptos ← aptos ∪ {professor}
        fim
    fim
    solucao[oferecimento] ← escolherAleatorio(aptos)
fim
```
## 3.2.2. Definição das Funções Objetivo
### Tabela de Aptidão:
| Aptidão | Custo | Significado / Interesse do Professor |
| :---: | :---: | :--- |
| 5 | 1.0 | Aptidão máxima (Muito interesse) |
| 4 | 1.3 | Alta aptidão |
| 3 | 1.5 | Aptidão média |
| 2 | 2.0 | Baixa aptidão |
| 1 | 2.5 | Menor aptidão (Mas ainda tem interesse) |
| 0 | 5.0 | Não se sente totalmente capacitado, mas aceita ministrar se necessário |
| -1 | 10.0 | Desinteresse total / Nenhuma aptidão (Gera a maior penalidade no sistema) |

*Nota*: O problema é de minimização, logo quanto menor o custo melhor a solução (os custos foram definidos pelo coordenador)


### Fórmula Geral: Custo de Atribuição

$$f_{atrib}(S) = \sum_{i \in P} \sum_{j \in D} x_{ij} c_{ij}$$

Onde:
* $P$: Conjunto de todos os professores
* $D$: Conjunto de todas as disciplinas (oferecimentos)
* $x_{ij}$: Variável de decisão binária (recebe 1 se o professor $i$ for atribuído à disciplina $j$, e 0 caso contrário)
* $c_{ij}$: Custo da atribuição do professor $i$ para a disciplina $j$ (baseado na tabela de aptidões)


#### Exemplo Prático: Cálculo do Custo de Atribuição

Dada a tabela de custos ($c_{ij}$) para os professores Natália (N) e Matheus (M) nas disciplinas A e B:

| Professor ($i$) | Custo Disc. A ($c_{i,a}$) | Custo Disc. B ($c_{i,b}$) |
| :--- | :---: | :---: |
| Natália (N) | 1.3 | 2.5 |
| Matheus (M) | 5.0 | 1.0 |

A equação da função objetivo $f_{atrib}(S)$ expandida para este cenário fica:

$$f_{atrib}(S) = (x_{N,a} \cdot 1.3) + (x_{N,b} \cdot 2.5) + (x_{M,a} \cdot 5.0) + (x_{M,b} \cdot 1.0)$$

*Sendo $x_{ij}$ = 1 se o professor for alocado à disciplina, e 0 caso contrário.*

Supondo uma solução onde:
* Natália ministra a Disciplina A ($x_{N,a} = 1$ e $x_{N,b} = 0$)
* Matheus ministra a Disciplina B ($x_{M,a} = 0$ e $x_{M,b} = 1$)

O custo total desta atribuição será:
$$f_{atrib}(S) = (1 \cdot 1.3) + (0 \cdot 2.5) + (0 \cdot 5.0) + (1 \cdot 1.0)$$
$$f_{atrib}(S) = 1.3 + 1.0 = \mathbf{2.3}$$

### Fórmula Geral: Custo de Distribuição (Balanceamento de Carga)

$$f_{dist}(S) = \sum_{i \in P} \rho(i,S)^2$$

Onde:
* $\rho(i,S)$: É o número total de créditos atribuídos a um dado professor $i$ na solução $S$
* $\sum_{i \in P}$: Somatório iterando sobre todos os professores do conjunto $P$

#### O Efeito da Penalização ao Quadrado
A função eleva o somatório das cargas didáticas ao quadrado justamente para penalizar cargas altas e forçar o algoritmo a buscar o balanceamento 

Exemplo prático (Distribuindo 6 créditos entre 3 professores):

* Cenário 1 (Perfeitamente Balanceado - Carga {2, 2, 2}):
  * Custo: $f_{dist} = 2^2 + 2^2 + 2^2 = 12$

* Cenário 2 (Levemente Desbalanceado - Carga {1, 2, 3}):
  * Custo: $f_{dist} = 1^2 + 2^2 + 3^2 = 14$ (Indica uma solução de qualidade mais baixa)

* Cenário 3 (Totalmente Desbalanceado - Carga {1, 1, 4}):
  * Custo: $f_{dist} = 1^2 + 1^2 + 4^2 = 18$ (Sendo a solução com pior custo e maior penalidade)

Como o algoritmo de Busca Local procura sempre o menor valor (minimização), ele naturalmente rejeitará o Cenário 3 e dará preferência ao Cenário 1.

### Função Objetivo Final: O Custo Combinado

$$f(S) = \frac{0.5}{4047} f_{dist}(S) + \frac{0.5}{152} f_{atrib}(S)$$

**Entendendo a Equação:**
A função de custo final $f(S)$ une os dois critérios em uma única métrica (minimização biobjetivo) para o algoritmo avaliar a qualidade global da solução.

* **Pesos Iguais (0.5):** Os autores definiram que tanto o balanceamento da carga horária quanto o respeito à aptidão dos professores têm exatamente a mesma importância (50% para cada).
* **Fatores de Normalização (4047 e 152):** Como os valores brutos de $f_{dist}$ (que ficam na casa dos milhares) e $f_{atrib}$ (que ficam na casa das centenas) operam em escalas matemáticas muito diferentes, foi necessário normalizá-los para que um critério não "engolisse" o outro na soma.
    * O valor **4047** é o custo aproximado de uma solução quando o algoritmo foi rodado para otimizar *exclusivamente* a distribuição de carga ($f_{dist}$).
    * O valor **152** é o custo aproximado de uma solução quando otimizada *exclusivamente* para a aptidão ($f_{atrib}$).
  
## 3.2.3. Classe Otimizador (Busca Local)

A classe `Otimizador` implementa a busca local propriamente dita.

- **Solução atual vs. melhor solução:** o algoritmo mantém duas instâncias de solução: `sol` (a que está sendo perturbada a cada iteração) e `best` (a melhor encontrada até o momento).
- **Vizinhança / Movimentos:** a cada iteração, `sol` é modificada alternando dois tipos de movimento:
  - **`swap`**: escolhe aleatoriamente **um** oferecimento e troca o professor atribuído por outro professor aleatório (dentre os aptos).
  - **`swap2`**: executa o processo de troca **duas vezes** em sequência (equivalente a aplicar `swap` duas vezes na mesma solução).

  Ou seja, a vizinhança $N(s)$ de uma solução é definida pelas soluções alcançáveis trocando a atribuição de um ou dois oferecimentos por vez.

- **Critério de aceitação:** após o movimento, o custo combinado $f(S)$ (Equação 3) da nova solução é calculado. Se for melhor que `best`, `best` é substituída por `sol`. Caso contrário, `sol` é revertida para `best`, ou seja, é uma busca local gulosa (hill climbing), sem aceitação de soluções piores.
- **Critério de parada:** limite de até 1 milhão de iterações, ou interrupção manual (os próprios autores relatam essa limitação e sugerem, como melhoria futura, parar automaticamente com base na taxa de melhoria do custo).

*Pontos para relacionar com a disciplina:* dá pra comentar que essa é uma busca local de vizinhança simples (troca única), sem diversificação (não é GRASP, não é SA, não tem perturbação para escapar de ótimos locais) — o que é uma limitação que vocês podem discutir na apresentação.

## 4. Resultados

Principais resultados apresentados:

- **Balanceamento de carga:** usando a planilha anual (dois semestres), a carga horária por professor ficou entre 12 e 16 créditos — uma diferença de apenas 4 créditos entre o professor com menor e maior carga, indicando boa distribuição.
- **Aptidão:** a maior parte das disciplinas foi alocada a professores com aptidão máxima (nível 5) ou alta (nível 4); nenhuma disciplina foi alocada a um professor com aptidão -1 (desinteresse total).
- **Convergência:** os gráficos de custo-atribuição e custo-distribuição ao longo das iterações mostram queda acentuada nas primeiras iterações, estabilizando depois (comportamento típico de busca local).
- **Comparação com a alocação real (1º semestre de 2025):** os autores compararam a distribuição real de carga horária e de aptidão (feita manualmente pela coordenação) com a que o algoritmo geraria para o mesmo conjunto de disciplinas. O resultado do algoritmo mostrou distribuição de carga mais equilibrada e maior concentração em aptidões altas do que a alocação manual.
- **Validação com o coordenador:** o programa foi apresentado ao coordenador responsável, que validou a abordagem como promissora, mas apontou limitações do modelo atual:
  - repetir a mesma disciplina para um professor deveria custar menos do que ministrar duas disciplinas diferentes;
  - cargas horárias iguais deveriam ter pesos diferentes dependendo de quantas disciplinas as compõem (ex.: 8h em 2 disciplinas ≠ 8h em 3 disciplinas);
  - o critério de parada deveria ser automático (baseado em % de melhoria), não manual.
- **Limitações reconhecidas pelos autores:** o modelo não trata restrições de grade horária (conflitos de horário, preferências de turno, regra de não dar aula à noite e de manhã seguinte, ocupação de salas) foi deixada para trabalhos futuros.

## 5. Conclusões

O trabalho mostra que uma busca local simples (troca de atribuições, função de custo biobjetivo com soma ponderada) já supera a alocação manual feita atualmente pela coordenação, tanto em equilíbrio de carga quanto em aderência à aptidão dos professores. É um bom caso para a apresentação porque:
- é uma aplicação real, com dados reais e validação por um "tomador de decisão" (o coordenador);
- ilustra bem os conceitos de solução inicial construtiva, vizinhança, movimento de busca local e ótimo local;
- tem limitações claras (não trata restrições de horário/sala), o que dá material para discussão crítica na apresentação.