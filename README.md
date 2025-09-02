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



## Acknowledgements
- Hugging Face MCP Course (Unit 2) – [link](https://huggingface.co/learn/mcp-course/unit2/introduction)
- Gradio team for MCP integration
- TextBlob library for sentiment analysis
