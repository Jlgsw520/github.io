<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>简易反馈表单</title>
    <style>
        body { max-width: 600px; margin: 2rem auto; padding: 0 1rem; font-family: sans-serif; }
        .form-group { margin: 1rem 0; }
        label { display: block; margin-bottom: 0.5rem; font-weight: 600; }
        input, textarea { width: 100%; padding: 0.8rem; border: 1px solid #ddd; border-radius: 4px; box-sizing: border-box; }
        button { padding: 0.8rem 2rem; background: #2196F3; color: white; border: none; border-radius: 4px; cursor: pointer; }
        button:hover { background: #1976D2; }
        #status { margin-top: 1rem; padding: 0.8rem; border-radius: 4px; display: none; }
        .success { background: #E8F5E9; color: #2E7D32; display: block; }
        .error { background: #FFEBEE; color: #C62828; display: block; }
    </style>
</head>
<body>
    <h2>用户反馈</h2>
    <div class="form-group">
        <label for="name">姓名（选填）</label>
        <input type="text" id="name" placeholder="请输入你的姓名">
    </div>
    <div class="form-group">
        <label for="feedback">反馈内容（必填）</label>
        <textarea id="feedback" rows="4" placeholder="请输入你的建议或问题"></textarea>
    </div>
    <button onclick="submitFeedback()">提交反馈</button>
    <div id="status"></div>

    <script>
        // 替换为你的 TinyWebDB 服务器地址
        const TINY_WEBDB_URL = "http://appinvtinywebdb.appspot.com"; 
        // 专用服务器需添加 user 和 secret
        // const USER = "你的用户名";
        // const SECRET = "你的密钥";

        function submitFeedback() {
            const name = document.getElementById("name").value.trim() || "匿名用户";
            const feedback = document.getElementById("feedback").value.trim();
            const statusEl = document.getElementById("status");

            // 内容校验
            if (!feedback) {
                statusEl.className = "error";
                statusEl.textContent = "请填写反馈内容！";
                return;
            }

            // 构造上传数据
            const data = {
                action: "update",
                // 用时间戳做唯一 tag，避免覆盖
                tag: `feedback_${new Date().getTime()}`,
                value: JSON.stringify({ name, feedback, time: new Date().toLocaleString() })
                // 专用服务器取消下面注释
                // user: USER,
                // secret: SECRET
            };

            // 上传到 TinyWebDB
            fetch(TINY_WEBDB_URL, {
                method: "POST",
                body: new URLSearchParams(data),
                headers: { "Content-Type": "application/x-www-form-urlencoded" }
            })
            .then(res => res.json())
            .then(res => {
                if (res.result === "OK") {
                    statusEl.className = "success";
                    statusEl.textContent = "反馈提交成功！感谢你的建议~";
                    document.getElementById("feedback").value = ""; // 清空输入框
                } else {
                    statusEl.className = "error";
                    statusEl.textContent = "提交失败，请稍后重试！";
                }
            })
            .catch(err => {
                statusEl.className = "error";
                statusEl.textContent = "网络错误，请检查连接！";
                console.error(err);
            });
        }
    </script>
</body>
</html>
