
Model Context Protocol (MCP)

MCP aik aisa protocol hai jo tools aur context ko LLM (Large Language Model) tak pohanchane ka standard way provide karta hai.

Simple Analogy:

Jaise USB-C port ek standard tarika hai devices ko connect karne ka — MCP bhi waise hi ek standard way hai AI models ko different data sources aur tools se connect karne ka.

Agents SDK mein MCP ka support

Agents SDK ke zariye aap MCP servers ka use karke apne Agents ko additional tools provide kar saktay ho.


---

MCP Servers ke Types

Abhi ke liye MCP spec do qisam ke servers define karta hai:

1. stdio servers:

Yeh servers aapki application ke subprocess ke tor par run hotay hain.

Simple language mein: yeh locally chal rahay hotay hain.



2. HTTP over SSE servers:

Yeh remotely run hotay hain.

In se URL ke zariye connect karte hain.




Aap MCPServerStdio aur MCPServerSse classes use karke in dono tarah ke servers se connect kar saktay ho.


---

Example (Filesystem MCP Server)

async with MCPServerStdio(
    params={
        "command": "npx",
        "args": ["-y", "@modelcontextprotocol/server-filesystem", samples_dir],
    }
) as server:
    tools = await server.list_tools()

Yeh example ek official filesystem MCP server ko connect karta hai aur us se tools ki list mangta hai.


---

MCP Servers ko Agent mein Add Karna

Jab bhi agent run hota hai, Agents SDK list_tools() ko call karta hai MCP servers par. Is se LLM ko pata chalta hai ke kin tools ka access hai.

Jab LLM kisi tool ko use karta hai jo MCP server se aata hai, to SDK call_tool() method se us tool ko invoke karta hai.

Example:

agent = Agent(
    name="Assistant",
    instructions="Kaam mukammal karne ke liye tools ka use karo",
    mcp_servers=[mcp_server_1, mcp_server_2]
)


---

Caching (Speed aur Efficiency)

Har dafa jab agent run hota hai to list_tools() call hoti hai — remote servers ke case mein yeh slow ho sakta hai.

Agar aapko pata hai ke tool list frequently change nahi hoti, to:

Aap cache_tools_list=True set kar saktay ho MCPServerStdio ya MCPServerSse mein.

Agar aap cache ko invalidate karna chaho (taake fresh tool list mile), to:

await server.invalidate_tools_cache()



---

End-to-End Examples

Aap full examples dekh saktay ho yahan:
examples/mcp


---

Tracing (Monitoring aur Debugging)

MCP ke operations automatically trace hote hain:

Jab list_tools() call hoti hai.

Jab MCP tools call kiye jatay hain.

Sab MCP-related info trace mein capture hoti hai.



