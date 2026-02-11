<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>GitHub Native SNS</title>
    <style>
        :root { --gh-blue: #0969da; --gh-bg: #f6f8fa; }
        body { font-family: -apple-system,BlinkMacSystemFont,"Segoe UI",Helvetica,Arial,sans-serif; background: var(--gh-bg); margin: 0; padding: 15px; display: flex; flex-direction: column; align-items: center; }
        .card { background: white; padding: 20px; border-radius: 10px; border: 1px solid #d0d7de; width: 100%; max-width: 600px; margin-bottom: 15px; }
        h1 { font-size: 20px; color: #24292f; margin: 0 0 15px; }
        .btn { display: inline-block; background: #2da44e; color: white; padding: 12px 20px; border-radius: 6px; font-weight: bold; text-decoration: none; text-align: center; width: 100%; box-sizing: border-box; }
        .post { background: white; padding: 15px; border-radius: 6px; border: 1px solid #d0d7de; margin-bottom: 10px; }
        .post-user { font-weight: bold; color: var(--gh-blue); }
        .post-time { font-size: 12px; color: #57606a; }
        .nav { display: flex; gap: 10px; margin-bottom: 15px; width: 100%; max-width: 600px; }
        .tab { flex: 1; padding: 8px; background: #eee; border-radius: 6px; text-align: center; cursor: pointer; font-size: 14px; }
        .active { background: var(--gh-blue); color: white; }
    </style>
</head>
<body>

<div class="card">
    <h1>🚀 blip-sns Portal</h1>
    <div class="nav">
        <div class="tab active">🏠 タイムライン</div>
        <div class="tab" onclick="location.href='https://github.com'">✉️ DM/チャット</div>
    </div>
    <p style="font-size:13px; color:#57606a;">写真は投稿時に画像を貼り付けるだけで反映されます。</p>
    <!-- GitHubのDiscussions新規投稿画面へ飛ばす -->
    <a href="https://github.com" class="btn">＋ 新規投稿・写真を貼る</a>
</div>

<div id="feed" style="width: 100%; max-width: 600px;">
    <p style="text-align:center;">データを読み込み中...</p>
</div>

<script>
    async function loadSNS() {
        try {
            // あなたのリポジトリのDiscussions(またはIssues)をAPIで取得
            const response = await fetch('https://api.github.com');
            const data = await response.json();
            
            const feed = document.getElementById('feed');
            if (!data || data.length === 0) {
                feed.innerHTML = '<div class="card">まだ投稿がありません。最初の投稿をしてみよう！</div>';
                return;
            }

            feed.innerHTML = data.map(item => `
                <div class="post">
                    <span class="post-user">@${item.user.login}</span> 
                    <span class="post-time">${new Date(item.created_at).toLocaleString()}</span>
                    <h3 style="margin:10px 0; font-size:16px;">${item.title}</h3>
                    <div style="font-size:14px; line-height:1.5;">${item.body.replace(/\n/g, '<br>')}</div>
                    <div style="margin-top:10px; border-top:1px solid #eee; padding-top:10px; font-size:12px;">
                        <a href="${item.html_url}" target="_blank" style="text-decoration:none; color:var(--gh-blue);">💬 返信・いいね・フォロー</a>
                    </div>
                </div>
            `).join('');
        } catch (e) {
            document.getElementById('feed').innerHTML = '<p>読み込みエラー。リポジトリの設定を確認してください。</p>';
        }
    }
    loadSNS();
</script>
</body>
</html>
