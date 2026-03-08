# NeuroAgent

Framework moderno, modular e simples para criar agentes de IA com poucas linhas de codigo.

Inspirado em LangChain, AutoGPT e CrewAI, com foco em integracao facil com sites.

## Arquitetura

```text
neuroagent/
  agents/
  memory/
  tools/
  planner/
  llm/
  team/
  server/
  cli/
  frontend/
  utils/
examples/
requirements.txt
```

## Instalação

```bash
python -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
```

## Criar agentes

```python
from neuroagent import Agent

agent = Agent(
    name="DevAgent",
    goal="Criar aplicacoes web modernas"
)

agent.add_tool("web_search")
agent.add_tool("code_executor")

print(agent.run("Crie um site moderno em React"))
```

## Criar tools personalizadas

```python
from neuroagent import Agent, tool

@tool()
def send_email(to: str, subject: str, body: str):
    return {"ok": True, "to": to, "subject": subject}

agent = Agent(name="SupportAgent", goal="Responder usuarios")
agent.add_tool(send_email)
print(agent.use_tool("send_email", to="user@example.com", subject="Oi", body="Mensagem"))
```

## Memoria

```python
agent.memory.save("Usuario prefere React")
print(agent.memory.short.latest())
print(agent.memory.long.search("React"))
print(agent.memory.vector.search("frontend em React"))
```

## Planejamento automatico

```python
steps = agent.plan("Criar um site com frontend, backend e deploy")
print(steps)
```

## Multi-agentes

```python
from neuroagent import Team

team = Team()
team.add_agent("ResearchAgent")
team.add_agent("DevAgent")
team.add_agent("DeployAgent")

print(team.run("Crie um SaaS completo"))
```

## Providers de modelo (LLMProvider)

```python
from neuroagent import Agent
from neuroagent.llm import OpenAIProvider, LocalModelProvider

openai_agent = Agent(
    name="CoderAI",
    goal="Criar aplicacoes completas",
    provider=OpenAIProvider(model="gpt-4o-mini")
)

local_agent = Agent(
    name="LocalAgent",
    goal="Responder com modelo local",
    provider=LocalModelProvider(model="llama3")
)
```

## API FastAPI

Subir servidor:

```bash
set NEUROAGENT_CORS_ORIGINS=http://localhost:3000,https://meusite.com
python -m neuroagent.cli.neuroagent_cli start-server --host 0.0.0.0 --port 8000
```

`NEUROAGENT_CORS_ORIGINS` permite liberar o domínio do site do usuário para chamadas do navegador.

Endpoint principal:

- `POST /agent/run`

Body:

```json
{
  "agent": "DevAgent",
  "message": "Crie uma landing page"
}
```

Resposta:

```json
{
  "agent": "DevAgent",
  "response": "..."
}
```

Endpoints adicionais:

- `GET /health`
- `GET /agents`
- `POST /agent/register`
- `GET /widget.js`

## WebSocket tempo real

Conectar em:

- `ws://localhost:8000/ws/{agent_name}`

Payload enviado:

```json
{
  "message": "Oi agente"
}
```

## Conectar o agente de IA com o site do usuario

### 1) Suba o servidor e registre o agente

```bash
set NEUROAGENT_CORS_ORIGINS=http://localhost:3000,https://meusite.com
python -m neuroagent.cli.neuroagent_cli start-server --host 0.0.0.0 --port 8000
```

```bash
curl -X POST http://localhost:8000/agent/register ^
  -H "Content-Type: application/json" ^
  -d "{\"name\":\"SupportAgent\",\"goal\":\"Responder duvidas de usuarios do site\"}"
```

### 2) Integracao via fetch (REST)

```html
<script>
  async function perguntarAoAgente(message) {
    const response = await fetch("http://localhost:8000/agent/run", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        agent: "SupportAgent",
        message
      })
    });

    const data = await response.json();
    return data.response;
  }
</script>
```

### 3) Integracao via widget pronto

O servidor expõe o widget em `GET /widget.js`:

```html
<script src="http://localhost:8000/widget.js"></script>
<script>
  NeuroAgent.init({
    apiUrl: "http://localhost:8000",
    agent: "SupportAgent"
  });
</script>
```

### 4) Tempo real com WebSocket no widget

```html
<script src="http://localhost:8000/widget.js"></script>
<script>
  NeuroAgent.init({
    apiUrl: "http://localhost:8000",
    wsUrl: "ws://localhost:8000",
    agent: "SupportAgent",
    realtime: true
  });
</script>
```

## CLI

Comandos:

```bash
neuroagent init my-project
neuroagent create-agent DevAgent --goal "Criar apps web"
neuroagent run DevAgent --message "Crie uma API"
neuroagent start-server --port 8000
```

## Gerador de projetos

`neuroagent init my-project` cria:

- estrutura basica do projeto
- `neuroagent.json` com configuracao inicial
- exemplos prontos para executar

## Exemplos

- `examples/website_agent.py`
- `examples/dev_agent.py`
- `examples/multi_agent_team.py`

## Exemplo final

```python
from neuroagent import Agent

coder = Agent(
    name="CoderAI",
    goal="Criar aplicacoes completas"
)

coder.add_tool("code_executor")
coder.add_tool("file_manager")
coder.add_tool("web_search")

print(coder.run("Crie um app SaaS em React e Node"))
```
