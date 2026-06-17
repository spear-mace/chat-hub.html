<!DOCTYPE html>
<html lang="en" class="dark">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Chat Hub • Live</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.6.0/css/all.min.css">
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600&display=swap');
        body { font-family: 'Inter', system_ui, sans-serif; }
        .chat-container { height: 100vh; display: grid; grid-template-columns: 320px 1fr; }
        .sidebar { background: #18181b; border-right: 1px solid #3f3f46; }
        .message-bubble { max-width: 70%; padding: 12px 16px; border-radius: 20px; animation: fadeIn 0.3s; }
        .me { background: #0a84ff; color: white; border-bottom-right-radius: 4px; }
        .them { background: #27272a; border-bottom-left-radius: 4px; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
        .typing { display: flex; gap: 4px; padding: 12px 16px; background: #27272a; border-radius: 20px; width: fit-content; }
        .dot { width: 5px; height: 5px; background: #9ca3af; border-radius: 50%; animation: typingBounce 1.2s infinite; }
        .dot:nth-child(2) { animation-delay: 0.2s; }
        .dot:nth-child(3) { animation-delay: 0.4s; }
        @keyframes typingBounce { 0%, 80%, 100% { transform: translateY(0); } 40% { transform: translateY(-5px); } }
    </style>
</head>
<body class="bg-zinc-950 text-zinc-100 overflow-hidden">
    <div class="chat-container flex h-screen">
        <!-- Sidebar -->
        <div class="sidebar w-80 flex flex-col">
            <div class="p-4 border-b border-zinc-700 flex items-center justify-between bg-zinc-900">
                <h1 class="font-semibold text-xl flex items-center gap-2">
                    <i class="fa-solid fa-comments text-blue-500"></i> Chat Hub
                </h1>
                <button onclick="showNewChatModal()" class="w-9 h-9 hover:bg-zinc-800 rounded-full flex items-center justify-center">
                    <i class="fa-solid fa-plus"></i>
                </button>
            </div>
            <div class="flex-1 overflow-y-auto p-2 space-y-1" id="chat-list"></div>
            
            <div class="p-4 border-t border-zinc-700 bg-zinc-900">
                <div class="flex items-center gap-3">
                    <div class="w-9 h-9 bg-blue-600 rounded-full flex items-center justify-center font-bold">JD</div>
                    <div>
                        <p class="font-medium">Josiah Dearness</p>
                        <p class="text-xs text-emerald-400">Online</p>
                    </div>
                </div>
            </div>
        </div>

        <!-- Main Chat -->
        <div class="flex-1 flex flex-col">
            <div class="px-6 py-4 border-b border-zinc-700 bg-zinc-900 flex items-center justify-between">
                <div class="flex items-center gap-4">
                    <div class="w-11 h-11 bg-gradient-to-br from-blue-500 to-cyan-500 rounded-full flex items-center justify-center text-3xl">🤖</div>
                    <div>
                        <p id="chat-title" class="font-semibold text-lg">Chat Hub Setup</p>
                        <p class="text-emerald-400 text-sm">Online</p>
                    </div>
                </div>
                <button onclick="showInviteModal()" class="flex items-center gap-2 px-5 py-2 bg-zinc-800 hover:bg-zinc-700 rounded-2xl">
                    <i class="fa-solid fa-user-plus"></i> Invite
                </button>
            </div>

            <div id="messages" class="flex-1 overflow-y-auto p-6 space-y-6 bg-zinc-950"></div>

            <div class="p-4 bg-zinc-900 border-t border-zinc-700">
                <div class="bg-zinc-800 rounded-3xl px-5 py-2 flex items-center gap-3">
                    <input id="message-input" type="text" placeholder="Type a message..." 
                           class="flex-1 bg-transparent outline-none text-base"
                           onkeydown="if(event.key === 'Enter') sendMessage()">
                    <button onclick="sendMessage()" class="text-blue-500 hover:text-blue-400">
                        <i class="fa-solid fa-paper-plane text-xl"></i>
                    </button>
                </div>
            </div>
        </div>
    </div>

    <!-- New Chat Modal -->
    <div id="new-chat-modal" class="hidden fixed inset-0 bg-black/70 flex items-center justify-center z-50">
        <div class="bg-zinc-900 rounded-3xl w-96 p-6">
            <h2 class="text-xl font-semibold mb-4">New Chat</h2>
            <input id="new-chat-name" type="text" placeholder="Chat name" class="w-full bg-zinc-800 rounded-2xl px-4 py-3 mb-4 outline-none">
            <div class="flex gap-3">
                <button onclick="hideNewChatModal()" class="flex-1 py-3 rounded-2xl border border-zinc-700">Cancel</button>
                <button onclick="createNewChat()" class="flex-1 py-3 bg-blue-600 rounded-2xl">Create</button>
            </div>
        </div>
    </div>

    <!-- Invite Modal -->
    <div id="invite-modal" class="hidden fixed inset-0 bg-black/70 flex items-center justify-center z-50">
        <div class="bg-zinc-900 rounded-3xl w-96 p-6">
            <h2 class="text-xl font-semibold mb-4">Invite People</h2>
            <p class="text-zinc-400 mb-4">This is a demo. For real shared chat with friends, let me know and I'll make a Firebase version.</p>
            <button onclick="hideInviteModal()" class="w-full py-3 bg-zinc-800 rounded-2xl">Close</button>
        </div>
    </div>

    <script>
        let chats = [
            {
                id: 0,
                name: "Chat Hub Setup",
                messages: [
                    { from: "them", text: "👋 Hi Josiah! Welcome to Chat Hub.", time: "11:32" },
                    { from: "me", text: "Can you walk me through the full setup?", time: "11:33" },
                    { from: "them", text: "Sure thing! Here's the complete guide.", time: "11:33" }
                ]
            }
        ];
        let currentChatId = 0;

        function renderChatList() {
            const container = document.getElementById('chat-list');
            container.innerHTML = '';
            chats.forEach(chat => {
                const isActive = chat.id === currentChatId ? 'bg-zinc-800' : 'hover:bg-zinc-800';
                const el = document.createElement('div');
                el.className = `flex items-center gap-3 p-3 rounded-2xl cursor-pointer ${isActive}`;
                el.innerHTML = `
                    <div class="w-12 h-12 bg-zinc-700 rounded-full flex items-center justify-center text-2xl">👥</div>
                    <div class="flex-1">
                        <p class="font-semibold">${chat.name}</p>
                    </div>
                `;
                el.onclick = () => switchChat(chat.id);
                container.appendChild(el);
            });
        }

        function switchChat(id) {
            currentChatId = id;
            renderChatList();
            renderMessages();
        }

        function renderMessages() {
            const chat = chats.find(c => c.id === currentChatId);
            const container = document.getElementById('messages');
            container.innerHTML = '';
            chat.messages.forEach(msg => {
                const div = document.createElement('div');
                div.className = `flex ${msg.from === 'me' ? 'justify-end' : 'justify-start'}`;
                div.innerHTML = `
                    <div class="${msg.from === 'me' ? 'me' : 'them'} message-bubble">
                        <p class="whitespace-pre-line">${msg.text}</p>
                        <p class="text-[10px] mt-1 opacity-70 text-right">${msg.time}</p>
                    </div>
                `;
                container.appendChild(div);
            });
            container.scrollTop = container.scrollHeight;
        }

        function sendMessage() {
            const input = document.getElementById('message-input');
            if (!input.value.trim()) return;

            const chat = chats.find(c => c.id === currentChatId);
            const time = new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });

            chat.messages.push({ from: "me", text: input.value, time });
            renderMessages();
            input.value = '';

            // AI reply
            setTimeout(() => {
                const typing = document.createElement('div');
                typing.id = 'typing';
                typing.className = 'flex justify-start';
                typing.innerHTML = `<div class="typing"><div class="dot"></div><div class="dot"></div><div class="dot"></div></div>`;
                document.getElementById('messages').appendChild(typing);
                document.getElementById('messages').scrollTop = 9999;

                setTimeout(() => {
                    typing.remove();
                    const replies = ["Got it, Josiah!", "Anything else I can help with?", "✅ Perfect!"];
                    chat.messages.push({
                        from: "them",
                        text: replies[Math.floor(Math.random() * replies.length)],
                        time: new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' })
                    });
                    renderMessages();
                }, 1500);
            }, 600);
        }

        function showNewChatModal() { document.getElementById('new-chat-modal').classList.remove('hidden'); }
        function hideNewChatModal() { document.getElementById('new-chat-modal').classList.add('hidden'); }
        function createNewChat() {
            const name = document.getElementById('new-chat-name').value.trim() || "New Group Chat";
            const newChat = {
                id: Date.now(),
                name: name,
                messages: [{ from: "them", text: `Welcome to ${name}!`, time: "now" }]
            };
            chats.push(newChat);
            hideNewChatModal();
            renderChatList();
            switchChat(newChat.id);
        }

        function showInviteModal() {
            document.getElementById('invite-modal').classList.remove('hidden');
        }
        function hideInviteModal() {
            document.getElementById('invite-modal').classList.add('hidden');
        }

        // Initialize
        window.onload = () => {
            renderChatList();
            renderMessages();
        };
    </script>
</body>
</html>
