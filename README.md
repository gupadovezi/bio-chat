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

O PDF da diretriz não entra neste repositório. O caminho já está na seção Configuration de cada arquivo.

## Como executar

Na seção Configuration, troque o placeholder pela sua chave:

```python
API_KEY = "set your mistral API key here"
```

Crie a chave em [console.mistral.ai](https://console.mistral.ai). A chave antiga, que estava no código, deve ser revogada.

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
python llm_chatbot_mistral_BioBERT.py
```

Para sair do loop, digite `exit`.

```bash
jupyter notebook notebooks/Answers_15_09_patients.ipynb
```

A primeira execução baixa o PubMedBERT e embute todos os trechos do PDF antes de aceitar perguntas.
