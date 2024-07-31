# Components

> **Version**: [0.0.2]
**Date**: [2024/7/31]

---

OmAgent offers standard, extendable interfaces and external integrations for building with LLMs. It implements some components, relies on third-party integrations for others, and uses a mix for the rest.

---

## Encoder `omagent-core.core.encoder`

Embedding models can transform text into a numerical vector format, where each vector is an array that encapsulates the text's semantic essence. This numerical representation enables the execution of mathematical operations, facilitating the identification of texts with closely related meanings. Such capabilities are foundational to advanced natural language search functions, and crucial in context retrieval tasks. In these tasks, large language models (LLMs) leverage the provided data vectors to generate responses that accurately align with the query's intent.

![Embedding Model](docs/img/Embedding Model.drawio.png)

The `EncoderBase` class is designed for interface with text embedding models. There are many different embedding model providers (OpenAI, Hugging Face, etc) and local models, and this class is designed to provide a base interface for all of them.

---

## Chat models`omagent-core.core.llm`

We define a class `BaseLLM` that inherits from `BotBase` and `ABC` (Abstract Base Class). This class is designed to interact with a Language Learning Model (LLM). It includes mechanisms for caching responses to reduce the need to call the underlying model for the same inputs repeatedly. 

We have some standardized parameters when constructing ChatModels based on `BaseLLM`:

- `model_id`: the id of the model
- `temperature`: the sampling temperature
- `vision`: model vision ability
- `max_tokens`: max tokens to generate
- `stop`: default stop sequences
- `response_format`: format of response
- `api_key`: API key for the model provider
- `base_url`: endpoint to send requests to
- `use_default_sys_prompt` : use default sys_prompt or not

Some important things to note:
﻿
standard parameters only apply to model providers that expose parameters with the intended functionality. For example, some providers do not expose a configuration for vision ability, so `vision` can't be supported on these. So  when  interfacing with different models, you should pay more attention to it.

ChatModels also accept other parameters that are specific to that integration. To find all the parameters supported by a ChatModel head to the API reference for that model. 

---

## Node`omagent-core.core.node`

### Base

The `Node` class is an abstract base class that represents the minimal processing unit in a pipeline. All nodes in a pipeline should share the same interface. This class inherits from `BotBase` and `ABC` (Abstract Base Class). And it has both synchronous and asynchronous execution paths.

And `Basedecider` and `Baseprocessor` are both defined that based on the `Node` class, aiming to process data in a pipeline. And the purpose of them are moving data from one processing step to the next according to the defined sequence. OmAgent also defined a loop class to implement the processing loop capacity.

### Dnc

Here comes our specific design about the task schedule, in which we use the divide and conqueror(dnc) theory. We have `TaskDivider` and `TaskConqueror`  to divide complex multimodal tasks and then execute them and finally conqueror the results, the process are implement by LLMs, LLM will decide how to divide and when to conqueror the final answers.

### Misc

Here OmAgent define a class `TaskRescue`, inherits from `BaseLLMBackend` and `BaseDecider` , this class is responsible for rescuing tasks that have failed through using a large language model to generate new prompts and executing tool tasks.

Here is the detail of tool call:

```python
toolcall_rescue_output_structure = {
                    "tool_status": rescue_execution_status,
                    "tool_result": rescue_execution_results,
                }
                self.callback.send_block(
                    f'{Fore.WHITE}\n{"-=" * 5}Tool Call {Fore.RED}Rescue{Style.RESET_ALL} Output{"=-" * 5}{Style.RESET_ALL}\n'
                    f"{Fore.BLUE}{json.dumps(toolcall_rescue_output_structure, indent=2, ensure_ascii=False)}{Style.RESET_ALL}"
                )
```

---

## Prompt templates`omagent-core.core.prompt`

Prompt templates convert user inputs and parameters into instructions for a language model, ensuring it generates relevant and coherent responses by understanding the context. 

Prompt templates use a dictionary where each key corresponds to a variable in the template to be filled as input.

Prompt templates generate a Prompt, which can be utilized by a Language Model (LLM) or a ChatModel. This Prompt is designed for flexibility, allowing it to be converted into either a string or a list of messages. The purpose behind the creation of Prompt is to simplify the transition between string formats and message lists.

There are a few different methods to create prompt from prompt templates:

**From example**

The `from_examples` method constructs a prompt from a list of examples, a suffix, and input variables, allowing for dynamic prompt creation. 

```python
def from_examples(
        cls,
        examples: List[str],
        suffix: str,
        input_variables: List[str],
        example_separator: str = "\n\n",
        prefix: str = "",
        **kwargs: Any,
    ) -> PromptTemplate:
	      """Take examples in list format with prefix and suffix to create a prompt."""
    
```

**From file**

The `from_file` method loads a prompt template from a file.

```python
def from_file(
        cls, template_file: Union[str, Path], **kwargs: Any
    ) -> PromptTemplate:
        """Load a prompt from a file."""
```

**From template**

The `from_template` method creates a prompt template directly from a template string, determining the input variables based on the template format. 

```python
def from_template(
        cls, template: str, template_format: str = "jinja2", **kwargs: Any
    ) -> PromptTemplate:
        """Load a prompt template from a template."""
```

**From config**

The `from_config` method loads a prompt template from a configuration dictionary, extracting the template and other parameters from the config.

```python
def from_config(cls, config: Dict) -> PromptTemplate:
        """Load a prompt template from a config."""
```

---

## Tool system`omagent-core.core.tool_system`

Tools are utilities intended to be invoked by a model: their inputs are crafted to be generated by models, and their outputs are meant to be returned to models. Tools are necessary when you want a model to manage parts of your code or interact with external APIs.

A tool consists of:

1. The name of the tool.
2. A description of what the tool does.
3. A function (and, also an async function to implement or pass it)
4. A optional type schema defining the inputs to tool.

When a tool is linked to a model, the name, description, and args schema are given as context to the model. With a list of tools and a set of instructions, a model can request to invoke one or more tools with specific inputs. A typical usage scenario might look like this:

```python
 #create a json file under the path: /workflows/your task, tools'info should also be defined in this json file
{     
    "llm": "...",
    "tools": [...]
}

#in run.py
def run_agent(task):
    ...
    bot_builder = Builder.from_file("workflows/your task")  #change this path
    ...
    return input.last_output
if __name__ == "__main__":
    run_agent("your query") #enter your query
```

We define a class`BaseTool` here, it has  `_run` method,  `_arun` method, `run` method, `arun` method and `generate_schema` method. 

The `_run` method is a private method that executes the function stored in `func` with the provided input arguments, the `_arun` method is an asynchronous version of `_run`.

The `run` method is a public method that first checks if `args_schema` is defined.After validation, it calls the `_run` method with the input arguments and any special parameters, similarly, the `arun` method is the asynchronous counterpart to `run`.

The `generate_schema` method generates a schema for the tool's input parameters. If `args_schema` is not defined, it returns a basic schema with a description and a single required input parameter. If `args_schema` is defined, it uses the `generate_schema` method of `ArgSchema` to create a detailed schema with properties and required fields.

---

## Handlers`omagent-core.handlers`

OmAgent provides different handle systems for callbacks, data, error and logs that allows you to hook into the various stages of your LLM application. This is useful for logging, monitoring and callbacks.

### Callback_handler

The `callback_handler` includes `Testcallback` and `Defaultcallback` based on `basecallback`. The `Testcallback` is a simple implementation for testing purposes. And `Defaultcallback` is a more complex implementation that logs messages, handles errors, and organizes logs in a dict structure.

To use these classes, you would create an instance of `DefaultCallback` or `TestCallback` and call the methods as needed. For example:

```python
callback = DefaultCallback(bot_id="123", endpoint="http://example.com")
callback.info("This is an info message")
callback.error(VQLError("This is an error message"))
callback.send_block({"key": "value"})
callback.finish()
```

### Data_handler

The data_handler has several class for interfacing to different kinds of databases: `milvushandler`, `sql_data_handler`, `video_handler` .

The `milvushandler` is for interacting with the Milvus database. Milvus is an open-source vector database specifically designed for handling large-scale vector data.

And the `sql_data_handler` is similar, as for `video_handler` ,it’s based on the `milvushandler` to store the vectors from video information. 

### Error_handler

We defined a custom exception class named `VQLError` here. This custom exception is designed to handle various error scenarios with specific error codes and messages.

```python
- 500: "Internal Error",
- 501: "Image Error: Unable to Read",
- 502: "Image Error: Corrupted Image",
- 503: "Image Error: Unable to Retrieve",
- 504: "Image Error: Unrecognizable",
- 505: "Image Error: Nonexistent Key",
- 506: "Image Error: Unable to Connect to Database",
- 511: "Request Error: Service Processing Failed",
- 515: "Request Error: Exceeded Retry Limit, Unable to Access Address",
- 516: "Request Error: Nonexistent Address",
- 517: "Request Error: Incorrect Request Format",
- 518: "Request Error: Illegal Request Address",
- 550: "IBase Error",
- 570: "Callback Error: Result Callback Failed"
```

### Log_handler

We define a custom logging class `Logger` that extends Python's built-in `_logging.Logger` class. This custom logger allows for enhanced logging with caller information and supports both console and file logging with rotation.

---

## Schemas`omagent-core.schemas`

OmAgent defined its specific schemas here, including `STM`, `BaseInterface` and `BaseTable` inherited from `BaseModel` and `SQLModel` :

- `STM` class: This class has five attributes: `tasks`, `memory`, `knowledge`, `summary` and `kwargs` in dict type for passing information between nodes.
- `BaseInterface` class: This class has two attributes:  `last_output` and `kwargs` . `last_output` represent the result that the node processed.
- `BaseTable` class: This class inherits from `SQLModel` , is used for database table representation.

And we also define a `Message` class here to process the `content` attribute of a `Message` object, which can contain both text and image placeholders.
The class can replace image placeholders with actual image URLs from a provided image cache, which is useful for handling messages that include dynamic image content.

---

## Utils`omagent-core.utils`

OmAgent first defines a class called `Builder` here, it can be used to init and run the bot system and provide methods to create instances from file or dict. It can also visualize the loop nodes we defined in `Baseloop` before. Apart from that, there are also LRUcache class for temp storage, HTTP request function and a `Registry` class to support multitype modules and dynamically importing modules.