# Base de Conhecimento

### 📂 Detalhamento dos Dados Utilizados

| Arquivo | Formato | Utilização no Agente |
|:---|:---|:---|
| `extrato.json` | JSON | **Fonte de Fatos:** Contém o histórico de transações e saldo atual. É usado pela "Análise de Saldo/Gastos" para identificar padrões de consumo e gatilhos de alerta. |
| `perfil_cliente.json`| JSON | **Contextualização:** Fornece dados como limite de crédito e metas financeiras, permitindo que a IA personalize o tom de voz e as sugestões de reserva. |
| `dicas_educativas.md` | Markdown | **Base de Conhecimento (RAG):** Repositório com diretrizes oficiais de educação financeira. Serve para "ancorar" as respostas da IA e evitar alucinações técnicas. |
| `categorias_gastos.csv`| CSV | **Classificação:** Tabela auxiliar para agrupar despesas (Ex: Lazer, Alimentação), permitindo que o agente calcule o "Custo de Oportunidade" com precisão. |

🔑 Os dados foram anonimizados antes de serem enviados para a API da LLM, seguindo as boas práticas da LGPD e as diretrizes de segurança do Bradesco.

---

## Estratégia de Integração

### Como os dados são carregados?
> Descreva como seu agente acessa a base de conhecimento.

### ⚙️ Estratégia de Integração de Dados

| Etapa | Processo Técnico | Objetivo |
|:---|:---|:---|
| **1. Ingestão** | Leitura de arquivos via `json.load()` ou `pd.read_csv()`. | Carregar os dados brutos da pasta `/data` para a memória do Python. |
| **2. Processamento** | Filtragem e agregação com [Pandas](https://pandas.pydata.org). | Transformar o extrato em "insights" (ex: gasto com lazer subiu 20%). |
| **3. Recuperação (RAG)** | Busca por palavras-chave em arquivos `.md` ou `.txt`. | Selecionar a dica educativa do [Bradesco](https://banco.bradesco) que combine com o gasto detectado. |
| **4. Augmentação** | Injeção de Contexto no **Prompt de Sistema**. | Combinar Perfil do Usuário + Insight de Gasto + Dica Educativa em uma única instrução para a IA. |
| **5. Geração** | Chamada de API (OpenAI/Gemini). | A IA processa o pacote de dados e gera a resposta final humanizada. |

---

### 📝 Descrição da Estratégia de Integração

*   **Processamento Eficiente de Dados:** A integração começa com o carregamento dos arquivos JSON e CSV utilizando a biblioteca [Pandas](https://pandas.pydata.org). Isso permite que o sistema identifique "gatilhos" financeiros (como saldo baixo ou gastos atípicos) através de lógica de programação pura, antes mesmo de acionar a inteligência artificial, economizando recursos e aumentando a precisão.

*   **Arquitetura de Recuperação (RAG):** Implementamos uma camada de **Retrieval-Augmented Generation (RAG)** que conecta os dados do usuário a uma base de conhecimento em Markdown. O sistema busca dinamicamente orientações de educação financeira do [Bradesco](https://banco.bradesco) que sejam relevantes para o cenário atual do cliente, garantindo que o conselho proativo seja fundamentado e seguro.

*   **Orquestração de Contexto:** A estratégia de integração culmina na **Injeção de Contexto**. Em vez de enviar dados brutos e desordenados para o modelo de linguagem (LLM), o sistema organiza um "pacote de informações" contendo o perfil do usuário, o insight gerado pela análise e a dica educativa recuperada. Isso garante que a IA atue estritamente como um **Mentor Financeiro** e evite alucinações.

*   **Segurança e Privacidade (LGPD):** Durante todo o fluxo de integração, os dados passam por uma camada de anonimização. Informações sensíveis são tratadas localmente e apenas o contexto necessário para a geração da resposta educativa é enviado para a API da IA Generativa, seguindo as diretrizes de ética e segurança do [Bradesco](https://banco.bradesco).

💡 Essa estratégia transforma o agente de uma simples ferramenta de consulta em um parceiro proativo, que utiliza dados para educar e prevenir problemas financeiros.

### Como os dados são usados no prompt?
> Os dados vão no system prompt? São consultados dinamicamente?

### 🧠 Estratégia de Uso de Dados no Prompt

A utilização dos dados no modelo de linguagem (LLM) segue uma arquitetura de **Injeção Dinâmica de Contexto**, garantindo que o agente seja sempre atualizado e preciso.

#### 1. System Prompt (Camada Estática)
O **System Prompt** funciona como o "manual de conduta" fixo do agente. Ele define a **Personalidade de Mentor Educativo** e as **Regras de Segurança**. Os dados do usuário **não** ficam armazenados aqui para evitar rigidez e exposição desnecessária.
*   **Conteúdo:** Diretrizes de tom de voz, restrições bancárias e instruções de antialucinação.

#### 2. Context Window (Camada Dinâmica)
Os dados do cliente e as orientações financeiras são consultados em tempo real e injetados no prompt no momento da interação. Essa estratégia é dividida em:

*   **Dados de Saldo/Extrato:** O código Python processa o arquivo `extrato.json` e insere apenas o resumo relevante (ex: saldo atual e alertas de gastos) como contexto imediato para a IA.
*   **Recuperação via RAG:** A dica educativa mais adequada é buscada na base de conhecimento e adicionada ao prompt, servindo como a "âncora de verdade" para a resposta do mentor.

#### 3. Exemplo de Estrutura de Prompt Enviada à IA:


| Camada | Conteúdo Exemplo |
|:---|:---|
| **System** | "Você é um Mentor Financeiro do Bradesco. Use os dados abaixo para educar o usuário sem solicitar senhas." |
| **Contexto (Dados)** | "Saldo: R$ 250,00. Gasto atípico detectado: R$ 400,00 em Restaurantes." |
| **Contexto (RAG)** | "Dica: Reduzir gastos variáveis em 10% pode criar uma reserva de emergência em 6 meses." |
| **User Query** | "Como está a minha conta hoje?" |

---
**Vantagem Técnica:** Esta abordagem de **Context Injection** reduz o consumo de tokens, evita que a IA confunda instruções com dados reais e permite que o agente responda de forma personalizada a cada alteração no extrato do usuário.


---

## Exemplo de Contexto Montado

> Mostre um exemplo de como os dados são formatados para o agente.

```
### [CONTEXTO DE ENTRADA DO AGENTE FINANCEIRO]

# PERFIL DO USUÁRIO (Dados do perfil.json)
- Cliente: João Silva
- Perfil de Investidor: Moderado
- Saldo Disponível: R$ 5.000,00

# HISTÓRICO RECENTE (Processado do extrato.json)
- 01/11: Supermercado (Alimentação) - R$ 450,00
- 03/11: Streaming (Lazer/Assinatura) - R$ 55,00

# ANÁLISE DE GATILHOS (Lógica Python/Pandas)
- Status de Saldo: Saudável (Positivo).
- Oportunidade: O saldo de R$ 5.000,00 está parado em conta corrente.
- Alerta: Perfil Moderado indica abertura para investimentos de baixo/médio risco.

# DICA EDUCATIVA RECUPERADA (Via RAG)
"O perfil Moderado busca o equilíbrio entre segurança e rentabilidade. Manter grandes quantias apenas em conta corrente resulta em perda de poder de compra pela inflação. Considere opções como CDBs com liquidez diária ou Fundos de Renda Fixa."

# CONSULTA DO USUÁRIO
"Como está o status da minha conta hoje?"

```
