
Handoffs – Agent ka doosray Agent ko kaam dena

Handoffs ka matlab hai ke aik agent kisi aur agent ko koi task de de. Yeh tab kaam aata hai jab har agent kisi specific kaam mein expert ho.

Example:
Customer support app mein ho saktay hain alag alag agents:

Order Status Agent

Refund Agent

FAQ Agent


Agar kisi user ka query refund ka ho, to system Refund Agent ko handoff kar deta hai.


---

LLM ke liye yeh handoffs tools ki tarah hotay hain

Jab LLM ko yeh handoff dikhai deta hai to usay yeh aik tool lagta hai.
Agar agent ka naam Refund Agent hai to tool ka naam hoga:

transfer_to_refund_agent


---

Handoff kaise banayein?

Har Agent ke paas aik handoffs parameter hota hai, jisme:

ya to seedha Agent diya ja sakta hai

ya handoff() function ka result (custom Handoff object)



---

Basic Example

from agents import Agent, handoff

billing_agent = Agent(name="Billing agent")
refund_agent = Agent(name="Refund agent")

triage_agent = Agent(
    name="Triage agent", 
    handoffs=[billing_agent, handoff(refund_agent)]
)

Yahan Triage agent do agents ko handoff de raha hai.


---

handoff() function se customization

handoff() se aap zyada control le saktay ho. Isme yeh options hain:

agent: Jis agent ko handoff dena hai

tool_name_override: Tool ka naam customize karna ho to

tool_description_override: Tool ka description customize karna ho to

on_handoff: Jab handoff hota hai to yeh callback function run hota hai

input_type: Agar LLM se kuch input lena ho handoff ke waqt

input_filter: Pichla conversation history filter karni ho


Example:

from agents import Agent, handoff, RunContextWrapper

def on_handoff(ctx: RunContextWrapper[None]):
    print("Handoff called")

agent = Agent(name="My agent")

handoff_obj = handoff(
    agent=agent,
    on_handoff=on_handoff,
    tool_name_override="custom_handoff_tool",
    tool_description_override="Custom description",
)


---

LLM se Input lena during Handoff

Agar aap chahte ho ke handoff ke waqt LLM kuch input de (jaise reason for escalation), to input_type define karna parta hai.

Example:

from pydantic import BaseModel
from agents import Agent, handoff, RunContextWrapper

class EscalationData(BaseModel):
    reason: str

async def on_handoff(ctx: RunContextWrapper[None], input_data: EscalationData):
    print(f"Escalation agent called with reason: {input_data.reason}")

agent = Agent(name="Escalation agent")

handoff_obj = handoff(
    agent=agent,
    on_handoff=on_handoff,
    input_type=EscalationData,
)


---

Input Filtering – Pichla conversation control karna

By default jab handoff hota hai, to next agent ko poori conversation history milti hai.

Agar aap chahte ho ke kuch part remove ho jaye (jaise tools ke calls), to input_filter use kar saktay ho.

Example:

from agents import Agent, handoff
from agents.extensions import handoff_filters

agent = Agent(name="FAQ agent")

handoff_obj = handoff(
    agent=agent,
    input_filter=handoff_filters.remove_all_tools,
)


---

LLM Prompts mein handoffs ka zikr zaroor karain (Recommended)

LLM ko handoff samajhne ke liye prompt mein instructions honi chahiye.
Aap yeh recommended prefix use kar saktay ho:

from agents import Agent
from agents.extensions.handoff_prompt import RECOMMENDED_PROMPT_PREFIX

billing_agent = Agent(
    name="Billing agent",
    instructions=f"""{RECOMMENDED_PROMPT_PREFIX}
    <Yahan baqi ka apna prompt likhein>.""",
)

Ya aap prompt_with_handoff_instructions() function bhi use kar saktay ho for auto integration.

