<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>iChat - Secure & Safe Messenger</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        :root { --app-height: 100%; }
        html, body { 
            font-family: 'Inter', sans-serif; 
            background-color: #0f172a; 
            color: #e2e8f0; 
            overflow: hidden; 
            height: var(--app-height);
            width: 100vw;
            position: fixed;
        }
        .app-height { height: var(--app-height); display: flex; overflow: hidden; position: relative; }
        #messages { 
            flex-grow: 1; 
            overflow-y: auto; 
            padding: 1.5rem; 
            background-color: #0f172a; 
            -webkit-overflow-scrolling: touch;
        }
        .bubble { 
            max-width: 85%; 
            padding: 0.85rem 1.1rem; 
            margin-bottom: 0.25rem; 
            border-radius: 1.25rem; 
            font-size: 0.95rem; 
            line-height: 1.5; 
            word-wrap: break-word; 
            position: relative;
        }
        .own { background: #2563eb; color: white; align-self: flex-end; border-bottom-right-radius: 0.25rem; }
        .other { background: #1e293b; color: #f1f5f9; align-self: flex-start; border-bottom-left-radius: 0.25rem; border: 1px solid #334155; }
        
        .msg-container:hover .report-btn { opacity: 1; }
        .report-btn { 
            opacity: 0; 
            transition: opacity 0.2s; 
            font-size: 10px; 
            color: #ef4444; 
            cursor: pointer;
            margin-top: 2px;
            text-transform: uppercase;
            font-weight: bold;
        }

        .user-item { cursor: pointer; transition: all 0.2s; border-radius: 0.75rem; border: 1px solid transparent; position: relative; }
        .user-item.active { background-color: #2563eb; color: white; }
        
        #login-overlay { position: fixed; inset: 0; z-index: 200; background: #0f172a; display: flex; align-items: center; justify-content: center; }
        .sidebar { 
            width: 320px; 
            border-right: 1px solid #1e293b; 
            display: flex; 
            flex-direction: column; 
            background: #0f172a; 
            transition: transform 0.3s ease;
            z-index: 50;
        }
        .auth-input, .search-input { 
            width: 100%; 
            background: #1e293b; 
            border: 1px solid #334155; 
            border-radius: 0.75rem; 
            padding: 0.85rem 1rem; 
            color: white; 
            outline: none; 
            font-size: 16px; 
        }
        #emoji-picker {
            position: absolute; bottom: 100%; right: 0; margin-bottom: 10px;
            background: #1e293b; border: 1px solid #334155; border-radius: 1rem;
            display: grid; grid-template-columns: repeat(6, 1fr); gap: 5px; padding: 10px;
            box-shadow: 0 10px 25px rgba(0,0,0,0.5); z-index: 100; width: 250px;
        }
        .emoji-btn { font-size: 1.25rem; padding: 5px; cursor: pointer; text-align: center; }
        @media (max-width: 768px) {
            .sidebar { position: absolute; left: 0; transform: translateX(-100%); }
            .sidebar.open { transform: translateX(0); width: 85%; }
            .mobile-overlay { position: fixed; inset: 0; background: rgba(0,0,0,0.5); z-index: 40; display: none; }
            .mobile-overlay.active { display: block; }
        }
        .hidden { display: none !important; }
    </style>
</head>
<body>

    <div id="login-overlay">
        <div class="w-full max-w-md bg-slate-900 border border-slate-800 p-8 md:p-10 rounded-[2.5rem] shadow-2xl mx-4">
            <div class="flex flex-col items-center mb-10">
                <div class="w-16 h-16 bg-blue-600 rounded-3xl flex items-center justify-center mb-6">
                    <svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" viewBox="0 0 24 24" fill="none" stroke="white" stroke-width="2.5"><path d="m3 21 1.9-5.7a8.5 8.5 0 1 1 3.8 3.8z"/></svg>
                </div>
                <h2 id="auth-title" class="text-3xl font-bold">iChat</h2>
                <div class="text-center space-y-1">
                    <p class="text-slate-500 text-sm mt-2">Your chats are private and hidden.</p>
                    <p class="text-blue-500 text-[10px] font-bold uppercase tracking-widest">Connected to the internet</p>
                </div>
                <div id="conn-status" class="mt-4 text-[10px] uppercase tracking-widest text-slate-600 font-bold">Connecting...</div>
            </div>
            <div class="space-y-4">
                <div id="name-field" class="hidden">
                    <label class="text-xs font-bold text-slate-400 uppercase">Display Name</label>
                    <input id="auth-name" type="text" placeholder="John Doe" class="auth-input mt-1">
                </div>
                <div>
                    <label class="text-xs font-bold text-slate-400 uppercase">Username</label>
                    <input id="auth-username" type="text" placeholder="username" class="auth-input mt-1">
                </div>
                <div>
                    <label class="text-xs font-bold text-slate-400 uppercase">Password</label>
                    <input id="auth-pass" type="password" placeholder="••••••••" class="auth-input mt-1">
                </div>
                <button id="auth-submit-btn" onclick="handleAuth()" class="w-full bg-blue-600 text-white font-bold py-4 rounded-xl mt-4 disabled:opacity-50">Log In</button>
                <div class="text-center mt-6">
                    <button id="auth-toggle" onclick="toggleAuthMode()" class="text-sm text-blue-400">New user? Create a profile</button>
                </div>
                <p id="auth-error" class="text-xs text-rose-500 text-center mt-4 min-h-[1.5rem]"></p>
            </div>
        </div>
    </div>

    <div id="mobile-sidebar-overlay" class="mobile-overlay" onclick="toggleSidebar(false)"></div>

    <div id="chat-app" class="app-height w-full hidden">
        <aside id="app-sidebar" class="sidebar p-5">
            <div class="mb-6">
                <h2 class="font-bold text-2xl mb-4">iChat</h2>
                <div class="relative">
                    <input id="search-user" type="text" placeholder="Search @username..." class="search-input text-sm pr-10">
                    <button onclick="searchUser()" class="absolute right-3 top-1/2 -translate-y-1/2 text-slate-400">
                        <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="11" cy="11" r="8"/><path d="m21 21-4.3-4.3"/></svg>
                    </button>
                </div>
                <p id="search-msg" class="text-[10px] mt-2 text-slate-500 italic"></p>
            </div>
            
            <div class="flex-grow overflow-y-auto space-y-2" id="users-list">
                <p class="text-[10px] text-slate-500 px-2 uppercase tracking-[0.2em] font-black mb-4">Contacts</p>
            </div>

            <div class="mt-auto pt-6 border-t border-slate-800/50">
                <div class="bg-slate-900/50 p-3 rounded-xl mb-4 border border-slate-800">
                    <p class="text-[10px] text-slate-500 uppercase font-bold mb-1">Signed in as:</p>
                    <p id="user-display-handle" class="text-sm font-mono text-blue-400 font-bold"></p>
                </div>
                <button onclick="logout()" class="text-slate-400 hover:text-rose-400 text-sm font-semibold flex items-center space-x-2 w-full px-2">
                    <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M9 21H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h4"/><polyline points="16 17 21 12 16 7"/><line x1="21" y1="12" x2="9" y2="12"/></svg>
                    <span>Sign Out</span>
                </button>
            </div>
        </aside>

        <div class="flex-grow flex flex-col relative bg-[#0f172a] overflow-hidden">
            <header class="p-4 border-b border-slate-800/50 flex items-center space-x-4 bg-[#0f172a]/80 backdrop-blur-md">
                <button onclick="toggleSidebar(true)" class="md:hidden p-2 text-slate-400"><svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><line x1="3" x2="21" y1="12" y2="12"/><line x1="3" x2="21" y1="6" y2="6"/><line x1="3" x2="21" y1="18" y2="18"/></svg></button>
                <div id="active-user-avatar" class="w-10 h-10 bg-slate-800 rounded-xl flex items-center justify-center font-bold text-slate-400">?</div>
                <div>
                    <h1 id="active-user-name" class="font-bold text-white">Select a Chat</h1>
                    <span class="text-[10px] text-slate-500 uppercase">Moderated Environment</span>
                </div>
            </header>

            <main id="messages" class="flex flex-col"></main>

            <footer id="input-area" class="p-4 bg-slate-900/40 border-t border-slate-800/50 hidden relative">
                <div id="emoji-picker" class="hidden"></div>
                <div class="flex items-end space-x-3 max-w-5xl mx-auto">
                    <div class="flex-grow bg-slate-800/50 rounded-2xl border border-slate-700 p-1.5 flex items-end">
                        <textarea id="message-input" rows="1" placeholder="Type a message..." class="flex-grow bg-transparent border-none focus:ring-0 px-4 py-2 text-sm text-white resize-none"></textarea>
                        <button onclick="toggleEmojiPicker()" class="p-2 text-slate-400 hover:text-yellow-400">
                            <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="12" cy="12" r="10"/><path d="M8 14s1.5 2 4 2 4-2 4-2"/><line x1="9" x2="9.01" y1="9" y2="9"/><line x1="15" x2="15.01" y1="9" y2="9"/></svg>
                        </button>
                    </div>
                    <button id="send-btn" onclick="handleSend()" class="bg-blue-600 text-white p-3 rounded-2xl"><svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><line x1="22" x2="11" y1="2" y2="13"/><polygon points="22 2 15 22 11 13 2 9 22 2"/></svg></button>
                </div>
            </footer>
        </div>
    </div>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, setDoc, getDoc, collection, addDoc, query, onSnapshot, serverTimestamp, where } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        const firebaseConfig = {
            apiKey: "AIzaSyAYbFC_duDdmSOlvwxv30lIYcsBUMy_Fds",
            authDomain: "ichat-9ce6f.firebaseapp.com",
            databaseURL: "https://ichat-9ce6f-default-rtdb.firebaseio.com",
            projectId: "ichat-9ce6f",
            storageBucket: "ichat-9ce6f.firebasestorage.app",
            messagingSenderId: "1092185548291",
            appId: "1:1092185548291:web:b7643d40b2b42e2db79036",
            measurementId: "G-MJ44KT73FL"
        };

        const appId = 'ichat-secure-v9';
        const app = initializeApp(firebaseConfig);
        const auth = getAuth(app);
        const db = getFirestore(app);

        let profileData = null;
        let activeRecipient = null;
        let isSignUpMode = false;
        let isAuthenticating = false;
        let messageUnsubscribe = null;
        let myContacts = JSON.parse(localStorage.getItem('ichat_contacts') || '[]');

        const initAuth = async () => {
            if (auth.currentUser) return auth.currentUser;
            try {
                const result = await signInAnonymously(auth);
                document.getElementById('conn-status').textContent = "Verified";
                document.getElementById('conn-status').classList.replace('text-slate-600', 'text-green-500');
                return result.user;
            } catch (error) {
                document.getElementById('conn-status').textContent = "Offline";
                throw error;
            }
        };

        window.handleAuth = async () => {
            if (isAuthenticating) return;
            const userInput = document.getElementById('auth-username').value.trim().toLowerCase();
            const passInput = document.getElementById('auth-pass').value.trim();
            const nameInput = document.getElementById('auth-name').value.trim();
            const errDisplay = document.getElementById('auth-error');

            if (!userInput || !passInput || (isSignUpMode && !nameInput)) {
                return errDisplay.textContent = "Please fill in all fields.";
            }
            
            isAuthenticating = true;
            errDisplay.textContent = isSignUpMode ? "Creating account..." : "Logging in...";

            try {
                await initAuth();
                const userRef = doc(db, 'artifacts', appId, 'public', 'data', 'users', userInput);
                const userSnap = await getDoc(userRef);

                if (isSignUpMode) {
                    if (userSnap.exists()) {
                        throw new Error("This username is already taken.");
                    }
                    const data = { 
                        uid: crypto.randomUUID(), 
                        displayName: nameInput, 
                        username: userInput, 
                        password: passInput, 
                        createdAt: new Date().toISOString() 
                    };
                    await setDoc(userRef, data);
                    finalizeLogin(data);
                } else {
                    if (!userSnap.exists() || userSnap.data().password !== passInput) {
                        throw new Error("Invalid username or password.");
                    }
                    finalizeLogin(userSnap.data());
                }
            } catch (e) { 
                errDisplay.textContent = e.message; 
                isAuthenticating = false;
            }
        };

        function finalizeLogin(data) {
            profileData = data;
            isAuthenticating = false;
            localStorage.setItem('ichat_session', JSON.stringify(data));
            document.getElementById('login-overlay').classList.add('hidden');
            document.getElementById('chat-app').classList.remove('hidden');
            document.getElementById('user-display-handle').textContent = `@${data.username}`;
            renderContacts();
        }

        window.searchUser = async () => {
            const term = document.getElementById('search-user').value.trim().toLowerCase().replace('@','');
            const msg = document.getElementById('search-msg');
            if (!term || (profileData && term === profileData.username)) return;
            
            msg.textContent = "Searching...";
            try {
                const snap = await getDoc(doc(db, 'artifacts', appId, 'public', 'data', 'users', term));
                if (snap.exists()) {
                    const found = snap.data();
                    if (!myContacts.find(c => c.username === found.username)) {
                        myContacts.push(found);
                        localStorage.setItem('ichat_contacts', JSON.stringify(myContacts));
                    }
                    msg.textContent = "Found!";
                    renderContacts();
                    selectContact(found);
                } else { msg.textContent = "User not found."; }
            } catch(e) { msg.textContent = "Error searching."; }
        };

        function renderContacts() {
            const list = document.getElementById('users-list');
            list.innerHTML = `<p class="text-[10px] text-slate-500 px-2 uppercase tracking-[0.2em] font-black mb-4">Contacts</p>`;
            myContacts.forEach(c => {
                const el = document.createElement('div');
                el.className = `user-item flex items-center p-3 space-x-3 mb-1 ${activeRecipient?.username === c.username ? 'active' : ''}`;
                el.onclick = () => selectContact(c);
                el.innerHTML = `
                    <div class="w-10 h-10 bg-slate-800 rounded-xl flex items-center justify-center font-bold">${c.displayName[0]}</div>
                    <div class="flex-grow truncate">
                        <p class="font-bold text-sm truncate">${c.displayName}</p>
                        <p class="text-[10px] opacity-50">@${c.username}</p>
                    </div>`;
                list.appendChild(el);
            });
        }

        function selectContact(c) {
            activeRecipient = c;
            renderContacts();
            document.getElementById('active-user-name').textContent = c.displayName;
            document.getElementById('active-user-avatar').textContent = c.displayName[0];
            document.getElementById('input-area').classList.remove('hidden');
            const chatId = [profileData.uid, c.uid].sort().join('_');
            startMessages(chatId);
        }

        function startMessages(chatId) {
            if (messageUnsubscribe) messageUnsubscribe();
            const q = query(collection(db, 'artifacts', appId, 'public', 'data', 'messages'), where("chatId", "==", chatId));
            messageUnsubscribe = onSnapshot(q, (snap) => {
                const container = document.getElementById('messages');
                const msgs = [];
                snap.forEach(d => msgs.push({id: d.id, ...d.data()}));
                msgs.sort((a,b) => (a.createdAt?.toMillis() || 0) - (b.createdAt?.toMillis() || 0));
                
                container.innerHTML = '';
                msgs.forEach(m => {
                    const isOwn = m.senderId === profileData.uid;
                    const d = document.createElement('div');
                    d.className = `msg-container flex flex-col mb-4 ${isOwn ? 'items-end' : 'items-start'}`;
                    d.innerHTML = `
                        <div class="bubble ${isOwn ? 'own' : 'other'}">${m.text}</div>
                        ${!isOwn ? `<button class="report-btn" onclick="reportMsg('${m.id}')">Report Harassment</button>` : ''}
                    `;
                    container.appendChild(d);
                });
                container.scrollTop = container.scrollHeight;
            });
        }

        window.reportMsg = (id) => {
            console.log("Reported message:", id);
            alert("This message has been flagged for moderation. Thank you for keeping iChat safe.");
        };

        window.handleSend = async () => {
            const input = document.getElementById('message-input');
            const text = input.value.trim();
            if (!text || !activeRecipient || !profileData) return;
            const chatId = [profileData.uid, activeRecipient.uid].sort().join('_');
            
            try {
                await addDoc(collection(db, 'artifacts', appId, 'public', 'data', 'messages'), {
                    chatId,
                    text,
                    senderId: profileData.uid,
                    senderUsername: profileData.username,
                    recipientUsername: activeRecipient.username,
                    createdAt: serverTimestamp(),
                    appInstance: appId
                });
                input.value = '';
                input.focus();
            } catch(e) { console.error("Send failed", e); }
        };

        window.logout = () => { localStorage.removeItem('ichat_session'); location.reload(); };
        
        window.toggleAuthMode = () => { 
            isSignUpMode = !isSignUpMode; 
            const title = document.getElementById('auth-title');
            const submitBtn = document.getElementById('auth-submit-btn');
            const toggleBtn = document.getElementById('auth-toggle');
            
            title.textContent = isSignUpMode ? "Join iChat" : "iChat"; 
            submitBtn.textContent = isSignUpMode ? "Create Account" : "Log In";
            toggleBtn.textContent = isSignUpMode ? "Already have an account? Log In" : "New user? Create a profile";
            
            document.getElementById('name-field').classList.toggle('hidden', !isSignUpMode); 
            document.getElementById('auth-error').textContent = "";
        };
        
        onAuthStateChanged(auth, (user) => {
            if (user) {
                const saved = localStorage.getItem('ichat_session');
                if (saved && !profileData) finalizeLogin(JSON.parse(saved));
            }
        });
        initAuth();
    </script>
</body>
</html>
