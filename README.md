# CRUD + AI: Churn Prediction with a Local LLM

A customer management system (CRUD) that uses a local language model to estimate each customer's churn risk, meaning how likely they are to stop using a service. Academic project for the Software Patterns and Architecture course of the Software Engineering program at USF.

The goal was to practice clean separation of responsibilities and to integrate an LLM into a traditional system, running it locally with no API key or cost.

## How it works

```text
SistemaCRUDIA (entry point)
   ├──► RepositorioCliente   create, read, update, delete and list customers
   └──► ChurnIA              sends the customer's activity score to the LLM
                               └──► Ollama (mistral) at localhost:11434
```

1. A customer is registered with an activity score from 0 to 100.
2. `ChurnIA` builds a prompt and asks the model for a JSON answer with the churn probability and risk level (HIGH, MEDIUM or LOW).
3. The answer is parsed and validated, and the result is saved back to the customer through the repository, along with the response time.

## Design decisions

- **Repository pattern:** `RepositorioCliente` hides how customers are stored. Today it is an in-memory dictionary; it could become a database without changing the rest of the code.
- **AI as a separate service:** `ChurnIA` holds everything related to the model (URL, model name, prompt, parsing), so the provider can be swapped in one place.
- **Single entry point:** `SistemaCRUDIA` combines both and exposes simple operations, keeping the business flow in one class.
- **Defensive parsing:** LLMs do not always return clean JSON, so the code extracts the JSON block from the answer and fails with a clear message when it is invalid.
- **Startup check:** the app verifies that Ollama is running before starting and explains how to fix it if not.

## Tech stack

Python, Requests, Ollama (`mistral` model).

## Running it

Requirements: Python 3.10+ and [Ollama](https://ollama.com).

```bash
# terminal 1: start the model
ollama run mistral

# terminal 2: run the project
git clone https://github.com/Feduzo/crud-ai-churn.git
cd crud-ai-churn
pip install requests
python crud_ia_ollama_final.py
```

Running the file executes three tests: the CRUD on its own, the AI on its own and the full integration.

## Limitations and next steps

In this version the prompt gives the model fixed thresholds (score below 30 is high risk, and so on). That kept the results predictable for the assignment, but simple rules like these would be faster and cheaper as plain code. The natural next step is to let the model do what rules cannot, for example analyzing free-text customer feedback or support tickets to explain why a customer might leave.

Other improvements: persist customers in SQLite and move the tests to pytest.

## License

[MIT](LICENSE)
