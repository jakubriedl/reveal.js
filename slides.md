# Building AI products 101

---

## I'm Jakub and I build them at

<div class="flex items-center justify-center px-20">
    <div class="w-1/2 text-center"><img src="/images/cultureamp.png" class="w-72 inline-block"></div>
    <div class="w-1/2 text-center"><img src="https://www.appear.sh/Appear%20white%20logo.png" class="w-52 inline-block"></div>
</div>

---

![Skeuomorphism](https://miro.medium.com/v2/resize:fit:4800/format:webp/0*7ckvHZgH4ITuSglb)

Skeuomorphism

---

- Temperature
- Tools
- USBs
- Agents
- Teams of Agents with Team leads
  - Crews with Managers (CrewAI)
- Knowledge
- Memory (Short and Long term)
- Reasoning

---

# LLMs

Large Language Models

---

## LLM = black box 3rd party API

to call with a "natural language" request/instructions

non-deterministic

always stateless

---

## LLM = 3rd party API

```ts
import OpenAI from "openai";
const openai = new OpenAI(/* API key comes here */);

async function callLLM() {
  const response = await openai.responses.create({
    model: "gpt-4.1",
    input: [
      // instructions
      { role: "developer", content: "You are a helpful assistant." },
      // user input
      { role: "user", content: "Hello!" },
    ],
    temperature: 0.6, // = randomness, optional, defaults to 1
  });
  console.log(response.output_text);
  // => Hello! How can I assist you today?
}
```

<!-- .element: class="h-full *:max-h-full! mt-0!" -->

---

## Prompt Engineering

🪦 2022 - 2024

A pseudo-science to write good input to LLM

---

## Context Engineering = putting the right data into LLM input

not too much, not too little

---

![Context Engineering](https://www.philschmid.de/static/blog/context-engineering/context.png)

---

![Information overload](/images/information-overload.png)

---

## Prompt management = strings versioning

have you heard about git?

---

## RAG = a database query you run before calling LLM

eager data loading

---

## Tools = functions that the LLM can tell you to run

lazy data loading + actions

---

<!-- .element: class="top-0! h-full" -->

```js
const tools = [
  {
    type: "function",
    name: "get_weather",
    // documentation for the LLM to know what it does
    description: "Get current temperature for a given location.",
    // json schema of parameters
    parameters: {
      type: "object",
      properties: {
        location: {
          type: "string",
          description: "City and country e.g. Bogotá, Colombia",
        },
      },
      required: ["location"],
      additionalProperties: false,
    },
  },
];

const response = await openai.responses.create({
  model: "gpt-4.1",
  input: [
    { role: "user", content: "What is the weather like in Paris today?" },
  ],
  tools,
});

console.log(response.output);
[
  {
    type: "function_call",
    id: "fc_12345xyz",
    call_id: "call_12345xyz",
    name: "get_weather",
    arguments: '{"location":"Paris, France"}',
  },
];

const response = await openai.responses.create({
  model: "gpt-4.1",
  input: [
    { role: "user", content: "What is the weather like in Paris today?" },
    {
      role: "tool_message",
      tool_call_id: "call_12345xyz",
      content: '{"condition": "sunny", "temperature": 20}',
    },
  ],
  tools,
});
```

<!-- .element: class="h-full *:max-h-full! mt-0!" -->

---

## MCP = standardized function interface

it's not an USB

---

## MCP Client

the code that got response from LLM to call tool

---

## MCP Server

the function that gets called

---

## Remote MCP = JSON-RPC API

```json
{
  "jsonrpc": "2.0",
  "id": "fc_12345xyz",
  "method": "get_weather",
  "params": { "location": "Paris, France" }
}
```

```json
{
  "jsonrpc": "2.0",
  "id": "fc_12345xyz",
  "result": { "weather": "sunny", "temperature": "20" }
}
```

---

## RAG = a database query you run before calling LLM

---

## Embeddings = Vectors = data index

that allows to search using semantic similarity

---

## Vector DB = a database

use postgres anyway

---

## Semantic similarity

pants = trousers

---

```js
// 1. convert user input `pants` into embedding `[3,1,2]` using embedding model
const embedding = await openai.embeddings.create({
  model: "text-embedding-ada-002",
  input: "Do I have pants in my wardrobe?",
});
```

```sql
-- 2. Query database to find 5 most similar items
SELECT * FROM items ORDER BY embedding <-> '[3,1,2]' LIMIT 5;
```

```js
// 3. pass the result of the query into the prompt
const values = rows.flatMap((r) => r.text);
const response = await openai.responses.create({
  model: "gpt-4.1",
  input: [
    {
      role: "user",
      content: `
        You're assistant helping with wardrobe.
        <WardrobeContent>
            ${rows.flatMap((r) => r.text).join("\n")}
        </WardrobeContent>`,
    },
    { role: "user", content: "Do I have trousers in my wardrobe?" },
  ],
});
```

---

## Hybrid search = multiple where statements

combine the similarity with full-text

---

## Evaluations = Tests

is it accurate

does it recall information we passed in

LLM-as-a-judge

---

## Security = same old

don't trust user inputs

---

breathe in ... breathe out

🧘

---

# Building AI products 102

---

## One-shot vs Multi-Shot

are you calling the API just once? or multiple times

---

## Agent

let LLM calls to affect control flow

---

## Agent in practice

A composable abstraction with a purpose

```python
# Agno framework example
agent = Agent(
    model=OpenAIChat(model="gpt-4o-mini"),
    description="You're a weather reporter",
    tools=[get_weather],
    knowledge=knowledge_base, # includes historical data about weather patterns
    memory=memory, # from memory knows we're in Melbourne
    response_model=ResponseDataStructure, # a data structure we can map to a UI component
    use_json_mode=True,
)
agent.print_response("Is it really cold today?", stream=True)
```

---

## Team/Crew = group of agents

```python
team = Team(members=[
    Agent(name="Agent 1", role="You answer questions in English"),
    Agent(name="Agent 2", role="You answer questions in Chinese"),
    Team(name="Team 1", role="You answer questions in French"),
])
team.print_response("Hey how are you?")
```

---

## Team lead/manager = entrypoint agent

```python
team = Team(
    name="Multi Language Team",
    mode="route",
    model=OpenAIChat("gpt-4.5-preview"),
    members=[
        english_agent,
        spanish_agent,
        japanese_agent,
    ],
    instructions=[
        "You are a language router that directs questions to the appropriate language agent.",
        "If the user asks in a language whose agent is not a team member, respond in English with:",
        "'I can only answer in the following languages: English, Spanish, Japanese, French and German. Please ask your question in one of these languages.'",
        "Always check the language of the user's input before routing to an agent.",
        "For unsupported languages like Italian, respond in English with the above message.",
    ],
    show_members_responses=True,
)
```

---

## Agent coordination = sequence of LLM calls

Route traffic to one team member = `route`

Call team members sequentially with sub-tasks = `coordinate`

Call team members in parallel and aggregate = `collaborate`

---

## Human-in-the-loop = ask for confirmation

---

## Observability is more impotant than ever

### because of non-determinisim

evaluation scores, cost tracking, waterfalls of slow calls, ...

---

# It's easier than it sounds

it's "just" engineering, have fun
