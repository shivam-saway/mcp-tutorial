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


