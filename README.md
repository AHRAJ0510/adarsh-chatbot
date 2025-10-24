# adarsh-chatbot
import gradio as gr
import requests
import json

# Function to ask Ollama
def ask_ollama(prompt, model="gpt-oss:120b-cloud"):
    url = "http://localhost:11434/api/generate"
    data = {"model": model, "prompt": prompt}
    reply = ""
    try:
        response = requests.post(url, json=data, stream=True)
        for line in response.iter_lines():
            if line:
                try:
                    data_line = json.loads(line.decode("utf-8"))
                    if "response" in data_line:
                        reply += data_line["response"]
                except json.JSONDecodeError:
                    continue
        return reply
    except Exception as e:
        return f"[Error]: {e}"

# Gradio interface
def chat_fn(message, chat_history):
    bot_reply = ask_ollama(message, model=model_select)
    chat_history.append((message, bot_reply))
    
    # Print message in CMD
    print(f"[User]: {message}")
    print(f"[Bot]: {bot_reply}\n")
    
    # Save chat log in a file
    with open("chat_log.txt", "a", encoding="utf-8") as f:
        f.write(f"[User]: {message}\n[Bot]: {bot_reply}\n---\n")
    
    return chat_history, chat_history

# Choose model
model_select = "gpt-oss:120b-cloud"  # cloud
# model_select = "mistral:latest"    # local

with gr.Blocks() as demo:
    gr.Markdown("<h1 style='text-align:center;'>🤖 Adarsh GPT Cloud Assistant</h1>")
    chatbot = gr.Chatbot()
    msg = gr.Textbox(placeholder="Type your message here...")
    state = gr.State([])
    msg.submit(chat_fn, inputs=[msg, state], outputs=[chatbot, state])
    gr.Button("Clear").click(lambda: [], outputs=[chatbot])

demo.launch(share=True)
