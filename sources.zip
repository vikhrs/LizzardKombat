Admin.html:

<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Buzzamix Admin</title>
    <style>
        body { background: #000; color: #0f0; font-family: monospace; padding: 20px; }
        .user-card, .post-card { border: 1px solid #0f0; padding: 15px; margin-bottom: 15px; background: #111; }
        button { background: #000; color: #0f0; border: 1px solid #0f0; padding: 8px 12px; cursor: pointer; margin: 2px; }
        button.danger { border-color: red; color: red; }
        button.allow { border-color: #00ff41; color: #00ff41; }
        input { background: #222; color: #0f0; border: 1px solid #0f0; padding: 10px; margin-bottom: 5px; }
        #admin-panel { display: none; }
        .verified-badge { display: inline-flex; align-items: center; justify-content: center; background: #00a8ff; color: #fff; border-radius: 50%; width: 14px; height: 14px; font-size: 10px; margin-left: 4px; font-weight: bold; vertical-align: middle; }
    </style>
</head>
<body>
    
    <div id="auth-screen" style="text-align: center; margin-top: 20vh;">
        <h1>ADMIN AUTHENTICATION</h1>
        <input type="password" id="admin-pass" placeholder="Enter Password">
        <button onclick="login()">LOGIN</button>
    </div>

    <div id="admin-panel">
        <h1>Консоль Управления Buzzamix</h1>
        
        <div style="margin-bottom: 20px;">
            <button onclick="showSection('users-section')">Пользователи</button>
            <button onclick="showSection('rooms-section')">Каналы и Группы</button>
            <button onclick="showSection('chats-section')">Чаты Пользователей</button>
            <button onclick="showSection('posts-section')">Управление Постами</button>
            <button onclick="showSection('forbidden-section')" style="border-color:red; color:red;">Запрещённые посты</button>
        </div>

        <div id="users-section">
            <h2 id="total-users-title">Все Пользователи</h2>
            <div id="users-list">Загрузка...</div>
        </div>

        <div id="rooms-section" style="display:none;">
            <h2>Все Каналы и Группы</h2>
            <div id="rooms-list">Загрузка...</div>
        </div>

        <div id="chats-section" style="display:none;">
            <h2>Переписки Пользователей</h2>
            <input id="chat-search-user" placeholder="Введите юзернейм" style="width: 250px;">
            <button onclick="loadUserChats()">Показать чаты</button>
            <div id="admin-chats-list" style="margin-top:20px;">Укажите юзернейм для поиска переписок...</div>
        </div>

        <div id="posts-section" style="display:none;">
            <h2 id="total-posts-title">Все посты платформы</h2>
            <button class="danger" onclick="deleteAllPosts()" style="margin-bottom: 15px;">Удалить ВСЕ посты платформы</button>
            <div id="posts-list">Загрузка...</div>
        </div>

        <div id="forbidden-section" style="display:none;">
            <h2>Запрещённые посты (Модерация)</h2>
            <div id="forbidden-list">Загрузка...</div>
        </div>
    </div>

    <script>
        let adminToken = '';
        const badgeHtml = `<span class="verified-badge">✓</span>`;
        let allUsersCache = {};
        let allRoomsCache = {};

        async function login() {
            const pwd = document.getElementById('admin-pass').value;
            const res = await fetch('/api/admin/users', { headers: { 'x-admin-pass': pwd } });
            if (res.status === 403) { alert('ДОСТУП ЗАПРЕЩЕН'); } else {
                adminToken = pwd;
                document.getElementById('auth-screen').style.display = 'none';
                document.getElementById('admin-panel').style.display = 'block';
                loadAll();
                setInterval(loadAll, 2000); // Автообновление
            }
        }

        function loadAll() {
            if (!adminToken) return;
            // Чтобы не выкидывало из меню ввода при накрутке, останавливаем обновление, если активен input
            if (document.activeElement && document.activeElement.tagName === 'INPUT') return;
            
            loadUsers(); loadRooms(); loadPosts(); loadForbidden();
        }

        function showSection(id) {
            ['users-section', 'posts-section', 'forbidden-section', 'rooms-section', 'chats-section'].forEach(s => {
                document.getElementById(s).style.display = 'none';
            });
            document.getElementById(id).style.display = 'block';
        }

        function renderUserCard(u) {
            return `
                <div class="user-card">
                    <h2 style="display:flex; align-items:center;">
                        <img src="${u.photo}" style="width:30px;height:30px;border-radius:50%;margin-right:10px;">
                        ${u.name} (@${u.username}) ${u.isVerified ? badgeHtml : ''} ${u.isBlocked ? '🚫 БАН' : ''}
                    </h2>
                    <p>Подписчиков: ${u.followers ? u.followers.length : 0} | Страйки: ${u.strikes || 0}</p>
                    
                    <div>
                        <input type="number" id="boost-user-${u.username}" placeholder="Кол-во" style="width:100px; padding:5px;">
                        <button onclick="boostFollowers('${u.username}', 'user')">Накрутить подписчиков</button>
                    </div>

                    <button onclick="actUser('${u.username}', 'block', ${!u.isBlocked})">${u.isBlocked ? 'Разблокировать' : 'Заблокировать'}</button>
                    <button onclick="actUser('${u.username}', 'verify', ${!u.isVerified})">${u.isVerified ? 'Снять галочку' : 'Выдать галочку'}</button>
                </div>
            `;
        }

        async function loadUsers() {
            const res = await fetch('/api/admin/users', { headers: { 'x-admin-pass': adminToken } });
            allUsersCache = await res.json();
            document.getElementById('total-users-title').innerText = `Всего зарегистрировано людей: ${Object.keys(allUsersCache).length}`;
            document.getElementById('users-list').innerHTML = Object.values(allUsersCache).map(renderUserCard).join('');
        }

        async function loadRooms() {
            const res = await fetch('/api/admin/rooms', { headers: { 'x-admin-pass': adminToken } });
            allRoomsCache = await res.json();
            document.getElementById('rooms-list').innerHTML = Object.keys(allRoomsCache).map(k => {
                const r = allRoomsCache[k];
                return `<div class="user-card">
                    <h2>@${k} (${r.type}) ${r.isVerified ? badgeHtml : ''} ${r.isBlocked ? '🚫 БАН' : ''}</h2>
                    <p>Участников: ${r.members ? r.members.length : 0}</p>
                    
                    <div>
                        <input type="number" id="boost-room-${k}" placeholder="Кол-во" style="width:100px; padding:5px;">
                        <button onclick="boostFollowers('${k}', 'room')">Накрутить участников</button>
                    </div>

                    <button onclick="actRoom('${k}', 'verify', ${!r.isVerified})">${r.isVerified ? 'Снять галочку' : 'Выдать галочку'}</button>
                    <button onclick="actRoom('${k}', 'block', ${!r.isBlocked})">${r.isBlocked ? 'Разблокировать' : 'Заблокировать'}</button>
                    <button class="danger" onclick="actRoom('${k}', 'delete', true)">Удалить полностью</button>
                </div>`;
            }).join('');
        }

        async function boostFollowers(target, type) {
            const count = document.getElementById(`boost-${type}-${target}`).value;
            if(!count || count <= 0) return alert("Введите корректное число");
            await fetch('/api/admin/boost', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json', 'x-admin-pass': adminToken },
                body: JSON.stringify({ target, type, count: parseInt(count) })
            });
            alert('Накручено!');
            loadAll();
        }

        async function loadUserChats() {
            const u = document.getElementById('chat-search-user').value.toLowerCase().trim();
            if(!u) return;
            const res = await fetch(`/api/admin/chats/${u}`, { headers: { 'x-admin-pass': adminToken } });
            const chats = await res.json();
            if(chats.error) return alert(chats.error);
            
            if (chats.length === 0) {
                document.getElementById('admin-chats-list').innerHTML = '<p>Переписок не найдено.</p>';
                return;
            }

            let html = '';
            chats.forEach(c => {
                html += `<div class="post-card"><b style="color: #00ff41;">Чат: @${c.u1} и @${c.u2}</b><hr style="border-color: #333;">`;
                c.msgs.forEach(m => {
                    const time = new Date(m.time).toLocaleString();
                    html += `<div style="margin-bottom: 5px;">[<span style="color:gray;font-size:12px;">${time}</span>] <b style="color:#00e5ff">${m.sender}</b>: ${m.text ? m.text : '📎 <i>Медиафайл</i>'}</div>`;
                });
                html += `</div>`;
            });
            document.getElementById('admin-chats-list').innerHTML = html;
        }

        async function actUser(u, action, value) {
            await fetch('/api/admin/action', { method: 'POST', headers: { 'Content-Type': 'application/json', 'x-admin-pass': adminToken }, body: JSON.stringify({ username: u, action, value }) });
            loadAll();
        }

        async function actRoom(roomId, action, value) {
            if(action === 'delete' && !confirm('Точно удалить комнату?')) return;
            await fetch('/api/admin/roomAction', { method: 'POST', headers: { 'Content-Type': 'application/json', 'x-admin-pass': adminToken }, body: JSON.stringify({ roomId, action, value }) });
            loadAll();
        }

        async function loadPosts() {
            const res = await fetch('/api/admin/posts', { headers: { 'x-admin-pass': adminToken } });
            const posts = await res.json();
            document.getElementById('total-posts-title').innerText = `Всего постов опубликовано: ${posts.length}`;
            if (posts.length === 0) { document.getElementById('posts-list').innerHTML = "Лента пуста."; return; }
            document.getElementById('posts-list').innerHTML = posts.map(p => `
                <div class="post-card"><b>@${p.author}</b><br><p>${p.content}</p><button class="danger" onclick="deleteSinglePost('${p.id}')">Удалить пост</button></div>
            `).join('');
        }

        async function loadForbidden() {
            const res = await fetch('/api/admin/forbidden', { headers: { 'x-admin-pass': adminToken } });
            const posts = await res.json();
            if (posts.length === 0) { document.getElementById('forbidden-list').innerHTML = "Нет постов на модерации."; return; }
            document.getElementById('forbidden-list').innerHTML = posts.map(p => `
                <div class="post-card" style="border-color:red;">
                    <b>Нарушитель: @${p.author}</b><br>
                    <p>Текст: ${p.content}</p>
                    <button class="allow" onclick="allowPost(${p.id})">Разрешить публикацию</button>
                </div>
            `).join('');
        }

        async function allowPost(id) {
            await fetch(`/api/admin/allowPost`, { method: 'POST', headers: { 'Content-Type': 'application/json', 'x-admin-pass': adminToken }, body: JSON.stringify({ id }) });
            loadAll();
        }

        async function deleteSinglePost(id) {
            if(!confirm('Удалить этот пост?')) return;
            await fetch(`/api/admin/posts/${id}`, { method: 'DELETE', headers: { 'x-admin-pass': adminToken } });
            loadAll();
        }

        async function deleteAllPosts() {
            if(!confirm('ВНИМАНИЕ! Точно удалить ВСЕ посты?')) return;
            await fetch(`/api/admin/posts`, { method: 'DELETE', headers: { 'x-admin-pass': adminToken } });
            loadAll();
        }
    </script>
</body>
</html>


index.html:

<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Buzzamix</title>
    <script src="/socket.io/socket.io.js"></script>
    <script>
        const UI = {
            showAlert: (text, title = "Внимание") => {
                document.getElementById('modal-title').innerText = title;
                document.getElementById('modal-text').innerHTML = text;
                document.getElementById('modal-cancel-btn').style.display = 'none';
                document.getElementById('custom-modal').classList.remove('hidden');
                document.getElementById('modal-ok-btn').onclick = () => {
                    document.getElementById('custom-modal').classList.add('hidden');
                };
            },
            showConfirm: (text, onConfirm, title = "Подтверждение") => {
                document.getElementById('modal-title').innerText = title;
                document.getElementById('modal-text').innerHTML = text;
                document.getElementById('modal-cancel-btn').style.display = 'inline-block';
                document.getElementById('custom-modal').classList.remove('hidden');
                document.getElementById('modal-ok-btn').onclick = () => {
                    document.getElementById('custom-modal').classList.add('hidden');
                    if(onConfirm) onConfirm();
                };
                document.getElementById('modal-cancel-btn').onclick = () => {
                    document.getElementById('custom-modal').classList.add('hidden');
                };
            },
            showPrompt: (text, onConfirm, title = "Ввод") => {
                document.getElementById('modal-title').innerText = title;
                document.getElementById('modal-text').innerHTML = `<div style="margin-bottom:10px;">${text}</div><input type="text" id="modal-prompt-input" style="width:100%; box-sizing:border-box; padding:10px; border-radius:5px; border:1px solid var(--border); background:var(--bg); color:var(--text);">`;
                document.getElementById('modal-cancel-btn').style.display = 'inline-block';
                document.getElementById('custom-modal').classList.remove('hidden');
                
                setTimeout(() => {
                    const inp = document.getElementById('modal-prompt-input');
                    if(inp) inp.focus();
                }, 100);

                document.getElementById('modal-ok-btn').onclick = () => {
                    const val = document.getElementById('modal-prompt-input').value;
                    document.getElementById('custom-modal').classList.add('hidden');
                    if(onConfirm) onConfirm(val);
                };
                document.getElementById('modal-cancel-btn').onclick = () => {
                    document.getElementById('custom-modal').classList.add('hidden');
                };
            }
        };

        const E2EE = {
            p: 340282366920938463463374607431768211297n, 
            modPow: function(b, e, m) { 
                let r = 1n; b = BigInt(b) % m; e = BigInt(e); 
                while(e > 0n) { 
                    if(e % 2n === 1n) r = (r * b) % m; 
                    e /= 2n; b = (b * b) % m; 
                } 
                return r; 
            },
            getSessionKey: (chatId, type) => {
                if (!App.me || !chatId || !App.db) return "default_key";
                
                if (type === 'room' || App.db.rooms[chatId]) {
                    let hash = 0;
                    for (let i = 0; i < chatId.length; i++) {
                        hash = (hash << 5) - hash + chatId.charCodeAt(i);
                        hash |= 0;
                    }
                    return "e2ee_room_" + Math.abs(hash);
                }

                if (!App.db.users[App.me] || !App.db.users[chatId]) return "default_key";
                if (!App.db.users[App.me].keys || !App.db.users[chatId].keys) return "default_key";

                try {
                    const myPriv = BigInt("0x" + App.db.users[App.me].keys.private);
                    const theirPub = BigInt("0x" + App.db.users[chatId].keys.public);
                    
                    const sharedSecret = E2EE.modPow(theirPub, myPriv, E2EE.p);
                    return "session_" + sharedSecret.toString(16);
                } catch(e) {
                    return "default_key";
                }
            },
            encrypt: (text, chatId, type = 'dm') => {
                if(!text) return text;
                const key = E2EE.getSessionKey(chatId, type);
                const iv = Math.random().toString(36).substring(2, 10);
                const fullKey = key + iv;
                let res = '';
                for(let i=0; i<text.length; i++) {
                    res += String.fromCharCode(text.charCodeAt(i) ^ fullKey.charCodeAt(i % fullKey.length));
                }
                try { return iv + ':' + btoa(unescape(encodeURIComponent(res))); } catch(e) { return text; }
            },
            decrypt: (encText, chatId, type = 'dm') => {
                if(!encText) return encText;
                if(!encText.includes(':')) {
                    try {
                        const govKey = "buzzamix_gov_key_2026_e2ee_voip";
                        const text = decodeURIComponent(escape(atob(encText)));
                        let res = '';
                        for(let i=0; i<text.length; i++) res += String.fromCharCode(text.charCodeAt(i) ^ govKey.charCodeAt(i % govKey.length));
                        return res;
                    } catch(e) { return encText; }
                }
                try {
                    const [iv, b64] = encText.split(':');
                    const key = E2EE.getSessionKey(chatId, type);
                    const fullKey = key + iv;
                    const text = decodeURIComponent(escape(atob(b64)));
                    let res = '';
                    for(let i=0; i<text.length; i++) {
                        res += String.fromCharCode(text.charCodeAt(i) ^ fullKey.charCodeAt(i % fullKey.length));
                    }
                    return res;
                } catch(e) { return encText; }
            }
        };

        const isAuth = !!localStorage.getItem('v_user');
        document.documentElement.style.setProperty('--auth-display', isAuth ? 'none' : 'block');
        document.documentElement.style.setProperty('--app-display', isAuth ? 'block' : 'none');
        
        if ('serviceWorker' in navigator) {
            navigator.serviceWorker.register('/sw.js').catch(err => console.log('SW setup failed', err));
            
            navigator.serviceWorker.addEventListener('message', event => {
                const { action, data } = event.data;
                if (!data) return;

                const checkInterval = setInterval(() => {
                    if (App.db && App.me) {
                        clearInterval(checkInterval);
                        
                        if (data.type === 'dm' || data.type === 'room') {
                            App.openChat(data.user, data.type);
                        } else if (data.type === 'call') {
                            if (action === 'accept') {
                                Call.pendingAccept = data.caller;
                                if (Call.incomingData && Call.incomingData.caller === data.caller) Call.accept();
                            } else if (action === 'decline') {
                                socket.emit('call-busy', { target: data.caller, from: App.me });
                                Call.end();
                            } else {
                                if (!Call.incomingData) {
                                    Call.answer({ caller: data.caller, video: true, offer: null });
                                }
                            }
                        }
                    }
                }, 200);
            });
        }

        const publicVapidKey = 'BGbMVqwr1UuB8ifCZTR_7erTZ6pLJUhoG4NV9b1g0aaT_9H1cExNnJK9CRoQbpusZ6i38HP4Zsl5bvCKa028c2c';

        function urlBase64ToUint8Array(base64String) {
            const padding = '='.repeat((4 - base64String.length % 4) % 4);
            const base64 = (base64String + padding).replace(/\-/g, '+').replace(/_/g, '/');
            const rawData = window.atob(base64);
            const outputArray = new Uint8Array(rawData.length);
            for (let i = 0; i < rawData.length; ++i) { outputArray[i] = rawData.charCodeAt(i); }
            return outputArray;
        }

        async function subscribeToPush() {
            if ('serviceWorker' in navigator && 'PushManager' in window) {
                try {
                    const reg = await navigator.serviceWorker.ready;
                    const sub = await reg.pushManager.subscribe({
                        userVisibleOnly: true,
                        applicationServerKey: urlBase64ToUint8Array(publicVapidKey)
                    });
                    await fetch('/api/subscribe', {
                        method: 'POST',
                        body: JSON.stringify({ subscription: sub, username: localStorage.getItem('v_user') }),
                        headers: { 'Content-Type': 'application/json' }
                    });
                } catch(e) { console.error('Ошибка подписки на Push', e); }
            }
        }
    </script>
    <style>
        :root { --bg: #000000; --text: #00e5ff; --panel: #0a0a0a; --border: #0077ff; --accent: #0077ff; }
        .light { --bg: #ffffff; --text: #000000; --panel: #f4f4f4; --border: #cccccc; --accent: #00a8ff; }
        
        body { background: var(--bg); color: var(--text); font-family: -apple-system, BlinkMacSystemFont, Roboto, sans-serif; margin: 0; padding-bottom: 120px; transition: 0.3s; }
        * { box-sizing: border-box; }
        .hidden { display: none !important; }
        
        #auth-screen { display: var(--auth-display); }
        #main-app { display: var(--app-display); }
        
        header { background: var(--panel); padding: 15px; font-weight: bold; border-bottom: 1px solid var(--border); position: sticky; top: 0; z-index: 50; display: flex; justify-content: space-between; align-items: center; }
        .header-btn { width: 33%; cursor: pointer; }
        .header-title { width: 34%; text-align: center; font-size: 1.2em; color: var(--accent); }
        .header-theme { width: 33%; text-align: right; cursor: pointer; font-size: 1.2em; }

        .burger-menu { position: fixed; top: 0; left: -100%; width: 250px; height: 100vh; background: var(--panel); z-index: 100; transition: 0.3s; padding: 20px; border-right: 1px solid var(--border); display: flex; flex-direction: column; overflow-y: auto;}
        .burger-menu.open { left: 0; }
        
        .nav-bar { position: fixed; bottom: 0; width: 100%; display: flex; background: var(--panel); border-top: 1px solid var(--border); z-index: 50; padding-bottom: env(safe-area-inset-bottom, 0px); }
        .nav-btn { flex: 1; text-align: center; padding: 15px 0; cursor: pointer; font-weight: bold; position: relative; font-size: 14px; }
        .nav-badge { background: red; color: white; border-radius: 50%; padding: 2px 6px; font-size: 10px; position: absolute; top: 5px; right: 20%; display: none; }
        
        .screen { padding: 15px; }
        input, textarea, select { width: 100%; padding: 12px; margin-bottom: 10px; background: var(--bg); color: var(--text); border: 1px solid var(--border); border-radius: 5px; font-size: 16px; }
        input:focus, textarea:focus, select:focus { outline: none; border-color: var(--text); }
        .btn { width: 100%; padding: 15px; background: var(--accent); color: #fff; border: none; border-radius: 5px; font-weight: bold; font-size: 16px; cursor: pointer; margin-bottom: 10px; }
        .btn:disabled { opacity: 0.6; cursor: not-allowed; }
        .btn-outline { background: transparent; color: var(--text); border: 1px solid var(--border); }
        .btn-danger { background: transparent; color: #ff3333; border: 1px solid #ff3333; }
        
        .post-card { background: var(--panel); border: 1px solid var(--border); border-radius: 8px; padding: 15px; margin-bottom: 15px; }
        .post-author { font-weight: bold; color: var(--accent); cursor: pointer; display: inline-block; margin-bottom: 5px; }
        .reactions-box { display: flex; gap: 10px; margin-top: 10px; font-size: 14px; }
        .reaction-btn { background: var(--bg); border: 1px solid var(--border); border-radius: 20px; padding: 4px 10px; cursor: pointer; color: var(--text); }
        
        .chat-list-item { padding: 15px; border-bottom: 1px solid var(--border); display: flex; align-items: center; cursor: pointer; }
        .chat-list-img { width: 40px; height: 40px; border-radius: 50%; margin-right: 15px; object-fit: cover; border: 1px solid var(--accent); }
        .unread-badge { background: red; color: white; padding: 2px 6px; border-radius: 10px; font-size: 12px; margin-left: auto; }
        
        .msg-bubble { max-width: 80%; padding: 10px; border-radius: 10px; margin-bottom: 10px; clear: both; word-wrap: break-word; }
        .msg-me { background: var(--accent); color: #fff; float: right; }
        .msg-them { background: var(--panel); border: 1px solid var(--border); float: left; color: var(--text); }
        
        .avatar { width: 80px; height: 80px; border-radius: 50%; object-fit: cover; border: 2px solid var(--accent); }
        .verified-badge { display: inline-flex; align-items: center; justify-content: center; background: #00a8ff; color: #fff; border-radius: 50%; width: 16px; height: 16px; font-size: 11px; margin-left: 4px; font-weight: bold; vertical-align: middle; }

        #chat-header {
            display: flex; justify-content: space-between; align-items: center;
            padding: 10px 15px; border-bottom: 1px solid var(--border);
            margin: -15px -15px 10px -15px; position: sticky; top: 51px;
            background: var(--bg); z-index: 45;
        }

        #call-overlay { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.95); z-index: 200; display: none; flex-direction: column; align-items: center; justify-content: center; color: white; }
        #call-videos { display: flex; flex-direction: column; align-items: center; width: 100%; }
        #call-remote-videos { display: flex; flex-wrap: wrap; justify-content: center; gap: 10px; margin-bottom: 10px; max-height: 50vh; overflow-y: auto; }
        .remote-vid-element { width: 90%; max-width: 300px; max-height: 40vh; background: transparent; border-radius: 10px; border: 2px solid var(--accent); object-fit: cover; }
        
        #local-video { width: 100px; height: 150px; background: transparent; border-radius: 10px; object-fit: cover; border: 1px solid var(--border); transform: scaleX(-1); }
        
        .call-controls { display: flex; gap: 10px; margin-top: 20px; flex-wrap: wrap; justify-content: center; }
        .call-btn { padding: 10px 15px; border-radius: 5px; border: none; cursor: pointer; font-weight: bold; }
        #password-modal { position: fixed; top: 30%; left: 10%; right: 10%; background: var(--panel); border: 1px solid var(--border); padding: 20px; z-index: 1000; border-radius: 8px; }
        
        #custom-modal { position: fixed; top:0; left:0; width:100%; height:100%; background:rgba(0,0,0,0.8); z-index:9999; display:flex; align-items:center; justify-content:center; }
        #custom-modal-content { background:var(--panel); padding:20px; border-radius:8px; border:1px solid var(--border); width:80%; max-width:400px; text-align:center; box-shadow: 0 4px 15px rgba(0,0,0,0.5); }
    </style>
</head>
<body>

<div id="custom-modal" class="hidden">
    <div id="custom-modal-content">
        <h3 id="modal-title" style="margin-top:0;"></h3>
        <div id="modal-text"></div>
        <div style="display:flex; gap:10px; justify-content:center; margin-top:20px;">
            <button id="modal-ok-btn" class="btn" style="margin:0;">OK</button>
            <button id="modal-cancel-btn" class="btn btn-outline" style="margin:0;">Отмена</button>
        </div>
    </div>
</div>

<div id="call-overlay">
    <div style="color: #00ff41; font-size: 14px; margin-bottom: 5px; text-align: center;">🔒 E2EE WebRTC Encrypted<br><span style="font-size:10px;">(Аудио/Видео полностью зашифрованы)</span></div>
    <h2 id="call-status">Звонок...</h2>
    <h3 id="call-target-name" style="color: var(--accent); text-align: center; padding: 0 10px;"></h3>
    <div id="call-videos">
        <div id="call-remote-videos"></div>
        <video id="local-video" autoplay playsinline muted></video>
    </div>
    <div class="call-controls">
        <button id="call-accept-btn" class="call-btn" style="background: #00ff41; color: black; display: none;" data-i18n="accept">Принять</button>
        <button id="call-mic-btn" class="call-btn" style="background: var(--panel); color: var(--text); border: 1px solid var(--border);" onclick="Call.toggleMute()">🎤 <span data-i18n="mic">Микрофон</span></button>
        <button id="call-cam-btn" class="call-btn" style="background: var(--panel); color: var(--text); border: 1px solid var(--border);" onclick="Call.switchCamera()">🔄 <span data-i18n="cam">Камера</span></button>
        <button id="call-spk-btn" class="call-btn" style="background: var(--panel); color: var(--text); border: 1px solid var(--border);" onclick="Call.switchSpeaker()">🔊 <span data-i18n="speaker">Динамик</span></button>
        <button class="call-btn" style="background: #ff3333; color: white;" onclick="Call.end()" data-i18n="decline">Отбой</button>
    </div>
</div>

<div id="auth-screen" class="screen">
    <h1 style="text-align: center; color: var(--accent);">Buzzamix</h1>
    <div id="reg-box">
        <input id="reg-n" placeholder="Имя (Обязательно)" data-i18n="name_req">
        <input id="reg-u" placeholder="Username (От 5 букв, без пробелов)" data-i18n="username_req">
        <input id="reg-p" type="password" placeholder="Пароль" data-i18n="password">
        <button class="btn" onclick="App.auth('reg')" data-i18n="register">Регистрация</button>
        <button class="btn btn-outline" onclick="App.toggleAuth()" data-i18n="already_have">Уже есть аккаунт? Войти</button>
    </div>
    <div id="login-box" class="hidden">
        <input id="log-u" placeholder="Username" data-i18n="username">
        <input id="log-p" type="password" placeholder="Пароль" data-i18n="password">
        <button class="btn" onclick="App.auth('login')" data-i18n="login">Войти</button>
        <button class="btn btn-outline" onclick="App.toggleAuth()" data-i18n="no_account">Нет аккаунта? Регистрация</button>
    </div>
</div>

<div id="main-app">
    <header>
        <div class="header-btn" onclick="App.toggleMenu()" data-i18n="menu_btn">☰ Меню</div>
        <div class="header-title">Buzzamix</div>
        <div class="header-theme" onclick="App.toggleTheme()">🌓</div>
    </header>

    <div id="burger" class="burger-menu">
        <div>
            <h2 style="margin-top:0; cursor:pointer;" onclick="App.toggleMenu()" data-i18n="close">✕ Закрыть</h2>
            <button class="btn btn-outline" onclick="App.nav('create-post')" data-i18n="create_post">Опубликовать пост</button>
            <button class="btn btn-outline" onclick="App.nav('search')" data-i18n="search_people">🔍 Поиск Людей</button>
            <button class="btn btn-outline" onclick="App.nav('search-room')" data-i18n="search_rooms">📢 Поиск Групп/Каналов</button>
            
            <hr style="border-color: var(--border); margin: 20px 0;">
            <p style="text-align: center;">Official channel:<br><b style="color:var(--accent); cursor:pointer; font-size: 1.1em;" onclick="App.handleMention('buzzamix');">@buzzamix</b></p>
            <button class="btn btn-danger" style="margin-top: auto;" onclick="App.logout()" data-i18n="logout">Выйти</button>
        </div>
    </div>

    <div id="scr-feed" class="screen"></div>
    
    <div id="scr-chats" class="screen hidden">
        <div style="display:flex; gap:10px;">
            <button class="btn" onclick="App.startNewChat()" data-i18n="write_msg">+ Написать</button>
            <button class="btn btn-outline" onclick="App.nav('create-room', 'group')" data-i18n="group">👥 Группа</button>
            <button class="btn btn-outline" onclick="App.nav('create-room', 'channel')" data-i18n="channel">📢 Канал</button>
        </div>
        <div id="chats-list"></div>
    </div>

    <div id="scr-chat-room" class="screen hidden" style="padding-bottom: 120px;">
        <div id="chat-header">
            <div id="chat-title" style="cursor:pointer; flex:1;"></div>
            <div id="chat-call-btns" style="display:flex; gap:5px;"></div>
        </div>
        <div id="chat-messages" style="display: flow-root;"></div>
        
        <div id="chat-input-area" style="position: fixed; bottom: 50px; left: 0; width: 100%; padding: 10px 10px 25px 10px; background: var(--bg); display: flex; align-items: center; border-top: 1px solid var(--border); z-index: 40;">
            <input type="file" id="chat-media-upload" accept="image/*,video/*" style="display: none;" onchange="App.sendMedia()">
            <button class="btn btn-outline" style="width:auto; margin:0 5px 0 0; padding:10px;" onclick="document.getElementById('chat-media-upload').click()">📎</button>
            
            <input id="msg-text" placeholder="Сообщение..." style="margin-bottom:0; flex:1;" data-i18n="msg_placeholder">
            <button class="btn" style="width:auto; margin:0 0 0 5px; padding:10px;" onclick="App.sendMsg()">➤</button>
        </div>
    </div>

    <div id="scr-profile" class="screen hidden">
        <div id="profile-header" style="text-align: center; margin-bottom: 20px; padding-top: 25px;"></div>
        <div id="profile-stats" style="text-align: center; margin-bottom: 20px; font-weight: bold; color: var(--text);"></div>
        <h3 style="border-bottom: 1px solid var(--border); padding-bottom: 5px;" data-i18n="posts_title">Посты:</h3>
        <div id="profile-posts"></div>
    </div>

    <div id="scr-settings" class="screen hidden">
        <h2 data-i18n="settings">Настройки</h2>
        
        <hr style="border-color: var(--border); margin: 20px 0;">
        <h3 data-i18n="lang">Язык / Language</h3>
        <select id="lang-select" onchange="App.changeLang()">
            <option value="ru">Русский</option>
            <option value="en">English</option>
        </select>

        <hr style="border-color: var(--border); margin: 20px 0;">
        <h3 data-i18n="security">Безопасность</h3>
        <button class="btn btn-outline" onclick="App.openPasswordModal()" data-i18n="change_pwd">Изменить пароль</button>
        <button class="btn btn-danger" onclick="App.deleteAccount()" data-i18n="del_acc">Удалить аккаунт навсегда</button>

        <hr style="border-color: var(--border); margin: 20px 0;">
        <h3 data-i18n="support">Служба поддержки</h3>
        <p style="margin:5px 0;">Support: <b style="color:var(--accent); cursor:pointer; font-size:18px;" onclick="App.openChat('support', 'dm')">@support</b></p>
        <p style="margin:5px 0;">Email: buzzamix.support@gmail.com</p>

        <hr style="border-color: var(--border); margin: 20px 0;">
        <h3 data-i18n="proxy">Настройки Прокси</h3>
        <p style="margin:5px 0; font-size:14px;">Status: <span id="proxy-status" style="color:gray; font-weight:bold;">Выкл</span></p>
        <select id="proxy-type">
            <option value="http">HTTP Proxy</option>
            <option value="socks5">SOCKS5</option>
        </select>
        <input id="proxy-ip" placeholder="IP Прокси" data-i18n="ip_proxy">
        <input id="proxy-port" placeholder="Порт" data-i18n="port">
        <input id="proxy-login" placeholder="Логин (необязательно)" data-i18n="login_opt">
        <input id="proxy-pass" type="password" placeholder="Пароль (необязательно)" data-i18n="pass_opt">
        <button class="btn btn-outline" onclick="App.setupProxy()" data-i18n="connect">Подключить</button>
    </div>

    <div id="password-modal" class="hidden">
        <h3 style="margin-top:0;" data-i18n="change_pwd">Смена пароля</h3>
        <input type="password" id="new-password-input" placeholder="Введите новый пароль" data-i18n="new_pwd">
        <button class="btn" onclick="App.submitNewPassword()" data-i18n="save">Сохранить</button>
        <button class="btn btn-outline" onclick="document.getElementById('password-modal').classList.add('hidden')" data-i18n="cancel">Отмена</button>
    </div>

    <div id="scr-room-info" class="screen hidden">
        <div id="room-info-content" style="text-align:center;"></div>
    </div>

    <div id="scr-create-post" class="screen hidden">
        <h2 data-i18n="new_post">Новый пост</h2>
        <textarea id="new-post-text" rows="4" placeholder="Что нового?" data-i18n="what_new"></textarea>
        <label data-i18n="attach">Прикрепить фото/видео:</label>
        <input type="file" id="new-post-file" accept="image/*,video/*" style="margin-bottom: 15px; display: block;">
        <button class="btn" onclick="App.publishPost()" data-i18n="publish">Опубликовать</button>
    </div>

    <div id="scr-create-room" class="screen hidden">
        <h2 id="create-room-title" data-i18n="create">Создать</h2>
        <input id="room-name" placeholder="Название" data-i18n="name">
        
        <div style="display:flex; align-items:center; margin-bottom:10px;">
            <input type="checkbox" id="room-private" style="width:auto; margin:0 10px 0 0;" onchange="App.togglePrivateRoom()">
            <label for="room-private" data-i18n="private_room">Приватная (случайная ссылка)</label>
        </div>
        
        <input id="room-username" placeholder="Юзернейм (от 5 букв, без пробелов)" data-i18n="username_req">
        <textarea id="room-desc" placeholder="Описание" data-i18n="desc"></textarea>
        <label data-i18n="avatar">Аватарка:</label>
        <input type="file" id="room-photo" accept="image/*" style="margin-bottom: 15px; display: block;">
        <button class="btn" onclick="App.saveRoom()" data-i18n="save">Сохранить</button>
    </div>

    <div id="scr-edit-profile" class="screen hidden">
        <h2 data-i18n="edit_profile">Редактировать профиль</h2>
        <input id="edit-name" placeholder="Имя" data-i18n="name">
        <textarea id="edit-bio" placeholder="О себе" data-i18n="about_me"></textarea>
        <label data-i18n="new_avatar">Новая аватарка:</label>
        <input type="file" id="edit-avatar" accept="image/*" style="margin-bottom: 20px; display: block;">
        <button class="btn" onclick="App.saveProfile()" data-i18n="save_profile">Сохранить профиль</button>
    </div>

    <div id="scr-search" class="screen hidden">
        <h2 data-i18n="search_people">Поиск Людей</h2>
        <input id="search-input" placeholder="Введите username" data-i18n="enter_username">
        <button class="btn" onclick="App.searchUser()" data-i18n="find">Найти</button>
    </div>

    <div id="scr-search-room" class="screen hidden">
        <h2 data-i18n="search_rooms">Поиск Групп/Каналов</h2>
        <input id="search-room-input" placeholder="Введите username группы/канала" data-i18n="enter_room">
        <button class="btn" onclick="App.searchRoom()" data-i18n="find">Найти</button>
    </div>

    <nav class="nav-bar">
        <div class="nav-btn" onclick="App.nav('feed')" data-i18n="feed">Лента</div>
        <div class="nav-btn" onclick="App.nav('chats')"><span data-i18n="chats">Чаты</span> <span id="nav-badge" class="nav-badge">0</span></div>
        <div class="nav-btn" onclick="App.nav('settings')" data-i18n="settings">Настройки</div>
        <div class="nav-btn" onclick="App.viewUser(App.me)" data-i18n="profile">Профиль</div>
    </nav>
</div>

<script>
    const socket = io();
    const badgeHtml = `<span class="verified-badge">✓</span>`;

    document.addEventListener('click', () => {
        if ("Notification" in window && Notification.permission !== "granted" && Notification.permission !== "denied") {
            Notification.requestPermission();
        }
    }, { once: true });

    document.getElementById('reg-u').addEventListener('input', function(e) { e.target.value = e.target.value.replace(/\s/g, ''); });
    document.getElementById('room-username').addEventListener('input', function(e) { e.target.value = e.target.value.replace(/\s/g, ''); });

    const Call = {
        localStream: null,
        peers: {},
        isVideo: true,
        target: null,
        roomTarget: null,
        iceQueue: {},
        incomingData: null,
        pendingAccept: null,
        facingMode: 'user',
        startTime: null,
        callAnswered: false,
        isCaller: false,
        config: { iceServers: [
            { urls: 'stun:stun.l.google.com:19302' },
            { urls: 'stun:stun1.l.google.com:19302' },
            { urls: 'stun:stun2.l.google.com:19302' },
            { urls: 'stun:stun3.l.google.com:19302' },
            { urls: 'stun:stun4.l.google.com:19302' }
        ] },
        
        async start(targetId, video, isRoom = false) {
            try {
                this.isVideo = video;
                this.target = isRoom ? null : targetId;
                this.roomTarget = isRoom ? targetId : null;
                this.isCaller = true;
                this.callAnswered = false;
                this.startTime = null;
                
                document.getElementById('call-overlay').style.display = 'flex';
                document.getElementById('call-target-name').innerText = targetId;
                document.getElementById('call-accept-btn').style.display = 'none';
                document.getElementById('call-status').innerText = App.lang === 'ru' ? 'Звоним...' : 'Calling...';
                document.getElementById('call-cam-btn').style.display = video ? 'inline-block' : 'none';
                document.getElementById('call-accept-btn').onclick = () => this.accept();
                
                const streamOk = await this.initStream();
                if (!streamOk) return this.end();

                if (isRoom) {
                    socket.emit('call-room', { room: targetId, caller: App.me, video });
                } else {
                    await this.createPeer(targetId, true);
                }
            } catch (error) { console.error(error); this.end(); }
        },
        
        async answer(data) {
            this.incomingData = data;
            this.target = data.caller; this.isVideo = data.video;
            this.isCaller = false;
            this.callAnswered = false;
            document.getElementById('call-overlay').style.display = 'flex';
            document.getElementById('call-target-name').innerText = data.caller;
            document.getElementById('call-status').innerText = App.lang === 'ru' ? 'Входящий звонок...' : 'Incoming call...';
            document.getElementById('call-accept-btn').style.display = 'inline-block';
            document.getElementById('call-accept-btn').onclick = () => this.accept();
            document.getElementById('call-cam-btn').style.display = this.isVideo ? 'inline-block' : 'none';
        },
        
        async accept() {
            try {
                document.getElementById('call-status').innerText = App.lang === 'ru' ? 'Соединение...' : 'Connecting...';
                document.getElementById('call-accept-btn').style.display = 'none';
                this.callAnswered = true;
                this.startTime = Date.now();
                
                const streamOk = await this.initStream();
                if (!streamOk) return this.end();

                if (this.incomingData && this.incomingData.offer) {
                    await this.createPeer(this.incomingData.caller, false, this.incomingData.offer);
                }
            } catch (error) { console.error(error); this.end(); }
        },

        answerRoom(data) {
            this.incomingData = data;
            this.roomTarget = data.room; this.isVideo = data.video;
            this.isCaller = false;
            this.callAnswered = false;
            document.getElementById('call-overlay').style.display = 'flex';
            document.getElementById('call-target-name').innerText = `${data.room} (${data.caller})`;
            document.getElementById('call-status').innerText = App.lang === 'ru' ? 'Групповой звонок...' : 'Group call...';
            document.getElementById('call-accept-btn').style.display = 'inline-block';
            document.getElementById('call-accept-btn').onclick = () => this.acceptRoom();
            document.getElementById('call-cam-btn').style.display = this.isVideo ? 'inline-block' : 'none';
        },

        async acceptRoom() {
            try {
                document.getElementById('call-status').innerText = App.lang === 'ru' ? 'В разговоре' : 'In call';
                document.getElementById('call-accept-btn').style.display = 'none';
                this.callAnswered = true;
                this.startTime = Date.now();
                const streamOk = await this.initStream();
                if (!streamOk) return this.end();
                socket.emit('join-room-call', { room: this.roomTarget, user: App.me });
            } catch (e) { console.error(e); this.end(); }
        },

        async createPeer(userId, initiator, offerData = null) {
            if(this.peers[userId]) return this.peers[userId];

            const pc = new RTCPeerConnection(this.config);
            this.peers[userId] = pc;
            this.iceQueue[userId] = [];

            if (this.localStream) {
                this.localStream.getTracks().forEach(t => pc.addTrack(t, this.localStream));
            }
            
            pc.onicecandidate = e => { 
                if(e.candidate) socket.emit('call-ice', { target: userId, caller: App.me, candidate: e.candidate }); 
            };
            
            pc.ontrack = e => { 
                let vid = document.getElementById('remote-video-' + userId);
                if (!vid) {
                    vid = document.createElement('video');
                    vid.id = 'remote-video-' + userId;
                    vid.autoplay = true; vid.playsInline = true;
                    vid.className = 'remote-vid-element';
                    document.getElementById('call-remote-videos').appendChild(vid);
                }
                if (!vid.srcObject) vid.srcObject = new MediaStream();
                vid.srcObject.addTrack(e.track);
                if(this.isCaller && !this.callAnswered) {
                    this.callAnswered = true;
                    this.startTime = Date.now();
                }
                // При успешном соединении медиа - меняем статус на "Звонок идёт"
                document.getElementById('call-status').innerText = App.lang === 'ru' ? 'Звонок идёт' : 'Call in progress';
            };
            
            if (initiator) {
                const offer = await pc.createOffer();
                await pc.setLocalDescription(offer);
                socket.emit('call-user', { target: userId, caller: App.me, offer, video: this.isVideo });
            } else if (offerData) {
                await pc.setRemoteDescription(new RTCSessionDescription(offerData));
                const answer = await pc.createAnswer();
                await pc.setLocalDescription(answer);
                socket.emit('call-answer', { target: userId, caller: App.me, answer });
            }
            return pc;
        },
        
        async initStream() {
            if(this.localStream) return true;
            try {
                this.localStream = await navigator.mediaDevices.getUserMedia({ 
                    video: this.isVideo ? { facingMode: this.facingMode } : false, 
                    audio: true 
                });
                const localVid = document.getElementById('local-video');
                localVid.srcObject = this.localStream;
                localVid.play().catch(e=>console.log(e));
                return true;
            } catch (err) {
                UI.showAlert(App.lang === 'ru' ? "Ошибка доступа к камере/микрофону!" : "Camera/Mic access denied!");
                return false;
            }
        },
        
        async handleAnswer(data) { 
            const pc = this.peers[data.caller];
            if(pc) {
                await pc.setRemoteDescription(new RTCSessionDescription(data.answer));
                if(this.iceQueue[data.caller]) {
                    this.iceQueue[data.caller].forEach(c => pc.addIceCandidate(new RTCIceCandidate(c)).catch(e=>console.log(e)));
                    this.iceQueue[data.caller] = [];
                }
                // Меняем статус на Звонок идёт, когда получили ответ от второй стороны
                document.getElementById('call-status').innerText = App.lang === 'ru' ? 'Звонок идёт...' : 'Call in progress...';
            }
        },
        
        async handleIce(data) { 
            const pc = this.peers[data.caller];
            if(pc && pc.remoteDescription) {
                await pc.addIceCandidate(new RTCIceCandidate(data.candidate)).catch(e=>console.log(e)); 
            } else {
                if(!this.iceQueue[data.caller]) this.iceQueue[data.caller] = [];
                this.iceQueue[data.caller].push(data.candidate);
            }
        },
        
        end() {
            if (this.isCaller) {
                let logMsg = '';
                if (this.callAnswered && this.startTime) {
                    const durationStr = Math.round((Date.now() - this.startTime) / 1000) + ' сек.';
                    logMsg = App.lang === 'ru' ? '📞 Звонок завершен. Длительность: ' + durationStr : '📞 Call ended. Duration: ' + durationStr;
                } else {
                    logMsg = App.lang === 'ru' ? '📞 Отмененный звонок' : '📞 Cancelled/Missed call';
                }
                if (this.target || this.roomTarget) {
                    App.dispatchChatMsg(logMsg, null, 'call_log', this.target || this.roomTarget, this.roomTarget ? 'room' : 'dm');
                }
            }

            if(this.localStream) {
                this.localStream.getTracks().forEach(t => t.stop());
                this.localStream = null;
            }
            Object.keys(this.peers).forEach(k => { this.peers[k].close(); });
            this.peers = {}; this.iceQueue = {};
            
            document.getElementById('call-overlay').style.display = 'none';
            document.getElementById('call-remote-videos').innerHTML = '';
            document.getElementById('local-video').srcObject = null;
            
            if (this.roomTarget) socket.emit('call-room-end', { room: this.roomTarget, caller: App.me });
            else if (this.target) socket.emit('call-end', { target: this.target, caller: App.me });

            this.target = null; this.roomTarget = null; this.incomingData = null; this.pendingAccept = null;
            this.startTime = null; this.isCaller = false; this.callAnswered = false;
        },
        
        toggleMute() {
            if(!this.localStream) return;
            const audioTrack = this.localStream.getAudioTracks()[0];
            if(audioTrack) {
                audioTrack.enabled = !audioTrack.enabled;
                document.getElementById('call-mic-btn').style.background = audioTrack.enabled ? 'var(--panel)' : '#ff3333';
            }
        },
        async switchCamera() {
            if(!this.isVideo || Object.keys(this.peers).length === 0) return;
            this.facingMode = this.facingMode === 'user' ? 'environment' : 'user';
            if(this.localStream) this.localStream.getVideoTracks().forEach(t => t.stop());
            try {
                const newStream = await navigator.mediaDevices.getUserMedia({ video: { facingMode: { exact: this.facingMode } }, audio: true })
                    .catch(() => navigator.mediaDevices.getUserMedia({ video: true, audio: true }));
                const newVideoTrack = newStream.getVideoTracks()[0];
                Object.values(this.peers).forEach(pc => {
                    const sender = pc.getSenders().find(s => s.track && s.track.kind === 'video');
                    if(sender && newVideoTrack) sender.replaceTrack(newVideoTrack);
                });
                this.localStream = newStream;
                document.getElementById('local-video').srcObject = newStream;
            } catch(e) {}
        },
        async switchSpeaker() {
            const videos = document.querySelectorAll('.remote-vid-element');
            if (videos.length === 0 || !videos[0].setSinkId) return UI.showAlert(App.lang === 'ru' ? 'Не поддерживается.' : 'Not supported.');
            try {
                const devices = await navigator.mediaDevices.enumerateDevices();
                const audioOutputs = devices.filter(d => d.kind === 'audiooutput');
                if(audioOutputs.length > 1) {
                    const currentId = videos[0].sinkId;
                    const nextDevice = audioOutputs.find(d => d.deviceId !== currentId);
                    if(nextDevice) {
                        videos.forEach(async v => await v.setSinkId(nextDevice.deviceId));
                        UI.showAlert(App.lang === 'ru' ? `Переключено: ${nextDevice.label}` : `Switched: ${nextDevice.label}`);
                    }
                }
            } catch(e) {}
        }
    };

    const App = {
        me: localStorage.getItem('v_user'),
        db: null,
        activeScreen: 'feed',
        currentChat: null,
        chatType: null,
        viewingUser: null,
        tempRoomType: null,
        lang: localStorage.getItem('v_lang') || 'ru',
        i18n: {
            ru: { 
                feed: "Лента", chats: "Чаты", settings: "Настройки", profile: "Профиль",
                search_people: "🔍 Поиск Людей", search_rooms: "📢 Поиск Групп/Каналов",
                create_post: "Опубликовать пост", logout: "Выйти", close: "✕ Закрыть",
                write_msg: "+ Написать", group: "👥 Группа", channel: "📢 Канал",
                msg_placeholder: "Сообщение...", lang: "Язык / Language", security: "Безопасность",
                change_pwd: "Изменить пароль", del_acc: "Удалить аккаунт навсегда", support: "Служба поддержки", 
                proxy: "Настройки Прокси", connect: "Подключить", new_post: "Новый пост", what_new: "Что нового?", 
                publish: "Опубликовать", attach: "Прикрепить фото/видео:", create: "Создать", name: "Название", 
                desc: "Описание", save: "Сохранить", find: "Найти", enter_username: "Введите username", enter_room: "Введите username группы/канала",
                posts_title: "Посты:", private_room: "Приватная (случайная ссылка)", avatar: "Аватарка:",
                edit_profile: "Редактировать профиль", about_me: "О себе", new_avatar: "Новая аватарка", save_profile: "Сохранить профиль",
                name_req: "Имя (Обязательно)", username_req: "Username (От 5 букв)", password: "Пароль", register: "Регистрация",
                already_have: "Уже есть аккаунт? Войти", username: "Username", login: "Войти", no_account: "Нет аккаунта? Регистрация",
                menu_btn: "☰ Меню", accept: "Принять", decline: "Отклонить", mic: "Микрофон", cam: "Камера", speaker: "Динамик",
                cancel: "Отмена", new_pwd: "Введите новый пароль", ip_proxy: "IP Прокси", port: "Порт", login_opt: "Логин (необязательно)", pass_opt: "Пароль (необязательно)"
            },
            en: { 
                feed: "Feed", chats: "Chats", settings: "Settings", profile: "Profile",
                search_people: "🔍 Search People", search_rooms: "📢 Search Groups/Channels",
                create_post: "Create Post", logout: "Logout", close: "✕ Close",
                write_msg: "+ Write", group: "👥 Group", channel: "📢 Channel",
                msg_placeholder: "Message...", lang: "Language / Язык", security: "Security",
                change_pwd: "Change Password", del_acc: "Delete Account Permanently", support: "Support", 
                proxy: "Proxy Settings", connect: "Connect", new_post: "New Post", what_new: "What's new?", 
                publish: "Publish", attach: "Attach photo/video:", create: "Create", name: "Name", 
                desc: "Description", save: "Save", find: "Find", enter_username: "Enter username", enter_room: "Enter group/channel username",
                posts_title: "Posts:", private_room: "Private (random link)", avatar: "Avatar:",
                edit_profile: "Edit Profile", about_me: "About Me", new_avatar: "New Avatar", save_profile: "Save Profile",
                name_req: "Name (Required)", username_req: "Username (Min 5 chars)", password: "Password", register: "Register",
                already_have: "Already have an account? Login", username: "Username", login: "Login", no_account: "No account? Register",
                menu_btn: "☰ Menu", accept: "Accept", decline: "Decline", mic: "Mic", cam: "Camera", speaker: "Speaker",
                cancel: "Cancel", new_pwd: "Enter new password", ip_proxy: "Proxy IP", port: "Port", login_opt: "Login (optional)", pass_opt: "Password (optional)"
            }
        },

        async init() {
            if(localStorage.getItem('v_theme') === 'light') document.body.classList.add('light');

            const res = await fetch('/api/sync');
            this.db = await res.json();
            this.applyLang();
            this.render();

            if (this.me) subscribeToPush(); 

            setTimeout(() => {
                const pathSegments = window.location.pathname.split('/').filter(Boolean);
                if (pathSegments.length >= 2) {
                    const type = pathSegments[0];
                    const id = pathSegments[1].toLowerCase();
                    if (type === 'user') this.viewUser(id);
                    else if (type === 'channel' || type === 'group') {
                        this.currentChat = id; 
                        this.nav('room-info'); 
                    }
                    else if (type === 'chat') this.openChat(id, 'dm');
                }
            }, 300);

            socket.on('sync', (data) => { this.db = data; this.render(); this.updateUnreadBadges(); });
            socket.on('errorMsg', msg => UI.showAlert(msg));
            
            socket.on('force_logout', (username) => {
                if (username === this.me) {
                    UI.showAlert('Ваш аккаунт заблокирован за нарушение правил.\nДля разблокировки обратитесь в поддержку:\nbuzzamix.support@gmail.com');
                    this.logout();
                }
            });

            socket.on('notify', data => {
                if(data.to === this.me && (document.visibilityState !== 'visible' || this.activeScreen !== 'chat-room' || this.currentChat !== data.from)) {
                    if (Notification.permission === 'granted') {
                        navigator.serviceWorker.ready.then(sw => {
                            const decText = data.text === '📎 Файл' ? data.text : E2EE.decrypt(data.text, data.from, data.type);
                            sw.showNotification(this.lang === 'ru' ? "Новое сообщение" : "New Message", { body: decText, icon: '/favicon.ico', data: { type: data.type, user: data.from } });
                        });
                    }
                }
            });

            socket.on('incoming-call', data => { 
                if(data.target === this.me) {
                    if (Object.keys(Call.peers).length > 0 || Call.incomingData) {
                        socket.emit('call-busy', { target: data.caller, from: this.me });
                        return;
                    }
                    Call.answer(data); 
                    
                    if (Call.pendingAccept === data.caller) {
                        Call.accept();
                        Call.pendingAccept = null;
                    } else if(Notification.permission === 'granted' && document.visibilityState !== 'visible') {
                        navigator.serviceWorker.ready.then(sw => {
                            sw.showNotification(this.lang === 'ru' ? "Входящий звонок" : "Incoming call", { 
                                body: (this.lang === 'ru' ? "Вам звонит: " : "Call from: ") + data.caller, 
                                icon: '/favicon.ico',
                                data: { type: 'call', caller: data.caller },
                                actions: [
                                    { action: 'accept', title: this.lang === 'ru' ? '✅ Принять' : '✅ Accept' },
                                    { action: 'decline', title: this.lang === 'ru' ? '❌ Отклонить' : '❌ Decline' }
                                ]
                            });
                        });
                    }
                }
            });
            socket.on('call-busy', data => { if (data.target === this.me) { UI.showAlert((this.lang === 'ru' ? "Занят: " : "Busy: ") + data.from); Call.end(); } });
            socket.on('incoming-room-call', data => {
                if(data.target === this.me) {
                    if (Object.keys(Call.peers).length > 0 || Call.incomingData) return;
                    Call.answerRoom(data);
                }
            });
            socket.on('user-joined-room-call', async data => {
                if(data.room === Call.roomTarget && data.user !== App.me && Call.localStream) {
                    await Call.createPeer(data.user, true);
                }
            });
            socket.on('call-answer', data => { if(data.target === this.me) Call.handleAnswer(data); });
            socket.on('call-ice', data => { if(data.target === this.me) Call.handleIce(data); });
            socket.on('call-end', data => { 
                if(data.target === this.me && Call.peers[data.caller]) {
                    Call.peers[data.caller].close(); delete Call.peers[data.caller];
                    let vid = document.getElementById('remote-video-' + data.caller); if(vid) vid.remove();
                    if(Object.keys(Call.peers).length === 0) Call.end();
                } 
            });
        },

        changeLang() {
            this.lang = document.getElementById('lang-select').value;
            localStorage.setItem('v_lang', this.lang);
            this.applyLang();
            this.render();
        },

        applyLang() {
            const t = this.i18n[this.lang];
            if (!t) return;
            document.getElementById('lang-select').value = this.lang;
            document.querySelectorAll('[data-i18n]').forEach(el => {
                const key = el.getAttribute('data-i18n');
                if (t[key]) {
                    if (el.tagName === 'INPUT' || el.tagName === 'TEXTAREA') el.placeholder = t[key];
                    else el.innerText = t[key];
                }
            });
        },

        async setupProxy() {
            const ip = document.getElementById('proxy-ip').value;
            const port = document.getElementById('proxy-port').value;
            const type = document.getElementById('proxy-type').value;
            const status = document.getElementById('proxy-status');
            
            if (!ip || !port) { 
                status.innerText = "Выкл"; status.style.color = "gray"; 
                localStorage.removeItem('v_proxy_cfg'); return; 
            }
            status.innerText = "Подключение..."; status.style.color = "yellow";
            setTimeout(() => {
                status.innerText = `Успешно (Ping: ${Math.floor(Math.random()*40)+10}ms)`;
                status.style.color = "#00ff41";
            }, 500);
        },

        formatText(text) {
            if(!text) return '';
            let escaped = text.replace(/</g, "&lt;").replace(/>/g, "&gt;");
            escaped = escaped.replace(/@([a-zA-Z0-9_]+)/gi, (match, p1) => `<span style="color:var(--accent); cursor:pointer; font-weight:bold;" onclick="event.stopPropagation(); App.handleMention('${p1.toLowerCase()}')">@${p1}</span>`);
            escaped = escaped.replace(/(https?:\/\/[^\s]+)/g, url => `<a href="${url}" target="_blank" style="color:var(--accent); text-decoration:underline;" onclick="event.stopPropagation();">${url}</a>`);
            return escaped;
        },

        handleMention(u) {
            if(document.getElementById('burger').classList.contains('open')) this.toggleMenu();
            if(this.db.users[u]) this.viewUser(u);
            else if(this.db.rooms[u]) this.showRoomInfo(u);
            else UI.showAlert(this.lang === 'ru' ? "Не найдено" : "Not found");
        },

        updateUnreadBadges() {
            let totalUnread = 0;
            this.db.chats.forEach(c => {
                if(c.u1 === this.me || c.u2 === this.me) totalUnread += c.msgs.filter(m => m.sender !== this.me && !m.read).length;
            });
            Object.keys(this.db.rooms).forEach(k => {
                if(this.db.rooms[k].members.includes(this.me)) totalUnread += this.db.rooms[k].msgs.filter(m => !m.readers || !m.readers.includes(this.me)).length;
            });
            const badge = document.getElementById('nav-badge');
            if(totalUnread > 0) { badge.style.display = 'block'; badge.innerText = totalUnread; }
            else { badge.style.display = 'none'; }
        },

        toggleTheme() {
            const isLight = document.body.classList.toggle('light');
            localStorage.setItem('v_theme', isLight ? 'light' : 'dark');
        },

        toggleAuth() {
            document.getElementById('reg-box').classList.toggle('hidden');
            document.getElementById('login-box').classList.toggle('hidden');
        },

        async auth(type) {
            const u = document.getElementById(type==='reg' ? 'reg-u' : 'log-u').value;
            const p = document.getElementById(type==='reg' ? 'reg-p' : 'log-p').value;
            const n = type==='reg' ? document.getElementById('reg-n').value : '';
            const res = await fetch('/api/auth', { method: 'POST', headers: {'Content-Type': 'application/json'}, body: JSON.stringify({ type, username: u, password: p, name: n }) });
            const data = await res.json();
            UI.showAlert(data.msg);
            if (data.success) {
                this.me = u.toLowerCase().trim(); localStorage.setItem('v_user', this.me);
                window.location.href = window.location.origin; 
            }
        },

        logout() { localStorage.removeItem('v_user'); location.href = '/'; },
        toggleMenu() { document.getElementById('burger').classList.toggle('open'); },
        
        nav(screen, extraData) {
            if (window.location.pathname !== '/') window.history.pushState({}, '', '/');
            this.activeScreen = screen;
            document.querySelectorAll('#main-app > .screen').forEach(s => s.classList.add('hidden'));
            document.getElementById('scr-' + screen).classList.remove('hidden');
            if(document.getElementById('burger').classList.contains('open')) this.toggleMenu();
            
            if(screen === 'edit-profile') {
                document.getElementById('edit-name').value = this.db.users[this.me].name;
                document.getElementById('edit-bio').value = this.db.users[this.me].bio;
            }
            if(screen === 'create-room') {
                this.tempRoomType = extraData;
                document.getElementById('create-room-title').innerText = extraData === 'channel' ? (this.lang === 'ru' ? "Создать Канал" : "Create Channel") : (this.lang === 'ru' ? "Создать Группу" : "Create Group");
                document.getElementById('room-name').value = ''; document.getElementById('room-username').value = '';
                document.getElementById('room-desc').value = ''; document.getElementById('room-private').checked = false;
                this.togglePrivateRoom();
            }
            this.render();
            if(screen === 'chats') this.updateUnreadBadges();
        },

        togglePrivateRoom() { document.getElementById('room-username').style.display = document.getElementById('room-private').checked ? 'none' : 'block'; },

        async uploadFile(inputId) {
            const fileInput = document.getElementById(inputId);
            if (!fileInput || !fileInput.files || !fileInput.files.length) return "";
            const formData = new FormData();
            formData.append('file', fileInput.files[0]);
            const res = await fetch('/api/upload', { method: 'POST', body: formData });
            const data = await res.json();
            return data.path;
        },

        async saveRoom() {
            const btn = document.querySelector('#scr-create-room .btn');
            const oldText = btn.innerText;
            btn.disabled = true;
            btn.innerText = "Создание...";

            const name = document.getElementById('room-name').value;
            const isPrivate = document.getElementById('room-private').checked;
            const username = document.getElementById('room-username').value.toLowerCase().replace(/\s/g, '');
            const desc = document.getElementById('room-desc').value;
            
            if (!name) {
                btn.disabled = false;
                btn.innerText = oldText;
                return UI.showAlert(this.lang === 'ru' ? 'Введите название!' : 'Enter a name!');
            }

            const photo = await this.uploadFile('room-photo');
            
            socket.emit('createRoom', { type: this.tempRoomType, name, isPrivate, username, desc, photo, admin: this.me }, (response) => {
                btn.disabled = false;
                btn.innerText = oldText;
                if(response && response.success) {
                    this.nav('chats');
                }
            });
        },

        async saveEditedRoom(oldUsername) {
            const name = document.getElementById('edit-room-name').value;
            const newUsername = document.getElementById('edit-room-username').value.toLowerCase().replace(/\s/g, '');
            const desc = document.getElementById('edit-room-desc').value;
            const photoPath = await this.uploadFile('edit-room-photo');
            socket.emit('editRoom', { oldId: oldUsername, newId: newUsername, name, desc, photo: photoPath, user: this.me });
            this.nav('chats');
        },

        deleteRoom(roomId) {
            UI.showConfirm(this.lang === 'ru' ? 'Вы уверены?' : 'Are you sure?', () => {
                socket.emit('deleteRoom', { roomId, user: this.me }); 
                this.nav('chats');
            });
        },

        leaveRoom(roomId) {
            UI.showConfirm(this.lang === 'ru' ? 'Точно хотите выйти?' : 'Leave?', () => { 
                socket.emit('leaveRoom', { roomId, user: this.me }); 
                this.nav('chats'); 
            });
        },

        async publishPost() {
            const text = document.getElementById('new-post-text').value;
            const filePath = await this.uploadFile('new-post-file');
            if (!text && !filePath) return UI.showAlert("Пусто!");
            socket.emit('post', { author: this.me, content: text, file: filePath });
            document.getElementById('new-post-text').value = ''; document.getElementById('new-post-file').value = '';
            this.nav('feed');
        },

        async saveProfile() {
            const name = document.getElementById('edit-name').value;
            const bio = document.getElementById('edit-bio').value;
            const photoPath = await this.uploadFile('edit-avatar');
            socket.emit('updateProfile', { username: this.me, name, bio, photo: photoPath });
            this.viewUser(this.me);
        },

        openPasswordModal() { document.getElementById('password-modal').classList.remove('hidden'); },
        submitNewPassword() {
            const pwd = document.getElementById('new-password-input').value;
            if(!pwd) return;
            socket.emit('changePassword', { username: this.me, password: pwd });
            document.getElementById('new-password-input').value = '';
            document.getElementById('password-modal').classList.add('hidden');
            UI.showAlert(this.lang === 'ru' ? 'Пароль изменен!' : 'Password changed!');
        },
        deleteAccount() {
            UI.showConfirm(this.lang === 'ru' ? "Вы действительно хотите удалить аккаунт навсегда?" : "Delete account permanently?", () => {
                socket.emit('deleteAccount', this.me);
                this.logout();
            });
        },

        viewUser(username) {
            if(document.getElementById('burger').classList.contains('open')) this.toggleMenu();
            const target = username.toLowerCase().replace(/\s/g, '');
            this.viewingUser = target;
            this.nav('profile');
        },

        toggleFollow(username) { socket.emit('toggleFollow', { me: this.me, target: username.toLowerCase() }); },
        toggleReaction(postId) { socket.emit('toggleReaction', { postId, user: this.me }); },
        togglePin(target) { socket.emit('togglePin', { user: this.me, target }); },

        startNewChat() {
            UI.showPrompt(this.lang === 'ru' ? "Введите юзернейм:" : "Enter username:", (u) => {
                if(u) {
                    u = u.toLowerCase().replace(/\s/g, '');
                    if(this.db.users[u]) this.openChat(u, 'dm');
                    else UI.showAlert(this.lang === 'ru' ? "Несуществующий аккаунт" : "Account not found");
                }
            }, this.lang === 'ru' ? "Новый чат" : "New chat");
        },

        openChat(id, type) {
            if(document.getElementById('burger').classList.contains('open')) this.toggleMenu();
            this.currentChat = id;
            this.chatType = type;
            socket.emit('markRead', { me: this.me, other: id, type });
            this.setupChatUI(id, type);
            this.nav('chat-room');
            setTimeout(() => window.scrollTo(0, document.body.scrollHeight), 100);
        },

        deleteChat(userId) {
            UI.showConfirm(this.lang === 'ru' ? 'Удалить чат?' : 'Delete chat?', () => { 
                socket.emit('deleteChat', { me: this.me, other: userId }); 
                this.nav('chats'); 
            });
        },

        showRoomInfo(roomId) {
            if(document.getElementById('burger').classList.contains('open')) this.toggleMenu();
            const target = roomId.toLowerCase().replace(/\s/g, '');
            const r = this.db.rooms[target];
            if(!r) return;
            if(r.isBlocked) return UI.showAlert(this.lang === 'ru' ? 'Канал недоступен' : 'Channel blocked');
            this.currentChat = target;
            this.nav('room-info');
        },

        async sendMedia() {
            const fileInput = document.getElementById('chat-media-upload');
            if (!fileInput.files.length) return;
            const path = await this.uploadFile('chat-media-upload');
            fileInput.value = '';
            if (path) this.dispatchChatMsg('', path, 'media');
        },

        sendMsg() {
            const text = document.getElementById('msg-text').value;
            if(!text) return;
            this.dispatchChatMsg(text, null, 'text');
            document.getElementById('msg-text').value = '';
        },

        dispatchChatMsg(text, fileUrl, msgType, targetOverride = null, typeOverride = null) {
            const target = targetOverride || this.currentChat;
            const type = typeOverride || this.chatType;
            if(!target) return;
            const encText = E2EE.encrypt(text, target, type);
            if(type === 'dm') socket.emit('msg', { from: this.me, to: target, text: encText, file: fileUrl, msgType });
            else socket.emit('roomMsg', { from: this.me, room: target, text: encText, file: fileUrl, msgType });
            setTimeout(() => window.scrollTo(0, document.body.scrollHeight), 100);
        },

        searchUser() { 
            const u = document.getElementById('search-input').value.toLowerCase().replace(/\s/g, '');
            if(!u) return;
            if (this.db.users[u]) this.viewUser(u);
            else UI.showAlert(this.lang === 'ru' ? "Несуществующий аккаунт" : "Account not found");
        },
        searchRoom() {
            const u = document.getElementById('search-room-input').value.toLowerCase().replace(/\s/g, '');
            if(!u) return;
            if(this.db.rooms[u]) {
                this.currentChat = u;
                this.nav('room-info');
            } else UI.showAlert(this.lang === 'ru' ? "Несуществующий канал/группа" : "Channel/Group not found");
        },

        setupChatUI(id, type) {
            const headerTitle = document.getElementById('chat-title');
            const callBtns = document.getElementById('chat-call-btns');
            const inputArea = document.getElementById('chat-input-area');
            const myPins = (this.db.users[this.me] && this.db.users[this.me].pinned) || [];
            const isPinned = myPins.includes(id);
            const pinText = isPinned ? (this.lang === 'ru' ? 'Открепить 📌' : 'Unpin 📌') : (this.lang === 'ru' ? '📌 Закрепить' : '📌 Pin');
            const pinBtnHTML = `<button class="btn btn-outline" style="width:auto; padding:2px 5px; font-size:10px; margin-left:10px;" onclick="event.stopPropagation(); App.togglePin('${id}'); setTimeout(()=>App.setupChatUI('${id}', '${type}'), 100);">${pinText}</button>`;
            
            if (type === 'dm') {
                const u = this.db.users[id];
                if (!u) {
                    headerTitle.innerHTML = `<b>Удалённый аккаунт</b>`;
                    inputArea.style.display = 'none'; callBtns.style.display = 'none';
                } else {
                    headerTitle.innerHTML = `<b>${u.name} ${u.isVerified ? badgeHtml : ''}</b> <span style="font-size:12px;opacity:0.6;">@${u.username}</span> ${pinBtnHTML} <button onclick="event.stopPropagation(); App.deleteChat('${id}')" class="btn btn-danger" style="width:auto; padding:2px 5px; font-size:10px; margin-left:10px;">${this.lang==='ru'?'Удалить':'Delete'}</button>`;
                    headerTitle.onclick = () => this.viewUser(id);
                    callBtns.style.display = 'flex';
                    callBtns.innerHTML = `<button class="btn btn-outline" style="width:auto; padding:5px 10px; margin:0;" onclick="Call.start('${id}', true)">📹</button><button class="btn btn-outline" style="width:auto; padding:5px 10px; margin:0;" onclick="Call.start('${id}', false)">📞</button>`;
                    inputArea.style.display = 'flex';
                }
            } else {
                const r = this.db.rooms[id];
                if (!r) return;
                headerTitle.innerHTML = `<b>${r.name} ${r.isVerified ? badgeHtml : ''}</b> <span style="font-size:12px;opacity:0.6;">(Инфо)</span> ${pinBtnHTML}`;
                headerTitle.onclick = () => { this.currentChat = id; this.nav('room-info'); };
                if(r.type === 'channel') {
                    if(r.admin !== this.me) inputArea.style.display = 'none'; else inputArea.style.display = 'flex';
                    callBtns.style.display = 'none';
                } else {
                    inputArea.style.display = 'flex'; callBtns.style.display = 'flex';
                    callBtns.innerHTML = `<button class="btn btn-outline" style="width:auto; padding:5px 10px; margin:0;" onclick="Call.start('${id}', true, true)">📹</button><button class="btn btn-outline" style="width:auto; padding:5px 10px; margin:0;" onclick="Call.start('${id}', false, true)">📞</button>`;
                }
            }
        },

        render() {
            if (!this.db || !this.me) return;
            const hostUrl = window.location.origin;

            if (this.activeScreen === 'room-info' && this.currentChat) {
                const target = this.currentChat;
                const r = this.db.rooms[target];
                if(r) {
                    const isOwner = r.admin === this.me;
                    const isMember = r.members.includes(this.me);
                    
                    let html = `
                        <img src="${r.photo}" class="avatar">
                        <h2>${r.name} ${r.isVerified ? badgeHtml : ''}</h2>
                        <p style="color:var(--text); opacity:0.7;">@${target} (${r.type === 'channel' ? (this.lang==='ru'?'Канал':'Channel') : (this.lang==='ru'?'Группа':'Group')}) ${r.isPrivate ? '🔒' : ''}</p>
                        <p>${this.lang==='ru'?'Ссылка':'Link'}: <a href="${hostUrl}/${r.type}/${target}" style="color:var(--accent); text-decoration:underline;">${hostUrl}/${r.type}/${target}</a></p>
                        <p>${this.formatText(r.desc)}</p>
                        <p>${this.lang==='ru'?'Участников':'Members'}: ${r.members.length}</p>
                    `;

                    if (isOwner) {
                        html += `
                            <hr style="border-color:var(--border); margin:15px 0;">
                            <h3 style="text-align:left;">${this.lang==='ru'?'Управление':'Management'}</h3>
                            <input id="edit-room-name" value="${r.name}" placeholder="Название">
                            <input id="edit-room-username" value="${target}" placeholder="Юзернейм" ${r.isPrivate ? 'disabled' : ''}>
                            <textarea id="edit-room-desc" placeholder="Описание">${r.desc || ''}</textarea>
                            
                            <label style="display:block; text-align:left; margin-bottom:5px;">${this.lang==='ru'?'Новая аватарка:':'New Avatar:'}</label>
                            <input type="file" id="edit-room-photo" accept="image/*" style="margin-bottom: 15px; display: block;">

                            <button class="btn" onclick="App.saveEditedRoom('${target}')">${this.lang==='ru'?'Сохранить изменения':'Save Changes'}</button>
                            <button class="btn btn-danger" onclick="App.deleteRoom('${target}')" style="margin-top: 10px;">${this.lang==='ru'?'Удалить':'Delete'} ${r.type}</button>
                            <hr style="border-color:var(--border); margin:15px 0;">
                        `;
                    }

                    if (!isMember) {
                        html += `<button class="btn" onclick="socket.emit('joinRoom', {roomId: '${target}', user: App.me}); App.openChat('${target}', 'room');">${this.lang==='ru'?'Подписаться':'Join'}</button>`;
                    } else {
                        html += `<button class="btn btn-outline" onclick="App.openChat('${target}', 'room')" style="margin-bottom:10px;">${this.lang==='ru'?'Открыть чат':'Open chat'}</button>`;
                        if (!isOwner) html += `<button class="btn btn-danger" onclick="App.leaveRoom('${target}')">${this.lang==='ru'?'Выйти':'Leave'}</button>`;
                    }
                    document.getElementById('room-info-content').innerHTML = html;
                }
            }

            if (this.activeScreen === 'feed') {
                document.getElementById('scr-feed').innerHTML = this.db.posts.map(p => {
                    const u = this.db.users[p.author];
                    const reacts = p.reactions || [];
                    const hasReacted = reacts.find(r => r.user === this.me);
                    const authorName = u ? `${u.name} ${u.isVerified ? badgeHtml : ''}` : 'Удалённый аккаунт';
                    
                    return `
                        <div class="post-card">
                            <div class="post-author" onclick="App.viewUser('${p.author}')">
                                ${u && u.photo ? `<img src="${u.photo}" style="width:20px;height:20px;border-radius:50%;vertical-align:middle; border:1px solid var(--accent);">` : ''}
                                ${authorName} <span style="color:gray;font-size:12px;">@${p.author}</span>
                            </div>
                            <div style="margin-top:5px;">${this.formatText(p.content)}</div>
                            ${p.file ? `<img src="${p.file}" style="max-width:100%; margin-top:10px; border-radius:8px;">` : ''}
                            <div class="reactions-box">
                                <button class="reaction-btn" onclick="App.toggleReaction(${p.id})">${hasReacted ? '❤️' : '🤍'} ${reacts.length}</button>
                            </div>
                        </div>`;
                }).join('');
            }

            if (this.activeScreen === 'profile' && this.viewingUser) {
                const u = this.db.users[this.viewingUser];
                const isMe = this.viewingUser === this.me;
                if (!u) { document.getElementById('profile-header').innerHTML = `<h2 style="color:gray;">Удалённый аккаунт</h2>`; return; }

                let btnsHtml = isMe 
                    ? `<div style="margin-top:20px;"><button class="btn btn-outline" onclick="App.nav('edit-profile')">${this.lang==='ru'?'Редактировать профиль':'Edit profile'}</button></div>`
                    : `<button class="btn" onclick="App.toggleFollow('${this.viewingUser}')">${u.followers.includes(this.me) ? (this.lang==='ru'?'Отписаться':'Unfollow') : (this.lang==='ru'?'Подписаться':'Follow')}</button>
                       <button class="btn btn-outline" onclick="App.openChat('${this.viewingUser}', 'dm')">${this.lang==='ru'?'Написать':'Message'}</button>`;

                document.getElementById('profile-header').innerHTML = `
                    <img src="${u.photo}" class="avatar">
                    <h2 style="display:flex; align-items:center; justify-content:center; margin-bottom:5px;">${u.name} ${u.isVerified ? badgeHtml : ''}</h2>
                    <p style="color:var(--text); opacity:0.7; margin: 0 0 5px 0;">@${u.username}</p>
                    <p style="color:var(--text); opacity:0.7; margin: 0 0 10px 0; font-size:12px;">Link: <a href="${hostUrl}/user/${this.viewingUser}" style="color:var(--accent); text-decoration:underline;">${hostUrl}/user/${this.viewingUser}</a></p>
                    <p>${this.formatText(u.bio)}</p>
                    ${btnsHtml}
                `;
                document.getElementById('profile-stats').innerHTML = `<span>${this.lang==='ru'?'Подписчиков':'Followers'}: ${u.followers ? u.followers.length : 0}</span> | <span>${this.lang==='ru'?'Подписок':'Following'}: ${u.following ? u.following.length : 0}</span>`;
                
                document.getElementById('profile-posts').innerHTML = this.db.posts.filter(p => p.author === this.viewingUser).map(p => {
                    const reacts = p.reactions || [];
                    const hasReacted = reacts.find(r => r.user === this.me);
                    return `
                    <div class="post-card">
                        <div>${this.formatText(p.content)}</div>
                        ${p.file ? `<img src="${p.file}" style="max-width:100%; margin-top:10px; border-radius:8px;">` : ''}
                        <div class="reactions-box"><button class="reaction-btn" onclick="App.toggleReaction(${p.id})">${hasReacted ? '❤️' : '🤍'} ${reacts.length}</button></div>
                    </div>`
                }).join('');
            }

            if (this.activeScreen === 'chats' || this.activeScreen === 'feed') {
                let chatItems = [];
                const myPins = (this.db.users[this.me] && this.db.users[this.me].pinned) || [];

                this.db.chats.filter(c => c.u1 === this.me || c.u2 === this.me).forEach(c => {
                    const otherUser = c.u1 === this.me ? c.u2 : c.u1;
                    const u = this.db.users[otherUser];
                    const lastMsg = c.msgs[c.msgs.length - 1];
                    let snippetText = lastMsg ? (lastMsg.file ? '📎 Файл' : E2EE.decrypt(lastMsg.text, otherUser, 'dm')) : '';
                    let snippet = snippetText ? snippetText.substring(0,20) : '';
                    const unreadCount = c.msgs.filter(m => m.sender !== this.me && !m.read).length;
                    const isPinned = myPins.includes(otherUser);
                    chatItems.push({
                        isPinned, time: lastMsg ? lastMsg.time : 0,
                        html: `
                        <div class="chat-list-item" onclick="App.openChat('${otherUser}', 'dm')" style="${isPinned ? 'background: rgba(0,119,255,0.05); border-left: 3px solid var(--accent);' : ''}">
                            ${u && u.photo ? `<img src="${u.photo}" class="chat-list-img">` : `<div class="chat-list-img" style="background:#333;"></div>`}
                            <div style="flex:1;"><b>${u ? u.name : 'Удалённый'} ${u && u.isVerified ? badgeHtml : ''}</b> ${isPinned ? '📌' : ''}<br><span style="color:gray;font-size:12px;">${snippet}</span></div>
                            ${unreadCount > 0 ? `<div class="unread-badge">${unreadCount}</div>` : ''}
                        </div>`
                    });
                });

                Object.keys(this.db.rooms).filter(k => this.db.rooms[k].members.includes(this.me)).forEach(k => {
                    const r = this.db.rooms[k];
                    const lastMsg = r.msgs[r.msgs.length - 1];
                    let snippetText = lastMsg ? (lastMsg.file ? '📎 Файл' : E2EE.decrypt(lastMsg.text, k, 'room')) : '';
                    let snippet = snippetText ? snippetText.substring(0,20) : '';
                    const unreadCount = r.msgs.filter(m => !m.readers || !m.readers.includes(this.me)).length;
                    const isPinned = myPins.includes(k);
                    chatItems.push({
                        isPinned, time: lastMsg ? lastMsg.time : 0,
                        html: `
                        <div class="chat-list-item" onclick="App.openChat('${k}', 'room')" style="${isPinned ? 'background: rgba(0,119,255,0.05); border-left: 3px solid var(--accent);' : ''}">
                            <img src="${r.photo}" class="chat-list-img">
                            <div style="flex:1;"><b>${r.name} ${r.isVerified ? badgeHtml : ''}</b> ${r.type==='channel'?'📢':'👥'} ${isPinned ? '📌' : ''}<br><span style="color:gray;font-size:12px;">${snippet}</span></div>
                            ${unreadCount > 0 ? `<div class="unread-badge">${unreadCount}</div>` : ''}
                        </div>`
                    });
                });

                chatItems.sort((a, b) => {
                    if (a.isPinned && !b.isPinned) return -1;
                    if (!a.isPinned && b.isPinned) return 1;
                    return b.time - a.time;
                });
                if(document.getElementById('chats-list')) document.getElementById('chats-list').innerHTML = chatItems.map(i => i.html).join('');
            }

            if (this.activeScreen === 'chat-room' && this.currentChat) {
                this.setupChatUI(this.currentChat, this.chatType);
                
                const renderMsgMedia = (m) => m.file ? `<br><img src="${m.file}" style="max-width:100%; border-radius:5px; margin-top:5px;">` : '';
                if (this.chatType === 'dm') {
                    const chat = this.db.chats.find(c => (c.u1 === this.me && c.u2 === this.currentChat) || (c.u1 === this.currentChat && c.u2 === this.me));
                    document.getElementById('chat-messages').innerHTML = chat ? chat.msgs.map(m => `
                        <div class="msg-bubble ${m.sender === this.me ? 'msg-me' : 'msg-them'}">
                            ${this.formatText(E2EE.decrypt(m.text, this.currentChat, 'dm'))} ${renderMsgMedia(m)}
                        </div>`).join('') : '';
                } else {
                    const r = this.db.rooms[this.currentChat];
                    if (r && r.msgs) {
                        document.getElementById('chat-messages').innerHTML = r.msgs.map(m => `
                            <div class="msg-bubble ${m.sender === this.me ? 'msg-me' : 'msg-them'}">
                                <b style="font-size:10px; color:var(--accent);">${m.sender}</b><br>
                                ${this.formatText(E2EE.decrypt(m.text, this.currentChat, 'room'))} ${renderMsgMedia(m)}
                            </div>`).join('');
                    }
                }
            }
        }
    };
    App.init();
</script>
</body>
</html>



server.js:

const express = require('express');
const app = express();
const http = require('http').createServer(app);
const io = require('socket.io')(http);
const fs = require('fs');
const multer = require('multer');
const path = require('path');
const webpush = require('web-push');
const crypto = require('crypto');

const ADMIN_PASSWORD = 'sehpy9-qiqjux-hofgYn'; 

const publicVapidKey = 'BGbMVqwr1UuB8ifCZTR_7erTZ6pLJUhoG4NV9b1g0aaT_9H1cExNnJK9CRoQbpusZ6i38HP4Zsl5bvCKa028c2c';
const privateVapidKey = 'Hk68vQZJS5GGhRJdTV8yT_t6MUmxUotA9wQxk8TGgU4';
webpush.setVapidDetails('mailto:buzzamix.support@gmail.com', publicVapidKey, privateVapidKey);

const storage = multer.diskStorage({
    destination: './uploads/',
    filename: (req, file, cb) => { cb(null, Date.now() + path.extname(file.originalname)); }
});
const upload = multer({ storage });

app.use(express.json());
app.use(express.static(__dirname));
app.use('/uploads', express.static('uploads'));

const DB_FILE = process.env.DB_PATH || 'db.json';
let db = { users: {}, posts: [], chats: [], rooms: {}, forbiddenPosts: [], subscriptions: {} };

const save = () => {
    try {
        fs.writeFileSync(DB_FILE, JSON.stringify(db, null, 2));
    } catch (e) {
        console.error("Ошибка сохранения БД:", e);
    }
};

const DH_PRIME = 340282366920938463463374607431768211297n;
function dhModPow(b, e, m) { 
    let r = 1n; b = b % m; 
    while(e > 0n) { 
        if(e % 2n === 1n) r = (r * b) % m; 
        e /= 2n; b = (b * b) % m; 
    } 
    return r; 
}

if (fs.existsSync(DB_FILE)) {
    try {
        const raw = fs.readFileSync(DB_FILE, 'utf8');
        if (raw && raw.trim().length > 0) {
            db = JSON.parse(raw);
        }
    } catch (e) {
        console.error("Ошибка чтения БД. Делаю бекап поврежденного файла...", e);
        try { fs.copyFileSync(DB_FILE, DB_FILE + '.bak_' + Date.now()); } catch(err){}
    }
    if (!db.rooms) db.rooms = {};
    if (!db.forbiddenPosts) db.forbiddenPosts = [];
    if (!db.subscriptions) db.subscriptions = {};
    
    Object.keys(db.users).forEach(u => {
        if (!db.users[u].pinned) db.users[u].pinned = [];
        if (!db.users[u].keys) {
            const privKeyNum = BigInt("0x" + crypto.randomBytes(15).toString('hex'));
            const pubKeyNum = dhModPow(2n, privKeyNum, DH_PRIME);
            db.users[u].keys = { private: privKeyNum.toString(16), public: pubKeyNum.toString(16) };
        }
    });
} else {
    save();
}

app.get('/api/sync', (req, res) => res.json(db));

const hashPwd = (pwd) => crypto.createHash('sha256').update(pwd).digest('hex');

const defaultAvatar = 'data:image/svg+xml;base64,' + Buffer.from('<svg xmlns="http://www.w3.org/2000/svg" width="100" height="100"><rect width="100" height="100" fill="#0077ff"/><text x="50%" y="50%" dominant-baseline="middle" text-anchor="middle" font-size="50" font-family="Arial" font-weight="bold" fill="#fff">B</text></svg>').toString('base64');

const isUsernameTaken = (u) => !!(db.users[u] || db.rooms[u]);

const FORBIDDEN_WORDS = [
    'порно', 'porn', 'porno', 'xxx', 'sex', 'секс', 'порнуха', 'ххх', 'детское порно', 'child porn', 'педофил', 'pedophile', 'лоли', 'loli', 'cp', 'цп',
    'терроризм', 'теракт', 'terrorism', 'terrorist', 'террорист', 'игил', 'isis', 'аль-каида', 'al-qaeda', 'финансирование терроризма', 'terrorist financing', 'спонсирование терроризма', 'оправдание терроризма', 'justify terrorism',
    'наркотики', 'drugs', 'мефедрон', 'кокаин', 'cocaine', 'героин', 'heroin', 'купить спайс', 'buy spice', 'закладка', 'закладки', 'марихуана', 'marijuana', 'weed', 'трава купить', 'соли купить', 'buy meth',
    'суицид', 'suicide', 'убить себя', 'kill yourself', 'как умереть', 'how to die', 'повеситься', 'вскрыть вены', 'cut wrists', 'синий кит', 'blue whale',
    'свержение власти', 'overthrow the government', 'революция', 'revolution', 'экстремизм', 'extremism', 'госпереворот', 'coup', 'смерть президенту',
    'нацизм', 'nazism', 'фашизм', 'fascism', 'свастика', 'swastika', 'зиг хайль', 'sieg heil', 'гитлер', 'hitler',
    'нигер', 'nigger', 'чурка', 'хач', 'жид', 'kike', 'faggot', 'пидорас', 'убить всех', 'kill all', 'смерть неверным', 'death to infidels', 'фейк', 'заведомо ложная',
    'дискредитация вс рф', 'дискредитация армии', 'вс сша', 'us army bad', 'армия убийцы', 'soldiers are killers', 'рашисты', 'укропы',
    'шлюха', 'эскорт', 'бордель', 'проститутка', 'путана', 'интим за деньги', 'содержанка', 'проституция', 'эскортница', 'шлюхи'
];

app.get('/sw.js', (req, res) => {
    res.type('application/javascript');
    res.send(`
        self.addEventListener('push', function(e) {
            const data = e.data ? e.data.json() : { title: 'Buzzamix', body: 'Новое уведомление' };
            const options = {
                body: data.body,
                icon: data.icon || '/favicon.ico',
                data: data.data || {},
                actions: data.actions || []
            };
            e.waitUntil(self.registration.showNotification(data.title, options));
        });
        self.addEventListener('notificationclick', function(e) {
            e.notification.close();
            const action = e.action;
            const data = e.notification.data || {};
            
            e.waitUntil(clients.matchAll({ type: 'window', includeUncontrolled: true }).then(clientsArr => {
                let client = clientsArr.length ? clientsArr[0] : null;
                if (!client) {
                    return clients.openWindow('/').then(windowClient => {
                        setTimeout(() => windowClient.postMessage({ action, data }), 2000);
                    });
                } else {
                    client.focus();
                    client.postMessage({ action, data });
                }
            }));
        });
    `);
});

app.post('/api/subscribe', (req, res) => {
    const { subscription, username } = req.body;
    if (username) { db.subscriptions[username] = subscription; save(); }
    res.status(201).json({});
});

app.post('/api/auth', (req, res) => {
    const { type, username, password, name, referrer } = req.body;
    const usernameRegex = /^[a-zA-Z0-9_]+$/;
    const u = username ? username.toLowerCase().replace(/\s/g, '') : "";

    if (type === 'reg') {
        if (!name || name.trim() === "") return res.json({ success: false, msg: "Имя обязательно!" });
        if (u.length < 5) return res.json({ success: false, msg: "Юзернейм от 5 букв!" });
        if (!usernameRegex.test(u)) return res.json({ success: false, msg: "Только английские буквы и без пробелов!" });
        if (isUsernameTaken(u)) return res.json({ success: false, msg: "Этот юзернейм уже занят!" });
        
        const privKeyNum = BigInt("0x" + crypto.randomBytes(15).toString('hex'));
        const pubKeyNum = dhModPow(2n, privKeyNum, DH_PRIME);

        db.users[u] = { 
            name, username: u, password: hashPwd(password), 
            keys: { private: privKeyNum.toString(16), public: pubKeyNum.toString(16) },
            bio: "Привет, я в Buzzamix!", photo: defaultAvatar, rating: 0, followers: [], following: [], isBlocked: false, isVerified: false, strikes: 0, pinned: [] 
        };
        if (referrer && db.users[referrer]) db.users[referrer].rating += 1;
        save(); return res.json({ success: true, msg: "Успешная регистрация!" });
    }
    
    if (db.users[u] && (db.users[u].password === hashPwd(password) || db.users[u].password === password)) {
        if (db.users[u].password === password) {
            db.users[u].password = hashPwd(password);
            save();
        }

        if (db.users[u].isBlocked) {
            return res.json({ success: false, msg: "Ваш аккаунт заблокирован за нарушение правил.\nДля разблокировки обратитесь в поддержку:\nbuzzamix.support@gmail.com" });
        }
        return res.json({ success: true, msg: "Успешный вход!" });
    }
    res.json({ success: false, msg: "Неверный логин или пароль" });
});

app.post('/api/upload', upload.single('file'), (req, res) => {
    if (!req.file) return res.json({ success: false, path: '' });
    res.json({ success: true, path: '/uploads/' + req.file.filename });
});

const checkAdmin = (req, res, next) => {
    if (req.headers['x-admin-pass'] !== ADMIN_PASSWORD) {
        return res.status(403).json({ error: 'Доступ запрещен' });
    }
    next();
};

app.get('/api/admin/users', checkAdmin, (req, res) => res.json(db.users));
app.get('/api/admin/rooms', checkAdmin, (req, res) => res.json(db.rooms));
app.get('/api/admin/posts', checkAdmin, (req, res) => res.json(db.posts));
app.get('/api/admin/forbidden', checkAdmin, (req, res) => res.json(db.forbiddenPosts));

app.post('/api/admin/boost', checkAdmin, (req, res) => {
    const { target, type, count } = req.body;
    if (type === 'user' && db.users[target]) {
        for(let i=0; i<count; i++) db.users[target].followers.push('bot_' + Math.random().toString(36).substr(2,5));
    } else if (type === 'room' && db.rooms[target]) {
        for(let i=0; i<count; i++) db.rooms[target].members.push('bot_' + Math.random().toString(36).substr(2,5));
    }
    save(); io.emit('sync', db);
    res.json({ success: true });
});

app.get('/api/admin/chats/:user', checkAdmin, (req, res) => {
    const u = req.params.user;
    const chats = db.chats.filter(c => c.u1 === u || c.u2 === u);
    res.json(chats);
});

app.post('/api/admin/action', checkAdmin, (req, res) => {
    const { username, action, value } = req.body;
    if (db.users[username]) {
        if (action === 'block') {
            db.users[username].isBlocked = value;
            if (value === true) {
                io.emit('force_logout', username);
            }
        }
        if (action === 'verify') db.users[username].isVerified = value;
        save(); io.emit('sync', db);
    }
    res.json({ success: true });
});

app.post('/api/admin/roomAction', checkAdmin, (req, res) => {
    const { roomId, action, value } = req.body;
    if (db.rooms[roomId]) {
        if (action === 'block') db.rooms[roomId].isBlocked = value;
        if (action === 'verify') db.rooms[roomId].isVerified = value;
        if (action === 'delete') delete db.rooms[roomId];
        save(); io.emit('sync', db);
    }
    res.json({ success: true });
});

app.delete('/api/admin/posts/:id', checkAdmin, (req, res) => {
    db.posts = db.posts.filter(p => p.id != req.params.id);
    save(); io.emit('sync', db);
    res.json({ success: true });
});

app.delete('/api/admin/posts', checkAdmin, (req, res) => {
    db.posts = [];
    save(); io.emit('sync', db);
    res.json({ success: true });
});

app.post('/api/admin/allowPost', checkAdmin, (req, res) => {
    const p = db.forbiddenPosts.find(p => p.id == req.body.id);
    if (p) {
        db.posts.unshift(p);
        db.forbiddenPosts = db.forbiddenPosts.filter(x => x.id != req.body.id);
        save(); io.emit('sync', db);
    }
    res.json({ success: true });
});

app.get('*', (req, res) => {
    res.sendFile(path.join(__dirname, 'index.html'));
});

io.on('connection', (socket) => {
    socket.emit('sync', db);
    
    socket.on('post', (p) => {
        const textLower = (p.content || '').toLowerCase();
        const isForbidden = FORBIDDEN_WORDS.some(word => textLower.includes(word));
        
        if (isForbidden) {
            let user = db.users[p.author];
            if (user) {
                user.strikes = (user.strikes || 0) + 1;
                if (user.strikes >= 2) {
                    user.isBlocked = true;
                    io.emit('force_logout', p.author);
                }
            }
            db.forbiddenPosts.unshift({ ...p, id: Date.now(), reactions: [] });
            save(); io.emit('sync', db);
            
            if (user && user.strikes >= 2) {
                socket.emit('errorMsg', 'Ваш аккаунт заблокирован за нарушение правил.\nДля разблокировки обратитесь в поддержку:\nbuzzamix.support@gmail.com');
            } else {
                socket.emit('errorMsg', 'Предупреждение: Ваш пост содержит запрещенные материалы и отправлен на модерацию!');
            }
        } else {
            db.posts.unshift({ ...p, id: Date.now(), reactions: [] }); 
            save(); io.emit('sync', db); 
        }
    });

    socket.on('changePassword', (data) => {
        if (db.users[data.username]) {
            db.users[data.username].password = hashPwd(data.password);
            save();
        }
    });

    socket.on('toggleReaction', (data) => {
        const post = db.posts.find(p => p.id === data.postId);
        if (post) {
            if (!post.reactions) post.reactions = [];
            const idx = post.reactions.findIndex(r => r.user === data.user);
            if (idx > -1) {
                post.reactions.splice(idx, 1);
            } else {
                post.reactions.push({ user: data.user });
            }
            save(); io.emit('sync', db);
        }
    });

    socket.on('deleteAccount', (username) => {
        if (db.users[username]) {
            delete db.users[username];
            db.posts = db.posts.filter(p => p.author !== username);
            db.chats = db.chats.filter(c => c.u1 !== username && c.u2 !== username);
            Object.keys(db.rooms).forEach(k => {
                db.rooms[k].members = db.rooms[k].members.filter(m => m !== username);
            });
            delete db.subscriptions[username];
            save(); io.emit('sync', db);
        }
    });
    
    socket.on('msg', (m) => {
        let chat = db.chats.find(c => (c.u1 === m.from && c.u2 === m.to) || (c.u1 === m.to && c.u2 === m.from));
        if(!chat) { chat = { u1: m.from, u2: m.to, msgs: [] }; db.chats.push(chat); }
        chat.msgs.push({ sender: m.from, text: m.text, file: m.file, msgType: m.msgType, time: Date.now(), read: false });
        save(); io.emit('sync', db); 
        io.emit('notify', { to: m.to, from: m.from, text: m.file ? '📎 Файл' : m.text, type: 'dm' });
        
        if (db.subscriptions[m.to]) {
            const payload = JSON.stringify({
                title: `Новое сообщение от ${m.from}`,
                body: m.file ? '📎 Файл' : 'Новое сообщение',
                icon: '/favicon.ico',
                data: { type: 'dm', user: m.from }
            });
            webpush.sendNotification(db.subscriptions[m.to], payload).catch(e => {});
        }
    });

    socket.on('deleteChat', (data) => {
        db.chats = db.chats.filter(c => !((c.u1 === data.me && c.u2 === data.other) || (c.u1 === data.other && c.u2 === data.me)));
        save(); io.emit('sync', db);
    });

    socket.on('createRoom', (r, callback) => {
        let roomId = r.username ? r.username.toLowerCase().replace(/\s/g, '') : '';
        if (r.isPrivate) {
            roomId = 'priv_' + Math.random().toString(36).substring(2, 10);
        } else {
            if (!roomId || roomId.length < 5) {
                socket.emit('errorMsg', 'Юзернейм должен быть не менее 5 букв');
                if(callback) callback({success: false}); return;
            }
            if (isUsernameTaken(roomId)) {
                socket.emit('errorMsg', 'Этот юзернейм уже занят!');
                if(callback) callback({success: false}); return;
            }
        }
        
        db.rooms[roomId] = { type: r.type, isPrivate: r.isPrivate, name: r.name, desc: r.desc, photo: r.photo || defaultAvatar, admin: r.admin, members: [r.admin], msgs: [], isVerified: false, isBlocked: false };
        save(); io.emit('sync', db);
        if(callback) callback({success: true});
    });

    socket.on('editRoom', (data) => {
        const { oldId, newId, name, desc, photo, user } = data;
        if(db.rooms[oldId] && db.rooms[oldId].admin === user) {
            let targetId = oldId;
            if (newId && newId !== oldId && !db.rooms[oldId].isPrivate) {
                if (db.rooms[newId]) return socket.emit('errorMsg', 'Занят!');
                db.rooms[newId] = db.rooms[oldId]; delete db.rooms[oldId]; targetId = newId;
            }
            if(name) db.rooms[targetId].name = name;
            if(desc !== undefined) db.rooms[targetId].desc = desc;
            if(photo) db.rooms[targetId].photo = photo;
            save(); io.emit('sync', db);
        }
    });

    socket.on('deleteRoom', (data) => { if(db.rooms[data.roomId] && db.rooms[data.roomId].admin === data.user) { delete db.rooms[data.roomId]; save(); io.emit('sync', db); } });
    socket.on('leaveRoom', (data) => { if(db.rooms[data.roomId]) { db.rooms[data.roomId].members = db.rooms[data.roomId].members.filter(m => m !== data.user); save(); io.emit('sync', db); } });
    socket.on('joinRoom', (data) => { if(db.rooms[data.roomId] && !db.rooms[data.roomId].members.includes(data.user)) { db.rooms[data.roomId].members.push(data.user); save(); io.emit('sync', db); } });

    socket.on('roomMsg', (m) => {
        if(db.rooms[m.room]) {
            db.rooms[m.room].msgs.push({ sender: m.from, text: m.text, file: m.file, msgType: m.msgType, time: Date.now(), readers: [m.from] });
            save(); io.emit('sync', db);

            db.rooms[m.room].members.forEach(member => {
                if (member !== m.from && db.subscriptions[member]) {
                    const payload = JSON.stringify({
                        title: `${db.rooms[m.room].name}`,
                        body: `${m.from}: ${m.file ? '📎 Файл' : 'Новое сообщение'}`,
                        icon: db.rooms[m.room].photo || '/favicon.ico',
                        data: { type: 'room', user: m.room }
                    });
                    webpush.sendNotification(db.subscriptions[member], payload).catch(e => {});
                }
            });
        }
    });

    socket.on('markRead', (data) => {
        if(data.type === 'dm') {
            let chat = db.chats.find(c => (c.u1 === data.me && c.u2 === data.other) || (c.u1 === data.other && c.u2 === data.me));
            if(chat) chat.msgs.forEach(m => { if(m.sender !== data.me) m.read = true; });
        } else if(db.rooms[data.other]) {
            db.rooms[data.other].msgs.forEach(m => { if(!m.readers) m.readers = []; if(!m.readers.includes(data.me)) m.readers.push(data.me); });
        }
        save(); io.emit('sync', db);
    });

    socket.on('toggleFollow', (data) => {
        const me = db.users[data.me]; const target = db.users[data.target];
        if(me && target) {
            const idx = me.following.indexOf(data.target);
            if(idx > -1) { me.following.splice(idx, 1); target.followers = target.followers.filter(f => f !== data.me); } 
            else { me.following.push(data.target); target.followers.push(data.me); }
            save(); io.emit('sync', db); 
        }
    });

    socket.on('togglePin', (data) => {
        if (!db.users[data.user]) return;
        if (!db.users[data.user].pinned) db.users[data.user].pinned = [];
        const idx = db.users[data.user].pinned.indexOf(data.target);
        if (idx > -1) db.users[data.user].pinned.splice(idx, 1); else db.users[data.user].pinned.push(data.target);
        save(); io.emit('sync', db);
    });

    socket.on('updateProfile', (data) => {
        if(db.users[data.username]) { 
            if(data.name) db.users[data.username].name = data.name; 
            if(data.bio) db.users[data.username].bio = data.bio; 
            if(data.photo) db.users[data.username].photo = data.photo; 
            save(); io.emit('sync', db);
        }
    });

    socket.on('call-user', data => {
        io.emit('incoming-call', data);
        if (db.subscriptions[data.target]) {
            const payload = JSON.stringify({
                title: 'Входящий звонок',
                body: `Вам звонит: ${data.caller}`,
                icon: '/favicon.ico',
                data: { type: 'call', caller: data.caller },
                actions: [
                    { action: 'accept', title: '✅ Принять' },
                    { action: 'decline', title: '❌ Отклонить' }
                ]
            });
            webpush.sendNotification(db.subscriptions[data.target], payload).catch(e=>{});
        }
    });
    
    socket.on('call-answer', data => io.emit('call-answer', data));
    socket.on('call-ice', data => io.emit('call-ice', data));
    socket.on('call-end', data => io.emit('call-end', data));
    socket.on('call-busy', data => io.emit('call-busy', data));
    socket.on('call-room', (data) => {
        const r = db.rooms[data.room];
        if(r) r.members.forEach(m => { if(m !== data.caller) io.emit('incoming-room-call', { room: data.room, caller: data.caller, video: data.video, target: m }); });
    });
    socket.on('join-room-call', data => io.emit('user-joined-room-call', data));
    socket.on('call-room-end', data => {
        const r = db.rooms[data.room];
        if(r) r.members.forEach(m => { if(m !== data.caller) io.emit('call-end', { target: m, caller: data.caller }); });
    });
});

http.listen(3000, () => console.log('Buzzamix Active'));



Так смотри сделвй так чтоб в админ панеле при нажатие на чаты с пользователями, и наход нужного человека, я могу взять его end-to-end его яатов, чтоб я могу этот ключ дать органам, чтоб все было как у всех мессенджеров.

Сделац end-to-end для чатов, не только для каждого чата, чтоб я мог дать ключ и органы просмотрели историю пользователя.


ДОБАВЬ ТОЛЬКО ТЕ ИЗМЕНЕНИЯ КОТОРЫЕ Я ТЕБЕ СКАЗАЛ, ЧТО Я НЕ ГОВОРИЛ ТРОГАТЬ НЕ ТРОГАЙ !

ПРИШЛИ МНЕ ПОЛНОСТЬЮ ИЗМЕНЕННЫЕ КОДЫ, ВСЕ

ЖДУ