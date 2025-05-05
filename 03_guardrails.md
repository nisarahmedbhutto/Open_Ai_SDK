

Guardrails – User input/output ki validation

Guardrails aapke agent ke parallel chaltay hain aur user ke input/output ka analysis karte hain — jaise security check ya policy enforcement.

Example Scenario:
Agar aapka agent aik powerful LLM use karta hai (jo mehnga ya slow ho sakta hai), to aap nahi chahenge ke koi user usay math homework solve karne ke liye use kare.
To aap aik guardrail laga saktay ho jo pehle fast/cheap model se check kare — agar misuse detect ho to wo turant execution rok dega.


---

Do tarah ke Guardrails hotay hain:

1. Input Guardrails – user ke input pe chaltay hain


2. Output Guardrails – agent ke output pe chaltay hain




---

Input Guardrails – User ka input validate karna

Working Steps:

1. Guardrail ko user ka wahi input milta hai jo agent ko milta.


2. Guardrail function run karta hai aur GuardrailFunctionOutput return karta hai.


3. Agar .tripwire_triggered true ho, to InputGuardrailTripwireTriggered exception raise hoti hai — jisse aap response ya flow control kar saktay ho.



Note:
Input guardrails sirf pehle agent pe chaltay hain. Har agent ke apnay guardrails hotay hain, isliye Runner.run() mein alag se pass nahi karte.


---

Output Guardrails – Agent ka output validate karna

Working Steps:

1. Guardrail ko same input milta hai.


2. Guardrail function run hota hai, aur GuardrailFunctionOutput return karta hai.


3. Agar .tripwire_triggered true ho, to OutputGuardrailTripwireTriggered exception raise hoti hai.



Note:
Output guardrails sirf aakhri agent ke output pe chaltay hain.


---

Tripwires – Jab rule break ho to guardrail trigger kare

Agar guardrail detect kare ke kuch galat ho gaya (input ya output mein), to tripwire trigger hota hai — jisse agent ki execution turant stop ho jati hai.


---

Guardrail banana – Example

Aapko aik function banana hota hai jo input le aur GuardrailFunctionOutput return kare. Neeche example mein hum check kar rahe hain ke user math homework to nahi karwa raha.

from pydantic import BaseModel
from agents import (
    Agent,
    GuardrailFunctionOutput,
    InputGuardrailTripwireTriggered,
    RunContextWrapper,
    Runner,
    TResponseInputItem,
    input_guardrail,
)

class MathHomeworkOutput(BaseModel):
    is_math_homework: bool
    reasoning: str

# Guardrail Agent jo check karega
guardrail_agent = Agent( 
    name="Guardrail check",
    instructions="Check if the user is asking you to do their math homework.",
    output_type=MathHomeworkOutput,
)

# Guardrail function
@input_guardrail
async def math_guardrail( 
    ctx: RunContextWrapper[None], agent: Agent, input: str | list[TResponseInputItem]
) -> GuardrailFunctionOutput:
    result = await Runner.run(guardrail_agent, input, context=ctx.context)

    return GuardrailFunctionOutput(
        output_info=result.final_output, 
        tripwire_triggered=result.final_output.is_math_homework,
    )

# Main Agent jisme guardrail laga hai
agent = Agent(  
    name="Customer support agent",
    instructions="You help customers with their questions.",
    input_guardrails=[math_guardrail],
)

# Test run
async def main():
    try:
        await Runner.run(agent, "Hello, can you help me solve for x: 2x + 3 = 11?")
        print("Guardrail didn't trip - this is unexpected")
    except InputGuardrailTripwireTriggered:
        print("Math homework guardrail tripped")


---

Output Guardrails ka Example

from pydantic import BaseModel
from agents import (
    Agent,
    GuardrailFunctionOutput,
    OutputGuardrailTripwireTriggered,
    RunContextWrapper,
    Runner,
    output_guardrail,
)

# Output ka structure
class MessageOutput(BaseModel): 
    response: str

class MathOutput(BaseModel): 
    reasoning: str
    is_math: bool

# Guardrail agent jo output check karega
guardrail_agent = Agent(
    name="Guardrail check",
    instructions="Check if the output includes any math.",
    output_type=MathOutput,
)

# Output guardrail function
@output_guardrail
async def math_guardrail(  
    ctx: RunContextWrapper, agent: Agent, output: MessageOutput
) -> GuardrailFunctionOutput:
    result = await Runner.run(guardrail_agent, output.response, context=ctx.context)

    return GuardrailFunctionOutput(
        output_info=result.final_output,
        tripwire_triggered=result.final_output.is_math,
    )

# Main agent with output guardrail
agent = Agent( 
    name="Customer support agent",
    instructions="You help customers with their questions.",
    output_guardrails=[math_guardrail],
    output_type=MessageOutput,
)

# Test run
async def main():
    try:
        await Runner.run(agent, "Hello, can you help me solve for x: 2x + 3 = 11?")
        print("Guardrail didn't trip - this is unexpected")
    except OutputGuardrailTripwireTriggered:
        print("Math output guardrail tripped")

