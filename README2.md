<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>回声洞</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; font-family: sans-serif; }
        body { max-width: 600px; margin: 2rem auto; padding: 0 1rem; background: #f5f5f5; }
        .container { background: white; padding: 2rem; border-radius: 8px; box-shadow: 0 2px 10px rgba(0,0,0,0.1); }
        h1 { text-align: center; color: #333; margin-bottom: 2rem; }
        .form-group { margin-bottom: 1.5rem; }
        textarea { width: 100%; padding: 1rem; border: 1px solid #ddd; border-radius: 4px; min-height: 100px; resize: none; }
        .btn-group { display: flex; gap: 1rem; justify-content: center; margin-bottom: 2rem; }
        button { padding: 0.8rem 1.5rem; border: none; border-radius: 4px; cursor: pointer; font-size: 1rem; }
        #submitBtn { background: #2196F3; color: white; }
        #randomBtn { background: #4CAF50; color: white; }
        #echoBox { padding: 1.5rem; border: 1px dashed #ddd; border-radius: 4px; text-align: center; min-height: 100px; color: #555; }
        .status { margin-top: 1rem; padding: 0.8rem; border-radius: 4px; text-align: center; display: none; }
        .success { background: #E8F5E9; color: #2E7D32; display: block; }
        .error { background: #FFEBEE; color: #C62828; display: block; }
    </style>
</head>
<body>
    <div class="container">
        <h1>回声洞</h1>
        <div class="form-group">
            <textarea id="message" placeholder="在这里留下你的话..."></textarea>
        </div>
        <div class="btn-group">
            <button id="submitBtn">发送回声</button>
            <button id="randomBtn">随机听回声</button>
        </div>
        <div id="echoBox">等待回声...</div>
        <div id="status" class="status"></div>
    </div>

    <script>
        // TinyWebDB 服务器地址，可替换为自建/专用服务器
        const TINY_WEBDB_URL = "http://appinvtinywebdb.appspot.com";
        // 专用服务器需添加 user 和 secret
        // const USER = "你的用户名";
        // const SECRET = "你的密钥";

        const messageEl = document.getElementById("message");
        const echoBoxEl = document.getElementById("echoBox");
        const statusEl = document.getElementById("status");
        const submitBtn = document.getElementById("submitBtn");
        const randomBtn = document.getElementById("randomBtn");

        // 存储所有留言的 tag 前缀
        const TAG_PREFIX = "echo_";

        // 提交留言
        submitBtn.addEventListener("click", () => {
            const message = messageEl.value.trim();
            if (!message) {
                showStatus("请输入要发送的内容！", "error");
                return;
            }

            // 生成唯一 tag：前缀 + 时间戳
            const tag = `${TAG_PREFIX}${new Date().getTime()}`;
            const data = {
                action: "update",
                tag: tag,
                value: message
                // 专用服务器取消注释
                // user: USER,
                // secret: SECRET
            };

            fetch(TINY_WEBDB_URL, {
                method: "POST",
                body: new URLSearchParams(data),
                headers: { "Content-Type": "application/x-www-form-urlencoded" }
            })
            .then(res => res.json())
            .then(res => {
                if (res.result === "OK") {
                    showStatus("回声发送成功！", "success");
                    messageEl.value = ""; // 清空输入框
                } else {
                    showStatus("发送失败，请稍后重试！", "error");
                }
            })
            .catch(err => {
                showStatus("网络错误，请检查连接！", "error");
                console.error(err);
            });
        });

        // 随机获取一条留言
        randomBtn.addEventListener("click", () => {
            // 先获取所有 tag（MIT 公共服务器可通过访问主页获取，自建服务器需额外接口）
            // 这里简化处理：随机生成近期时间戳 tag 尝试获取
            const recentTime = Date.now() - 30 * 24 * 60 * 60 * 1000; // 近30天
            const randomTime = Math.floor(Math.random() * (Date.now() - recentTime)) + recentTime;
            const randomTag = `${TAG_PREFIX}${randomTime}`;

            const data = {
                action: "get",
                tag: randomTag
                // 专用服务器取消注释
                // user: USER,
                // secret: SECRET
            };

            fetch(TINY_WEBDB_URL, {
                method: "POST",
                body: new URLSearchParams(data),
                headers: { "Content-Type": "application/x-www-form-urlencoded" }
            })
            .then(res => res.json())
            .then(res => {
                if (res.result === "OK" && res.value) {
                    echoBoxEl.textContent = res.value;
                    showStatus("听到一条回声~", "success");
                } else {
                    echoBoxEl.textContent = "暂时没有听到回声，换个运气试试吧！";
                    showStatus("未找到留言", "error");
                }
            })
            .catch(err => {
                echoBoxEl.textContent = "网络出错了...";
                console.error(err);
            });
        });

        // 显示状态提示
        function showStatus(text, type) {
            statusEl.textContent = text;
            statusEl.className = `status ${type}`;
            // 3秒后自动隐藏提示
            setTimeout(() => {
                statusEl.className = "status";
            }, 3000);
        }
    </script>
</body>
</html>
