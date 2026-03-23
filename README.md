# Telco Customer Churn Analysis: Engenharia de Dados, UI/UX e Estratégia de Retenção

## 📌 1. O Desafio de Negócio (O Problema)
No setor de Telecomunicações, o Custo de Aquisição de Clientes (CAC) é extremamente alto. Portanto, reter o cliente pelo maior tempo possível (aumentando o LTV - *Lifetime Value*) é o principal motor de lucratividade. 

Este projeto End-to-End visa resolver um problema clássico e custoso: o **Churn (Evasão de Clientes)**. Atuando como uma ponte entre a engenharia de dados e a Diretoria Comercial, o objetivo foi construir um pipeline completo — desde a modelagem no banco de dados até o design de interface (UI/UX) — para identificar onde a empresa está perdendo dinheiro e por quê.

### ❓ Perguntas de Negócio a serem respondidas:
1. Qual é o impacto financeiro (Receita Perdida) exato gerado pela evasão atual?
2. Quais perfis demográficos (Idade, Estrutura Familiar) apresentam maior propensão ao cancelamento?
3. O modelo de contrato atual está protegendo a receita ou facilitando a saída do cliente?
4. Existe algum gargalo técnico ou serviço específico "expulsando" a base de usuários?
5. Estratégias de *Cross-sell* (venda de serviços adicionais) são eficazes para reter o cliente no ecossistema?

---

## 🛠️ 2. Arquitetura e Stack Tecnológico
Para garantir governança, performance e uma experiência visual de alto nível, a solução integrou as seguintes ferramentas:

* **SGBD (Sistema de Gerenciamento de Banco de Dados):** PostgreSQL.
* **Linguagem de Consulta e Estruturação:** SQL (DDL e DQL).
* **UI/UX Design e Prototipagem:** Figma.
* **Conexão de Dados:** Import Data Mode (Power BI).
* **Linguagem de Modelagem Analítica:** DAX (Data Analysis Expressions).
* **Visualização:** Power BI Desktop.

---

## 🎨 3. UI/UX Design e Prototipagem (Figma)
Um dashboard só gera valor se for intuitivo. Antes de importar os dados para o Power BI, toda a interface visual foi planejada e desenhada no **Figma**, aplicando conceitos rigorosos de design de interfaces e redução de carga cognitiva.

* **Wireframing e Layout:** Utilização do sistema de Grids (ex: Grade 2x3 para a página de serviços) para garantir alinhamento perfeito e fluidez de leitura (padrão Z e Top-Down). 
* **Custom Backgrounds:** Criação de fundos personalizados em formato de imagem (Dark Mode em tom de azul profundo), exportados do Figma para o Power BI. Isso reduziu o tempo de renderização da página, pois substituiu múltiplos elementos visuais pesados por um único *wallpaper* otimizado.
* **Componentização:** Os cartões (*cards*) dos gráficos foram desenhados com fundos claros, sombras suaves (Drop Shadow) e bordas arredondadas (10px) para criar profundidade e separar claramente os blocos de informação.
* **Psicologia das Cores:** Aplicação de regras estritas de formatação condicional. 
  * **Vermelho:** Associado ao perigo/alerta. Usado exclusivamente para destacar ofensores (Evasão alta, Contratos Mensais, Ausência de Suporte).
  * **Azul Escuro:** Associado à segurança/retenção. Usado para cenários saudáveis.
* **Navegação (App-like Experience):** Criação de botões customizados minimalistas (como a seta branca sem fundo para navegação) importados do Figma, transformando o relatório em uma experiência de aplicativo fluido.

---

## 🗄️ 4. Engenharia de Dados e Modelagem (PostgreSQL)
A base bruta original consistia em uma única tabela *flat file* desnormalizada contendo mais de 7.000 registros e 21 colunas misturadas. 

Para estruturar um modelo analítico otimizado e performático, projetei um **Star Schema (Esquema Estrela)** diretamente no **PostgreSQL** através de scripts SQL.

### Scripts de Criação do Star Schema (DDL):

```sql
-- 1. Tabela Dimensão Cliente (Características Demográficas)
CREATE TABLE dim_cliente AS
SELECT DISTINCT
    customerID, 
    gender AS genero, 
    SeniorCitizen AS idoso, 
    Partner AS temparceiro, 
    Dependents AS temdependentes
FROM telco_raw_data;

ALTER TABLE dim_cliente ADD CONSTRAINT pk_cliente PRIMARY KEY (customerID);

-- 2. Tabela Dimensão Contrato (Dados de Faturamento)
CREATE TABLE dim_contrato AS
SELECT DISTINCT
    customerID, 
    Contract AS tipocontrato, 
    PaperlessBilling AS faturamentodigital, 
    PaymentMethod AS metodopagamento
FROM telco_raw_data;

ALTER TABLE dim_contrato ADD CONSTRAINT pk_contrato PRIMARY KEY (customerID);

-- 3. Tabela Dimensão Serviços (Portfólio e Produtos Ativos)
CREATE TABLE dim_servicos AS
SELECT DISTINCT
    customerID, 
    PhoneService, 
    MultipleLines, 
    InternetService, 
    OnlineSecurity, 
    OnlineBackup, 
    DeviceProtection, 
    TechSupport, 
    StreamingTV, 
    StreamingMovies
FROM telco_raw_data;

ALTER TABLE dim_servicos ADD CONSTRAINT pk_servicos PRIMARY KEY (customerID);

-- 4. Tabela Fato Churn (Eventos, Métricas Financeiras e Status)
CREATE TABLE fato_churn AS
SELECT 
    customerID, 
    tenure AS mesespermanencia, 
    MonthlyCharges AS valormensal, 
    TotalCharges AS valortotal, 
    Churn AS statuschurn
FROM telco_raw_data;

-- Criação das Chaves Estrangeiras (Relacionamentos 1:N)
ALTER TABLE fato_churn ADD CONSTRAINT fk_fato_cliente FOREIGN KEY (customerID) REFERENCES dim_cliente(customerID);
ALTER TABLE fato_churn ADD CONSTRAINT fk_fato_contrato FOREIGN KEY (customerID) REFERENCES dim_contrato(customerID);
ALTER TABLE fato_churn ADD CONSTRAINT fk_fato_servicos FOREIGN KEY (customerID) REFERENCES dim_servicos(customerID);


🧮 5. Desenvolvimento Analítico (DAX)
Dentro do Power BI, desenvolvi uma série de cálculos avançados e variáveis de engenharia (Feature Engineering) para traduzir o banco de dados em KPIs comerciais.

Colunas Calculadas (Clusterização e Estratégia):
Sensibilidade ao Preço: Agrupamento de tickets mensais para análise de ruptura financeira no bolso do consumidor.

Snippet de código
Faixa de Mensalidade = 
SWITCH(
    TRUE(),
    'fato_churn'[valormensal] <= 30, "1. Básico (Até $30)",
    'fato_churn'[valormensal] <= 60, "2. Intermediário ($31 a $60)",
    'fato_churn'[valormensal] <= 90, "3. Avançado ($61 a $90)",
    "4. Premium (Acima de $90)"
)
Índice de Efeito Lock-in (Cross-sell): Quantificação do engajamento do cliente no ecossistema de produtos.

Snippet de código
Qtd Servicos Adicionais = 
IF('dim_servicos'[OnlineSecurity] = "Yes", 1, 0) +
IF('dim_servicos'[OnlineBackup] = "Yes", 1, 0) +
IF('dim_servicos'[DeviceProtection] = "Yes", 1, 0) +
IF('dim_servicos'[TechSupport] = "Yes", 1, 0) +
IF('dim_servicos'[StreamingTV] = "Yes", 1, 0) +
IF('dim_servicos'[StreamingMovies] = "Yes", 1, 0)
Principais Medidas de Negócio:
Total de Clientes = COUNTROWS('fato_churn')

Clientes Perdidos = CALCULATE([Total de Clientes], 'fato_churn'[statuschurn] = "Yes")

Taxa de Churn = DIVIDE([Clientes Perdidos], [Total de Clientes], 0)

Receita Perdida = CALCULATE(SUM('fato_churn'[valormensal]), 'fato_churn'[statuschurn] = "Yes")

Tempo de Casa Medio (Meses) = AVERAGE('fato_churn'[mesespermanencia])

📈 6. Storytelling de Dados e Dashboards
O fluxo analítico foi estruturado em 3 páginas lógicas (Drill-down conceitual), guiando o gestor do macro (financeiro) para o micro (causa raiz).

📄 Página 1: Resumo Executivo (O Tamanho do Problema)
Objetivo: O impacto global na companhia.

Componentes Chave: Cartões de KPIs com destaque para a Receita Perdida. O visual principal é o Raio-X do Churn: Análise de Causa Raiz Demográfica, que utiliza Inteligência Artificial (Árvore de Decomposição) para descobrir o caminho exato que eleva as taxas de cancelamento.

<img width="1116" height="621" alt="image" src="https://github.com/user-attachments/assets/a33effef-c4ca-44f1-a013-4ec9ff4fed2d" />

📄 Página 2: Perfil do Cliente (Quem está saindo?)
Objetivo: Investigação comportamental e demográfica.

Componentes Chave: A Matriz de Risco Crítico: Churn por Estrutura Familiar e o gráfico de Impacto Familiar na Retenção (Dependentes).

Insights Gerados: Clientes solteiros e sem dependentes (fatia vermelha) apresentam risco desproporcional. A análise do Perfil da Evasão: Proporção do Público Sênior (60+) reforça a necessidade de atendimento especializado, já que a quebra de contrato ocorre com frequência no público de maior idade.

<img width="1121" height="623" alt="image" src="https://github.com/user-attachments/assets/9edd2df9-2c76-4df4-81ec-d7bb7756de66" />

📄 Página 3: Serviços e Contratos (Por que estão saindo?)
Objetivo: Investigação de falhas de produto e precificação.

Componentes Chave & Insights:

Sangria de Caixa: O gráfico de Receita Perdida por Modalidade de Contrato escancara que o contrato "Mensal" é o grande ralo financeiro.

Gargalo Técnico: O visual de Taxa de Evasão por Tecnologia de Internet revela uma anomalia grave: a Fibra Óptica é o principal ofensor. Aliado a isso, o Risco de Cancelamento por Suporte Técnico mostra que a falta de assistência triplica o cancelamento.

A Solução (Lock-in): O visual em linha Efeito 'Lock-in': Impacto do Cross-sell traz a resposta estratégica. A matemática comprova: vender 4 ou mais serviços extras derruba a evasão para patamares irrisórios.

<img width="1118" height="628" alt="image" src="https://github.com/user-attachments/assets/b3757618-c64f-4e01-a534-4b04d40eb3c1" />

🚀 7. Conclusão e Plano de Ação
A união do design focado no usuário (Figma), engenharia de dados robusta (PostgreSQL) e regras de negócios aplicadas ao Power BI permitiu transformar incertezas em diretrizes táticas. As recomendações para a Diretoria são:

Reestruturação Comercial: Migrar massivamente campanhas de marketing para incentivar contratos de 1 a 2 anos, abandonando o foco no contrato mensal.

Estratégia de Cross-sell: Criar ofertas do tipo "Combo Segurança + Suporte" visando forçar o aumento da métrica de Serviços Adicionais por cliente, aproveitando o efeito algema (Lock-in).

Auditoria Técnica: Abrir um chamado imediato com a engenharia de redes para diagnosticar quedas de qualidade ou falhas na instalação da Fibra Óptica.

👨‍💻 Autor
Arthur Mesquita
Estudante de Administração | Analista de Dados & BI.
linkedin.com/in/arthur-g-mesquita
📍 Recife, PE
