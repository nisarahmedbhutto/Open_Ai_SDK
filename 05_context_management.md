

Context Management (Siyaaq ka Intizaam)

"Context" ka matlab alag alag jagah par alag hota hai. Is SDK mein 2 tareeqay ka context important hai:


---

1. Local Context (Aapke Code ke Andar Jo Data Available Hai)

Yeh wo data hai jo aap tools, callbacks (jaise on_handoff), aur lifecycle hooks ke dauraan istemal kartay ho.

Isko handle karta hai:

RunContextWrapper[T]


---

Local Context Workflow:

1. Aap koi bhi Python object bana saktay ho (usually @dataclass ya Pydantic class).


2. Phir is object ko Runner.run() mein context= ke zariye pass karte ho.


3. Har tool ya function jo chalega usko yeh context wrapper mein milay ga:

wrapper: RunContextWrapper[T]



> Important: Har agent run ke tools, hooks, waghera sab mein ek hi type ka context hona chahiye.




---

Use Cases:

User-specific info: Jaise name, uid, etc.

Dependencies: Jaise logger, fetchers, etc.

Helper methods


> Note: Yeh context LLM ko nahi bheja jata. Sirf aapke local functions ke liye hota hai.




---

Example Code: Local Context

import asyncio
from dataclasses import dataclass

from agents import Agent, RunContextWrapper, Runner, function_tool

@dataclass
class UserInfo:  
    name: str
    uid: int

@function_tool
async def fetch_user_age(wrapper: RunContextWrapper[UserInfo]) -> str:  
    return f"User {wrapper.context.name} is 47 years old"

async def main():
    user_info = UserInfo(name="John", uid=123)

    agent = Agent[UserInfo](  
        name="Assistant",
        tools=[fetch_user_age],
    )

    result = await Runner.run(  
        starting_agent=agent,
        input="What is the age of the user?",
        context=user_info,
    )

    print(result.final_output)  
    # Output: User John is 47 years old.

if _name_ == "_main_":
    asyncio.run(main())


---

2. Agent/LLM Context (Jo LLM Ko Nazar Aata Hai)

LLM sirf conversation history dekh sakta hai. Agar aap kisi nayi info ko model tak le jana chahte ho, to usay history ka hissa banana padega.


---

LLM Context Dene ke Tareeqay:

1. Agent instructions (System Prompt):

Jaise: "You are a helpful assistant for user John."

Yeh static ya dynamic dono ho sakti hai (context-based function se).



2. Runner input mein include karna:

Jaise: "User's name is John. What can I help with?"



3. Function tools ke zariye expose karna:

LLM jab chahe tab tool ko call karke data fetch kar sakta hai.



4. Retrieval ya Web search tools:

Files, database ya web se relevant info laane ke liye (grounded response ke liye).





---

Summary (Mukhtasir Fikr):

Type	Purpose	LLM ko Dikhta Hai?

Local Context	Code ke andar tools/hooks ko info dena	Nahi
Agent/LLM Context	LLM ko data provide karna	Haan


