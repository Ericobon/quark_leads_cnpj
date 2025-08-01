# 📊 Quark Leads - Análise e Enriquecimento de Empresas

**Projeto de qualificação e análise de empresas baseado em dados abertos da Receita Federal do Brasil (CNPJ) com foco em leads de alta renda e prospecção B2B.**

Este projeto realiza o **cruzamento e enriquecimento** de uma base de CNPJs adquirida pela **Quark Investimentos** com dados públicos estruturados da RFB, gerando insights comerciais, segmentações e análises preditivas para qualificação de leads empresariais.

---

## 🎯 Objetivos Principais

### **📈 Enriquecimento de Dados**
- ✅ **Cruzar base comprada Quark** (~44K empresas) com dados públicos RFB (~550K empresas pré-qualificadas)
- ✅ **Identificar match rate** e cobertura dos CNPJs adquiridos (98%+ de match esperado)
- ✅ **Enriquecer com dados estruturados**: razão social, capital social, CNAE, endereço, contatos, situação cadastral
- ✅ **Marcar empresas da base comprada** com flag `MATCH` na base pública total

### **🧠 Análises Estratégicas**
- ✅ **Análise comparativa** entre empresas compradas vs não compradas
- ✅ **Segmentação por capital social**: Micro, Pequeno, Médio, Grande porte
- ✅ **Clusterização semântica** por setores/CNAEs usando NLP
- ✅ **Distribuição geográfica** e perfil de contatos (telefone/email)
- ✅ **Scoring de qualificação** para priorização comercial

### **📊 Outputs e Entregáveis**
- ✅ **Datasets enriquecidos** em formatos Parquet e CSV
- ✅ **Análise exploratória completa** via Jupyter Notebook
- ✅ **Visualizações executivas** (gráficos de distribuição, clusters)
- ✅ **Base qualificada para CRM** e campanhas de marketing

---

## 📁 Estrutura do Projeto

```
quark_leads/
├── 01_analise_exploratoria_quark.ipynb    # 🔬 Notebook principal de análise
├── README.md                              # 📖 Esta documentação
├── CLAUDE.md                              # 🤖 Instruções específicas Claude
├── dados_bq/                              # 📂 Datasets BigQuery originais
│   ├── leads_prequalificados.parquet      # 🏛️ Base RFB pré-qualificada (550K)
│   └── quark_leads_enriquecidos.parquet   # 🎯 Base Quark enriquecida (44K)
├── dados_output/                          # 💾 Outputs finais processados
│   ├── leads_prequalificados_clusterizado.csv  # 📊 Base RFB + clusters NLP
│   └── quark_leads_clusterizado.csv            # 📊 Base Quark + clusters NLP
└── socios_administradores_limpo.csv        # 👥 Dados de sócios limpos
```

## 🧱 Fontes de Dados

### **🏛️ Base Pública RFB (BigQuery)**
- **View**: `silent-text-458716-c9.cnpj_dados_rfb.view_lead_prequalificados_total`
- **Volume**: ~550K empresas pré-qualificadas
- **Filtros**: Capital ≥ R$ 1Mi, situação ativa, não optantes Simples/MEI
- **Campos**: Dados cadastrais, endereço, contatos, CNAE, capital social

### **🎯 Base Comprada Quark (BigQuery)**
- **View**: `silent-text-458716-c9.cnpj_dados_rfb.view_quark_enriquecida_rfb`
- **Volume**: ~44K empresas adquiridas
- **Origem**: Base comercial terceirizada para prospecção
- **Enriquecimento**: Cruzada com dados RFB + flag `MATCH_CNPJ_COMPLETO`

---

## 🧠 Metodologia de Análise

### **🔍 1. Extração e Preparação**
```python
# Conexão BigQuery e extração das views
quark_leads_df = client.query("SELECT * FROM view_quark_enriquecida_rfb").to_dataframe()
leads_prequalificados_df = client.query("SELECT * FROM view_lead_prequalificados_total").to_dataframe()

# Salvamento em Parquet para performance
quark_leads_df.to_parquet('dados/quark_leads_enriquecidos.parquet')
leads_prequalificados_df.to_parquet('dados/leads_prequalificados.parquet')
```

### **📊 2. Análise Exploratória**
- **Estatísticas descritivas**: Capital social, anos de atividade, distribuições
- **Análise geográfica**: Concentração por UF (SP, RJ, MG, RS dominam)
- **Perfil de contatos**: 98%+ têm telefone, 84%+ têm email na base pré-qualificada
- **Regime tributário**: Base Quark tem mix Simples/Lucro Real, base RFB apenas Lucro Real

### **🧠 3. Clusterização Semântica (NLP)**
```python
# TF-IDF + K-Means nos CNAEs
vectorizer = TfidfVectorizer(stop_words=stopwords_pt)
kmeans = KMeans(n_clusters=8, random_state=42)

# Clusters identificados: comércio, construção, serviços, etc.
```

### **✅ 4. Regras de Qualificação**
- **Capital mínimo**: R$ 1 milhão (base pré-qualificada)
- **Situação**: Apenas empresas ativas (código '02')
- **Regime**: Exclusão de optantes Simples Nacional/MEI
- **Contatos**: Priorização de empresas com telefone + email
- **Idade**: Empresas com 3+ anos de atividade

---

## 🚀 Como Utilizar

### **📥 1. Preparação do Ambiente**
```bash
# Instalar dependências
pip install pandas google-cloud-bigquery pyarrow matplotlib seaborn scikit-learn nltk

# Configurar credenciais GCP
export GOOGLE_APPLICATION_CREDENTIALS="/path/to/service-account.json"
export GOOGLE_CLOUD_PROJECT="silent-text-458716-c9"
```

### **🔬 2. Executar Análise Completa**
```bash
# Abrir Jupyter Notebook
jupyter notebook 01_analise_exploratoria_quark.ipynb

# Ou executar via Python
python -c "import pandas as pd; df = pd.read_parquet('dados/quark_leads_enriquecidos.parquet')"
```

### **📊 3. Principais Consultas SQL**

```sql
-- Base Quark enriquecida
SELECT * FROM `silent-text-458716-c9.cnpj_dados_rfb.view_quark_enriquecida_rfb`
WHERE capital_social_valor > 1000000
  AND situacao_cadastral = '02';

-- Base pré-qualificada RFB  
SELECT * FROM `silent-text-458716-c9.cnpj_dados_rfb.view_lead_prequalificados_total`
WHERE faixa_capital = 'GRANDE'
  AND tem_telefone = True
  AND tem_email = True;

-- Análise de match rate
SELECT 
  COUNT(*) as total_quark,
  SUM(CASE WHEN tipo_match = 'MATCH_CNPJ_COMPLETO' THEN 1 ELSE 0 END) as matches,
  ROUND(100.0 * SUM(CASE WHEN tipo_match = 'MATCH_CNPJ_COMPLETO' THEN 1 ELSE 0 END) / COUNT(*), 2) as match_rate_pct
FROM `silent-text-458716-c9.cnpj_dados_rfb.view_quark_enriquecida_rfb`;
```

---

## 📈 Principais Insights

### **💰 Perfil de Capital Social**
| Base | Capital Médio | Faixa Dominante | Empresas |
|------|---------------|-----------------|----------|
| **Quark** | R$ 995K | Pequeno (63%) | 44.420 |
| **RFB** | R$ 4.3Bi | Grande (100%) | 550.789 |

### **🌍 Distribuição Geográfica**
- **SP**: 40%+ de ambas as bases (concentração nacional)
- **RJ, MG, RS**: 35%+ do volume restante
- **Demais UFs**: Representação proporcional ao PIB estadual

### **📞 Qualidade de Contatos**
| Indicador | Quark | RFB | Gap |
|-----------|-------|-----|-----|
| **Telefone** | 98.7% | 94.3% | +4.4% |
| **Email** | 70.9% | 83.9% | -13.0% |
| **Ambos** | 70.2% | 79.1% | -8.9% |

### **🏭 Setores Dominantes (Clusters)**
1. **Comércio** - 35%+ das empresas
2. **Construção** - 20%+ (imobiliário forte)
3. **Serviços** - 15%+ (consultoria, TI)
4. **Participações** - 10%+ (holdings)

---

## 🔧 Tecnologias

| Categoria | Tecnologia | Versão |
|-----------|------------|--------|
| **Linguagem** | Python | 3.11+ |
| **Data** | Pandas, PyArrow | Latest |
| **Cloud** | Google BigQuery | API v2 |
| **ML** | Scikit-learn, NLTK | Latest |
| **Viz** | Matplotlib, Seaborn | Latest |

---

## 📊 Outputs Gerados

### **📁 Arquivos de Dados**

#### **📂 Dados Originais (BigQuery)**
- [`dados_bq/quark_leads_enriquecidos.parquet`](dados_bq/quark_leads_enriquecidos.parquet) - Base Quark original (44.420 empresas)
- [`dados_bq/leads_prequalificados.parquet`](dados_bq/leads_prequalificados.parquet) - Base RFB pré-qualificada (550.789 empresas)

#### **💾 Outputs Processados (Prontos para CRM)**

> 📂 **Pasta compartilhada OneDrive Quantum Invest**  
> 🔗 **Acesso para colaboradores**: Entre em contato com erico.bonilha@quarkinvestimentos.com.br

#### **📥 Downloads Individuais:**

- **🎯 Base Quark Clusterizada** (30MB) - Base Quark + clusters semânticos NLP  
  📁 `dados_output/quark_leads_clusterizado.csv`  
  📥 [**DOWNLOAD DIRETO**](https://quantuminvest-my.sharepoint.com/:x:/g/personal/erico_bonilha_quarkinvestimentos_com_br/EQpgyl0X1KlEiB_4dlX8dY0BG9iwsIzivG50Q1SztE6CXQ?e=8u56r6)

- **🏛️ Base RFB Clusterizada** (296MB) - Base RFB + clusters semânticos NLP  
  📁 `dados_output/leads_prequalificados_clusterizado.csv`  
  📥 [**DOWNLOAD DIRETO**](https://quantuminvest-my.sharepoint.com/:x:/g/personal/erico_bonilha_quarkinvestimentos_com_br/EYbCfI6lswVKpZpVmKbAqBAButfvuiky2-uVwYbR6rPbQQ?e=fak1rL)

- **👥 Sócios Administrativos** (6.6MB) - Incluído no repositório ✅  
  📁 [`socios_administradores_limpo.csv`](socios_administradores_limpo.csv)

#### **📊 Como Acessar os Dados:**
1. **Arquivos pequenos**: Clone o repositório normalmente
2. **Arquivos grandes**: Downloads diretos via OneDrive Quark Investimentos
3. **BigQuery**: Execute as queries SQL documentadas na seção anterior

#### **🔐 Acesso Empresarial:**
- **Hospedagem**: OneDrive Business - Quantum Invest
- **Responsável**: erico.bonilha@quarkinvestimentos.com.br
- **Controle**: Links corporativos com estatísticas de acesso

### **📈 Visualizações**
- Distribuição geográfica por UF
- Capital social por faixa/porte
- Clusters semânticos por setor
- Análise comparativa bases

### **🎯 Use Cases**
- **CRM**: Importação direta dos CSVs clusterizados
- **Marketing**: Segmentação por capital + localização
- **Sales**: Priorização por scoring de qualificação
- **BI**: Dashboards executivos com KPIs

---

## 🚀 Próximos Passos

- [ ] **Scoring preditivo** com ML para qualificação de leads
- [ ] **API REST** para consulta em tempo real
- [ ] **Dashboard Power BI** com filtros interativos
- [ ] **Automação** de atualização mensal dos dados
- [ ] **Integração CRM** (HubSpot, Salesforce)

---

*✅ Documentação atualizada - Julho 2025*  
*Quark Leads - Análise de 594K+ empresas com foco em alta renda*

