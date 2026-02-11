<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Scratchers Pro SNS</title>
    <script src="https://cdn.jsdelivr.net"></script>
    <style>
        body { font-family: sans-serif; background: #f0f2f5; margin: 0; padding: 20px; display: flex; flex-direction: column; align-items: center; }
        .card { background: white; padding: 25px; border-radius: 15px; box-shadow: 0 4px 15px rgba(0,0,0,0.1); width: 100%; max-width: 450px; margin-bottom: 20px; }
        input, textarea { width: 100%; padding: 12px; margin: 8px 0; border: 1px solid #ddd; border-radius: 8px; box-sizing: border-box; }
        button { width: 100%; padding: 12px; background: #855cd6; color: white; border: none; border-radius: 8px; font-weight: bold; cursor: pointer; }
        .hidden { display: none; }
        .post { background: white; padding: 15px; border-radius: 10px; margin-bottom: 10px; border-left: 5px solid #855cd6; }
    </style>
</head>
<body>

<!-- 1. ログイン・登録画面 -->
<div id="auth-section" class="card">
    <h2 id="auth-title">🔑 ログイン / 新規登録</h2>
    <input type="email" id="email" placeholder="メールアドレス">
    <input type="password" id="password" placeholder="パスワード">
    <button onclick="signUp()">新規登録（メール承認）</button>
    <button onclick="signIn()" style="background:#4CAF50; margin-top:10px;">ログイン</button>
    <p style="font-size:0.8em; color:gray;">※登録後、メールボックスを確認してください。</p>
</div>

<!-- 2. SNSメイン画面（ログイン後のみ表示） -->
<div id="main-section" class="hidden" style="width:100%; max-width:450px;">
    <div class="card">
        <h3>🚀 いま何してる？</h3>
        <p id="user-info" style="font-size:0.9em; color:#855cd6;"></p>
        <textarea id="post-content" placeholder="投稿内容..."></textarea>
        <button onclick="sendPost()">投稿する</button>
        <button onclick="signOut()" style="background:#aaa; margin-top:10px; padding:5px;">ログアウト</button>
    </div>
    <div id="feed">読み込み中...</div>
</div>

<script>
    // --- ⚠️ Supabaseの設定をここに貼る ⚠️ ---
    const SB_URL = "https://あなたのURL.supabase.co";
    const SB_KEY = "あなたのAPIキー";
    const supabase = window.supabase.createClient(SB_URL, SB_KEY);

    // ログイン状態のチェック
    supabase.auth.onAuthStateChange((event, session) => {
        if (session) {
            document.getElementById('auth-section').classList.add('hidden');
            document.getElementById('main-section').classList.remove('hidden');
            document.getElementById('user-info').innerText = `ログイン中: ${session.user.email}`;
            fetchPosts();
        } else {
            document.getElementById('auth-section').classList.remove('hidden');
            document.getElementById('main-section').classList.add('hidden');
        }
    });

    // 新規登録
    async function signUp() {
        const email = document.getElementById('email').value;
        const password = document.getElementById('password').value;
        const { error } = await supabase.auth.signUp({ email, password });
        if (error) alert(error.message);
        else alert("確認メールを送りました！承認してからログインしてね。");
    }

    // ログイン
    async function signIn() {
        const email = document.getElementById('email').value;
        const password = document.getElementById('password').value;
        const { error } = await supabase.auth.signInWithPassword({ email, password });
        if (error) alert(error.message);
    }

    // ログアウト
    async function signOut() { await supabase.auth.signOut(); }

    // 投稿
    async function sendPost() {
        const content = document.getElementById('post-content').value;
        const user = (await supabase.auth.getUser()).data.user;
        if (!content) return;

        const { error } = await supabase.from('posts').insert([{ 
            username: user.email.split('@')[0], // メールの@より前を名前にする
            content: content 
        }]);
        if (error) alert("投稿エラー: " + error.message);
        else { document.getElementById('post-content').value = ""; fetchPosts(); }
    }

    // タイムライン取得
    async function fetchPosts() {
        const { data } = await supabase.from('posts').select('*').order('created_at', { ascending: false });
        const feed = document.getElementById('feed');
        feed.innerHTML = data ? data.map(p => `
            <div class="post">
                <b>@${p.username}</b><br>${p.content}
            </div>
        `).join('') : "投稿がありません。";
    }
</script>

</body>
</html>
