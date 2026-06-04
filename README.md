# ainfera-pydantic

**[Pydantic AI](https://pydantic.dev/docs/ai) + [Ainfera Routing](https://ainfera.ai).** Typed agent outputs, routed through the gateway, signed audit per turn.

## Install

```bash
pip install ainfera-pydantic
```

## Quickstart (typed result through Ainfera)

```bash
export AINFERA_API_KEY=ainfera_...  # https://app.ainfera.ai/signup
```

```python
import asyncio
from pydantic import BaseModel
from pydantic_ai import Agent
from ainfera_pydantic import ainfera_model

class City(BaseModel):
    name: str
    country: str
    population_millions: float

async def main() -> None:
    agent = Agent(
        model=ainfera_model(),  # → ainfera-inference (routing engine picks backend)
        result_type=City,
        system_prompt="Extract the city the user asks about.",
    )
    result = await agent.run("Tell me about Singapore.")
    print(result.data)
    # City(name='Singapore', country='Singapore', population_millions=5.9)

asyncio.run(main())
```

That's the whole story. Pydantic AI's typed-output discipline + Ainfera's routing engine + a signed receipt per turn.

## Model choice

| Model arg | What it does |
|---|---|
| `ainfera_model()` (default) | `model="ainfera-inference"` — routing engine picks the backend per call. Recommended; exercises the moat. |
| `ainfera_model(model="claude-opus-4-7")` | Pin Anthropic Claude Opus 4.7 |
| `ainfera_model(model="gpt-5-5")` | Pin OpenAI GPT-5.5 |
| `ainfera_model(model="gemini-3-1-pro")` | Pin Google Gemini 3.1 Pro |
| `ainfera_model(model="grok-4")` | Pin xAI Grok 4 |
| `ainfera_model(model="mistral-large-3")` | Pin Mistral Large 3 |

Live catalog: `GET https://api.ainfera.ai/v1/models`.

## Configuration

| Env var | Default | Required |
|---|---|---|
| `AINFERA_API_KEY` | (none) | **Yes** — must start with `ainfera_` |
| `AINFERA_API_URL` | `https://api.ainfera.ai/v1` | No |

You can also pass `api_key=`, `base_url=`, `model=` directly to `ainfera_model(...)`. Explicit args win.

## Why route through Ainfera?

- **Typed outputs + signed audit.** Pydantic AI guarantees the result shape; Ainfera guarantees the result lineage. Every call is hash-chained + verifiable.
- **One billing surface, all backends.** Pre-paid wallet model; per-call cost-plus margin; switching backends is a model-string change.
- **Routing-decision moat.** Each call adds to `q_empirical` — the proprietary signal that makes the routing better over time.

## Streaming + tool-use

Both pass through. Pydantic AI's `Agent.run_stream(...)` + tool-calling work end-to-end via the routed model — the Ainfera gateway forwards both shapes to the picked backend.

## What this adapter is NOT

- Not a bypass — every call goes through the gateway.
- Not a payment system — billing lives on the Ainfera side.
- Not a wrapper around Pydantic AI's whole surface — you use Pydantic AI's `Agent` exactly as you would normally; the model is what we configure for you.

## Family

This is one of the Ainfera framework adapters. All share the same `AINFERA_API_KEY` + `AINFERA_API_URL` contract:

- [ainfera-openai-compatible](https://github.com/ainfera-ai/ainfera-openai-compatible)
- [ainfera-langchain](https://github.com/ainfera-ai/ainfera-langchain)
- [ainfera-langgraph](https://github.com/ainfera-ai/ainfera-langgraph)
- [ainfera-crewai](https://github.com/ainfera-ai/ainfera-crewai)
- [ainfera-llamaindex](https://github.com/ainfera-ai/ainfera-llamaindex)
- [ainfera-letta](https://github.com/ainfera-ai/ainfera-letta)
- [ainfera-google-adk](https://github.com/ainfera-ai/ainfera-google-adk)
- [ainfera-hermes](https://github.com/ainfera-ai/ainfera-hermes)
- [ainfera-mcp](https://github.com/ainfera-ai/ainfera-mcp)
- [ainfera-openclaw](https://github.com/ainfera-ai/ainfera-openclaw)
- [ainfera-microsoft-agent-framework](https://github.com/ainfera-ai/ainfera-microsoft-agent-framework)
- **ainfera-pydantic** (this repo)

## License

Apache 2.0.
