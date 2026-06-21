from flask import Flask, render_template, request, jsonify
import sqlite3
import datetime
import os

app = Flask(__name__)
app.config['SECRET_KEY'] = 'your-secret-key'

# Setup database
def init_db():
    conn = sqlite3.connect('ai_platform.db')
    c = conn.cursor()
    c.execute('''CREATE TABLE IF NOT EXISTS history
                 (id INTEGER PRIMARY KEY, type TEXT, input TEXT, output TEXT, timestamp TEXT)''')
    conn.commit()
    conn.close()

init_db()

# Fake AI functions - replace these with real API calls
def ai_chat_response(prompt):
    # Replace with OpenAI, Llama, etc API call
    return f"This is a demo AI response to: '{prompt}'. Connect a real LLM API here."

def ai_image_response(prompt):
    # Replace with Stability AI, DALL-E, Meta AI etc
    # For now return a placeholder
    return f"https://via.placeholder.com/512x512.png?text={prompt.replace(' ', '+')}"

@app.route('/')
def home():
    return render_template('index.html')

@app.route('/api/chat', methods=['POST'])
def chat():
    user_input = request.json.get('message')
    response = ai_chat_response(user_input)

    # Save to DB
    conn = sqlite3.connect('ai_platform.db')
    c = conn.cursor()
    c.execute("INSERT INTO history (type, input, output, timestamp) VALUES (?,?,?,?)",
              ('chat', user_input, response, datetime.datetime.now().isoformat()))
    conn.commit()
    conn.close()

    return jsonify({'response': response})

@app.route('/api/image', methods=['POST'])
def image():
    user_prompt = request.json.get('prompt')
    image_url = ai_image_response(user_prompt)

    # Save to DB
    conn = sqlite3.connect('ai_platform.db')
    c = conn.cursor()
    c.execute("INSERT INTO history (type, input, output, timestamp) VALUES (?,?,?,?)",
              ('image', user_prompt, image_url, datetime.datetime.now().isoformat()))
    conn.commit()
    conn.close()

    return jsonify({'image_url': image_url})

@app.route('/api/history')
def get_history():
    conn = sqlite3.connect('ai_platform.db')
    c = conn.cursor()
    c.execute("SELECT type, input, output, timestamp FROM history ORDER BY id DESC LIMIT 20")
    rows = c.fetchall()
    conn.close()
    return jsonify([{'type': r[0], 'input': r[1], 'output': r[2], 'time': r[3]} for r in rows])

if __name__ == '__main__':
    app.run(debug=True)
