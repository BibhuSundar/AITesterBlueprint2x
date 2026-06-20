## Switchable judge LLMs

Set `JUDGE_PROVIDER` to one of:

- **`openai`** → uses `OPENAI_API_KEY` and `gpt-4o-mini` (override with `JUDGE_MODEL_OPENAI`)
- **`groq`** → uses `GROQ_API_KEY` and `openai/gpt-oss-120b` (override with `JUDGE_MODEL_GROQ`)
- **`ollama`** → uses local Ollama at `http://localhost:11434/v1` and `gpt-oss:20b` (override with `JUDGE_MODEL_OLLAMA`)

The same `CompatibleJudge` class works for all three because OpenAI, Groq, and Ollama all expose an OpenAI-compatible endpoint. `instructor` handles structured-output schema extraction uniformly.


## Quick start

```bash
cd 03_deepeval_framework
pip install -r requirements.txt

# Pick judge
export JUDGE_PROVIDER=groq
export GROQ_API_KEY=gsk_...

# (or OpenAI)
# export JUDGE_PROVIDER=openai
# export OPENAI_API_KEY=sk-...

# (or fully local)
# export JUDGE_PROVIDER=ollama

# Make sure Subsystem A and B are running
# (see their READMEs)

python run_all.py
open reports/report.html
```

## File map

```
03_deepeval_framework/
├── run_all.py              one-command runner
├── pytest.ini              markers + html report config
├── conftest.py             fixtures: judge, chatbot, rag, goldens
├── llm_providers/
│   ├── base.py             CompatibleJudge (works for OpenAI/Groq/Ollama)
│   └── factory.py          reads JUDGE_PROVIDER and builds the judge
├── targets/
│   ├── chatbot.py          HTTP client → Subsystem A
│   └── rag_pipeline.py     HTTP client → Subsystem B
├── datasets/
│   ├── chatbot_goldens.py  8 golden Q&A + 5 adversarial safety prompts
│   └── rag_goldens.py      8 golden Q&A with expected sources/keywords
└── tests/
    └── test_NN_*.py        one file per metric (20 files)
```

## Scoring conventions

| Direction | Metrics |
|-----------|---------|
| Higher is better (threshold = floor) | answer relevancy, faithfulness, contextual precision/recall/relevancy, summarization, all G-Evals, conversation relevancy |
| Lower is better (threshold = ceiling) | hallucination, bias, toxicity |

DeepEval handles the inversion automatically; just use `is_successful()`.