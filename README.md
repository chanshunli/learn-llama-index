# Learn llama_index

## Basic ReActAgent Definition

```python
import os
from llama_index.core.agent import ReActAgent
from llama_index.llms.ollama import Ollama
from llama_index.core.tools import FunctionTool
from llama_index.core import Settings
from llama_index.core.callbacks import CallbackManager
from langfuse.llama_index import LlamaIndexCallbackHandler
import subprocess
import sys
import json

# Set up the Langfuse callback handler
langfuse_callback_handler = LlamaIndexCallbackHandler()
Settings.callback_manager = CallbackManager([langfuse_callback_handler])

langfuse_callback_handler.set_trace_params(
  user_id="user-123",
  session_id="session-abc",
  tags=["ReAct shell"]
)

# Define the functions to be used as tools
def multiply(a: float, b: float) -> float:
    """Multiply two numbers and returns the product"""
    return a * b
multiply_tool = FunctionTool.from_defaults(fn=multiply)

def add(a: float, b: float) -> float:
    """Add two numbers and returns the sum"""
    return a + b
add_tool = FunctionTool.from_defaults(fn=add)

def run_shell_command(command: str) -> str:
    """Runs a shell command and returns the output or error"""
    try:
        result = subprocess.run(command, shell=True, check=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
        return result.stdout.decode('utf-8')
    except subprocess.CalledProcessError as e:
        return e.stderr.decode('utf-8')
shell_command_tool = FunctionTool.from_defaults(fn=run_shell_command)

# Initialize the LLM and agent
llm = Ollama(model="llama3.1:latest", request_timeout=120.0)
agent = ReActAgent.from_tools([multiply_tool, add_tool, shell_command_tool], llm=llm, verbose=True)

question = sys.argv[1]
response = agent.chat(question)

print(response)

```

## Using LangChain to Implement ReAct 

```python
import os
from langchain.agents import initialize_agent, Tool
from langchain.llms import OpenAI

# Define Tools
def lookup_wikipedia(query):
    if query.lower() == 'paris':
        return "Paris is the capital city of France, known for its art, gastronomy, and culture. Founded in the 3rd century BC..."
    else:
        return "No information found."

wikipedia_tool = Tool(
    name='Wikipedia',
    func=lookup_wikipedia,
    description='Provides summaries from Wikipedia articles.'
)

tools = [wikipedia_tool]

# Initialize Agent
llm = OpenAI(temperature=0)
agent = initialize_agent(
    tools=tools,
    llm=llm,
    agent="zero-shot-react-description",
    verbose=True
)

# Run Agent
question = "What is the capital of France, and can you provide a brief history of the city?"
response = agent.run(question)
print(response)
```
