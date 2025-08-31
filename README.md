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

### Q1 Why do we need to create a virtual env for this project ?

### Answer:

Here’s the idea in plain terms:

- Python libraries are like ingredients.
- If you cook all your recipes in the same kitchen without labeling, soon you’ll have spice jars everywhere, some fresh, some expired, some with different versions of the same spice. Messy!

A virtual environment is like a mini-kitchen.
You set up a clean, empty space just for this one recipe (your Gradio MCP server). You only put in the ingredients you need, in the right versions.

#### Why it matters for you here:

The Gradio MCP server depends on some specific libraries (like gradio, mcp, maybe fastapi).

If your global Python already has different versions of these libraries (say, for another project), they can clash and cause errors.

With a virtual environment, you isolate dependencies so the tutorial works exactly as intended.

When you run the below command

```
python -m venv venv
```

a folder named venv/ appears in your project. Inside it, Python quietly sets up:

A copy of the Python interpreter → so your project uses a fixed Python version.

A separate site-packages/ directory → where all your pip install ... libraries go, only for this project.

Activation scripts → small shell/batch scripts (activate, activate.bat) that tell your terminal:

“Use this Python”

“Look for packages here, not globally”

So when you activate it (e.g., source venv/bin/activate on Linux/Mac or .\venv\Scripts\activate on Windows), your shell switches into that private mini-kitchen. Anything you pip install goes inside the venv folder, not your system Python.

When you’re done, you just deactivate (deactivate command), and your terminal goes back to “normal.”

### Q2 what is textblob ?
### Answer:
Textblob is a library used to implement the text sentiment analysis.

### Q3. Some MCP client does not support SSE, in that case we are recommended to use `mcp-remote`. Why ?
### Answer:
- When you run your Gradio app with mcp_server=True, it exposes an MCP server endpoint at:
- `http://localhost:7860/gradio_api/mcp/sse`
- Notice the SSE at the end? That stands for Server-Sent Events → a way for the server to push data continuously to the client over HTTP.

So normally, your MCP client (like a model, tool, or app) connects directly to this SSE-based endpoint.

#### The Problem

Not all MCP clients know how to talk SSE.
Some only understand standard JSON-RPC over stdin/stdout or plain HTTP, not this “streaming events” style.

So if the client doesn’t support SSE, it won’t be able to connect directly to the Gradio server.

#### The solution: `mcp-remote`

- Think of mcp-remote as a translator or adapter.

```
{
  "mcpServers": {
    "gradio": {
      "command": "npx",
      "args": [
        "mcp-remote",
        "http://localhost:7860/gradio_api/mcp/sse"
      ]
    }
  }
}
```
- This tells your MCP client:
“Instead of connecting directly to the SSE server, run mcp-remote. It will connect to the Gradio SSE server on your behalf, and then speak plain MCP (JSON-RPC) to you.”

So, mcp-remote = middleman that bridges SSE → standard MCP protocol.

## Acknowledgements
- Hugging Face MCP Course (Unit 2) – [link](https://huggingface.co/learn/mcp-course/unit2/introduction)
- Gradio team for MCP integration
- TextBlob library for sentiment analysis
