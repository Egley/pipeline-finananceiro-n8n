## pipeline-finananceiro-n8n
# 💸 Pipeline Financeiro Automatizado — N8N + Gmail + Google Sheets

Automação completa de lançamentos financeiros pessoais: do extrato bancário à planilha categorizada, sem intervenção manual.

---

## 📌 Sobre o projeto

Este projeto surgiu de uma necessidade real: consolidar e analisar meus gastos de múltiplos bancos numa única base de dados padronizada, sem depender de apps financeiros de terceiros que limitam o acesso e a personalização dos dados.

A solução foi construir um pipeline ETL automatizado que:
1. **Detecta** automaticamente e-mails com extratos bancários em formato OFX
2. **Extrai** e parseia as transações do arquivo binário
3. **Categoriza** cada transação com base em um dicionário de regras personalizadas
4. **Insere** os dados já tratados numa planilha Google Sheets, pronta para análise

---

## 🏗️ Arquitetura

```
Gmail (egleyfinance)
      ↓
[Gmail Trigger] — detecta e-mail do Nubank a cada minuto
      ↓
[Get a Message] — baixa o e-mail completo com anexo binário
      ↓
[Verifica se é OFX] — condicional: só processa se tiver .ofx
      ↓
[Processa arquivo OFX] ──────────────────────────┐
      ↓                                           ↓
[Merge] ←── [Lê Dicionário de Categorias] ───────┘
      ↓
[Aplica Dicionário de Categorias]
      ↓
[Insere na Planilha — Google Sheets]
```

---

## 🛠️ Tecnologias utilizadas

| Ferramenta | Uso |
|---|---|
| **N8N Cloud** | Orquestração do fluxo de automação |
| **Gmail API (OAuth2)** | Monitoramento da caixa de entrada e download de anexos |
| **Google Sheets API (OAuth2)** | Leitura do dicionário e inserção das transações |
| **JavaScript (N8N Code Node)** | Parser OFX e motor de categorização |
| **OFX (Open Financial Exchange)** | Formato de extrato bancário utilizado pelo Nubank |

---

## ⚙️ Como funciona

### 1. Gatilho automático (Gmail Trigger)
O N8N monitora a caixa `egleyfinance@gmail.com` a cada minuto. Quando detecta um e-mail não lido do remetente `todomundo@nubank.com.br`, dispara o fluxo automaticamente.

### 2. Download do anexo (Get a Message)
Um segundo nó Gmail busca o conteúdo completo do e-mail, incluindo o arquivo `.ofx` como dado binário — necessário porque o Gmail Trigger não baixa anexos por padrão.

### 3. Parser OFX (Code Node — JavaScript)
Lê o arquivo binário, decodifica para texto e extrai os campos de cada transação:
- Data de lançamento (`DTPOSTED`) e data da compra (`DTUSER`)
- Valor (`TRNAMT`)
- Descrição (`MEMO`)

Para boletos (água, luz, internet), a data de compra é automaticamente igualada à data de pagamento — uma regra de modelagem de dados própria.

### 4. Dicionário de categorias (Google Sheets)
Uma aba separada na planilha (`dicionario`) funciona como motor de regras: o sistema lê todas as palavras-chave e as cruza com a descrição de cada transação para determinar:

| Campo | Exemplo |
|---|---|
| **Identificação** | "Netflix" (padronizado) |
| **Tipo** | Fixo / Variável previsível / Variável extraordinária |
| **CC** | Lazer |
| **Categoria** | Bem-estar |

Transações sem correspondência no dicionário recebem o valor `Não identificado`, facilitando a revisão manual e a expansão incremental das regras.

### 5. Inserção na planilha (Google Sheets)
Os dados categorizados são inseridos automaticamente na aba `transacoes`, no formato:

`Competência-Caixa | Data da compra | Data do pagamento | Banco | Descrição | Valor | Pgto | Identificação | Tipo | CC | Categoria`

---

## 📊 Modelagem dos dados

A estrutura da planilha foi desenhada para análise em tabela dinâmica e dashboards. O campo **Identificação** é o diferencial: padroniza descrições inconsistentes do banco (ex: `Netflix.Com` e `Netflix Entretenimento` viram simplesmente `Netflix`), garantindo agrupamentos corretos nas análises.

O campo **Competência-Caixa** permite alocar um gasto no mês de competência correto, independente da data de lançamento — especialmente útil para compras parceladas e faturas de cartão.

---

## 🗺️ Roadmap

- [x] Fase 1 — Nubank (cartão de crédito via OFX)
- [ ] Fase 2 — Banco do Brasil (extrato débito via OFX)
- [ ] Fase 3 — Banco do Brasil (fatura crédito via PDF)
- [ ] Fase 4 — Inter (a definir conforme formato de exportação)
- [ ] Dashboard de análise no Looker Studio / Power BI

---

## 👩‍💻 Autora

**Égley** — [LinkedIn](https://www.linkedin.com/in/) | [GitHub](https://github.com/Egley)

Profissional em transição para a área de dados, com experiência em operações, automação de processos e análise.
