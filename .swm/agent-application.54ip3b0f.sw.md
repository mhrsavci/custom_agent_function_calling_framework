---
title: Agent Application
---
# Introduction

This document will walk you through the implementation of the "Agent Application" feature.

The feature allows the application to perform a series of actions based on user input, including mathematical operations, scheduling, and fetching information.

We will cover:

1. How actions are defined and stored.
2. How actions are processed and executed.
3. How the results are communicated back to the user.

# Defining and storing actions

<SwmSnippet path="app.py" line="1">

---

Actions are defined as a list of strings. Each string represents a sequence of tasks that the agent needs to perform.

```
import ostc
import config as cfg
```

---

</SwmSnippet>

<SwmSnippet path="app.py" line="3">

---

This code snippet performs multiple actions in sequence. First, it calculates the result of multiplying 4 and 6. Then, it schedules a meeting with Johan at a date and time of its choice. Finally, it provides a fact about penguins.

```


actions = ["Get the result of multiplying 4 and 6 then scheduling a meeting with Johan at a date and time of your choice. Next, tell me a fact about penguins."]
```

---

</SwmSnippet>

&nbsp;

<SwmSnippet path="/app.py" line="7">

---

This code snippet assigns the value of `cfg.llm` to the variable `model`.

```python

model = cfg.llm

```

---

</SwmSnippet>

&nbsp;

<SwmSnippet path="/app.py" line="10">

---

This code multiplies two integers `a` and `b` and returns the result integer. It also prints the result of the multiplication.

```python
def multiply(a: int, b: int) -> int:
    """Multiply two integers and returns the result integer"""
    result = a * b
    print(f"The result of multiplying {a} and {b} is {result}")
    return a * b
```

---

</SwmSnippet>

.

<SwmSnippet path="/app.py" line="15">

---

This code snippet schedules a meeting with a specified `attendee` at a given `time`. It optionally takes a `description` parameter to provide additional details.

```python

def create_meeting(attendee, time, description=None):
    
    print(f"Scheduled a meeting with {attendee} at {time}.")
```

---

</SwmSnippet>

&nbsp;

<SwmSnippet path="/app.py" line="23">

---

This code defines an array of `tools` objects. Each object represents a tool with a `name`, `description`, and `parameters`. The `parameters` object specifies the type and description of the required properties for each tool.

```python
tools = [
        { 
            "name": "multiply",
            "description": "multiple two integers",
            "parameters": {
                "type": "object",
                "properties": {
                    "a": {
                        "type": "int",
                        "description": "The first integer to multiply"
                    },
                    "b": {
                        "type": "int",
                        "description": "The second integer to multiply"
                    }
                },
                "required": ["a", "b"]
            }
        },
        {
            "name": "create_meeting",
            "description": "Schedule a meeting for the user with the specified details",
            "parameters": {
                "type": "object",
                "properties": {
                    "attendee": {
                        "type": "string",
                        "description": "The person to schedule the meeting with"
                    },
                    "time": {
                        "type": "datetime",
                        "description": "The date and time of the meeting"
                    }
                },
                "required": [
                    "attendee",
                    "time"
                ]
            },
        },
        {
            "name": "chat_response",
            "description": (
                "Use chat responses if a conversation is appropriate instead of a function."
        ),
            "parameters": {
                "type": "object",
                "properties": {
                    "response": {
                        "type": "string",
                        "description": "Standard chat response.",
                    },
                },
                "required": ["response"],
        },
}
]
```

---

</SwmSnippet>

&nbsp;

<SwmSnippet path="/app.py" line="80">

---

This code snippet searches for specific tools in a list of `tools`. It looks for a tool with the name 'multiply', 'create_meeting', and 'chat_response' respectively. If found, it assigns the tool to the `multiply_tool`, <SwmToken path="/app.py" pos="81:0:0" line-data="create_meeting_tool = next((tool for tool in tools if tool[&quot;name&quot;] == &quot;create_meeting&quot;), None)">`create_meeting_tool`</SwmToken>  , and `chat_response_tool` variables respectively. If a tool with the specified name is not found, the variable is assigned `None`.

```python
multiply_tool = next((tool for tool in tools if tool["name"] == "multiply"), None)
create_meeting_tool = next((tool for tool in tools if tool["name"] == "create_meeting"), None)
chat_response_tool = next((tool for tool in tools if tool["name"] == "chat_response"), None)
```

---

</SwmSnippet>

&nbsp;

<SwmSnippet path="/app.py" line="84">

---

This code snippet defines a dictionary `functions` that maps function names to their corresponding implementations. The functions included are `multiply`, `create_meeting`, and `chat_response`.

```python
functions = {
    "multiply": multiply,
    "create_meeting": create_meeting,
    "chat_response": chat_response

}
```

---

</SwmSnippet>

<SwmSnippet path="/app.py" line="91">

---

&nbsp;

```python
agent_creators = [
    ostc.AgentCreator(name="tool_user", tools=[multiply_tool], description="Can use multiply tool to multiply two integers"),
    ostc.AgentCreator(name="meeting_agent", tools=[create_meeting_tool], description="Can use create_meeting tool to schedule a meeting"),
    ostc.AgentCreator(name="can_chat", tools=[chat_response_tool], description="Can use chat responses to respond to user prompts"),

]
```

---

</SwmSnippet>

<SwmSnippet path="/app.py" line="98">

---

&nbsp;

```python
agents = [creator.create_agent() for creator in agent_creators]

action_agents = ostc.AgentCreator.assign_agents(model, agents, actions)
```

---

</SwmSnippet>

&nbsp;

<SwmSnippet path="app.py" line="103">

---

This code snippet invokes and runs a series of actions based on a given `llm` object and `action_agents` dictionary. It generates a response using `ostc.CallingFormat.generate_response` and checks if the result is a string. If it is, it prints an error message and continues to the next action. If the result is not a string, it iterates over the result and performs specific functions based on the `function_name` and `arguments`. It appends the results of these functions to a `results` list. Finally, it returns the `results`.

```
def invoke_and_run(llm, action_agents):
    results = []
    for action, agent in action_agents.items():
        result = ostc.CallingFormat.generate_response(llm, agent.tools, action)
        if isinstance(result, str):
            print(f"Error: {result}")
            continue
        for res in result:
            function_name = res['tool']
            arguments = res.get('tool_input', {})
            if function_name in functions:
                results.append(functions[function_name](**arguments))
            else:
                print(f"Error: Unknown function '{function_name}'")
    return results

result = invoke_and_run(model, action_agents)
```

---

</SwmSnippet>

&nbsp;

&nbsp;

# Processing and executing actions

The actions list is processed to extract individual tasks. Each task is then executed in sequence. This involves performing mathematical operations, scheduling meetings, and fetching information.

# Communicating results

The results of the actions are communicated back to the user. This includes the result of the mathematical operation, confirmation of the scheduled meeting, and the fetched information.

This approach ensures that the agent can handle complex sequences of tasks efficiently.

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBY3VzdG9tX2FnZW50X2Z1bmN0aW9uX2NhbGxpbmdfZnJhbWV3b3JrJTNBJTNBbWhyc2F2Y2k=" repo-name="custom_agent_function_calling_framework"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
