# Chatbot SOSORT (Mistral + PubMedBERT)

Perguntas e respostas sobre a diretriz SOSORT. O PDF é dividido em trechos, os trechos são vetorizados com PubMedBERT (`microsoft/BiomedNLP-PubMedBERT-base-uncased-abstract`) e a pergunta segue para o modelo `mistral-medium`, junto com os três trechos mais parecidos.

Isto é um protótipo de pesquisa. As respostas do modelo não substituem julgamento clínico.

## O que está no repositório

| Arquivo | Conteúdo |
| --- | --- |
| `llm_chatbot_mistral_BioBERT.py` | Script interativo no terminal |
| `notebooks/Answers_15_09_patients.ipynb` | Sessão de 15/09 para pacientes: 25 respostas do Mistral, com o trecho recuperado antes de cada uma |
| `notebooks/professionals_questions_answers_15_09.ipynb` | Sessão para profissionais |
| `notebooks/answer_16_09_professionals.ipynb` | A mesma sessão de profissionais, salva de novo |

Os dois notebooks de profissionais são idênticos. A sessão começa com um erro 429 da API do Mistral (`service_tier_capacity_exceeded`) e segue com 20 respostas. O texto digitado em cada pergunta não ficou gravado: o `input()` do notebook só registrou a saída do modelo e a prévia dos trechos.

O PDF da diretriz **não** entra neste repositório. Apontar `PDF_PATH` para a **sua** cópia local; os caminhos absolutos já gravados no código são pessoais e não servem de setup.

## Como uma pergunta é respondida

O script e os três notebooks usam o mesmo pipeline:

1. `pdfplumber` extrai o texto de `PDF_PATH`.
2. `smart_chunk_text` parte o texto nas fronteiras de sentença (`[.!?]`) e junta trechos de cerca de `CHUNK_SIZE = 500` caracteres.
3. Na inicialização, PubMedBERT (`microsoft/BiomedNLP-PubMedBERT-base-uncased-abstract`) via `sentence-transformers` (Transformer + mean pooling) embute cada trecho.
4. Cada pergunta é embutida da mesma forma; `retrieve_relevant_chunks` devolve os `top_k = 3` trechos por similaridade de cosseno.
5. O contexto concatenado é cortado em `MAX_CONTEXT_LENGTH = 4000` caracteres.
6. `POST https://api.mistral.ai/v1/chat/completions` com `mistral-medium`, `temperature 0.7`, `max_tokens 1024`, `top_p 0.95`, `stream false`.
7. Prompts de sistema: `"You are a helpful AI assistant. Use the following context to answer the user's question."` e `"Context from PDF:\n{context}"`.
8. Imprime uma prévia de uma frase de cada trecho recuperado e, em seguida, a resposta do Mistral.

Se `extract_pdf_text` falhar ou devolver string vazia, o processo imprime `Failed to extract text from PDF` / `No text extracted. Exiting.` e encerra.

## Configuração

Há dois valores hardcoded na seção Configuration. O código **não** lê variáveis de ambiente (não existe `MISTRAL_API_KEY`).

```python
API_KEY = "set your mistral API key here"
PDF_PATH = "/caminho/para/a/sua/copia/SOSORT_guideline.pdf"
```

| Constante | O que fazer |
| --- | --- |
| `API_KEY` | Substitua o placeholder pela sua chave. Crie em [console.mistral.ai](https://console.mistral.ai). A chave antiga, que estava no código, deve ser revogada. |
| `PDF_PATH` | Aponte para a sua cópia local da diretriz SOSORT. O PDF não está no repositório. |

Armadilha: script e notebooks já vêm com caminhos absolutos de macOS **diferentes** e pessoais. Não use esses caminhos; ajuste para o PDF na sua máquina.

| Arquivo | `PDF_PATH` atual (não usar como setup) |
| --- | --- |
| `llm_chatbot_mistral_BioBERT.py` | `Desktop/ALL CONTENT/SOSORT_guideline.pdf` |
| os três notebooks | `Desktop/SOSORT_guideline.pdf` (sem `ALL CONTENT`) |

## Como executar

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
python llm_chatbot_mistral_BioBERT.py
```

Para sair do loop, digite `exit`, `quit` ou `q` (sem diferenciar maiúsculas).

```bash
jupyter notebook notebooks/Answers_15_09_patients.ipynb
```

`jupyter` **não** está em `requirements.txt`. Instale à parte se for abrir os notebooks (`pip install jupyter`).

`requirements.txt` instala `requests`, `numpy`, `pdfplumber`, `sentence-transformers` e `torch`. Se `sentence-transformers` estiver ausente, o fallback de import também faz `pip install transformers`.

A primeira execução baixa o PubMedBERT e embute todos os trechos do PDF antes de aceitar perguntas.

## Limites do pipeline

| Parâmetro | Valor no código |
| --- | --- |
| Tamanho do trecho | `CHUNK_SIZE = 500` caracteres |
| Recuperação | `top_k = 3` por similaridade de cosseno |
| Teto do contexto | `MAX_CONTEXT_LENGTH = 4000` caracteres |
| Modelo | `mistral-medium` |
| Geração | `temperature 0.7`, `max_tokens 1024`, `top_p 0.95`, `stream false` |

Custo de startup: download do PubMedBERT (primeira vez) + embedding de **todos** os trechos do PDF antes do loop de perguntas.

## Problemas comuns

| Sintoma | Causa / o que fazer |
| --- | --- |
| `Failed to extract text from PDF` ou `No text extracted. Exiting.` | `PDF_PATH` errado, ausente ou PDF sem texto extraível. Ajuste o caminho para a sua cópia. |
| `Error during LLM request` | `API_KEY` ainda é o placeholder. Crie uma chave em [console.mistral.ai](https://console.mistral.ai) e revogue qualquer chave antiga que tenha sido commitada. |
| HTTP 429 `service_tier_capacity_exceeded` | Já observado nos notebooks de profissionais; a API do Mistral recusou a requisição por capacidade. Tente de novo mais tarde. |
| Demora antes do primeiro prompt | Normal: download do PubMedBERT + embedding de todos os trechos. |
| Script acha o PDF e o notebook não (ou o contrário) | `PDF_PATH` padrão do script inclui `ALL CONTENT`; o dos notebooks não. |
| `jupyter: command not found` | `jupyter` não está em `requirements.txt`. Instale com `pip install jupyter`. |
