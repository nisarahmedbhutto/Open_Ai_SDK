
**Agents SDK kyun use karein?**
SDK ke do ahem design principles hain:

1. Itni features hon ke use karna faida mand ho, lekin itni kam complexity ke seekhna asaan ho.
2. Out of the box achha kaam karta hai, lekin agar aap chahen to har cheez ko customize bhi kar sakte hain.

**SDK ki kuch main features ye hain:**

* **Agent loop:** Built-in loop jo automatically tools ko call karta hai, results LLM ko bhejta hai, aur loop chalaata hai jab tak LLM ka kaam complete nahi hota.
* **Python-first:** Aap Python ke built-in features ka use karke agents ko chain aur orchestrate kar sakte hain, naye abstractions seekhne ki zarurat nahi.
* **Handoffs:** Ye feature aapko multiple agents ke darmiyan coordination aur delegation mein madad deta hai.
* **Guardrails:** Inputs ki validation aur checking parallel mein agents ke saath chalte hain — agar validation fail ho jaaye to turant rok diya jaata hai.
* **Function tools:** Kisi bhi Python function ko tool mein tabdeel karein — schema automatic ban jaata hai aur Pydantic ka validation included hota hai.
* **Tracing:** Built-in tracing se aap apne workflows ko visualize, debug, aur monitor kar sakte hain — aur OpenAI ke evaluation, fine-tuning, aur distillation tools ka bhi use kar sakte hain.

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

* **deep informition**
Zaroor! Niche mein **SDK ke main features** ko **1 agent ke perspective** se explain kiya gaya hai — jese ek user ne input diya, to agent kaise kaam karta hai aur har feature ka kya role hai. Ye ek **step-by-step numbered flow** hai:

---

### **Agents SDK ka Agent Workflow (Step-by-Step):**

**1. User Input deta hai → Guardrails ka kaam start hota hai**
🔹 **Guardrails** sabse pehle input ki **validation aur safety checks** perform karta hai (e.g., toxic language, incomplete data).
🔹 Agar input valid nahi hota, to **workflow yahin ruk jaata hai**.

**2. Input valid hone ke baad → Agent Loop activate hota hai**
🔹 **Agent loop** main brain hai — ye decide karta hai ke **LLM se kya poochhna hai**, aur **kis tool ki zarurat hai**.
🔹 Ye har step pe LLM se response leta hai, aur loop tab tak chalta hai jab tak task complete nahi hota.

**3. Tool ki zarurat ho → Function Tools ka kaam start hota hai**
🔹 **Function tools** kisi bhi Python function ko ek **callable tool** bana dete hain.
🔹 Agent in tools ko LLM ke kehne par **automatically call** karta hai, aur **inputs validate** karta hai using Pydantic.

**4. Agar dusre agents ki zarurat ho → Handoffs activate hota hai**
🔹 Kabhi kabhi kaam complex hota hai ya dusri expertise chahiye hoti hai.
🔹 **Handoffs** ka kaam hai dusre agents ko **delegate** karna — ek agent dusre agent ko kaam handover karta hai.

**5. Workflow chal raha hota hai → Tracing parallel mein hoti rehti hai**
🔹 **Tracing** aapko har step ka **record, debug information aur visualization** deti hai.
🔹 Ye baad mein aapko fine-tuning, evaluation, aur monitoring mein madad karti hai.

---

### 🔄 Loop continues...

Agar LLM response incomplete ho ya aur tools chahiye hon, agent loop wapas step 2 par jaata hai, jab tak final answer milta.

---

**Ye poora process ek powerful aur flexible agent system ka hissa hai**, jahan har feature apni jagah pe kaam karta hai.


------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

* **handsoff*
Bohat acha sawal hai! Aaiye **`handoffs`** feature ko detail se samajhtay hain — **kaise kaam karta hai**, aur **code me kis attribute ya function se yeh trigger hota hai.**

---

### 🔁 **Handoff ka matlab kya hai?**

Handoff ka matlab hai:

> **Ek agent ka kisi aur agent ko kaam de dena** — agar usko lagay ke doosra agent is kaam mein zyada expert hai.

---

### 🧠 **Code Explanation:**

```python
triage_agent = Agent(
    name="Triage Agent",
    instructions="You determine which agent to use based on the user's homework question",
    handoffs=[history_tutor_agent, math_tutor_agent]
)
```

**Yahan kya ho raha hai:**

* `triage_agent` ek **meta-agent** hai.
* Isko `handoffs` list di gayi hai, jisme:

  * `history_tutor_agent`
  * `math_tutor_agent`

Ye dono **doosre agents** hain jo specific domains handle karte hain.

---

### 💡 `handoffs` Attribute ka Role

* Jab `triage_agent` ko koi **input milta hai**, to LLM (via agent loop) dekhta hai ke input kis domain ka hai.
* Agar input math ka hai, to wo internally decide karta hai:

  ➤ **"Is kaam ko `math_tutor_agent` handle kare"**,
  ➤ Aur **usko handoff kar deta hai**.

Ye logic `Agent` class ke andar LLM prompt aur tools ke beech run hota hai.

---

### ⚙️ **Kaunsa function decide karta hai ke handoff ho?**

Agar aap SDK ka andar ka code dekhein to:

1. `Agent.run()` ya `Agent.__call__()` method hota hai.
2. Ye internally **LLM ke output** ko parse karta hai.
3. Agar output me koi agent ka naam ya identifier aata hai (jo `handoffs` list me ho), to:

   * Agent system us agent ko call karta hai.
   * Ye decision **LLM ke reasoning** par based hota hai — e.g., `"Send this to math_tutor_agent"`.

> **Matlab: handoff explicit function call nahi karta, balkay `handoffs=[...]` list LLM ko batati hai ke kaun se agents available hain for delegation.**

---

### ✅ Summary:

| Cheez            | Kaam                                                                                           
| `handoffs` param | Ye list batati hai ke kis kis agent ko handover kiya ja sakta hai.                                                 |
| LLM + Agent loop | LLM input samajhta hai aur decide karta hai ke handoff karna hai ya nahi.                                          |
| Function Trigger | `Agent.run()` ya `__call__()` ke through yeh logic internally chalta hai.                                          |
| Custom Logic     | Aap agar chahein to LLM prompt me clearly define kar sakte hain: `"If it's a math question, use math_tutor_agent"` |


* *handsoff_discreption*
Great question! 🔍

### ✅ `handoff_description` ka kaam:

> Jab ek **triage agent ya koi doosra agent** handoff karne ka decision le raha hota hai, to us waqt LLM ko ye samajhne me madad milti hai ke **har specialist agent kis kaam me expert hai** — aur ye hi kaam `handoff_description` karta hai.

---

## 🎯 Purpose of `handoff_description`:

* Ye **LLM ko guidance deta hai** handoff decision ke waqt.
* Batata hai: “Ye agent kis type ke sawalon ke liye hai.”

---

### 💡 For example:

```python
history_tutor_agent = Agent(
    name="History Tutor",
    handoff_description="Specialist agent for historical questions",
    ...
)
```

To jab **triage agent** ko user ka input milta hai, maan lo:

> `"Who was Julius Caesar?"`

To LLM ye dekhta hai:

* Hmm, ye historical question lagta hai.
* Mujhe kaunse agent ka handoff\_description match karta hai?

It finds:

```text
"History Tutor" => "Specialist agent for historical questions"
```

So, **handoff decision** confidently karta hai: ➡️ `history_tutor_agent`

---

## 🧠 LLM ke liye yeh metadata hai

Jese insaan ko kisi resume me "expertise: math teaching" likha mile —
waise hi LLM ko har agent ke liye yeh short tag milta hai for **decision-making help**.

---

### 🧪 Internally kya hota hai?

`handoff_description` SDK ke andar:

* LLM prompt ke ek part me **list of possible handoffs** ke sath inject hota hai, e.g.:

```
You can delegate to one of the following agents:
- History Tutor: Specialist agent for historical questions
- Math Tutor: Specialist agent for math questions
```

LLM phir **match karta hai input ke context se** ke kis agent ko handoff kiya jaye.

---

### ✅ Summary Table:

| Attribute             | Purpose                                                               |
| --------------------- | --------------------------------------------------------------------- |
| `handoff_description` | **LLM ko batata hai** ye agent kis type ke questions handle karta hai |
| Used by               | **Triage agents** ya kisi bhi agent jo delegation karte hain          |
| Benefit               | Accurate, explainable handoffs based on agent specialization          |




* *Persona*
✅ 2. instructions ka kaam:
python
Copy
Edit
instructions="You determine which agent to use based on the user's homework question"
📌 Kya karta hai?
Ye agent ko LLM ke through ek role aur behavior define karta hai.

Ye ek tarah ka prompt injection hai jo LLM ko kehta hai:

“Tumhara role ye hai: tum user ka input analyse karo aur decide karo kiska agent use hona chahiye.”

🧠 Example use:
Agar instructions ye ho:
"You are a helpful math tutor. Answer only math questions."
To agent sirf math questions ke liye reply karega.

Agar ye ho:
"You are a triage agent. Your job is to forward the question to the correct domain expert."
To agent khud se jawab nahi dega, balkay handoff karega.



* *guardrails*
Aap ne excellent example diya hai — chaliye detail se samajhtay hain ke:

> ✅ **Guardrail kya hai, kaise kaam karta hai, kis jagah laga hai, aur iska result kya hota hai?**

---

## 🛡️ Guardrail: Overview

### 🔑 **Guardrail ka role hota hai input validate karna** before ya during agent execution.

Yani:

> "Agar input expected criteria match nahi karta, to agent ruk jaye ya alternate logic follow kare."

---

## 🔍 **Aap ke code ka analysis**

```python
class HomeworkOutput(BaseModel):
    is_homework: bool
    reasoning: str
```

Ye ek **Pydantic model** hai jo guardrail agent ka expected output define karta hai.

| Field         | Purpose                                   |
| ------------- | ----------------------------------------- |
| `is_homework` | Kya input homework se related hai ya nahi |
| `reasoning`   | LLM ne kis logic se ye decision liya      |

---

### 🧠 `guardrail_agent` setup:

```python
guardrail_agent = Agent(
    name="Guardrail check",
    instructions="Check if the user is asking about homework.",
    output_type=HomeworkOutput,
)
```

Yahan:

* **LLM se bola gaya hai**: "Tumhara kaam ye check karna hai ke input homework question hai ya nahi"
* Agent output karega: `HomeworkOutput` type ka response

  * Example: `{"is_homework": True, "reasoning": "Because the user asked about an assignment topic"}`

---

### ✅ `homework_guardrail` function:

```python
async def homework_guardrail(ctx, agent, input_data):
    result = await Runner.run(guardrail_agent, input_data, context=ctx.context)
    final_output = result.final_output_as(HomeworkOutput)
    return GuardrailFunctionOutput(
        output_info=final_output,
        tripwire_triggered=not final_output.is_homework,
    )
```

### ⚙️ Kya ho raha hai?

1. **Run `guardrail_agent`** with `input_data`

   * Example input: `"Tell me how to solve this math problem"`

2. **LLM agent output karta hai**:

   ```json
   {
     "is_homework": True,
     "reasoning": "The user is asking how to solve a specific academic problem"
   }
   ```

3. **Output parsed hota hai** into `HomeworkOutput`.

4. `tripwire_triggered` ka matlab hai:

   * Agar `is_homework == False`, to **guardrail trip ho jata hai** (invalid input)
   * Agar `True`, to **pass ho jata hai**

---

## 🎯 **Guardrail laga kahan?**

`homework_guardrail(...)` function ek **custom guardrail function hai** jo aap kisi bhi agent ke **execution se pehle ya parallel** run kar sakte hain.

Aap is function ko:

* **Agent run se pehle check ke liye**
* Ya **SDK ke internal guardrail system me register karne ke liye**
  use karte hain.

---

## 📤 Example Output:

### ✅ Input:

```python
input_data = "Tell me how to solve this algebra homework"
```

### ✅ Output:

```python
GuardrailFunctionOutput(
    output_info=HomeworkOutput(
        is_homework=True,
        reasoning="The question includes 'homework' and a subject-specific topic"
    ),
    tripwire_triggered=False
)
```

---

### ❌ Input (not homework):

```python
input_data = "How's the weather in Lahore?"
```

### ❌ Output:

```python
GuardrailFunctionOutput(
    output_info=HomeworkOutput(
        is_homework=False,
        reasoning="The question is about general knowledge and not schoolwork"
    ),
    tripwire_triggered=True  # <-- Guardrail failed
)
```

---

## ✅ Summary Table:

| Item                 | Purpose                                             |
| -------------------- | --------------------------------------------------- |
| `guardrail_agent`    | LLM-based agent that checks input validity          |
| `HomeworkOutput`     | Defines the schema for valid result                 |
| `tripwire_triggered` | True = validation failed, False = pass              |
| Use-case             | Early exit or alternate flow if input is irrelevant |






