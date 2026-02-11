<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8"><title>Easy Scratch SNS</title>
    <script src="https://cdn.jsdelivr.net"></script>
    <style>
        body { font-family: sans-serif; background: #f0f2f5; padding: 20px; display: flex; flex-direction: column; align-items: center; }
        .card { background: white; padding: 20px; border-radius: 12px; box-shadow: 0 4px 10px rgba(0,0,0,0.1); width: 100%; max-width: 450px; margin-bottom: 15px; }
        input, textarea { width: 100%; padding: 10px; margin: 5px 0; border: 1px solid #ddd; border-radius: 8px; box-sizing: border-box; }
        button { width: 100%; padding: 10px; background: #855cd6; color: white; border: none; border-radius: 8px; cursor: pointer; font-weight: bold; }
        .hidden { display: none; }
        .post { background: white; padding: 15px; border-radius: 10px; margin-bottom: 10px; border-left: 5px solid #855cd6; }
    </style>
</head>
<body>

<!-- ログイン/登録（メアド形式のIDとパスワードだけでOK） -->
<div id="auth-ui" class="card">
    <h2 style="color:#855cd6; margin-top:0;">🚀 かんたんログイン</h2>
    <input type="text" id="email" placeholder="名前 (例: user1@test.com)">
    <input type="password" id="pw" placeholder="パスワード (6文字以上)">
    <button onclick="auth('signUp')">新規登録</button>
    <button onclick="auth('signIn')" style="background:#4CAF50; margin-top:5px;">ログイン</button>
    <p id="msg" style="font-size:0.8em; color:red;"></p>
</div>

<!-- メイン画面 -->
<div id="sns-ui" class="hidden" style="width:100%; max-width:450px;">
    <div class="card">
        <textarea id="text" placeholder="今なにしてる？"></textarea>
        <button onclick="post()">投稿する</button>
        <button onclick="location.reload()" style="background:#666; margin-top:5px; font-size:0.7em;">ログアウト</button>
    </div>
    <div id="feed"></div>
</div>

<script>
    // --- ⚠️ 自分のURLとKeyに書き換える ⚠️ ---
    const URL = "https://あなたのURL.supabase.co";
    const KEY = "あなたのAPIキー";
    const supabase = window.supabase.createClient(URL, KEY);

    async function auth(type) {
        const email = document.getElementById('email').value;
        const password = document.getElementById('pw').value;
        const { error } = await supabase.auth[type]({ email, password });
        if (error) document.getElementById('msg').innerText = error.message;
        else if (type === 'signUp') alert("登録完了！そのままログインしてね");
    }

    supabase.auth.onAuthStateChange((ev, session) => {
        if (session) {
            document.getElementById('auth-ui').classList.add('hidden');
            document.getElementById('sns-ui').classList.remove('hidden');
            load();
        }
    });

    async function post() {
        const user = (await supabase.auth.getUser()).data.user;
        const content = document.getElementById('text').value;
        await supabase.from('posts').insert([{ username: user.email.split('@')[0], content: content }]);
        document.getElementById('text').value = ""; load();
    }

    async function load() {
        const { data } = await supabase.from('posts').select('*').order('created_at', { ascending: false });
        document.getElementById('feed').innerHTML = data.map(p => `<div class="post"><b>@${p.username}</b><br>${p.content}</div>`).join('');
    }
</script>
</body>
</html>
