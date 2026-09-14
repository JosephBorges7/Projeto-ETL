### 🏗️ Arquitetura do Projeto

O projeto foi estruturado seguindo o modelo de pipeline **ETL (Extract, Transform, Load)**, garantindo a separação de responsabilidades entre a coleta, o tratamento e a persistência dos dados.

---

### 🚀 Guia de Execução (Passo a Passo)

Para garantir a integridade dos dados, siga rigorosamente a ordem abaixo:

#### 1. Preparação do Ambiente

(Dentro de um terminal do Git BASH)

```bash
# Entrar na pasta do projeto
cd Teste-Estagio-IntuitiveCare

# Criar o ambiente virtual
python -m venv .venv

# Ativar o ambiente virtual
# Windows
source .venv/Scripts/activate

# Linux/macOS
source .venv/bin/activate

# Instalar dependências principais
pip install pandas requests beautifulsoup4 sqlalchemy psycopg2-binary fastapi uvicorn
```

---

#### 2. Configuração do Banco de Dados

- Crie um banco de dados PostgreSQL chamado `intuitive_db`.
- Altere a constante DB_URL no arquivo importacao.py e api.py com suas credenciais.
- Execute o script `schema.sql` para criar tabelas, chaves primárias, chaves estrangeiras e índices.

---

#### 3. Execução do Pipeline ETL

- **Extração:** `python extracao.py` (Download e extração via Web Scraping resiliente).
- **Tratamento:** `python transformacao.py` (Limpeza e padronização inicial).
- **Enriquecimento:** `python enriquecimento.py` (Cruzamento de dados entre operadoras e despesas).
- **Inteligência:** `python agregacao.py` (Geração do arquivo de KPIs estatísticos).
- **Carga (Load):** `python importacao.py` (Persistência no PostgreSQL com tratamento de encoding e limpeza via TRUNCATE CASCADE).

Observação: execute esses comandos no mesmo terminal do Git BASH.

---

#### 4. Disponibilização e Visualização

Execute:
```bash
python api.py
```

Interface: Abra o arquivo index.html em seu navegador para visualizar o relatório reativo (eu uso a extensão do Live Server por ser mais prático e rápido, você encontra ela facilmente na aba de extensões do VScode).

---

#### 5. Validação e Análise

Execute as queries contidas em `analise.sql` no cliente SQL (ex: pgAdmin).

---

### 🧠 Decisões de Engenharia e Trade-offs

#### Processamento e Memória
Estratégia: Processamento Incremental por arquivos.

Justificativa: Para suportar o volume massivo da ANS (centenas de milhares de linhas por trimestre), evitamos o carregamento em lote na RAM, prevenindo erros de **Stack Overflow** ou **Out of Memory**.

#### Modelagem de Dados
Abordagem: **Opção B (Tabelas Normalizadas)**.

Justificativa:

**Escalabilidade:** Reduz a redundância de dados cadastrais (Razão Social, UF) que se repetem milhões de vezes nas despesas.

**Integridade:** Uso de **Foreign Keys (FK)** para garantir que nenhuma despesa seja órfã de uma operadora cadastrada.

#### Precisão Financeira
Tipo de Dado: **DECIMAL(18,2)**.

**Justificativa:** Em sistemas de back-end contábil, o uso de FLOAT é evitado devido à imprecisão binária em grandes somas. O DECIMAL garante que cálculos de bilhões de reais sejam exatos.

#### Arquitetura da Etapa 4 (API & Front-end)
**Escolha:** FastAPI + Vue.js (via CDN). 

**Justificativa:**

**Pragmatismo:** Como o foco é Back-end, optei pelo FastAPI pela alta performance e documentação automática.

**Desacoplamento:** O uso de Vue.js via CDN permitiu criar uma interface reativa e moderna para consumir a API sem a complexidade desnecessária de um ambiente de build Node.js, mantendo o projeto leve e focado na integração de dados.

### 🔍 Qualidade e Higiene de Dados (Etapa 1.3 & 3.3)

Para garantir a confiabilidade dos relatórios, implementei:

**Regex Sanitization:** Extração de metadados diretamente dos nomes dos arquivos para evitar erros de digitação nas planilhas.

**Normalização de Tipos:** Conversão automática de strings/vírgulas em formatos numéricos compatíveis com o PostgreSQL durante a importação.

**Deduplicação Inteligente:** Lógica de **keep last** para manter apenas a versão mais atualizada da razão social de cada operadora.

---

### 📂 Estrutura de Arquivos

- `saida/`
- `schema.sql`
- `importacao.py`

