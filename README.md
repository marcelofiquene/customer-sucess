# 🎯 Customer Success — Clusterização de Clientes

> Projeto avaliativo da Pós-graduação em **Data Analytics e Inteligência Artificial Aplicada a Negócios** — [FNAT (Fundação de Negócios, Analytics e Tecnologia)](https://fnat.com.br)
>
> O case e a base de dados foram fornecidos pela FNAT.

---

## 📋 Sobre o projeto

Este projeto aplica técnicas de **clusterização não supervisionada (K-Means)** para segmentar a base de clientes da Insight Academy — uma plataforma brasileira de educação online com mais de 45 mil alunos cadastrados em cursos de tecnologia, machine learning e análise de dados.

O objetivo é substituir uma abordagem homogênea de marketing e relacionamento por uma **segmentação orientada a dados**, identificando grupos com perfis comportamentais distintos e propondo ações específicas de retenção, reativação e fidelização para cada segmento.

---

## 🗂️ Estrutura do repositório

```
📦 customer-success-clustering
├── 📓 customer_segmentation_analysis.ipynb   # Notebook principal com toda a análise
├── 📄 business_recommendations.docx          # Recomendações de negócio por cluster
├── 📊 base_customer.xlsx                     # Base de dados original (fornecida pela FNAT)
└── 📊 new_base_customer.xlsx                 # Base de dados fictícia para testar o predict
```

---

## 🔍 Contexto do problema

A liderança da Insight Academy identificou que o crescimento da base de clientes **não estava acompanhado de engajamento proporcional**:

- Clientes compravam cursos mas raramente acessavam a plataforma
- Assinantes recorrentes permaneciam meses sem qualquer interação
- Campanhas de marketing enviadas para toda a base apresentavam taxas de conversão decrescentes
- Não havia segmentação — todos os clientes eram tratados de forma idêntica

A equipe de analytics foi acionada para desenvolver uma **segmentação data-driven** que apoiasse os times de marketing, retenção e vendas.

---

## 🧪 Metodologia

### 1. Seleção de features

Das 27 colunas disponíveis, 5 foram selecionadas como inputs do modelo, com justificativa documentada para cada decisão:

| Feature | Justificativa |
|---|---|
| `recorrente` | Diferencia modelo de pagamento (assinatura vs. avulso) |
| `ativo` | Indica status atual do cliente na plataforma |
| `n_acessos` | Mede intensidade de uso — proxy de valor percebido |
| `dias_sem_acessar` | Forte indicador de risco de churn |
| `total_parcelas` | Indica comprometimento financeiro do cliente |

### 2. Pré-processamento

- Encoding binário da coluna `ativo` (sim/não → 1/0)
- Imputação de `dias_sem_acessar` para clientes que nunca acessaram, calculada a partir da data de compra
- Padronização com `StandardScaler` (obrigatória para K-Means, sensível à escala)
- Validação de multicolinearidade via heatmap de correlação

### 3. Escolha do K

Dois critérios combinados:

- **Método do Cotovelo (Elbow):** curva de inércia (WCSS) testada de k=1 a k=10, com desaceleração clara a partir de k=5
- **Silhouette Score:** K=5 obteve score de **0.547**, superior a K=3 (0.502) e K=4 (0.480), e próximo a K=6 (0.583) com menor complexidade de interpretação

### 4. Modelagem

```python
model = KMeans(n_clusters=5, random_state=42, n_init='auto')
model.fit(df_scaled)
```

Modelo salvo com `joblib` para classificação de novos clientes via `predict()` sem necessidade de retreinamento.

---

## 📊 Resultados — Perfil dos clusters

| Cluster | Nome | Clientes | % Base | Recorrente | Ativo | Acessos | Dias inativo | Renovação |
|---|---|---:|---:|---:|---:|---:|---:|---:|
| C0 | Inativos de Longo Prazo | 3.334 | 7,3% | 6% | 81% | 9,6 | 337,5 | 26% |
| C1 | Assinantes Ativos | 12.393 | 27,1% | 100% | 100% | 22,4 | 117,5 | 12% |
| C2 | Assinantes em Risco | 7.968 | 17,4% | 100% | 0% | 6,9 | 141,9 | 0% |
| C3 | Avulsos Engajados | 19.078 | 41,6% | 0% | 100% | 26,0 | 103,6 | 11% |
| C4 | Dormentes Recentes | 3.042 | 6,6% | 0% | 0% | 1,1 | 47,0 | 0% |

---

## 🎯 Ações de negócio recomendadas

| Prioridade | Cluster | Ação principal |
|---|---|---|
| 1° — Crítica | C2 Assinantes em Risco | Onboarding de emergência + gatilho automático após 30 dias sem acesso |
| 2° — Urgente | C4 Dormentes Recentes | Sequência de e-mails de boas-vindas + checklist de primeiros passos |
| 3° — Alta | C1 Assinantes Ativos | Programa de fidelidade + campanha de renovação 60 dias antes do vencimento |
| 4° — Média | C0 Inativos de Longo Prazo | Campanha de reativação segmentada + pesquisa de motivo de abandono |
| 5° — Baixa | C3 Avulsos Engajados | Oferta de conversão para assinatura com desconto + trial de 30 dias |

> Detalhamento completo das ações disponível em [`business_recommendations.docx`](./business_recommendations.docx)

---

## 🔮 Classificação de novos clientes

O modelo serializado permite classificar novos clientes automaticamente, aplicando os mesmos pré-processamentos da base de treino:

```python
import joblib
import pandas as pd

model = joblib.load('cluster_model.pkl')

# Aplicar os mesmos tratamentos da base de treino
features = ['recorrente', 'ativo', 'n_acessos', 'dias_sem_acessar', 'total_parcelas']
X_new = df_new[features]

df_new['cluster'] = model.predict(X_new)
```

---

## 🛠️ Tecnologias utilizadas

| Biblioteca | Uso |
|---|---|
| `pandas` | Manipulação e transformação de dados |
| `numpy` | Operações numéricas |
| `matplotlib` / `seaborn` | Visualizações gráficas |
| `scikit-learn` | K-Means, StandardScaler, Silhouette Score |
| `joblib` | Serialização e reutilização do modelo |

---
## 📄 Licença

Base de dados e case de negócio © FNAT — Fundação de Negócios, Analytics e Tecnologia. Uso restrito a fins acadêmicos.
