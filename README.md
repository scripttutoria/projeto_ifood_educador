# 🧠 Explicação do Script `projeto_do_chat_ifood_educador_final.ipynb`

O notebook é um **Tutor Inteligente de PDFs com modo Anki**, feito em **Streamlit** e rodando no Google Colab, com LLM da **Groq** e busca semântica via **FAISS + HuggingFace Embeddings**.

## 📦 Estrutura em 3 células

### Célula 1 — Instalação de dependências
Instala bibliotecas-chave:
- `streamlit` → interface web
- `langchain-groq` / `langchain-core` → integração LLM Groq
- `langchain-huggingface` + `sentence-transformers` → embeddings locais
- `langchain-community` + `faiss-cpu` → vectorstore
- `pypdf` → leitura de PDFs
- `python-dotenv` → variáveis de ambiente

> O warning do `requests` é apenas conflito de versões do Colab — não impede nada.

---

### Célula 2 — Cria o `app.py` (o aplicativo em si)

#### 🔹 Configuração e estado
- Define tema, ícone 🧠 e layout wide.
- Usa `st.session_state` para guardar: API Key, histórico, retriever, arquivos ativos, modo Anki e flashcards.

#### 🔹 Funções auxiliares
| Função | O que faz |
|---|---|
| `get_embeddings()` | Carrega `all-MiniLM-L6-v2` (cacheada) |
| `get_available_models()` | Lista modelos **ativos na conta Groq** do usuário (filtra whisper/tts/guard), priorizando `openai/gpt-oss-120b` |
| `load_llm()` | Cria instância do `ChatGroq` com modelo/temperatura escolhidos |
| `clean_thinking()` | Remove blocos `</think>` de modelos reasoning |
| `process_pdfs()` | Salva PDFs em temp, divide em chunks (1000/200), gera embeddings e FAISS retriever (k=5). Guarda `full_text` com os 25 primeiros chunks |
| `generate_flashcards()` | Prompt que gera flashcards em **JSON** e faz parsing com regex |

#### 🔹 Barra lateral
- Campo da Groq API Key (mostra ✅/⚠️)
- Selectbox dinâmico de modelos + slider de temperatura + slider de qtd. de flashcards
- Upload **múltiplo** de PDFs → reprocessa ao detectar mudança de arquivo
- Botões: 🧹 Limpar histórico / 🗂️ Abrir Anki

#### 🔹 Área principal — dois modos

**1) Modo Chat (RAG):**
- Exibe histórico de mensagens
- Se o usuário digitar gatilhos como `"anki"`, `"flashcards"`, `"gerar perguntas"`, `"quiz"` → gera flashcards e entra no Anki
- Caso contrário → faz RAG: busca os 5 chunks mais relevantes + histórico recente → prompt do "tutor especialista" → resposta

**2) Modo Anki:**
- Mostra a pergunta, botão 👁️ Ver Resposta (revela em verde)
- Navegação ⬅️ Anterior / Próximo ➡️
- Botões de avaliação **Errei / Bom / Fácil** (Errei recoloca o card no fim da fila — spaced repetition básico)
- Botão ❌ Sair do Anki volta ao chat

---

### Célula 3 — Deploy público via Cloudflare Tunnel
- `pkill streamlit` (reinicia)
- Instala `cloudflared`
- Sobe o Streamlit na porta 8501
- Abre túnel público (`trycloudflare.com`) para acessar o app **fora do Colab**

> ⚠️ O link gerado expira ao reiniciar a célula — é só para demonstração.

---

## 🎯 Resumo em 1 frase
> Um app Streamlit que **transforma qualquer PDF em um tutor conversacional** (RAG com Groq + FAISS) e que, a pedido do usuário, **gera flashcards interativos estilo Anki** a partir do conteúdo estudado — tudo hospedado no Colab com URL pública temporária.

---

# 📝 Descrição do Repositório (GitHub — "About")

Cole no campo **Description** do repositório:

> 🧠 Tutor inteligente de PDFs com IA (Groq + LangChain + FAISS) e modo Flashcards estilo Anki. Feito com Streamlit e executável no Google Colab.

**Topics sugeridos:**
`streamlit` `langchain` `groq` `faiss` `rag` `flashcards` `anki` `python` `educacao` `pdf` `llm` `huggingface`


---

## ✨ Funcionalidades

- 📄 **Upload de múltiplos PDFs** — o app vira "especialista" no material enviado
- 💬 **Chat RAG** (Retrieval-Augmented Generation) — respostas baseadas **exclusivamente** no conteúdo dos PDFs
- 🗂️ **Modo Anki** — gera flashcards automaticamente com botões de avaliação (Errei / Bom / Fácil)
- 🤖 **Seleção dinâmica de modelos Groq** — lista todos os modelos ativos na sua conta
- 🔍 **Busca semântica** com FAISS + embeddings HuggingFace (`all-MiniLM-L6-v2`)
- 🌐 **URL pública temporária** via Cloudflare Tunnel (sem precisar de conta)
- 🎯 Ativação do Anki por gatilho de texto: `anki`, `flashcards`, `gerar perguntas`, `quiz`, etc.

---

## 🚀 Como executar

### Opção 1 — Google Colab (mais simples)

1. Abra o notebook [`projeto_do_chat_ifood_educador_final.ipynb`](projeto_do_chat_ifood_educador_final.ipynb) no Colab
2. Execute a **célula 1** (instala dependências)
3. Execute a **célula 2** (cria o `app.py`)
4. Execute a **célula 3** (sobe o servidor + Cloudflare Tunnel)
5. Copie a URL `https://xxxxx.trycloudflare.com` que aparece no output e acesse no navegador
6. Cole sua **Groq API Key** na barra lateral


## 🔑 Obtendo uma Groq API Key

1. Acesse [console.groq.com/keys](https://console.groq.com/keys)
2. Crie uma conta gratuita
3. Gere uma nova chave (começa com `gsk_...`)
4. Cole no campo **Groq API Key** na barra lateral do app

> O app **não armazena** sua chave — ela fica apenas na `session_state` do Streamlit.

---

## 🧩 Como usar

### 1️⃣ Carregar o material
- Barra lateral → **📂 Arquivos PDF** → faça upload de 1 ou mais PDFs
- O app processa, indexa e avisa: *"Arquivo carregado: ... Agora sou especialista neste material!"*

### 2️⃣ Conversar (modo Chat)
Digite qualquer pergunta no campo inferior, por exemplo:
- *"Resuma o capítulo 3"*
- *"Explique o conceito de X"*
- *"Quais são as etapas do processo Y?"*

### 3️⃣ Estudar com Anki
Digite no chat (ou clique em **🗂️ Anki** na sidebar):
- `gerar perguntas`
- `flashcards`
- `anki`
- `quiz`

Será aberto o **Modo Anki** com cartões de pergunta/resposta e botões:
- 🔴 **Errei** → reenvia o cartão para o final da fila
- 🟡 **Bom** → avança
- 🟢 **Fácil** → avança

---

## 🏗️ Arquitetura


       ┌──────────────┐
       │  PDF (upload)│
       └──────┬───────┘
              │  PyPDFLoader
              ▼
       ┌──────────────────┐
       │ Recursive Splitter│   chunk=1000 / overlap=200
       └──────┬───────────┘
              │
              ▼
       ┌──────────────────────────┐
       │ HuggingFace Embeddings   │   all-MiniLM-L6-v2
       └──────┬───────────────────┘
              │
              ▼
       ┌───────────┐        ┌─────────────┐
       │  FAISS    │◄──────►│  Retriever  │ k=5
       └───────────┘        └──────┬──────┘
                                   │
                                   ▼
                        ┌────────────────────┐
                        │  ChatGroq (LLM)    │
                        │  + Prompt RAG      │
                        └────────────────────┘
                                   │
                                   ▼
                          ┌─────────────────┐
                          │ Resposta/Cards  │
                          └─────────────────┘


---

## 🛠️ Stack

| Camada | Tecnologia |
|---|---|
| Interface | Streamlit |
| Orquestração | LangChain |
| LLM | Groq (Llama 3, GPT-OSS, etc.) |
| Embeddings | HuggingFace `all-MiniLM-L6-v2` |
| Vector Store | FAISS (CPU) |
| Leitura de PDF | pypdf |
| Tunelamento | Cloudflare Tunnel |
| Ambiente | Google Colab |

---

## ⚙️ Configurações disponíveis

| Parâmetro | Onde fica | Padrão |
|---|---|---|
| Modelo LLM | Sidebar | Prioriza `openai/gpt-oss-120b` |
| Temperatura | Sidebar | 0.4 |
| Qtd. de flashcards | Sidebar | 5 (mín. 3 / máx. 15) |
| Chunk size | Código | 1000 |
| Chunk overlap | Código | 200 |
| Chunks do `full_text` | Código | 25 primeiros |
| Retriever `k` | Código | 5 |

---

## 📋 Requisitos

- Python 3.10+
- Groq API Key (gratuita)
- ~2 GB de RAM (para embeddings)
- Conexão com internet

---

## 📁 Estrutura do repositório

```
projeto_ifood_educador/
├── projeto_do_chat_ifood_educador_final.ipynb   # Notebook principal
├── app.py                                       # Gerado pela célula 2 (opcional)
├── README.md
└── .env.example                                 # Exemplo de variáveis
```

---

## 🐛 Solução de problemas

| Problema | Solução |
|---|---|
| `ERROR: pip's dependency resolver... requests` | Aviso do Colab, **pode ignorar** |
| `GROQ_API_KEY` não encontrada | Insira a chave na sidebar (não precisa de `.env` no Colab) |
| URL do Cloudflare não abre | Aguarde ~10s; recarregue; verifique se o Streamlit subiu |
| Erro ao gerar flashcards | Aumente a temperatura ou diminua a qtd. de cartões |
| Modelo não aparece | Clique em "Limpar" e recarregue a página (cache de 5 min) |


---

## 🙌 Créditos

 [@scripttutoria](https://github.com/scripttutoria)
```

