# Disclaimer
This repository is based on the Unit 2 MCP course by Hugging Face. It is intended for learning and demonstration purposes only. The original idea and content belong to Hugging Face; I’ve just followed along and implemented the tutorial.

# mcp-tutorial
Hugging Face MCP course - e2e MCP application

## Prerequisites

- Python
- Node.js above version 18
- Hugging Face account (For deployment)

## Abstract
##### Server Side

- Uses Gradio to create a web interface and MCP server via gr.Interface
- Implements a sentiment analysis tool using TextBlob
- Exposes the tool through both HTTP and MCP protocols
  
##### Client Side

- Implements a HuggingFace.js client
- Or, creates a smolagents Python client
- Demonstrates how to use the same server with different client implementations

### Building a Gradio MCP server

First of all, we need to create a seperate virtual env for working with this project to indenpendently manage the python packages and dependencies.
Run the commands in the code snippets below

```
mkdir gradio-mcp-server
cd gradio-mcp-server
python -m venv venv
./venv/Scripts/Activate
pip install "gradio[mcp]" textblob
```

Gradio provides us features to automatically convert function in python scripts to mcp tools. When we run the gradio server by setting the flag `mcp_server=True` in `launch()`.

Followings are the features which gradio provides

- Automatically converts python functions into tools in mcp server
- Automatically determines the input and output schemas for MCP communication from function definition.
- Sets up JSON-RPC over HTTP+SSE for client-server communication
- Creates both a web interface and an MCP server endpoint

`app.py` defines the MCP server.

While defining the python functions, One should follow few best practices as mentioned below:

- Type hints helps gradio define the input/output schema
- The docstring is crucial as it helps Gradio generate the MCP tool schema

### Building Clients for MCP server

We will build the client for javascript app using `huggingface.js` and a python based client using `smolagents`.

#### Configuration files for MCP hosts

As per the architecture, the MCP clients needs a host environment to work. It can be a UI app or a MCP Host. 

The MCP HOST applications like Cursor IDE or smolagents uses configuration files to maintain the connection of clients with MCP server. The configuration files are very simple. These files follow a consistent naming convention and structure across the MCP hosts. For example, the standard configuration file for MCP is named as `mcp.json`.

```
{
  "servers": [
  {
    "name": "MCP Server",
    "transport": {
      "type": "sse",
      "url": "http://localhost:7860/gradio_api/mcp/sse" // we are hosting a local MCP server. so localhost in url. If server is hosted on cloud or is available remotely then remote server url will be pasted here.
    }
  }
  ]
}

```

If we want to expose a local script as MCP server to our client app then we can use `stdio` transport type instead of `sse`. The server configuration would change as follows,

Say you’ve built the app.py Gradio MCP server from the tutorial, but instead of deploying to Hugging Face Spaces or running via localhost:7860, you want to run it locally as a script and connect to it directly.

Normally, python app.py starts the HTTP server (http://localhost:7860) with sse transport.

But if you instead want your MCP client (e.g., Cursor, smolagents, etc.) to talk to the script directly via stdin/stdout, you’d use stdio transport.

```
{
  "servers": [
    {
      "name": "Local Sentiment MCP Server",
      "transport": {
        "type": "stdio",
        "command": "python",
        "args": ["app.py"]
      }
    }
  ]
}


```

###### How it works

- type: "stdio" → tells the MCP client:
- “Don’t use HTTP or SSE. Instead, spawn this process and talk over stdin/stdout.”

- command: "python" → the command to run.

- args: ["app.py"] → arguments passed to the command (so it runs your app.py).

##### Key Difference from sse

- sse = client opens a URL and listens to events (http://localhost:7860/...).

- stdio = client spawns the server process directly and communicates via pipes.

This is much simpler for local development because:

- No networking required.

- No ports or URLs.

- Just a subprocess connection.

#### Configuration files for UI applications

When it is needed to connect a UI app with a MCP server. we can configure the MCP client in UI app using the `config.json` file.
Here is how to setup it.

```
{
  "mcpServers": {
    "mcp": {
      "url": "http://localhost:7860/gradio_api/mcp/sse"
    }
  }
}
```

#### Configuring MCP clients in Cursor IDE

[link](https://huggingface.co/learn/mcp-course/unit2/clients#configuring-an-mcp-client-within-cursor-ide)

### Using MCP in your AI coding assistants

#### Connecting MCP servers with local and open source models

This section deals with connecting the local and open source models with MCP. It uses a tool called `Continue`.

##### Continue

TODO: Add something about continue here.

##### Setup Continue

- Download the Continue Extension from VS code market place.
- Install the extension and verify installation by seeing a extension icon the left bar

##### Setup Local Models

There are many models which are compatible with `Continue`. Three popular options are Ollama, Llama.cpp & LM Studio. Ollama is an open source tool that allows users to run LLM locally.
Llama.cpp is a C++ library for running LLMs that also includes an OpenAI-compatible server. While LM Studio provides a graphical interface for running local models. You can access local models from the Hugging Face Hub and get commands and quick links for all major local inference apps.

If we take an example of `llamacpp`, it provides `llama-server` which we can use to run the LLMs locally. we can run any model from hugging face using `llama-server` like as shown below

``` 
llama-server -hf <model name>
e.g., llama-server -hf unsloth/Devstral-Small-2505-GGUF:Q4_K_M
```

To use the local model in your text editor, we need to do some changes in our workspace directory and `Continue` will automatically handle rest of the things,

- Create a folder called .continue/models
- Create a file called local-model.yaml
- add the local model yaml with the following structure.

```
name: Llama.cpp model
version: 0.0.1
schema: v1
models:
  - provider: llama.cpp
    model: unsloth/Devstral-Small-2505-GGUF
    apiBase: http://localhost:8080
    defaultCompletionOptions:
      contextLength: 8192 # Adjust based on the model
    name: Llama.cpp Devstral-Small
    roles:
      - chat
      - edit
```
###### Note: Continue supports multiple local model providers. You can use different models for different tasks or switch models as needed. This section focuses on local-first solutions, but Continue does work with popular providers like OpenAI, Anthropic, Microsoft/Azure, Mistral, and more. You can also run your own model provider.

##### Local Model Integration with MCP

Do the following to connect the local model with MCP server

- Create a folder called `.continue/mcpServers`
- Add the manifest for the MCP server
- For example if we want to connect with sentiment analysis server then the manifest will look something like this

```
name: Sentiment Analysis MCP Server
version: 0.0.1
schema: v1
mcpServers:
  - name: Sentiment Analysis
    transport:
      type: sse
      url: http://localhost:7860/gradio_api/mcp/sse
```

Now you can prompt the model as follows

```
Please analyze the sentiment of the following sentences using the Sentiment Analysis tool:

1. "I love how easy this library is to use!"
2. "This bug makes me so frustrated."
3. "The product is okay, not great but not terrible."


```

### Building MCP client using Gradio

The blog [Building MCP client using Gradio](https://huggingface.co/learn/mcp-course/unit2/gradio-client) says the following statement in very first paragraph.

```Gradio is best suited to the creation of UI clients and MCP servers, but it is also possible to use it as an MCP Client and expose that as a UI.```

This statement is bit confusing specially with respect to UI clients parts. Here is the simplification of the above dense words

- Normally
  - Gradio is most commonly used in two ways:
    - To make a user interface (UI) for your ML models or tools.
    - To serve your ML model/tool as an MCP Server (so that other MCP clients like Cursor/VSCode can call it).
   
  - But also (the special case here):
    - You can flip the role: use Gradio not just to provide a UI or server, but actually as an MCP Client itself.
      - That means Gradio connects to another MCP Server.
      - Then it exposes the connected tools to users through a Gradio-based chat interface.
     
So the blog is saying:
👉 “Gradio is usually for building servers or UIs. But you can also use it as a client, connect it to an MCP server, and then give users a Gradio-powered chat UI to use those server tools.”

##### Example Scenario

Imagine:

- You already built an MCP Server that provides math tools (like prime factorization, sentiment analysis, etc.).
- Now, instead of opening Cursor IDE or VSCode to use those tools, you build a Gradio app that:
  - Connects to your MCP Server (as an MCP Client).
  - Lets end users interact with it directly in a web UI (chat window).

So the confusing line just means:

✅ Gradio isn’t limited to making UIs or MCP servers.
✅ It can also act as a bridge client, pulling tools from an MCP server and showing them in a Gradio chat UI.

##### Connect to an example MCP server

To connnect to a MCP server, we can use MCPClient Class from `smolagents.mcp_client` library.

```
With MCPClient(
  "url": "url of mcp server",
) as tools:
  print("\n".join(f"${t.name}: ${t.description}");
```

Now connect to MCP server from gradio app

First we need `smolagents`, `gradio` and `mcp_client` libraries.

```pip install "smolagents[mcp]" "gradio[mcp]" mcp fastmcp```

Now we can create a simple gradio interface which uses MCP client to the other MCP server.

```
import gradio as gr
import os

from mcp import StdioServerParameters
from smolagents import InferenceClientModel, CodeAgent, ToolCollection, MCPClient
```

Next we can connect to MCP server and get the list of MCP tools, which we can use to answer the question

```
mcp_client = MCPClient(
    {"url": "https://abidlabs-mcp-tool-http.hf.space/gradio_api/mcp/sse"} # This is the MCP Client we created in the previous section
)
tools = mcp_client.get_tools()
```

Now we can create a simple agent using `smolagents` and chat interface using `gradio`'s InferenceClientModel class.

```
model = InferenceClientModel(token=os.getenv("HF_TOKEN"))
agent = CodeAgent(tools=[*tools], model=model)
```

Chat Interface

```
demo = gr.ChatInterface(
    fn=lambda message, history: str(agent.run(message)),
    type="messages",
    examples=["Prime factorization of 68"],
    title="Agent with MCP Tools",
    description="This is a simple agent that uses MCP tools to answer questions."
)

demo.launch()
```

And that’s it! We’ve created a simple Gradio interface that uses the MCP Client to connect to the MCP Server and answer questions.

Complete Full code is available [here](https://huggingface.co/learn/mcp-course/unit2/gradio-client#complete-example)


## Acknowledgements
- Hugging Face MCP Course (Unit 2) – [link](https://huggingface.co/learn/mcp-course/unit2/introduction)
- Gradio team for MCP integration
- TextBlob library for sentiment analysis
