* *agents*

**Agents**
Agents aapke apps mein core building blocks hote hain. Ek agent ek large language model (LLM) hota hai, jo instructions aur tools ke saath configure kiya jata hai.

### Basic Configuration

Jab aap agent configure karte hain, to kuch common properties hoti hain jo aap configure karenge:

1. **Instructions**: Isay developer message ya system prompt bhi kaha jata hai.
2. **Model**: Ye wo LLM hota hai jo aap use karte hain, aur optional model\_settings ke through aap model tuning parameters jaise temperature, top\_p wagaira ko configure kar sakte hain.
3. **Tools**: Ye wo tools hote hain jo agent apne tasks achieve karne ke liye use karta hai.

```python
from agents import Agent, ModelSettings, function_tool

@function_tool
def get_weather(city: str) -> str:
    return f"The weather in {city} is sunny"

agent = Agent(
    name="Haiku agent",
    instructions="Always respond in haiku form",
    model="o3-mini",
    tools=[get_weather],
)
```

### Context

Agents apne context type mein generic hote hain. Context ek dependency-injection tool hota hai: Ye ek object hota hai jo aap create karte hain aur `Runner.run()` ko pass karte hain, jise har agent, tool, handoff wagaira mein pass kiya jata hai, aur ye agent run ke liye dependencies aur state ka grab bag ka kaam karta hai. Aap koi bhi Python object context ke roop mein provide kar sakte hain.

```python
@dataclass
class UserContext:
    uid: str
    is_pro_user: bool

    async def fetch_purchases() -> list[Purchase]:
        return ...

agent = Agent[UserContext](
    ...,
)
```

### Output Types

By default, agents plain text (i.e. str) output karte hain. Agar aap chahein ke agent kisi specific type ka output de, to aap `output_type` parameter use kar sakte hain. Ek common choice Pydantic objects hote hain, lekin hum koi bhi aisi type support karte hain jo Pydantic TypeAdapter mein wrap ho sakti ho - jaise dataclasses, lists, TypedDict, etc.

```python
from pydantic import BaseModel
from agents import Agent

class CalendarEvent(BaseModel):
    name: str
    date: str
    participants: list[str]

agent = Agent(
    name="Calendar extractor",
    instructions="Extract calendar events from text",
    output_type=CalendarEvent,
)
```

**Note:**
Jab aap `output_type` pass karte hain, to model ko ye batata hai ke structured outputs use kare plain text responses ke bajaye.

### Handoffs

Handoffs wo sub-agents hote hain jinhe agent delegate kar sakta hai. Aap handoffs ki ek list provide karte hain, aur agent relevant hone par unhe use kar sakta hai. Ye ek powerful pattern hai jo modular, specialized agents ko orchestrate karne ki suvidha deta hai, jo ek specific task mein expert hote hain.

```python
from agents import Agent

booking_agent = Agent(...)
refund_agent = Agent(...)

triage_agent = Agent(
    name="Triage agent",
    instructions=(
        "Help the user with their questions."
        "If they ask about booking, handoff to the booking agent."
        "If they ask about refunds, handoff to the refund agent."
    ),
    handoffs=[booking_agent, refund_agent],
)
```

### Dynamic Instructions

Zyada cases mein, aap agent banate waqt instructions provide karte hain. Lekin aap dynamic instructions bhi de sakte hain jo ek function ke zariye provide ki jaati hain. Ye function agent aur context ko receive karta hai aur prompt return karta hai. Dono regular aur async functions accepted hain.

```python
def dynamic_instructions(
    context: RunContextWrapper[UserContext], agent: Agent[UserContext]
) -> str:
    return f"The user's name is {context.context.name}. Help them with their questions."

agent = Agent[UserContext](
    name="Triage agent",
    instructions=dynamic_instructions,
)
```

### Lifecycle Events (Hooks)

Kabhi kabhi, aapko agent ke lifecycle ko observe karna hota hai. Jaise, aap events ko log karna chahein ya kisi specific event ke hone par data pre-fetch karna chahein. Aap agent lifecycle mein hooks property ke through hook kar sakte hain. `AgentHooks` class ko subclass karke, aap wo methods override kar sakte hain jo aapko chahiye.

### Guardrails

Guardrails aapko user input par checks/validations chalane ki suvidha dete hain, jo agent ke run ke parallel hoti hain. Misaal ke taur pe, aap user ke input ko relevance ke liye screen kar sakte hain.

### Cloning/Copying Agents

Agar aapko agent ka duplicate banana ho, toh aap `clone()` method ka use kar sakte hain, aur uski properties ko modify kar sakte hain.

```python
pirate_agent = Agent(
    name="Pirate",
    instructions="Write like a pirate",
    model="o3-mini",
)

robot_agent = pirate_agent.clone(
    name="Robot",
    instructions="Write like a robot",
)
```

### Forcing Tool Use

Tools ka list dena hamesha ye nahi garanti karta ke LLM tool use karega. Aap tool use karwane ke liye `ModelSettings.tool_choice` set kar sakte hain. Valid values hain:

* **auto**: LLM decide karega ke tool use karna hai ya nahi.
* **required**: LLM ko tool use karna hi padega (lekin wo intelligently decide karega ki konsa tool use karna hai).
* **none**: LLM ko tool use karne ki ijazat nahi hai.
* Specific tool ka name bhi de sakte hain jaise `my_tool`, jo LLM ko specific tool use karne ka keh raha ho.

**Note:**
Infinite loops ko rokne ke liye, framework automatically `tool_choice` ko "auto" reset kar deta hai tool call ke baad. Agar aap chahte hain ke Agent tool call ke baad puri tarah se ruk jaye (auto mode ke bajaye), to aap `[Agent.tool_use_behavior="stop_on_first_tool"]` set kar sakte hain, jo directly tool output ko final response ke taur par use karega, bina further LLM processing ke.

==========================================================================================================================================================================================

* *Running Agents*

**Running Agents**
Aap agents ko **Runner class** ke zariye run kar sakte hain. Aapke paas 3 options hain:

1. **Runner.run()**: Ye async method hota hai aur RunResult return karta hai.
2. **Runner.run\_sync()**: Ye ek sync method hai, jo internally .run() ko run karta hai.
3. **Runner.run\_streamed()**: Ye async method hota hai aur RunResultStreaming return karta hai. Ye LLM ko streaming mode mein call karta hai aur events ko stream karte hue return karta hai.

```python
from agents import Agent, Runner

async def main():
    agent = Agent(name="Assistant", instructions="You are a helpful assistant")

    result = await Runner.run(agent, "Write a haiku about recursion in programming.")
    print(result.final_output)
    # Code within the code,
    # Functions calling themselves,
    # Infinite loop's dance.
```

**Read more in the results guide.**

### The Agent Loop

Jab aap **Runner** mein `run` method use karte hain, to aap ek starting agent aur input pass karte hain. Input ek string ho sakti hai (jo user message consider kiya jata hai), ya ek list of input items ho sakti hai, jo OpenAI Responses API mein hote hain.

**Runner loop** kuch is tarah se kaam karta hai:

1. Hum current agent aur input ke saath LLM ko call karte hain.
2. LLM apna output generate karta hai.
3. Agar LLM final\_output return kare, to loop end ho jata hai aur result return hota hai.
4. Agar LLM handoff karta hai, to hum current agent aur input ko update karte hain aur loop ko dobara run karte hain.
5. Agar LLM tool calls generate karta hai, to hum un tool calls ko run karte hain, unke results ko append karte hain aur loop ko dobara run karte hain.
6. Agar hum max\_turns exceed karte hain, to **MaxTurnsExceeded exception** raise hota hai.

**Note:**
LLM output ko "final output" mana jata hai agar wo text output ke sath desired type produce kare, aur koi tool calls na ho.

### Streaming

Streaming ka matlab hai ke aap LLM ke run hone ke dauran streaming events ko bhi receive kar sakte hain. Jab stream complete ho jata hai, to **RunResultStreaming** mein poori run information hoti hai, including saare naye outputs. Aap **.stream\_events()** method use karke streaming events ko access kar sakte hain. Iske baare mein zyada details streaming guide mein hain.

### Run Config

Aap **run\_config** parameter ke zariye agent run ke liye kuch global settings configure kar sakte hain:

* **model**: Ek global LLM model set karta hai jo har agent ke model ke bajaye use hoga.
* **model\_provider**: Model provider, jo model names ko lookup karta hai (default OpenAI hota hai).
* **model\_settings**: Agent-specific settings ko override karta hai, jaise global temperature ya top\_p set karna.
* **input\_guardrails, output\_guardrails**: Input ya output guardrails ki list, jo saare runs mein include hoti hain.
* **handoff\_input\_filter**: Global input filter jo handoffs ke liye apply hota hai, agar handoff mein filter na ho.
* **tracing\_disabled**: Isse aap puri run ke liye tracing disable kar sakte hain.
* **trace\_include\_sensitive\_data**: Isse configure karte hain ke traces mein sensitive data include ho ya nahi.
* **workflow\_name, trace\_id, group\_id**: Tracing workflow ka name, trace ID aur group ID set karte hain.
* **trace\_metadata**: Saare traces mein metadata include karte hain.

### Conversations/Chat Threads

**Run methods** ko call karte waqt ek ya zyada agents run ho sakte hain (aur is tarah se ek ya zyada LLM calls). Lekin, ye ek logical turn represent karta hai ek chat conversation mein. Misaal ke taur par:

* **User turn**: User text enter karta hai.
* **Runner run**: Pehla agent LLM ko call karta hai, tools run karta hai, ek doosre agent ko handoff karta hai, aur phir output generate karta hai.

Run ke end mein, aap decide kar sakte hain ke aap user ko kya dikhayenge. Jaise, aap user ko saare naye items dikhayein jo agents ne generate kiye, ya sirf final output dikhayein. Agar user phir se follow-up question kare, to aap **run method** dobara call kar sakte hain.

Aap **RunResultBase.to\_input\_list()** method use karke agle turn ke inputs bhi le sakte hain.

```python
async def main():
    agent = Agent(name="Assistant", instructions="Reply very concisely.")

    with trace(workflow_name="Conversation", group_id=thread_id):
        # First turn
        result = await Runner.run(agent, "What city is the Golden Gate Bridge in?")
        print(result.final_output)
        # San Francisco

        # Second turn
        new_input = result.to_input_list() + [{"role": "user", "content": "What state is it in?"}]
        result = await Runner.run(agent, new_input)
        print(result.final_output)
        # California
```

### Exceptions

SDK kuch exceptions raise karta hai jab kuch issues hoti hain. Full list aapko **agents.exceptions** mein milti hai. Ek overview ke liye:

* **AgentsException**: Ye base class hai jo SDK mein sabhi exceptions ke liye hoti hai.
* **MaxTurnsExceeded**: Ye exception tab raise hoti hai jab run methods mein diye gaye **max\_turns** ko exceed kar liya jata hai.
* **ModelBehaviorError**: Ye exception tab raise hoti hai jab model invalid outputs generate karta hai, jaise malformed JSON ya non-existent tools ka use.
* **UserError**: Ye exception tab raise hoti hai jab aap (SDK ka use karte waqt) koi error karte hain.
* **InputGuardrailTripwireTriggered, OutputGuardrailTripwireTriggered**: Ye exceptions tab raise hoti hain jab guardrail trip ho jata hai.


==========================================================================================================================================================================================

* *Results*

**Results**
Jab aap **Runner.run** methods ko call karte hain, to aapko do cheezen mil sakti hain:

* **RunResult**: Agar aap **run** ya **run\_sync** call karte hain.
* **RunResultStreaming**: Agar aap **run\_streamed** call karte hain.

Dono **RunResultBase** se inherit karte hain, jahan se most useful information milti hai.

### Final Output

**final\_output** property last agent ka final output hota hai. Ye do forms mein ho sakta hai:

* Agar last agent ne **output\_type** define nahi kiya ho, to yeh ek **str** hoga.
* Agar agent ne **output\_type** define kiya ho, to yeh us type ka object hoga, jo **last\_agent.output\_type** ke corresponding hoga.

**Note:**
**final\_output** ka type **Any** hota hai, kyun ke hum statically type nahi kar sakte. Agar handoffs ho, to hum nahi jaan sakte ke final agent ka output kis type ka hoga.

### Inputs for the Next Turn

Aap **result.to\_input\_list()** method use karke result ko input list mein convert kar sakte hain, jo aapke original input ko agent run ke dauran generate hue items ke sath concatenate karta hai. Isse yeh convenient ho jata hai ke aap ek agent run ke outputs ko doosre run mein pass kar sakein, ya har turn ke baad naye user inputs append kar sakein.

### Last Agent

**last\_agent** property mein aapko last agent milta hai jo run hua tha. Aap isko next user input ke liye use kar sakte hain. Misaal ke taur par, agar aapka frontline triage agent hai jo language-specific agent ko handoff karta hai, to aap last agent ko store kar sakte hain aur next time use kar sakte hain jab user phir se agent se message kare.

### New Items

**new\_items** property mein wo naye items hote hain jo run ke dauran generate hote hain. Ye items **RunItems** hote hain. Har run item ko LLM ne generate kiya hota hai.

Types of run items:

* **MessageOutputItem**: Ye LLM ka generated message hota hai.
* **HandoffCallItem**: Ye LLM ka tool call item hota hai jab handoff tool call kiya jata hai.
* **HandoffOutputItem**: Ye wo response hota hai jo handoff tool call ke baad aata hai. Aap isme source aur target agents ko bhi access kar sakte hain.
* **ToolCallItem**: Ye wo item hota hai jab LLM ne tool invoke kiya ho.
* **ToolCallOutputItem**: Ye wo raw item hota hai jab tool call hota hai. Aap tool ka output bhi isme access kar sakte hain.
* **ReasoningItem**: Ye wo reasoning item hota hai jo LLM ne generate kiya ho.

### Other Information

**Guardrail Results**:
Agar aapne guardrails configure kiye hain, to **input\_guardrail\_results** aur **output\_guardrail\_results** properties mein unke results milte hain. Guardrail results aapko kuch useful information de sakte hain jo aap log ya store karna chahenge.

**Raw Responses**:
**raw\_responses** property mein aapko LLM se generated **ModelResponses** milte hain.

**Original Input**:
**input** property mein aapko wo original input milta hai jo aapne **run** method mein diya tha. Most cases mein aapko iski zaroorat nahi hoti, lekin agar chahein to aap isse access kar sakte hain.


==========================================================================================================================================================================================

* *Streaming*

Yeh raha tumhara diya gaya code aur uska explanation Urdu Roman mein:


---

Streaming

Streaming ka matlab hai ke aap agent ke run ke dauraan uski progress ya response updates ko live subscribe kar saktay ho. Ye end-user ko real-time progress dikhane ke liye kaafi faida mand hai.

Streaming ke liye Runner.run_streamed() call karo. Yeh aapko RunResultStreaming deta hai. Uske baad result.stream_events() se async stream milti hai jo StreamEvent objects deta hai.


---

Raw Response Events

RawResponsesStreamEvent woh events hain jo LLM se direct aate hain, OpenAI Responses API format mein. Har event ka aik type hota hai (jaise response.created, response.output_text.delta, etc.) aur data.

Yeh tab kaam aate hain jab aap chahte ho ke LLM ka text user ko token-by-token dikhaya jaye.

Code Example (LLM ka text stream karna token-by-token):

import asyncio
from openai.types.responses import ResponseTextDeltaEvent
from agents import Agent, Runner

async def main():
    agent = Agent(
        name="Joker",
        instructions="Tum aik madadgar assistant ho.",
    )

    result = Runner.run_streamed(agent, input="Mujhe 5 mazak sunao.")
    async for event in result.stream_events():
        if event.type == "raw_response_event" and isinstance(event.data, ResponseTextDeltaEvent):
            print(event.data.delta, end="", flush=True)

if _name_ == "_main_":
    asyncio.run(main())


---

Run Item Events aur Agent Events

RunItemStreamEvents thode high-level events hote hain. Ye batate hain jab koi complete item generate ho gaya ho, jaise: "message ban gaya", "tool chala", waghera.
Isi tarah AgentUpdatedStreamEvent tab hota hai jab current agent update ho (e.g. handoff ke baad).

Code Example (High-level updates dikhana):

import asyncio
import random
from agents import Agent, ItemHelpers, Runner, function_tool

@function_tool
def how_many_jokes() -> int:
    return random.randint(1, 10)

async def main():
    agent = Agent(
        name="Joker",
        instructions="Pehle how_many_jokes tool ko call karo, phir utne jokes sunao.",
        tools=[how_many_jokes],
    )

    result = Runner.run_streamed(
        agent,
        input="Hello",
    )
    print("=== Run shuru ho gaya hai ===")

    async for event in result.stream_events():
        if event.type == "raw_response_event":
            continue  # Raw events ko ignore kar rahe hain

        elif event.type == "agent_updated_stream_event":
            print(f"Agent update hua: {event.new_agent.name}")

        elif event.type == "run_item_stream_event":
            if event.item.type == "tool_call_item":
                print("-- Tool call hua")
            elif event.item.type == "tool_call_output_item":
                print(f"-- Tool ka output: {event.item.output}")
            elif event.item.type == "message_output_item":
                print(f"-- Message ka output:\n {ItemHelpers.text_message_output(event.item)}")

    print("=== Run mukammal ho gaya ===")

if _name_ == "_main_":
    asyncio.run(main())


==========================================================================================================================================================================================

* *Tools*

Yeh raha tumhara diya gaya "Tools" ka sara content Roman Urdu mein convert kiya gaya, asaan aur technical explanation ke saath:


---

Tools (Aalat)

Tools agent ko actions perform karne ki ijazat detay hain — jaise data lana, code chalana, external APIs call karna, ya computer use karna.

Agent SDK mein 3 qisam ke tools hote hain:

1. Hosted Tools – yeh LLM servers par directly chalaye jatay hain.


2. Function Calling Tools – koi bhi Python function ko tool bana saktay ho.


3. Agents as Tools – ek agent dusray agent ko tool ke tor par use kar sakta hai, bina control handoff kiye.




---

Hosted Tools

OpenAI kuch built-in hosted tools provide karta hai jab aap OpenAIResponsesModel use karte ho:

WebSearchTool: Web pe search karta hai.

FileSearchTool: OpenAI Vector Store se info nikalta hai.

ComputerTool: Computer ke tasks automate karta hai.


Example:

from agents import Agent, FileSearchTool, Runner, WebSearchTool

agent = Agent(
    name="Assistant",
    tools=[
        WebSearchTool(),
        FileSearchTool(
            max_num_results=3,
            vector_store_ids=["VECTOR_STORE_ID"],
        ),
    ],
)

async def main():
    result = await Runner.run(agent, "Main kis coffee shop jaon, meri pasand aur SF ka weather dekhte hue?")
    print(result.final_output)


---

Function Tools

Koi bhi Python function tool ban sakta hai. SDK isay khud setup karta hai:

Tool ka naam function ka naam hota hai (ya custom naam de saktay ho).

Description function ke docstring se aata hai.

Inputs ka schema function arguments se generate hota hai.

Har input ka explanation bhi docstring se liya jata hai (agar disable na kiya ho).


Example:

from typing_extensions import TypedDict, Any
from agents import Agent, FunctionTool, RunContextWrapper, function_tool

class Location(TypedDict):
    lat: float
    long: float

@function_tool  
async def fetch_weather(location: Location) -> str:
    """Diye gaye location ka weather laata hai."""
    return "sunny"

@function_tool(name_override="fetch_data")  
def read_file(ctx: RunContextWrapper[Any], path: str, directory: str | None = None) -> str:
    """File ke contents read karta hai."""
    return "<file contents>"

agent = Agent(
    name="Assistant",
    tools=[fetch_weather, read_file],  
)


---

Custom Function Tools

Kabhi kabhi aap function_tool decorator nahi use karna chahte. Aap manually FunctionTool bana saktay ho:

Name

Description

Params ka JSON schema

Async on_invoke_tool function jo JSON string input leta hai aur output return karta hai.


Example:

from pydantic import BaseModel
from agents import RunContextWrapper, FunctionTool

class FunctionArgs(BaseModel):
    username: str
    age: int

async def run_function(ctx: RunContextWrapper[Any], args: str) -> str:
    parsed = FunctionArgs.model_validate_json(args)
    return f"{parsed.username} ki age {parsed.age} hai"

tool = FunctionTool(
    name="process_user",
    description="User data process karta hai",
    params_json_schema=FunctionArgs.model_json_schema(),
    on_invoke_tool=run_function,
)


---

Automatic Argument aur Docstring Parsing

inspect module function ke signature se arguments ka type samajh ke schema banata hai.

griffe docstring se tool aur arguments ka description nikalta hai.

pydantic se validation aur model generation hoti hai.

Supported formats: Google, Sphinx, Numpy docstrings.


Agar chaho to docstring parsing disable bhi kar saktay ho.


---

Agents as Tools

Agar aap chaho ke aik central agent multiple agents ko control kare (bina handoff kiye), to agents ko tool ke tor par model kar saktay ho.

Example:

from agents import Agent, Runner
import asyncio

spanish_agent = Agent(
    name="Spanish agent",
    instructions="User ka message Spanish mein translate karo.",
)

french_agent = Agent(
    name="French agent",
    instructions="User ka message French mein translate karo.",
)

orchestrator_agent = Agent(
    name="orchestrator_agent",
    instructions="Tum aik translation agent ho. Tools use karo translate karne ke liye.",
    tools=[
        spanish_agent.as_tool(
            tool_name="translate_to_spanish",
            tool_description="User ka message Spanish mein translate karta hai.",
        ),
        french_agent.as_tool(
            tool_name="translate_to_french",
            tool_description="User ka message French mein translate karta hai.",
        ),
    ],
)

async def main():
    result = await Runner.run(orchestrator_agent, input="Say 'Hello, how are you?' in Spanish.")
    print(result.final_output)


---

Customize karna Agent Tools ko

agent.as_tool() ek asaan method hai agent ko tool mein badalne ka, lekin is mein har config ka support nahi hota (e.g. max_turns).
Advance config ke liye Runner.run() ko manually tool function ke andar call karo.

Example:

@function_tool
async def run_my_agent() -> str:
    agent = Agent(name="My agent", instructions="...")
    result = await Runner.run(
        agent,
        input="...",
        max_turns=5,
        run_config=...
    )
    return str(result.final_output)


---

Function Tool mein Error Handle karna

Agar @function_tool use kar rahe ho to aap failure_error_function provide kar saktay ho:

Default: tool crash hone par LLM ko generic error batata hai.

Custom error function bhi de saktay ho.

Agar None pass karo to exception raise hoga (ModelBehaviorError, UserError, etc).

Agar manually FunctionTool bana rahe ho, to on_invoke_tool ke andar hi error handle karna hoga.


