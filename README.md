# CRUD + IA: previsão de churn com uma LLM local

Um sistema de cadastro de clientes (CRUD) que usa um modelo de linguagem local para estimar o risco de churn de cada cliente, ou seja, a chance de ele deixar de usar um serviço. Projeto acadêmico da disciplina de Padrões e Arquitetura de Software do curso de Engenharia de Software da USF.

O objetivo era praticar separação de responsabilidades e integrar uma LLM a um sistema tradicional, rodando tudo localmente, sem chave de API e sem custo.

## Como funciona

```text
SistemaCRUDIA (ponto de entrada)
   ├──► RepositorioCliente   cria, lê, atualiza, remove e lista clientes
   └──► ChurnIA              envia o score de atividade do cliente para a LLM
                               └──► Ollama (mistral) em localhost:11434
```

1. O cliente é cadastrado com um score de atividade de 0 a 100.
2. A `ChurnIA` monta um prompt e pede ao modelo uma resposta em JSON com a probabilidade de churn e o nível de risco (ALTO, MÉDIO ou BAIXO).
3. A resposta é interpretada e validada, e o resultado volta para o cliente pelo repositório, junto com o tempo de resposta.

## Decisões de projeto

- **Padrão Repository:** o `RepositorioCliente` esconde como os clientes são guardados. Hoje é um dicionário em memória; poderia virar um banco sem mudar o resto do código.
- **IA como serviço separado:** a `ChurnIA` concentra tudo que é do modelo (URL, nome do modelo, prompt, interpretação), então dá para trocar de provedor em um lugar só.
- **Ponto de entrada único:** o `SistemaCRUDIA` junta os dois e expõe operações simples, mantendo o fluxo de negócio numa classe só.
- **Leitura defensiva:** LLMs nem sempre devolvem um JSON limpo, então o código extrai o bloco de JSON da resposta e falha com uma mensagem clara quando ele é inválido.
- **Checagem na inicialização:** o sistema verifica se o Ollama está rodando antes de começar e explica como resolver se não estiver.

## Stack

Python, Requests, Ollama (modelo `mistral`).

## Como rodar

Requisitos: Python 3.10+ e [Ollama](https://ollama.com).

```bash
# terminal 1: sobe o modelo
ollama run mistral

# terminal 2: roda o projeto
git clone https://github.com/Feduzo/crud-ai-churn.git
cd crud-ai-churn
pip install requests
python crud_ia_ollama_final.py
```

Rodar o arquivo executa três testes: o CRUD sozinho, a IA sozinha e a integração completa.

## Limitações e próximos passos

Nesta versão, o prompt passa ao modelo regras fixas (score abaixo de 30 é risco alto, e assim por diante). Isso deixou os resultados previsíveis para o trabalho, mas regras simples assim seriam mais rápidas e baratas como código comum. O próximo passo natural é usar o modelo no que regra nenhuma resolve, por exemplo analisar feedbacks e chamados de suporte em texto livre para explicar por que um cliente pode ir embora.

Outras melhorias: guardar os clientes em SQLite e passar os testes para o pytest.

## Licença

[MIT](LICENSE)
