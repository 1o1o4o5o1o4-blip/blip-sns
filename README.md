<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Scratchers Portal & SNS</title>
    <script src="https://cdn.jsdelivr.net"></script>
    <style>
        :root { --main: #855cd6; --sub: #4CAF50; --bg: #f0f2f5; }
        body { font-family: sans-serif; background: var(--bg); margin: 0; padding: 10px; display: flex; flex-direction: column; align-items: center; }
        .card { background: white; padding: 20px; border-radius: 15px; box-shadow: 0 4px 10px rgba(0,0,0,0.1); width: 100%; max-width: 500px; margin-bottom: 15px; box-sizing: border-box; }
        input, textarea { width: 100%; padding: 10px; margin: 5px 0; border: 1px solid #ddd; border-radius: 8px; box-sizing: border-box; }
        button { width: 100%; padding: 12px; background: var(--main); color: white; border: none; border-radius: 8px; font-weight: bold; cursor: pointer; margin-top: 5px; }
        .hidden { display: none; }
        .post { background: white; padding: 15px; border-radius: 10px; margin-bottom: 10px; border-left: 5px solid var(--main); box-shadow: 0 2px 5px rgba(0,0,0,0.05); }
        .tabs { display: flex; gap: 10px; margin-bottom: 15px; }
        .tab-btn { flex: 1; padding: 10px; background: #ddd; border-radius: 8px; text-align: center; cursor: pointer; font-weight: bold; font-size: 0.9em; }
        .tab-btn.active { background: var(--main); color: white; }
        .search-res { background: #f9f7ff; border: 1px solid #e1d5f5; padding: 10px; border-radius: 8px; margin-bottom: 10px; }
    </style>
</head>
<body>

<!-- 1. 認証 (ログイン前のみ表示) -->
<div id="auth-ui" class="card">
    <h2 style="color:var(--main); margin-top:0;">🚀 Portal ログイン</h2>
    <input type="email" id="email" placeholder="メールアドレス">
    <input type="password" id="password" placeholder="パスワード(6文字以上)">
    <button onclick="auth('signUp')">新規登録</button>
    <button onclick="auth('signIn')" style="background:var(--sub);">ログイン</button>
    <p id="err" style="color:red; font-size:0.8em;"></p>
</div>

<!-- 2. ポータルメイン (ログイン後表示) -->
<div id="main-ui" class="hidden" style="width:100%; max-width:500px;">
    <div class="tabs">
        <div id="tab-sns" class="tab-btn active" onclick="switchTab('sns')">💬 SNS交流</div>
        <div id="tab-search" class="tab-btn" onclick="switchTab('search')">🔍 爆速検索</div>
    </div>

    <!-- SNS機能 -->
    <div id="sns-section">
        <div class="card">
            <textarea id="post-msg" placeholder="いま何してる？"></textarea>
            <input type="text" id="post-img" placeholder="画像URL (任意)">
            <button onclick="sendPost()">投稿する</button>
        </div>
        <div id="sns-feed">読み込み中...</div>
    </div>

    <!-- 検索機能 (規制回避型) -->
    <div id="search-section" class="hidden">
        <div class="card" style="border-top: 5px solid #2196F3;">
            <h3 style="color:#2196F3; margin-top:0;">🔍 フォーラム検索</h3>
            <p style="font-size:0.8em; color:gray;">Googleのインデックスを利用するため、100%規制を回避します。</p>
            <input type="text" id="search-q" placeholder="キーワード (例: 拡張機能)">
            <input type="text" id="search-u" placeholder="ユーザ名 (任意)">
            <button onclick="runSearch()" style="background:#2196F3;">Googleで結果を表示</button>
        </div>
    </div>

    <button onclick="supabase.auth.signOut(); location.reload();" style="background:#666; margin-top:20px; font-size:0.8em;">ログアウト</button>
</div>

<script>
    // --- ⚠️ 自分のURLとKeyに書き換える ⚠️ ---
    const URL = "https://あなたのプロジェクトURL.supabase.co";
    const KEY = "あなたのAPIキー";
    const supabase = window.supabase.createClient(URL, KEY);

    // 認証
    async function auth(type) {
        const e = document.getElementById('email').value;
        const p = document.getElementById('password').value;
        const { error } = await supabase.auth[type]({ email: e, password: p });
        if (error) document.getElementById('err').innerText = error.message;
        else if (type==='signUp') alert("登録完了！ログインしてください");
    }

    // 状態監視
    supabase.auth.onAuthStateChange((ev, session) => {
        if (session) {
            document.getElementById('auth-ui').classList.add('hidden');
            document.getElementById('main-ui').classList.remove('hidden');
            fetchSns();
        }
    });

    // タブ切り替え
    function switchTab(t) {
        document.getElementById('sns-section').classList.toggle('hidden', t !== 'sns');
        document.getElementById('search-section').classList.toggle('hidden', t !== 'search');
        document.getElementById('tab-sns').classList.toggle('active', t === 'sns');
        document.getElementById('tab-search').classList.toggle('active', t === 'search');
    }

    // 爆速検索実行 (新しいタブで開く)
    function runSearch() {
        const q = document.getElementById('search-q').value;
        const u = document.getElementById('search-u').value;
        let query = `site:scratch.mit.edu/discuss ${q}`;
        if(u) query += ` "${u}"`;
        window.open(`https://www.google.com{encodeURIComponent(query)}`, '_blank');
    }

    // SNS投稿取得
    async function fetchSns() {
        const { data } = await supabase.from('posts').select('*').order('created_at', { ascending: false });
        const feed = document.getElementById('sns-feed');
        feed.innerHTML = data ? data.map(p => `
            <div class="post">
                <b>@${p.username}</b> <small style="color:gray;">${new Date(p.created_at).toLocaleTimeString()}</small><br>
                <div style="margin-top:5px;">${p.content}</div>
                ${p.image_url ? `<img src="${p.image_url}" style="width:100%; border-radius:8px; margin-top:10px;">` : ''}
            </div>
        `).join('') : "投稿がありません。";
    }

    // SNS投稿送信
    async function sendPost() {
        const msg = document.getElementById('post-msg').value;
        const img = document.getElementById('post-img').value;
        const user = (await supabase.auth.getUser()).data.user;
        if(!msg) return;

        const { error } = await supabase.from('posts').insert([{ 
            username: user.email.split('@')[0], 
            content: msg, 
            image_url: img 
        }]);
        
        if (error) alert(error.message);
        else { document.getElementById('post-msg').value = ""; fetchSns(); }
    }
</script>
</body>
</html>
