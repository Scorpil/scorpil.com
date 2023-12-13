+++
title = "Ensuring JSON Response Format in OpenAI Assistant API"
description = "Function Calling for enforcing response schema"
tags = [
    "ai", "openai", "python"
]
date = 2023-12-13T11:50:00Z
author = "Scorpil"
images = [
]
+++

_No time to read? Complete Python code can be found [here](https://gist.github.com/Scorpil/7559f072b53d2b62b50c54d530e4050f). Enjoy!_

Recently, OpenAI has released their newest set of APIs, designed to create robust AI Assistant applications. These APIs have numerous advantages over their "classical" chat completion counterparts, such as:

- **Persistent threads**: Assistant "remembers" the conversation without the need of manual context management on the application level.
- **Access to advanced tools** that allow Assistant to browse the web and execute Python code.
- **Ability to read files** and refer to them in the answers.

This functionality, familiar to GPT-4 users, can now be integrated into external applications, enhancing them with AI-driven capabilities. However, as the APIs are currently in beta, some familiar functions are not yet implemented. One such function is [json-mode](https://platform.openai.com/docs/guides/text-generation/json-mode), which forces the model to always respond in `json` format. This feature is highly convenient for integration into existing software, so understandably some users were disappointed to find it missing.

This post will showcase an improved method to not only ensure JSON output formatting but also guarantee that responses comply with a predefined schema, achieved through the utilization of the Function Calling feature.

## Understanding OpenAI's Function Calling Feature

Function Calling is not exclusive to the Assistant API; it also exists in the chat completion APIs. Even with json-mode available, using function calling for JSON formatting is a common practice among developers.

Despite its convenience and simplicity, json-mode has a major flaw: it enforces only the JSON response format without a reliable method to define the response schema. Relying solely on direct queries to the model and hoping for accurate results is not foolproof. Given the inherently non-deterministic nature of the model's output, this approach risks receiving responses with fields your application does not expect.

Using Function Calling to address this issue may not be immediately intuitive, as this feature is primarily designed for AI to request additional information or interact with external systems. The official documentation does not explicitly describe repurposing Function Calling for communication, though it could be considered a form of external system integration.

Function Calling enables developers to instruct the model on how to use a specific function. Instead of directly calling a function, the model generates a special type of response. This response instructs the user to execute a particular function with defined arguments and relay the result back to the model.

To perform this call, model must "shape" its output to match the schema of the function's arguments. This can be leveraged to enforce any desired schema on the function's output, simply by requesting the model to always use a response function when responding to the input.

## Practical example

_To follow examples you'll first need to create an API key in [your OpenAI dashboard](https://platform.openai.com/account/api-keys). Once generated, set it as an environment variable in your execution environment, for example with:_

```bash
OPENAI_API_KEY=<your-api-key>
```

You will also need to install Python version of OpenAI SDK:

```bash
pip install openai
```

In this example, we will create a translate function using the OpenAI Assistant API. This function will translate a given sentence from any language supported by GPT-4 into English, French, Spanish, and German. The function accepts a string as input and returns a dictionary containing the translations.

We'll start by importing the official OpenAI SDK and initiating our client:

```python
# file: ai.py
import os
from openai import OpenAI

def translate(input):
    """
    translate function takes a sentence in any language and translates
    it into English, French, Spanish and German.
    """
	client = OpenAI(api_key=os.environ.get("OPENAI_API_KEY"))
```

Now we'll use client to access beta version of Assistant API to create a new Assistant. This is where most of the magic happens, because here we will:

- define a function along with its arguments whose schema will match the schema of the response we are looking for.
- define instruction set to prompt the model for its purpose and request it to use the defined function for response communication.

```python
import os
from openai import OpenAI

def translate(input):
    """
    translate function takes a sentence in any language and translates
    it into English, French, Spanish and German.
    """

    client = OpenAI(api_key=os.environ.get("OPENAI_API_KEY"))
    assistant = client.beta.assistants.create(
        name="Structured Response Assistant",
        model="gpt-4-1106-preview", # newest GPT-4 model at the time of writing
        instructions=(
            "Detect the input language and translate user input into "
            "other languages: English, French, Spanish and German. "
			# you need to explicitly ask the model to use the response tool
            "Imporant: always use the response tool to respond to the user. "
            "Never add any other text to the response."
        ),
        tools=[{
            "type": "function",
            "function": {
                "name": "response",
                "description": "Translate user input into other languages",
                "parameters": {
                    "type": "object",
                    "required": [
                        "input",
                        "original_language",
                        "english",
                        "french",
                        "spanish",
                        "german",
                    ],
                    "properties": {
                        "input": {
                            "type": "string",
                            "description": "User input"
                        },
                        "original_language": { # detects original language
                            "type": "string",
                            "description": "Language of the user input"
                        },
                        # requests translation
                        # note that model uses both field names and descriptions
                        # to correctly fill in the arguments
                        "english": {
                            "type": "string",
                            "description": "Translation to English"
                        },
                        "french": {
                            "type": "string",
                            "description": "Translation to French"
                        },
                        "spanish": {
                            "type": "string",
                            "description": "Translation to Spanish"
                        },
                        "german": {
                            "type": "string",
                            "description": "Translation to German"
                        },
                    }
                },
            }
        }]
    )
```

A crucial aspect of this Assistant definition is the response tool, whose parameters accept an OpenAPI schema definition to shape the desired output format. In my experience, LLMs sometimes omit optional arguments when faced with complex translations, such as proverbs, idioms, or culturally nuanced phrases like, for example, "a penny for your thoughts". Defining all attributes as required in the schema can mitigate this issue.

Next, we create a thread (a "conversation" abstraction), define our input to the Assistant (sentence we want translated), and execute it all by creating a "run" object.

```python
def translate():
    # ... <truncated for brevity>
    thread = client.beta.threads.create()
    client.beta.threads.messages.create(
        thread_id=thread.id,
        role="user",
        content=input,
    )

    run = client.beta.threads.runs.create(
        thread_id=thread.id,
        assistant_id=assistant.id,
    )
```

Another major limitation of the current beta version of the Assistant API is that there is no good way of streaming or long-polling the thread replies. Simplest solution right now is to simply query the state of the run object in a loop until the status changes. In our case the status should become `requires_action`, because Assistant expects us to execute the function. Instead, we will just parse the argument payload and use it in our code.

```python
import json
import time
# <...>

def translate():
    # ... <truncated for brevity>

    # 60 retries max, which shouldn't take much more than 1 minute
    for i in range(60):
        print(f"Waiting for response... ({i} seconds)")

        # get the latest run state
        result = client.beta.threads.runs.retrieve(
            thread_id=thread.id,
            run_id=run.id
        )

        if result.status == "requires_action": # run has executed
            # parse structured response from the tool call
            structured_response = json.loads(
				# fetch json from function arguments
                result.required_action.submit_tool_outputs.\
                    tool_calls[0].function.arguments
            )
            return structured_response

        # wait 1 second before retry
        time.sleep(1)
    raise Exception("Timeout")
```

That's it. All that's let is to execute the `translate` function to see the result:

```text
>>> from pprint import pprint
>>> from ai import translate
>>> pprint(translate("Quidquid latine dictum sit, altum videtur"))
Waiting for response... (0 seconds)
Waiting for response... (1 seconds)
Waiting for response... (2 seconds)
Waiting for response... (3 seconds)
Waiting for response... (4 seconds)
Waiting for response... (5 seconds)
Waiting for response... (6 seconds)
{'input': 'Quidquid latine dictum sit, altum videtur',
 'original_language': 'la',
 'english': 'Whatever is said in Latin seems profound',
 'french': 'Tout ce qui est dit en latin semble profond',
 'german': 'Alles, was auf Latein gesagt wird, scheint tiefgründig zu sein',
 'spanish': 'Todo lo que se dice en latín parece profundo'}
```

As you can see, using Function Calling to structure OpenAI Assistant responses is fairly easy. This method not only ensures JSON formatting standards but also introduces a schema adherence and greater predictability in AI-generated responses. Complete Python code can be found [in this gist](https://gist.github.com/Scorpil/7559f072b53d2b62b50c54d530e4050f).