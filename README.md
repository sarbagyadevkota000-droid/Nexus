# Nexus
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Nexus AI | Live</title>
    <link href="https://fonts.googleapis.com/css2?family=Bricolage+Grotesque:wght@600;800&family=Figtree:wght@300;400;600&display=swap" rel="stylesheet">
    <script src="https://cdn.jsdelivr.net/npm/marked/marked.min.js"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/11.9.0/styles/tokyo-night-dark.min.css">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/11.9.0/highlight.min.js"></script>
    <script src="https://unpkg.com/lucide@latest"></script>

    <style>
        :root { --bg: #050508; --sidebar: #0d0e14; --card: #11121a; --border: #222436; --accent: #7c6cf0; --text: #e4e2ee; --dim: #8b8aa0; }
        * { margin:0; padding:0; box-sizing:border-box; }
        body { font-family:'Figtree', sans-serif; background:var(--bg); color:var(--text); height:100dvh; display:flex; overflow: hidden; }
        .sidebar { width: 260px; background: var(--sidebar); border-right: 1px solid var(--border); display: flex; flex-direction: column; padding: 15px; }
        .new-chat { background: var(--accent); color: white; border: none; padding: 12px; border-radius: 12px; cursor: pointer; font-weight: 600; margin-bottom: 20px; }
        .history-list { flex: 1; overflow-y: auto; display: flex; flex-direction: column; gap: 8px; }
        .history-item-container { position: relative; display: flex; align-items: center; border-radius: 10px; cursor: pointer; }
        .history-item-container:hover, .history-item-container.active { background: var(--card); }
        .history-link { flex: 1; padding: 12px; font-size: 13px; color: var(--dim); overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }
        .main { flex: 1; display: flex; flex-direction: column; position: relative; }
        .nav { padding: 20px; border-bottom: 1px solid var(--border); display: flex; justify-content: space-between; align-items: center; }
        .logo { font-family: 'Bricolage Grotesque'; font-size: 20px; color: var(--accent); font-weight: 800; }
        .view { flex: 1; overflow-y: auto; padding: 20px 10%; display: flex; flex-direction: column; gap: 24px; }
        .msg { max-width: 85%; line-height: 1.6; padding: 14px 18px; border-radius: 18px; }
        .msg.user { align-self: flex-end; background: var(--accent); color: white; border-bottom-right-radius: 4px; }
        .msg.bot { align-self: flex-start; background: var(--card); border: 1px solid var(--border); border-bottom-left-radius: 4px; }
        .input-box { padding: 20px 10%; border-top: 1px solid var(--border); }
        .bar { background: var(--card); border: 1px solid var(--border); border-radius: 16px; display: flex; align-items: center; padding: 8px 12px; }
        textarea { flex: 1; background: transparent; border: none; color: white; padding: 10px; outline: none; resize: none; font-family: inherit; }
        .send-btn { background: var(--accent); color: white; border: none; width: 40px; height: 40px; border-radius: 12px; cursor: pointer; }
    </style>
</head>
<body>

<div class="sidebar">
    <button class="new-chat" onclick="startNewChat()">+ New Chat</button>
    <div class="history-list" id="historyList"></div>
    <div style="font-size: 10px; color: var(--dim); text-align: center; margin-top: 10px;">DEV: SARBAGYA DEVKOTA</div>
</div>

<div class="main">
    <div class="nav">
        <div class="logo">NEXUS AI</div>
        <div id="status" style="font-size: 11px; color: #4ade80;">● Multi-Engine Active</div>
    </div>
    <div class="view" id="chatWin"></div>
    <div class="input-box">
        <div class="bar">
            <textarea id="userInput" placeholder="Ask Nexus..." rows="1"></textarea>
            <button class="send-btn" onclick="processInput()">➤</button>
        </div>
    </div>
</div>

<script>
    const chatWin = document.getElementById('chatWin');
    const inputEl = document.getElementById('userInput');
    let chats = JSON.parse(localStorage.getItem('nexus_vfinal')) || [];
    let currentId = Date.now();

    function renderMessage(role, text) {
        const div = document.createElement('div');
        div.className = `msg ${role}`;
        div.innerHTML = marked.parse(text);
        chatWin.appendChild(div);
        chatWin.scrollTop = chatWin.scrollHeight;
        return div;
    }

    async function processInput() {
        const query = inputEl.value.trim();
        if (!query) return;
        renderMessage('user', query);
        inputEl.value = '';
        const low = query.toLowerCase();

        // 1. Creator Identity (ONLY when asked)
        if (low.includes("who created you") || low.includes("who is your creator")) {
            renderMessage('bot', "I was created and developed by **Sarbagya Devkota**.");
            return;
        }

        // 2. Human Expressions
        const feels = ["hello", "hi", "how are you", "thanks", "thank you"];
        if (feels.some(f => low.startsWith(f))) {
            renderMessage('bot', "Hello! I am functioning perfectly. How can I help you with your work?");
            return;
        }

        // 3. Multi-language Translation
        if (low.includes("translate") || low.includes("how to say")) {
            const bot = renderMessage('bot', 'Translating...');
            try {
                const res = await fetch(`https://api.mymemory.translated.net/get?q=${encodeURIComponent(query)}&langpair=en|ne`);
                const data = await res.json();
                bot.innerHTML = `<b>Translation:</b> ${data.responseData.translatedText}`;
            } catch { bot.innerText = "Translation error."; }
            return;
        }

        // 4. Wikipedia Research
        const bot = renderMessage('bot', 'Researching Wikipedia...');
        try {
            const res = await fetch(`https://en.wikipedia.org/w/api.php?action=query&format=json&origin=*&prop=extracts&exintro&explaintext&titles=${encodeURIComponent(query)}`);
            const data = await res.json();
            const pages = data.query.pages;
            const pageId = Object.keys(pages)[0];
            bot.innerHTML = pageId === "-1" ? "No Wikipedia data found." : marked.parse(pages[pageId].extract);
        } catch { bot.innerText = "Connection error."; }
    }

    function startNewChat() { location.reload(); }
</script>
</body>
</html>
