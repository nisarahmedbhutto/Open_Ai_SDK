

Tracing (Saragh Rasd)

Agents SDK mein built-in tracing hoti hai jo agent ke run hone ke dauraan har important cheez ka record rakhti hai — jaise ke:

LLM generations (model ka jawab dena)

Tool calls

Handoffs

Guardrails

Aur custom events


Is tracing ka faida yeh hai ke aap debug, visualize, aur monitor kar saktay ho apni workflows ko — development aur production dono mein.


---

Tracing Default On Hoti Hai

Agar aapko disable karni ho tracing to 2 tareeqay hain:

1. Globally:

OPENAI_AGENTS_DISABLE_TRACING=1


2. Sirf ek run ke liye:

RunConfig(tracing_disabled=True)



Note:
Agar aapki organization Zero Data Retention (ZDR) policy follow karti hai, to tracing available nahi hoti.


---

Trace vs Span

Trace: Puri workflow ka record — jaise "Customer Support Workflow"

Span: Us trace ka ek hissa — jaise LLM ka jawab ya tool ka call


Trace Properties:

workflow_name (e.g., “Joke Workflow”)

trace_id (unique ID)

group_id (optional)

disabled (True/False)

metadata (optional info)


Span Properties:

started_at, ended_at

trace_id (kis trace se related hai)

parent_id (agar kisi aur span ke andar hai)

span_data (specific data, jaise generation info)



---

Default Tracing Kya Kya Record Karta Hai

Yeh sab tracing mein hota hai:

Runner.run() / run_sync() / run_streamed() → trace()

Agent ke har run → agent_span()

LLM response → generation_span()

Tool call → function_span()

Guardrail → guardrail_span()

Handoff → handoff_span()

Speech-to-text input → transcription_span()

Text-to-speech output → speech_span()

Audio group → speech_group_span()


Default name: "Agent trace"
Aap custom naam bhi de saktay ho using trace() ya RunConfig.


---

Custom Trace kaise Banayein (Higher-Level)

Agar aap chahte ho ke multiple Runner.run() ek hi trace ka hissa ho, to aap trace() ka context use karo:

from agents import Agent, Runner, trace

async def main():
    agent = Agent(name="Joke generator", instructions="Tell funny jokes.")

    with trace("Joke workflow"): 
        first_result = await Runner.run(agent, "Tell me a joke")
        second_result = await Runner.run(agent, f"Rate this joke: {first_result.final_output}")
        print(f"Joke: {first_result.final_output}")
        print(f"Rating: {second_result.final_output}")


---

Trace Banana: 2 tareeqay

1. Recommended (automatic):

with trace("My trace name") as my_trace:
    ...


2. Manual (advanced):

my_trace = trace("Manual trace")
my_trace.start(mark_as_current=True)
...
my_trace.finish(reset_current=True)




---

Custom Spans

Agar aapko khud ka custom span banana hai, to custom_span() use karo.
Ye spans automatically current trace ka hissa ban jate hain.


---

Sensitive Data Settings

generation_span() aur function_span() input/output data capture kartay hain (sensitive ho sakta hai)

Isko disable karne ke liye:

RunConfig(trace_include_sensitive_data=False)

Audio data base64 form mein capture hoti hai — isko disable karne ke liye:

VoicePipelineConfig.trace_include_sensitive_audio_data=False



---

Custom Tracing Processors

Tracing ka backend structure kuch is tarah hota hai:

1. TraceProvider: Har trace banata hai


2. BatchTraceProcessor: Traces ko batch mein backend bhejta hai


3. BackendSpanExporter: Backend tak data pohchata hai



Aap do tareeqay se customize kar saktay ho:

1. add_trace_processor()

OpenAI backend ke saath saath dusri jagah bhi data bhejna.

2. set_trace_processors()

Sirf apni processing karni ho, aur OpenAI backend na use karna ho.


---

Supported External Tracing Processors

Yeh third-party tools use ho saktay hain tracing ke liye:

Weights & Biases

Arize-Phoenix

Future AGI

MLflow (Self-hosted & Databricks)

Braintrust

Pydantic Logfire

AgentOps

Scorecard

Keywords AI

LangSmith

Maxim AI

Comet Opik

Langfuse

Langtrace

Okahu-Monocle

