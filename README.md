# Detecção de Transações Suspeitas em Dados Bancários

Este projeto desenvolve uma abordagem para identificar, pontuar e priorizar transações financeiras suspeitas em um cenário sem rótulos prévios de fraude (*unlabeled data*), utilizando análise exploratória de dados (EDA) e criação de um sistema de regras e pontuação de risco.

---

## 📌 Contexto do Problema
O conjunto de dados bancários não possui uma variável alvo que indique previamente se uma transação foi fraude ou não. Por essa razão, a análise não utiliza aprendizado supervisionado tradicional; em vez disso, foca em **critérios objetivos de risco operacional e comportamental** para ranquear as movimentações que exigem revisão prioritária por analistas de segurança.

---

## 📊 Base de Dados
* **Fonte:** *Bank Transaction Dataset for Fraud Detection* (Kaggle)
* **Licença:** Apache 2.0
* **Volume:** 2.512 transações financeiras distribuídas entre 495 contas.
* **Variáveis exploradas:** Identificador de transação/conta, valor, tipo (débito/crédito), canal (ATM, Branch, Online), cidade, dados do cliente (idade, ocupação, saldo) e telemetria básica (IP, dispositivo, duração e tentativas de login).

---

## ⚙️ Metodologia e Etapas do Projeto

### 1. Limpeza e Preparação dos Dados
* **Tratamento de Inconsistências:** A coluna `PreviousTransactionDate` continha datas posteriores à data da própria transação em 100% dos registros. Por ser um dado não confiável, a coluna foi descartada para evitar cálculos enviesados.
* **Padronização:** Renomeação das variáveis para o português.
* **Engenharia de Variáveis Temporais e Demográficas:** 
  * Extração de `hora`, `ano_mes` e `dia_semana`.
  * Criação de `faixa_etaria` agrupada em: 18-29, 30-44, 45-59 e 60+ anos.

### 2. Análise Exploratória de Dados (EDA)
* **Categorias:** Identificação de proporções equilibradas entre os canais de atendimento e profissões, com predominância de transações de débito (77,4%) sobre crédito (22,6%).
* **Distribuição de Valores:** A média (~298) apresentou-se significativamente superior à mediana (~211), evidenciando cauda longa à direita com presença de *outliers* de até 1.919. Com isso, definiu-se que o limite estatístico de "valor alto" deveria usar a amplitude interquartil (**IQR**), e não a média.
* **Análise por Grupos:** Boxplots e agregações mostraram que o valor médio não varia significativamente por canal, indicando que o canal isoladamente não deve ser usado como critério de risco.
* **Padrões Temporais:** Concentração das operações exclusivamente entre as 16h00 e 18h59 e maior volume em dias úteis (especialmente segundas-feiras). Como o horário não varia ao longo do dia/madrugada, substituiu-se a ideia de "horário incomum" pela métrica de "duração incomum".

### 3. Definição dos Sinais de Risco (Heurísticas)
Foram estabelecidos 5 sinais objetivos com verificação binária (0 ou 1):
1. **Tentativas excessivas de login:** Transações precedidas por 3 ou mais tentativas de senha incorreta.
2. **Valor muito alto (IQR):** Valores superiores ao limite estatístico de corte (corte estatístico: Q3 + 1,5 × IQR ≈ 913,50).
3. **Valor maior que o saldo:** Transações que excedem o saldo disponível na conta no momento da operação.
4. **Dispositivo compartilhado por múltiplas contas:** dispositivos associados a 8 ou mais contas diferentes (limite calibrado para evitar falsos alarmes causados por compartilhamento comum de hardware).
5. **Duração incomum:** Operações nos extremos de tempo (os 5% mais rápidos [< 24 s] ou os 5% mais lentos [> 265 s]).

### 4. Pontuação e Classificação de Risco
A pontuação final é dada pela soma dos sinais ativos (de 0 a 5), categorizada em quatro níveis:
* **Baixo (0 sinais):** Comportamento padrão (1.871 transações).
* **Moderado (1 sinal):** Alertas isolados (557 transações).
* **Alto (2 sinais):** Combinação de anomalias (80 transações).
* **Crítico (3 ou mais sinais):** Múltiplos indicadores críticos simultâneos (4 transações prioritárias para auditoria).

---

## 🛠️ Tecnologias Utilizadas
* **Linguagem:** Python
* **Bibliotecas:** 
  * `pandas` e `numpy` para análise estrutural, limpeza e métricas.
  * `matplotlib` e `seaborn` para visualizações e análises de distribuição.

---

## 📁 Estrutura do Repositório
* `transações_bancárias_em_2.ipynb`: Caderno com o código executado, limpeza, gráficos de exploração e tabela de priorização final.
* `README.md`: Documentação e justificativas de negócio do projeto.
