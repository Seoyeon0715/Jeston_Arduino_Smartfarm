

## **Gradio-Based Sensor Data Visualization - Failed Attempt**

In `src/failed_attempts/gradio_graph_attempt.md`, we attempted to enhance **the existing sensor data query functions** by adding **a graph generation feature**.
**Descriptions of the individual sensor functions** (`moisture_sensor_info`, `light_sensor_info`, `dht_sensor_info`) can be found in **chatbot_ui.md**.

This document summarizes **the failed code and reasons** for **the attempt to generate and display graphs via the Gradio UI**. By documenting this failure, we aim to identify potential solutions and provide a reference for future improvements.

---

In this attempt, the **graph_from_file function** was added to the existing Gradio chatbot to **generate graphs based on CSV data** and display them in the Gradio interface.

- **New Feature**: The `graph_from_file` function samples sensor data, generates a graph, and saves it.
- **Issue Encountered**: **The graph image could not be displayed** in the Gradio chatbot interface despite successful graph generation.



---

## **1. Installation and Execution**

**Install Required Libraries**
```bash
pip install gradio pandas matplotlib openai
```

**Run the Script**
```bash
python script_name.py
```

---

## **2. Newly Added Function**

### **graph_from_file**

This function reads data from a CSV file, **samples the data, and generates a graph**.

**code**:
```python
import pandas as pd
import matplotlib.pyplot as plt

def graph_from_file(file_path, x_column, y_columns, sample_step=27):
    try:
        # Read CSV file
        data = pd.read_csv(file_path)
        
        # Sample data: Skip rows based on sample_step
        sampled_data = data.iloc[::sample_step, :].reset_index(drop=True)

        # Remove % and C symbols and convert to integers
        for column in y_columns:
            if '%' in sampled_data[column].iloc[0]:
                sampled_data[column] = sampled_data[column].str.rstrip('%').astype(int)
            elif 'C' in sampled_data[column].iloc[0]:
                sampled_data[column] = sampled_data[column].str.rstrip('C').astype(int)

        # Create the graph
        plt.figure(figsize=(10, 6))
        for column in y_columns:
            plt.plot(sampled_data[x_column], sampled_data[column], label=column)
        
        plt.xlabel(x_column)
        plt.ylabel("Values")
        plt.title("Sampled Sensor Data Visualization")
        plt.xticks(rotation=45)
        plt.legend()
        plt.tight_layout()

        # Save the graph
        graph_path = "sensor_data_graph.png"
        plt.savefig(graph_path)
        plt.close()

        return {"graph_path": graph_path, "message": "Graph created successfully with sampled data."}
    except Exception as e:
        return {"error": str(e)}
```

---

## **3. Updated sensor_functions**

The `sensor_functions` array, now including the **`graph_from_file`** function, is as follows:

```python
sensor_functions = [
    # Refer to chatbot_ui.md for existing moisture_sensor_info, light_sensor_info, dht_sensor_info functions
    {
        "type": "function",
        "function": {
            "name": "graph_from_file",
            "description": "Generates a graph using data from a CSV file with specified x and y columns.",
            "parameters": {
                "type": "object",
                "properties": {
                    "file_path": {
                        "type": "string",
                        "description": "The path to the CSV file containing the sensor data."
                    },
                    "x_column": {
                        "type": "string",
                        "description": "The column name to use as the x-axis (e.g., 'Timestamp')."
                    },
                    "y_columns": {
                        "type": "array",
                        "items": {"type": "string"},
                        "description": "A list of column names to use as the y-axis (e.g., 'Soil Moisture (%)')."
                    },
                    "sample_step": {
                        "type": "integer",
                        "description": "The step size for sampling data points to reduce graph density. Default is 27.",
                        "default": 27
                    }
                },
                "required": ["file_path", "x_column", "y_columns"]
            }
        }
    }
]
```

---

## **4. Gradio UI**

The Gradio UI, integrated with the `graph_from_fil`e function, is implemented as follows:

```python
import gradio as gr
import json
from openai import OpenAI

# Function to process user messages
def ask_openai(llm_model, messages, user_message, functions):
    client = OpenAI()
    proc_messages = messages.copy()

    if user_message:
        proc_messages.append({"role": "user", "content": user_message})

    response = client.chat.completions.create(
        model=llm_model, messages=proc_messages, tools=functions, tool_choice="auto"
    )
    response_message = response.choices[0].message
    tool_calls = response_message.tool_calls

    if tool_calls:
        available_functions = {
            "graph_from_file": graph_from_file  # Newly added function
        }

        proc_messages.append(response_message)

        for tool_call in tool_calls:
            function_name = tool_call.function.name
            function_to_call = available_functions.get(function_name)
            if function_to_call:
                try:
                    function_args = json.loads(tool_call.function.arguments)
                    function_response = function_to_call(**function_args)
                    proc_messages.append({
                        "tool_call_id": tool_call.id,
                        "role": "tool",
                        "name": function_name,
                        "content": json.dumps(function_response),
                    })
                except Exception as e:
                    proc_messages.append({
                        "tool_call_id": tool_call.id,
                        "role": "tool",
                        "name": function_name,
                        "content": json.dumps({"error": str(e)}),
                    })

        second_response = client.chat.completions.create(
            model=llm_model, messages=proc_messages
        )
        assistant_message = second_response.choices[0].message.content
    else:
        assistant_message = response_message.content

    proc_messages.append({"role": "assistant", "content": assistant_message})
    return proc_messages, assistant_message

# Gradio Interface
messages = []

def process(user_message, chat_history):
    proc_messages, ai_message = ask_openai("gpt-4o", messages, user_message, functions=sensor_functions)
    chat_history.append((user_message, ai_message))
    return "", chat_history

with gr.Blocks() as demo:
    gr.Markdown("# Sensor Data Query and Graph Generation")
    chatbot = gr.Chatbot(label="Sensor Data Query and Visualization")
    user_textbox = gr.Textbox(label="Input")
    user_textbox.submit(process, [user_textbox, chatbot], [user_textbox, chatbot])

demo.launch(share=True, debug=True)
```

---

## **5. Execution Example**

### **Input Example**
```
"Generate a graph from the CSV file. Use 'Timestamp' as the x-axis and 'Soil Moisture (%)' and 'Temperature (C)' as the y-axes."
```

### **Output**
- Generated graph image: `sensor_data_graph.png`
- x-axis: `Timestamp`
- y-axes: `Soil Moisture (%)`, `Temperature (C)`

- ---

## **6. Analysis of Failure**

### **Observed Issue**
- The Gradio server executed successfully, and the graph seemed to be generated. However, the graph image was not displayed in the chatbot interface.
- Server logs indicated that the graph image file was saved, but only text responses were returned in the Gradio chatbot.

### **Suspected Causes**
1. **Gradio File Output Issue**:
   - The Gradio `Chatbot` component is optimized for **text responses**.
   - To display image files in the chatbot interface, a separate `gr.Image` component is required.

2. **Path Issue**:
   - The generated image file `sensor_data_graph.png` was saved with a relative path, which might have made it inaccessible to the Gradio UI.

3. **Missing frpc File**:
   - **Sharing link failure** prevented the local Gradio server from being properly exposed externally.
   - The required **frpc_linux_aarch64_v0.2** file, mentioned in the message, was missing.

---

## **7. Attempts to Resolve**

- **Attempt 1**: Modified the code to use the `gr.Imag`e component for returning the graph image.
- **Attempt 2**: Changed the image file path to an absolute path.
- **Attempt 3**: Manually downloaded the frpc file and placed it in the specified directory.

Despite these efforts, **the graph image could not be displayed in the Gradio chatbot interface**.

---

## **8. Purpose of Documenting Failed Code**

1. **Root Cause Analysis**:
   To understand the reasons behind the failure and avoid repeating the same mistakes in future implementations.

2. **Future Improvements**:
   - Research methods for properly returning images using `gr.Image` or other Gradio components.
   - Install the missing **frpc file** to enable proper server sharing functionality.

3. **Learning Resource**:
   Documenting failed code serves as a valuable reference for **debugging** and provides insights for **future solutions**.

---

## **9. Next Steps**

1. Refer to Gradio documentation to review **appropriate methods for displaying images**.
2. **Install the missing frpc file** and restore the sharing functionality.
3. Inspect and adjust file access permissions and path configurations for potential fixes.

---
