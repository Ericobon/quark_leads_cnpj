# CLAUDE.md - Quark Leads Analysis

## 🎯 Contexto do Projeto

Análise de enriquecimento de leads empresariais da **Quark Investimentos** utilizando dados públicos da Receita Federal do Brasil (RFB). O projeto cruza uma base comercial adquirida (~44K empresas) com dados estruturados da RFB (~550K empresas pré-qualificadas) para gerar insights comerciais e segmentações estratégicas.

## 📊 Datasets Principais

### **🎯 Base Quark (Comercial)**
- **Arquivo**: `dados_bq/quark_leads_enriquecidos.parquet`
- **Registros**: 44.420 empresas
- **Origem**: Base adquirida de terceiros para prospecção
- **BigQuery**: `view_quark_enriquecida_rfb`
- **Match Rate**: 100% (todas as empresas têm flag `MATCH_CNPJ_COMPLETO`)

### **🏛️ Base RFB (Pré-qualificada)**
- **Arquivo**: `dados_bq/leads_prequalificados.parquet`
- **Registros**: 550.789 empresas
- **Filtros**: Capital ≥ R$ 1Mi, situação ativa, não Simples/MEI
- **BigQuery**: `view_lead_prequalificados_total`
- **Perfil**: Empresas de grande porte apenas

## 🔬 Análise Técnica Realizada

### **1. Data Wrangling**
```python
# Correção de tipos de dados
quark_leads_df['tem_telefone'] = quark_leads_df['tem_telefone'].map({'SIM': True, 'NÃO': False})
quark_leads_df['data_inicio_atividade'] = pd.to_datetime(quark_leads_df['data_inicio_atividade'])
quark_leads_df['capital_social_valor'] = pd.to_numeric(quark_leads_df['capital_social_valor'], errors='coerce')
```

### **2. Análise Descritiva**
| Base | Capital Médio | Std | Min | Max |
|------|---------------|-----|-----|-----|
| **Quark** | R$ 995K | R$ 135Mi | R$ 0 | R$ 28Bi |
| **RFB** | R$ 4.3Bi | R$ 19Bi | R$ 1Mi | R$ 800Bi |

### **3. Distribuições Chave**

#### **Faixa de Capital**
- **Quark**: 63% Pequeno, 18% Micro, 15% Médio, 3% Grande
- **RFB**: 100% Grande (filtro pré-aplicado)

#### **Distribuição Geográfica**
- **SP**: Concentração dominante em ambas as bases (40%+)
- **RJ, MG, RS**: Juntos representam 35%+ do volume
- **Demais UFs**: Distribuição proporcional ao PIB regional

#### **Qualidade de Contatos**
| Indicador | Quark | RFB | Gap |
|-----------|-------|-----|-----|
| **Telefone** | 98.7% | 94.3% | +4.4% |
| **Email** | 70.9% | 83.9% | -13.0% |

### **4. Clusterização Semântica (NLP)**
```python
# TF-IDF + K-Means nos CNAEs
vectorizer = TfidfVectorizer(stop_words=stopwords_pt)
kmeans = KMeans(n_clusters=8, random_state=42)

# Clusters principais identificados:
# - comércio, construção, serviços, participações, etc.
```

## 📁 Outputs Gerados

### **💾 Datasets Finais**
1. **`dados_output/quark_leads_clusterizado.csv`**
   - Base Quark original + clusters semânticos NLP
   - 44.420 registros com 70+ campos
   - Pronto para importação CRM

2. **`dados_output/leads_prequalificados_clusterizado.csv`**
   - Base RFB pré-qualificada + clusters semânticos NLP  
   - 550.789 registros com 50+ campos
   - Empresas de grande porte para prospecção

3. **`socios_administradores_limpo.csv`**
   - Dados de sócios com perfil administrativo
   - Extraído da base societária da RFB

### **📈 Visualizações Criadas**
- Distribuição geográfica por UF (gráficos horizontais)
- Capital social por faixa/porte (barras + histogramas)
- Clusters semânticos por quantidade e capital
- Análise comparativa entre bases

## 🧮 Stack Tecnológico

| Categoria | Ferramentas |
|-----------|-------------|
| **Data** | Pandas, PyArrow, NumPy |
| **ML** | Scikit-learn, NLTK |
| **Viz** | Matplotlib, Seaborn |
| **Cloud** | Google BigQuery |
| **NLP** | TF-IDF, K-Means Clustering |

## 🎯 Comandos para Executar

### **Carregar Dados do BigQuery**
```python
import os
from google.cloud import bigquery
import pandas as pd

# Configurações
os.environ["GOOGLE_APPLICATION_CREDENTIALS"] = "/path/to/service-account.json"
os.environ["GOOGLE_CLOUD_PROJECT"] = "silent-text-458716-c9"

# Cliente
client = bigquery.Client()

# Queries
query_quark = "SELECT * FROM `silent-text-458716-c9.cnpj_dados_rfb.view_quark_enriquecida_rfb`"
query_rfb = "SELECT * FROM `silent-text-458716-c9.cnpj_dados_rfb.view_lead_prequalificados_total`"

# DataFrames
quark_df = client.query(query_quark).to_dataframe()
rfb_df = client.query(query_rfb).to_dataframe()
```

### **Executar Clusterização**
```python
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.cluster import KMeans
import nltk

# Download stopwords
nltk.download('stopwords')
stopwords_pt = nltk.corpus.stopwords.words('portuguese')

# Aplicar clustering
def clusterizar_cnaes(df, n_clusters=8):
    vectorizer = TfidfVectorizer(stop_words=stopwords_pt)
    kmeans = KMeans(n_clusters=n_clusters, random_state=42)
    # ... implementação completa no notebook
```

## 💡 Insights Principais

### **📊 Perfil das Bases**
- **Base Quark**: Mix de portes, foco em empresas menores
- **Base RFB**: Apenas grandes empresas (capital ≥ R$ 1Mi)
- **Sobreposição**: Bases complementares para diferentes estratégias

### **🌍 Concentração Geográfica**
- **Eixo SP-RJ-MG**: 75%+ das oportunidades
- **Mercados emergentes**: GO, PR, RS com potencial
- **Cauda longa**: Demais UFs para estratégias regionais

### **🏭 Setores Dominantes**
1. **Comércio** - Volume alto, capital médio
2. **Construção/Imobiliário** - Capital alto, volume médio  
3. **Serviços** - Diversificado (consultoria, TI, etc.)
4. **Participações/Holdings** - Capital muito alto

### **📞 Estratégia de Contatos**
- **Prioridade 1**: Empresas com telefone + email (70%+ da base Quark)
- **Prioridade 2**: Telefone apenas (28% adicional)
- **Enriquecimento**: Complementar emails faltantes na base Quark

## 🚀 Use Cases Recomendados

### **Para CRM/Sales**
- Importar CSVs clusterizados diretamente
- Segmentar por cluster_semantico + faixa_capital
- Priorizar por presença de contatos

### **Para Marketing**
- Campanhas por cluster (comércio, construção, etc.)
- Geo-targeting (SP, RJ, MG como prioritários)
- Scoring por capital social + idade empresa

### **Para BI/Analytics**
- Dashboards com filtros por cluster e UF
- KPIs de qualificação de leads
- Análise de conversão por segmento

---

*📋 Instruções específicas para Claude - Agosto 2025*  
*Quark Leads: 594K+ empresas analisadas com NLP e clustering*