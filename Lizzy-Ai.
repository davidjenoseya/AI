<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Lizzy AI</title>
<style>
  * { box-sizing: border-box; margin: 0; padding: 0; }
  body { font-family: Arial, sans-serif; background: #f4f1fb; color: #222; }
  header { background: #6c4ad0; color: #fff; padding: 14px; text-align: center; font-size: 20px; }
  nav { display: flex; background: #fff; border-bottom: 1px solid #ddd; }
  nav button { flex: 1; padding: 12px; border: none; background: none; font-size: 15px; }
  nav button.active { border-bottom: 3px solid #6c4ad0; color: #6c4ad0; font-weight: bold; }
  section { display: none; padding: 12px; }
  section.active { display: block; }
  #messages { height: 55vh; overflow-y: auto; background: #fff; border-radius: 10px; padding: 10px; margin-bottom: 10px; }
  .msg { margin: 6px 0; padding: 8px 12px; border-radius: 12px; max-width: 85%; }
  .user { background: #6c4ad0; color: #fff; margin-left: auto; }
  .bot { background: #eee; }
  .row { display: flex; gap: 6px; }
  input, textarea { flex: 1; padding: 10px; border: 1px solid #ccc; border-radius: 8px; font-size: 15px; width: 100%; }
  .row button, .btn { padding: 10px 14px; background: #6c4ad0; color: #fff; border: none; border-radius: 8px; font-size: 15px; }
  ul { list-style: none; margin-top: 10px; }
  li { background: #fff; padding: 10px; margin-bottom: 6px; border-radius: 8px; display: flex; justify-content: space-between; align-items: center; }
  li.done span { text-decoration: line-through; color: #888; }
  li button { background: #e55; color: #fff; border: none; border-radius: 6px; padding: 4px 8px; }
  .small { font-size: 12px; color: #666; margin: 6px 0; }
</style>
</head>
<body>
<header>Lizzy AI</header>
<nav>
  <button class="active" onclick="show('chat', this)">Chat</button>
  <button onclick="show('plan', this)">Planner</button>
  <button onclick="show('notes', this)">Notes</button>
  <button onclick="show('settings', this)">Settings</button>
</nav>

<section id="chat" class="active">
  <div id="messages"></div>
  <div class="row">
    <input id="input" placeholder="Ask Lizzy anything..." onkeydown="if(event.key==='Enter')send()">
    <button onclick="send()">Send</button>
  </div>
</section>

<section id="plan">
  <h3>Daily Planner</h3>
  <div class="row">
    <input id="taskInput" placeholder="Add a task..." onkeydown="if(event.key==='Enter')addTask()">
    <button onclick="addTask()">Add</button>
  </div>
  <ul id="tasks"></ul>
</section>

<section id="notes">
  <h3>Shared Notes</h3>
  <textarea id="noteText" rows="12" placeholder="Write notes, ideas, plans..." oninput="saveNotes()"></textarea>
  <p class="small">Saved automatically on this phone.</p>
</section>

<section id="settings">
  <h3>Settings</h3>
  <p class="small">Your name</p>
  <input id="userName" placeholder="Your name" oninput="saveSettings()">
  <p class="small">AI API key (kept only on this phone)</p>
  <input id="apiKey" type="password" placeholder="Paste API key here" oninput="saveSettings()">
  <p class="small">Anthropic API key from console.anthropic.com. Without it, Lizzy gives simple built-in replies.</p>
</section>

<script>
function show(id, btn) {
  document.querySelectorAll('section').forEach(s => s.classList.remove('active'));
  document.querySelectorAll('nav button').forEach(b => b.classList.remove('active'));
  document.getElementById(id).classList.add('active');
  btn.classList.add('active');
}

const msgBox = document.getElementById('messages');
let history = JSON.parse(localStorage.getItem('lizzyHistory') || '[]');

function render() {
  msgBox.innerHTML = '';
  history.forEach(m => addBubble(m.role === 'user' ? 'user' : 'bot', m.content));
  msgBox.scrollTop = msgBox.scrollHeight;
}
function addBubble(cls, text) {
  const d = document.createElement('div');
  d.className = 'msg ' + cls;
  d.textContent = text;
  msgBox.appendChild(d);
  msgBox.scrollTop = msgBox.scrollHeight;
}

async function send() {
  const input = document.getElementById('input');
  const text = input.value.trim();
  if (!text) return;
  input.value = '';
  history.push({ role: 'user', content: text });
  addBubble('user', text);
  const reply = await getReply();
  history.push({ role: 'assistant', content: reply });
  addBubble('bot', reply);
  localStorage.setItem('lizzyHistory', JSON.stringify(history.slice(-30)));
}

async function getReply() {
  const key = localStorage.getItem('lizzyKey');
  const name = localStorage.getItem('lizzyName') || 'friend';
  if (!key) {
    return 'Hi ' + name + '! Add your API key in Settings and I can chat properly. For now, try the Planner and Notes tabs.';
  }
  try {
    const res = await fetch('https://api.anthropic.com/v1/messages', {
      method: 'POST',
      headers: {
        'content-type': 'application/json',
        'x-api-key': key,
        'anthropic-version': '2023-06-01',
        'anthropic-dangerous-direct-browser-access': 'true'
      },
      body: JSON.stringify({
        model: 'claude-sonnet-4-5',
        max_tokens: 600,
        system: 'You are Lizzy, a warm, practical personal assistant helping ' + name + ' and their partner with daily life: planning, advice, motivation, ideas. Keep answers clear and kind.',
        messages: history.slice(-12)
      })
    });
    const data = await res.json();
    if (data.content && data.content[0]) return data.content[0].text;
    return 'Error: ' + (data.error ? data.error.message : 'unknown');
  } catch (e) {
    return 'Could not reach the AI: ' + e.message;
  }
}

let tasks = JSON.parse(localStorage.getItem('lizzyTasks') || '[]');
function renderTasks() {
  const ul = document.getElementById('tasks');
  ul.innerHTML = '';
  tasks.forEach((t, i) => {
    const li = document.createElement('li');
    if (t.done) li.className = 'done';
    const span = document.createElement('span');
    span.textContent = t.text;
    span.onclick = () => { tasks[i].done = !tasks[i].done; saveTasks(); };
    const del = document.createElement('button');
    del.textContent = 'X';
    del.onclick = () => { tasks.splice(i, 1); saveTasks(); };
    li.appendChild(span);
    li.appendChild(del);
    ul.appendChild(li);
  });
}
function saveTasks() { localStorage.setItem('lizzyTasks', JSON.stringify(tasks)); renderTasks(); }
function addTask() {
  const el = document.getElementById('taskInput');
  const v = el.value.trim();
  if (!v) return;
  tasks.push({ text: v, done: false });
  el.value = '';
  saveTasks();
}

function saveNotes() { localStorage.setItem('lizzyNotes', document.getElementById('noteText').value); }
function saveSettings() {
  localStorage.setItem('lizzyName', document.getElementById('userName').value);
  localStorage.setItem('lizzyKey', document.getElementById('apiKey').value);
}

document.getElementById('noteText').value = localStorage.getItem('lizzyNotes') || '';
document.getElementById('userName').value = localStorage.getItem('lizzyName') || '';
document.getElementById('apiKey').value = localStorage.getItem('lizzyKey') || '';
render();
renderTasks();
</script>
</body>
</html>
