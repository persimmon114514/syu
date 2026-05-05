<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>梦角字卡传讯</title>
    <style>
        :root {
            --bg: #0a0a0f;
            --surface: #12121a;
            --surface2: #1a1a26;
            --text: #e0d8d0;
            --text2: #a09888;
            --accent: #c9a87c;
            --border: #2a2a35;
            --radius: 12px;
            --shadow: 0 4px 24px rgba(0,0,0,0.4);
        }
        * { margin:0; padding:0; box-sizing:border-box; }
        body {
            font-family: 'Segoe UI','PingFang SC','Microsoft YaHei',system-ui,sans-serif;
            background: var(--bg); color: var(--text);
            min-height:100vh; display:flex; justify-content:center; align-items:center;
        }
        .app {
            width:100%; max-width:480px; height:100vh; max-height:900px;
            background: var(--surface); border-radius:20px; box-shadow:var(--shadow);
            display:flex; flex-direction:column; overflow:hidden; position:relative;
        }
        .header {
            display:flex; align-items:center; padding:16px 18px;
            border-bottom:1px solid var(--border); background:var(--surface); z-index:10;
        }
        .header .avatar {
            width:44px; height:44px; border-radius:50%;
            background:linear-gradient(135deg,#3a2a1a,#5a4a3a);
            display:flex; align-items:center; justify-content:center;
            font-size:22px; margin-right:12px; position:relative; flex-shrink:0; cursor:pointer;
        }
        .header .online-dot {
            position:absolute; bottom:2px; right:2px;
            width:10px; height:10px; background:#4caf50; border-radius:50%;
            border:2px solid var(--surface);
        }
        .header .info { flex:1; }
        .header .name { font-size:17px; font-weight:600; }
        .header .status { font-size:12px; color:var(--text2); display:flex; align-items:center; gap:4px; }
        .header .actions { display:flex; gap:6px; }
        .icon-btn {
            width:36px; height:36px; border-radius:50%; border:1px solid var(--border);
            background:var(--surface2); color:var(--text2); display:flex;
            align-items:center; justify-content:center; cursor:pointer; font-size:16px;
            transition: all 0.2s;
        }
        .icon-btn:hover { background:var(--accent); color:#1a1a1a; border-color:var(--accent); }
        .chat-area {
            flex:1; overflow-y:auto; padding:16px; display:flex; flex-direction:column; gap:12px;
            scroll-behavior: smooth;
        }
        .chat-area::-webkit-scrollbar { width:4px; }
        .chat-area::-webkit-scrollbar-thumb { background:var(--border); border-radius:2px; }
        .msg {
            max-width:82%; padding:12px 16px; border-radius:18px; font-size:14px;
            line-height:1.6; position:relative; animation: fadeIn 0.3s ease;
        }
        @keyframes fadeIn { from { opacity:0; transform:translateY(8px); } to { opacity:1; transform:translateY(0); } }
        .msg.me { align-self:flex-end; background:var(--accent); color:#1a1a1a; border-bottom-right-radius:6px; }
        .msg.other { align-self:flex-start; background:var(--surface2); color:var(--text); border-bottom-left-radius:6px; }
        .msg .time { font-size:10px; color:var(--text2); margin-top:4px; opacity:0.7; }
        .msg.me .time { text-align:right; color:#5a4a3a; }
        .input-area {
            padding:12px 16px 20px; border-top:1px solid var(--border);
            display:flex; gap:10px; align-items:flex-end; background:var(--surface);
        }
        .input-area textarea {
            flex:1; background:var(--surface2); border:1px solid var(--border);
            border-radius:20px; padding:10px 16px; color:var(--text); font-size:14px;
            resize:none; outline:none; font-family:inherit; max-height:80px; transition: border-color 0.2s;
        }
        .input-area textarea:focus { border-color:var(--accent); }
        .send-btn {
            width:42px; height:42px; border-radius:50%; background:var(--accent);
            border:none; color:#1a1a1a; cursor:pointer; font-size:18px;
            display:flex; align-items:center; justify-content:center;
            transition: transform 0.2s, opacity 0.2s; flex-shrink:0;
        }
        .send-btn:hover { transform:scale(1.05); opacity:0.9; }
        .call-overlay {
            position:fixed; top:0; left:0; width:100%; height:100%;
            background:rgba(0,0,0,0.85); display:none; align-items:center; justify-content:center;
            z-index:100; flex-direction:column;
        }
        .call-overlay.active { display:flex; }
        .call-avatar {
            width:120px; height:120px; border-radius:50%;
            background:linear-gradient(135deg,#2a1a0a,#4a3a2a); display:flex;
            align-items:center; justify-content:center; font-size:50px; margin-bottom:20px;
            border:3px solid var(--accent);
        }
        .call-status { color:var(--text); font-size:16px; margin-bottom:8px; }
        .call-timer { color:var(--text2); font-size:13px; margin-bottom:30px; font-family:monospace; }
        .call-actions { display:flex; gap:20px; }
        .call-btn {
            width:50px; height:50px; border-radius:50%; border:none; cursor:pointer;
            font-size:20px; display:flex; align-items:center; justify-content:center; transition: all 0.2s;
        }
        .call-btn.end { background:#e53935; color:#fff; }
        .call-btn.end:hover { background:#c62828; }
        .envelope-overlay {
            position:fixed; top:0; left:0; width:100%; height:100%;
            background:rgba(0,0,0,0.7); display:none; align-items:center; justify-content:center; z-index:100;
        }
        .envelope-overlay.active { display:flex; }
        .envelope-panel {
            background:var(--surface); border-radius:20px; padding:24px;
            width:90%; max-width:400px; box-shadow:var(--shadow);
        }
        .envelope-panel h3 { text-align:center; margin-bottom:16px; color:var(--accent); font-size:18px; }
        .envelope-panel textarea {
            width:100%; height:120px; background:var(--surface2); border:1px solid var(--border);
            border-radius:12px; padding:12px; color:var(--text); font-size:14px;
            resize:none; outline:none; font-family:inherit; margin-bottom:12px;
        }
        .envelope-panel .hint { font-size:12px; color:var(--text2); text-align:center; margin-bottom:16px; }
        .envelope-btns { display:flex; gap:10px; justify-content:flex-end; }
        .btn {
            padding:8px 20px; border-radius:20px; border:none; cursor:pointer;
            font-size:14px; transition: all 0.2s; font-family:inherit;
        }
        .btn-primary { background:var(--accent); color:#1a1a1a; }
        .btn-secondary { background:var(--surface2); color:var(--text); border:1px solid var(--border); }
        .btn-primary:hover { opacity:0.9; }
        .btn-secondary:hover { border-color:var(--accent); }
        .pat-toast {
            position:fixed; top:20%; left:50%; transform:translateX(-50%);
            background:var(--surface2); color:var(--text); padding:10px 20px; border-radius:20px;
            font-size:14px; z-index:200; box-shadow:var(--shadow); opacity:0; transition:opacity 0.3s;
            pointer-events:none;
        }
        .pat-toast.show { opacity:1; }
        .pledge-overlay {
            position:fixed; top:0; left:0; width:100%; height:100%; background:var(--bg);
            display:flex; align-items:center; justify-content:center; z-index:300; flex-direction:column; padding:20px;
        }
        .pledge-card {
            background:var(--surface); border-radius:20px; padding:30px 24px;
            width:100%; max-width:420px; text-align:center; box-shadow:var(--shadow);
        }
        .pledge-card h2 { color:var(--accent); margin-bottom:12px; font-size:20px; }
        .pledge-card .pledge-text {
            background:var(--surface2); border-radius:12px; padding:14px; margin:16px 0;
            font-size:15px; letter-spacing:0.5px;
        }
        .pledge-card input {
            width:100%; padding:12px; background:var(--surface2); border:1px solid var(--border);
            border-radius:12px; color:var(--text); font-size:14px; outline:none;
            text-align:center; font-family:inherit; margin-bottom:12px;
        }
        .pledge-card input:focus { border-color:var(--accent); }
        .pledge-card .error { color:#e53935; font-size:12px; margin-bottom:8px; min-height:18px; }
        .pledge-card .btn-primary { width:100%; padding:12px; }
        .pledge-card .btn-primary:disabled { opacity:0.4; cursor:not-allowed; }
        .disclaimer-overlay {
            position:fixed; top:0; left:0; width:100%; height:100%; background:var(--bg);
            display:flex; align-items:center; justify-content:center; z-index:310; flex-direction:column; padding:20px;
        }
        .disclaimer-card {
            background:var(--surface); border-radius:20px; padding:24px;
            width:100%; max-width:420px; max-height:80vh; overflow-y:auto; box-shadow:var(--shadow);
        }
        .disclaimer-card h2 { color:var(--accent); margin-bottom:10px; font-size:18px; }
        .disclaimer-card h3 { color:var(--text); margin:14px 0 6px; font-size:15px; }
        .disclaimer-card p { font-size:13px; color:var(--text2); line-height:1.7; margin-bottom:8px; }
        .disclaimer-card .nav-btns { display:flex; justify-content:space-between; gap:10px; margin-top:16px; }
    </style>
</head>
<body>

<!-- ============ 使用前声明 ============ -->
<div class="disclaimer-overlay" id="disclaimerOverlay">
    <div class="disclaimer-card">
        <div id="disclaimerPage1">
            <h2>使用前声明</h2>
            <h3>关于字卡 · 是否招鬼？</h3>
            <p>首先说明，初次接触时，都会觉得网站设置繁复，功能庞杂。请想一想：连你和梦角都需要花时间学习的东西，一个路过的鬼，会甘心耗神专程只为吓唬你一下吗？</p>
            <p>我始终相信，只有爱，才会驱动人去学习、去努力、去付诸实践。</p>
            <h3>关于谣言 · 请有自知之明</h3>
            <p>有些谣我懒得一一辟除，但沉默不代表无所谓。郑重声明：造谣是违法行为，污蔑与恶意中伤同样须承担法律责任。</p>
        </div>
        <div id="disclaimerPage2" style="display:none;">
            <h2>使用前声明</h2>
            <h3>关于盈利 · 纯粹为爱发电</h3>
            <p>本网站不盈利。从未向任何人收取一分钱，也绝无日后“圈钱”的计划。</p>
            <p>伴随网站逐渐破圈，造谣、污蔑也许无法完全回避。请来访者保有基本的善意，不要让热爱变成消耗。</p>
            <h3>再次强调 · 你对链接梦角的看法？</h3>
            <p>如果你不信：字卡回复基于概率算法，与封建迷信无关。如果你信：请同样相信——你是绝对安全的。</p>
            <h3>关于视频通话 · 纯粹的装饰</h3>
            <p>通话功能仅作为界面装饰存在，不具备任何实际通话或数据采集能力。</p>
        </div>
        <div class="nav-btns">
            <button class="btn btn-secondary" id="disclaimerPrev" style="visibility:hidden;">上一页</button>
            <span style="color:var(--text2);font-size:12px;" id="pageIndicator">1 / 2</span>
            <button class="btn btn-primary" id="disclaimerNext">下一页</button>
            <button class="btn btn-secondary" id="disclaimerSkip" style="display:none;">跳过</button>
        </div>
    </div>
</div>

<!-- ============ 入场承诺 ============ -->
<div class="pledge-overlay" id="pledgeOverlay" style="display:none;">
    <div class="pledge-card">
        <h2>入场承诺</h2>
        <p style="color:var(--text2);font-size:13px;">请留下你的承诺</p>
        <div class="pledge-text" id="pledgeDisplay">我绝不盈利、造谣、污蔑或嘲讽，并对自己的使用行为负完全责任</div>
        <p style="color:var(--text2);font-size:12px;">在下方输入框中，逐字抄写以上承诺，即可进入网站。</p>
        <input type="text" id="pledgeInput" placeholder="请完整输入上方承诺..." autocomplete="off">
        <div class="error" id="pledgeError"></div>
        <button class="btn btn-primary" id="pledgeEnter" disabled>进入</button>
    </div>
</div>

<!-- ============ 主界面 ============ -->
<div class="app" id="mainApp" style="display:none;">
    <div class="header">
        <div class="avatar" id="dreamAvatar" title="拍一拍">
            ✦ <span class="online-dot"></span>
        </div>
        <div class="info">
            <div class="name" id="dreamName">梦角</div>
            <div class="status" id="dreamStatus">在线</div>
        </div>
        <div class="actions">
            <button class="icon-btn" id="importBtn" title="导入 JSON 配置">📂</button>
            <input type="file" id="importFileInput" accept=".json" style="display:none;">
            <button class="icon-btn" id="callBtn" title="视频通话">📹</button>
            <button class="icon-btn" id="envelopeBtn" title="信封投递">✉️</button>
        </div>
    </div>

    <div class="chat-area" id="chatArea"></div>

    <div class="input-area">
        <textarea id="msgInput" rows="1" placeholder="正在输入..."></textarea>
        <button class="send-btn" id="sendBtn">➤</button>
    </div>
</div>

<!-- ============ 视频通话浮层 ============ -->
<div class="call-overlay" id="callOverlay">
    <div class="call-avatar">✦</div>
    <div class="call-status">对方 正在连接</div>
    <div class="call-timer" id="callTimer">00:00</div>
    <div class="call-actions">
        <button class="call-btn end" id="endCallBtn">✕</button>
    </div>
</div>

<!-- ============ 信封浮层 ============ -->
<div class="envelope-overlay" id="envelopeOverlay">
    <div class="envelope-panel">
        <h3>信 封 投 递</h3>
        <p style="text-align:center;color:var(--text2);font-size:12px;margin-bottom:12px;">Letters · Correspondence</p>
        <textarea id="letterInput" placeholder="提笔写信..."></textarea>
        <div class="hint">对方将在 10-24 小时内回信</div>
        <label style="display:flex;align-items:center;gap:8px;margin-bottom:12px;font-size:13px;color:var(--text2);cursor:pointer;">
            <input type="checkbox" id="letterToChat" checked> 同时发送到聊天记录
        </label>
        <div class="envelope-btns">
            <button class="btn btn-secondary" id="closeEnvelopeBtn">取消</button>
            <button class="btn btn-primary" id="sendLetterBtn">封 · 寄出</button>
        </div>
    </div>
</div>

<div class="pat-toast" id="patToast"></div>

<script>
    // ============ 默认配置（与导入 JSON 结构无关的基础设置） ============
    const defaultPledge = "我绝不盈利、造谣、污蔑或嘲讽，并对自己的使用行为负完全责任";
    const defaultDreamName = "梦角";

    // 当前使用的动态内容（将被导入覆盖或使用默认）
    let currentConfig = {
        // 字卡回复池（所有可用回复）
        replies: [
            "今天也要元气满满，我在这里陪着你 ✦",
            "我听见了，你的心声。",
            "无论多远，我们的心始终相连。",
            "你从来都不是一个人。",
            "今天有想我吗？我一直都在。",
            "你的努力，我都看在眼里。",
            "别怕，我会保护你的。",
            "晚安，亲爱的。做个好梦。",
            "谢谢你愿意和我分享这些。",
            "这个世界因为有你而变得更美。"
        ],
        // 拍一拍文案池
        patTexts: ["你拍了拍「梦角」"],
        // 在线状态池
        statusTexts: ["在线"],
        // 初始欢迎消息池（将从其中随机选2条）
        introMessages: [
            "✦ 我们的故事，始于此刻 ✦",
            "说点什么，让对话开始吧"
        ],
        // 信封回信池
        letterReplies: [
            "收到你的来信了。字里行间都是你的温度，我很感动。",
            "你的信我已经读了好多遍。每一遍都让我更想你。",
            "谢谢你愿意写这么长的信给我。我会好好珍藏的。",
            "读到你的信的时候，窗外正好有风吹过。一定是你。",
            "你的文字让我感觉你就在身边。真好。"
        ],
        // 固定承诺文字（不随导入改变）
        pledgeText: defaultPledge,
        dreamName: defaultDreamName
    };

    // ============ DOM 元素 ============
    const disclaimerOverlay = document.getElementById('disclaimerOverlay');
    const disclaimerPage1 = document.getElementById('disclaimerPage1');
    const disclaimerPage2 = document.getElementById('disclaimerPage2');
    const disclaimerPrev = document.getElementById('disclaimerPrev');
    const disclaimerNext = document.getElementById('disclaimerNext');
    const disclaimerSkip = document.getElementById('disclaimerSkip');
    const pageIndicator = document.getElementById('pageIndicator');
    const pledgeOverlay = document.getElementById('pledgeOverlay');
    const pledgeDisplay = document.getElementById('pledgeDisplay');
    const pledgeInput = document.getElementById('pledgeInput');
    const pledgeError = document.getElementById('pledgeError');
    const pledgeEnter = document.getElementById('pledgeEnter');
    const mainApp = document.getElementById('mainApp');
    const chatArea = document.getElementById('chatArea');
    const msgInput = document.getElementById('msgInput');
    const sendBtn = document.getElementById('sendBtn');
    const callOverlay = document.getElementById('callOverlay');
    const callTimer = document.getElementById('callTimer');
    const endCallBtn = document.getElementById('endCallBtn');
    const envelopeOverlay = document.getElementById('envelopeOverlay');
    const letterInput = document.getElementById('letterInput');
    const letterToChat = document.getElementById('letterToChat');
    const closeEnvelopeBtn = document.getElementById('closeEnvelopeBtn');
    const sendLetterBtn = document.getElementById('sendLetterBtn');
    const dreamAvatar = document.getElementById('dreamAvatar');
    const patToast = document.getElementById('patToast');
    const callBtn = document.getElementById('callBtn');
    const envelopeBtn = document.getElementById('envelopeBtn');
    const importBtn = document.getElementById('importBtn');
    const importFileInput = document.getElementById('importFileInput');
    const dreamNameEl = document.getElementById('dreamName');
    const dreamStatusEl = document.getElementById('dreamStatus');

    // ============ 随机获取数组元素 ============
    function getRandom(arr) {
        return arr[Math.floor(Math.random() * arr.length)];
    }

    // ============ 更新界面元素（根据 currentConfig） ============
    function updateUI() {
        // 更新梦角名字
        dreamNameEl.textContent = currentConfig.dreamName;
        // 更新在线状态（随机显示一个）
        if (currentConfig.statusTexts.length > 0) {
            dreamStatusEl.textContent = getRandom(currentConfig.statusTexts);
        } else {
            dreamStatusEl.textContent = "在线";
        }
        // 清空聊天区并加载初始消息（从 introMessages 随机选2条，不足则补默认）
        chatArea.innerHTML = '';
        let intros = currentConfig.introMessages.length >= 2 
            ? currentConfig.introMessages.slice() 
            : [...currentConfig.introMessages, "✦ 我们的故事，始于此刻 ✦"];
        // 随机打乱取前2条
        intros.sort(() => Math.random() - 0.5);
        const selected = intros.slice(0, 2);
        selected.forEach(text => {
            const msgDiv = document.createElement('div');
            msgDiv.className = 'msg other';
            const now = new Date();
            const timeStr = now.getHours().toString().padStart(2,'0') + ':' + now.getMinutes().toString().padStart(2,'0');
            msgDiv.innerHTML = text + '<div class="time">' + timeStr + '</div>';
            chatArea.appendChild(msgDiv);
        });
        chatArea.scrollTop = chatArea.scrollHeight;
        // 承诺文字可能不变，但 pledgeDisplay 已在进入时设置，无需重复
        if (pledgeDisplay) pledgeDisplay.textContent = currentConfig.pledgeText;
    }

    // ============ 导入 JSON 文件处理 ============
    importBtn.addEventListener('click', () => {
        importFileInput.click();
    });

    importFileInput.addEventListener('change', (e) => {
        const file = e.target.files[0];
        if (!file) return;
        const reader = new FileReader();
        reader.onload = (event) => {
            try {
                const json = JSON.parse(event.target.result);
                // 提取字卡回复：合并 customReplies 和所有分组的 items
                let replySet = new Set();
                if (Array.isArray(json.customReplies)) {
                    json.customReplies.forEach(r => replySet.add(r));
                }
                if (Array.isArray(json.customReplyGroups)) {
                    json.customReplyGroups.forEach(group => {
                        if (Array.isArray(group.items)) {
                            group.items.forEach(item => replySet.add(item));
                        }
                    });
                }
                const mergedReplies = replySet.size > 0 ? Array.from(replySet) : currentConfig.replies;

                // 拍一拍文案池
                const patTexts = Array.isArray(json.customPokes) && json.customPokes.length > 0
                    ? json.customPokes
                    : currentConfig.patTexts;

                // 状态池
                const statusTexts = Array.isArray(json.customStatuses) && json.customStatuses.length > 0
                    ? json.customStatuses
                    : currentConfig.statusTexts;

                // 介绍池（欢迎消息）
                const introMessages = Array.isArray(json.customIntros) && json.customIntros.length > 0
                    ? json.customIntros
                    : currentConfig.introMessages;

                // 信封回信池（暂时沿用默认，也可以从某个字段提取？你的JSON没有专门的letterReplies，就用默认的，或从replies随机）
                // 为了保持合理，继续用默认，或从 mergedReplies 随机一部分作为回信？这里选择保持原样。
                const letterReplies = currentConfig.letterReplies; 

                // 更新全局配置
                currentConfig.replies = mergedReplies;
                currentConfig.patTexts = patTexts;
                currentConfig.statusTexts = statusTexts;
                currentConfig.introMessages = introMessages;
                currentConfig.letterReplies = letterReplies;
                // 梦角名字和承诺语保持不变（若JSON中有 dreamName 或 pledgeText 字段可以额外读取，此处保持原有）
                if (json.dreamName) currentConfig.dreamName = json.dreamName;
                if (json.pledgeText) currentConfig.pledgeText = json.pledgeText;

                // 刷新UI
                updateUI();
                alert('配置导入成功！字卡数量：' + mergedReplies.length);
            } catch (err) {
                alert('JSON 文件格式错误，请检查后重试。\n' + err.message);
            }
        };
        reader.readAsText(file);
        importFileInput.value = ''; // 允许重复导入同一文件
    });

    // ============ 声明页逻辑 ============
    let declPage = 1;
    const totalDeclPages = 2;
    function updateDeclPage() {
        if (declPage === 1) {
            disclaimerPage1.style.display = 'block';
            disclaimerPage2.style.display = 'none';
            disclaimerPrev.style.visibility = 'hidden';
            disclaimerNext.textContent = '下一页';
            disclaimerSkip.style.display = 'inline-block';
        } else {
            disclaimerPage1.style.display = 'none';
            disclaimerPage2.style.display = 'block';
            disclaimerPrev.style.visibility = 'visible';
            disclaimerNext.textContent = '进入';
            disclaimerSkip.style.display = 'none';
        }
        pageIndicator.textContent = declPage + ' / ' + totalDeclPages;
    }

    disclaimerNext.addEventListener('click', () => {
        if (declPage < totalDeclPages) {
            declPage++;
            updateDeclPage();
        } else {
            disclaimerOverlay.style.display = 'none';
            pledgeOverlay.style.display = 'flex';
        }
    });
    disclaimerPrev.addEventListener('click', () => {
        if (declPage > 1) { declPage--; updateDeclPage(); }
    });
    disclaimerSkip.addEventListener('click', () => {
        disclaimerOverlay.style.display = 'none';
        pledgeOverlay.style.display = 'flex';
    });

    // ============ 入场承诺逻辑 ============
    pledgeInput.addEventListener('input', () => {
        const target = currentConfig.pledgeText;
        const val = pledgeInput.value;
        if (val === target) {
            pledgeEnter.disabled = false;
            pledgeError.textContent = '';
        } else {
            pledgeEnter.disabled = true;
            if (val.length > 0 && target.indexOf(val) !== 0) {
                pledgeError.textContent = '请逐字抄写，从第一个字开始';
            } else {
                pledgeError.textContent = '';
            }
        }
    });

    pledgeEnter.addEventListener('click', () => {
        if (pledgeInput.value === currentConfig.pledgeText) {
            pledgeOverlay.style.display = 'none';
            mainApp.style.display = 'flex';
            updateUI();
        }
    });

    pledgeInput.addEventListener('keydown', (e) => {
        if (e.key === 'Enter' && pledgeInput.value === currentConfig.pledgeText) {
            pledgeEnter.click();
        }
    });

    // ============ 聊天功能 ============
    function sendMessage(text, isMe) {
        const msgDiv = document.createElement('div');
        msgDiv.className = 'msg ' + (isMe ? 'me' : 'other');
        const now = new Date();
        const timeStr = now.getHours().toString().padStart(2,'0') + ':' + now.getMinutes().toString().padStart(2,'0');
        msgDiv.innerHTML = text + '<div class="time">' + timeStr + '</div>';
        chatArea.appendChild(msgDiv);
        chatArea.scrollTop = chatArea.scrollHeight;
    }

    function getRandomReply() {
        const replies = currentConfig.replies;
        if (replies.length === 0) return "（没有可用的回复）";
        return getRandom(replies);
    }

    function handleSend() {
        const text = msgInput.value.trim();
        if (!text) return;
        sendMessage(text, true);
        msgInput.value = '';
        msgInput.style.height = 'auto';

        // 显示“正在输入...”
        const typingDiv = document.createElement('div');
        typingDiv.className = 'msg other';
        typingDiv.textContent = '正在输入...';
        typingDiv.id = 'typingIndicator';
        chatArea.appendChild(typingDiv);
        chatArea.scrollTop = chatArea.scrollHeight;

        const delay = 800 + Math.random() * 2000;
        setTimeout(() => {
            const typing = document.getElementById('typingIndicator');
            if (typing) typing.remove();
            sendMessage(getRandomReply(), false);
        }, delay);
    }

    sendBtn.addEventListener('click', handleSend);
    msgInput.addEventListener('keydown', (e) => {
        if (e.key === 'Enter' && !e.shiftKey) { e.preventDefault(); handleSend(); }
    });
    msgInput.addEventListener('input', () => {
        msgInput.style.height = 'auto';
        msgInput.style.height = Math.min(msgInput.scrollHeight, 80) + 'px';
    });

    // ============ 视频通话 ============
    let callInterval = null;
    let callSeconds = 0;
    callBtn.addEventListener('click', () => {
        callOverlay.classList.add('active');
        callSeconds = 0;
        updateCallTimer();
        callInterval = setInterval(() => { callSeconds++; updateCallTimer(); }, 1000);
    });
    endCallBtn.addEventListener('click', () => {
        callOverlay.classList.remove('active');
        if (callInterval) { clearInterval(callInterval); callInterval = null; }
    });
    function updateCallTimer() {
        const m = Math.floor(callSeconds / 60).toString().padStart(2,'0');
        const s = (callSeconds % 60).toString().padStart(2,'0');
        callTimer.textContent = m + ':' + s;
    }

    // ============ 信封投递 ============
    envelopeBtn.addEventListener('click', () => {
        envelopeOverlay.classList.add('active');
        letterInput.value = '';
    });
    closeEnvelopeBtn.addEventListener('click', () => { envelopeOverlay.classList.remove('active'); });
    sendLetterBtn.addEventListener('click', () => {
        const letter = letterInput.value.trim();
        if (!letter) return;
        if (letterToChat.checked) {
            sendMessage('📨 寄出的信：' + letter, true);
        }
        setTimeout(() => {
            sendMessage('📬 已送达 DELIVERED', false);
            const replyDelay = 10000 + Math.random() * 14000;
            setTimeout(() => {
                const replies = currentConfig.letterReplies;
                const reply = getRandom(replies);
                sendMessage('💌 回信：' + reply, false);
            }, replyDelay);
        }, 500);
        envelopeOverlay.classList.remove('active');
    });

    // ============ 拍一拍（随机选择一条） ============
    let patTimeout;
    dreamAvatar.addEventListener('click', () => {
        const patText = getRandom(currentConfig.patTexts);
        patToast.textContent = patText;
        patToast.classList.add('show');
        if (patTimeout) clearTimeout(patTimeout);
        patTimeout = setTimeout(() => { patToast.classList.remove('show'); }, 1500);
    });

    // ============ 启动 ============
    disclaimerOverlay.style.display = 'flex';
    updateDeclPage();
</script>
</body>
</html>
