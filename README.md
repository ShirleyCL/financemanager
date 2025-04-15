# LJL的记账小本本

<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>智能购物记录本</title>
    <!-- 引入Tesseract.js OCR库 -->
    <script src="https://unpkg.com/tesseract.js@v4.0.2/dist/tesseract.min.js"></script>
    <style>
        /* 响应式布局 */
        :root { font-family: -apple-system, sans-serif; }
        .container { padding: 12px; max-width: 600px; margin: 0 auto; }
        
        /* 上传按钮样式 */
        .upload-btn {
            background: #007AFF;
            color: white;
            padding: 12px;
            border-radius: 8px;
            text-align: center;
            margin: 10px 0;
        }
        
        /* 商品列表 */
        .item-card {
            border: 1px solid #ddd;
            padding: 12px;
            margin: 8px 0;
            border-radius: 6px;
        }
        
        /* 评价输入 */
        .comment-input {
            width: 100%;
            padding: 8px;
            border: 1px solid #ccc;
            border-radius: 4px;
            margin-top: 8px;
        }
        
        /* 移动端优化 */
        @media (max-width: 480px) {
            .container { padding: 8px; }
        }
    </style>
</head>
<body>
    <div class="container">
        <!-- 首页 -->
        <div id="homeView">
            <input type="file" id="receiptUpload" accept="image/*" hidden>
            <div class="upload-btn" onclick="document.getElementById('receiptUpload').click()">
                📷 上传小票照片
            </div>
            
            <div id="searchBox">
                <input type="text" placeholder="搜索商品..." style="width:100%;padding:8px;" 
                       oninput="searchItems(this.value)">
            </div>
            
            <div id="itemList"></div>
        </div>

        <!-- 详情页模板 -->
        <template id="detailTemplate">
            <div class="item-card">
                <h3>{{name}}</h3>
                <p>价格: ¥{{price}}</p>
                <input type="text" class="comment-input" placeholder="输入评价（30字内）" 
                       value="{{comment}}" maxlength="30" onchange="saveComment('{{id}}', this.value)">
            </div>
        </template>
    </div>

    <script>
        // 数据存储结构
        let receipts = JSON.parse(localStorage.getItem('receipts')) || [];
        
        // OCR处理器
        const worker = Tesseract.createWorker({
            logger: m => console.log(m)
        });

        (async () => {
            await worker.load();
            await worker.loadLanguage('jpn');
            await worker.initialize('jpn');
        })();

        // 图片上传处理
        document.getElementById('receiptUpload').addEventListener('change', async (e) => {
            const file = e.target.files[0];
            const { data: { text } } = await worker.recognize(file);
            const parsedData = parseReceipt(text);
            receipts.push(parsedData);
            saveData();
            renderItems();
        });

        // 小票解析逻辑
        function parseReceipt(text) {
            const lines = text.split('\n');
            let receipt = {
                id: Date.now(),
                date: new Date().toISOString(),
                total: 0,
                items: []
            };

            lines.forEach(line => {
                // 匹配日期（示例：2025/ 4/13）
                if (line.match(/\d{4}\/\s?\d{1,2}\/\d{1,2}/)) {
                    receipt.date = line.replace(/\s/g, '');
                }
                
                // 匹配商品行（示例：雪印 特濃    268※）
                const itemMatch = line.match(/(.+?)\s+(\d+)\s*[※¥]/);
                if (itemMatch) {
                    receipt.items.push({
                        id: Date.now() + Math.random(),
                        name: itemMatch[1].trim(),
                        price: parseInt(itemMatch[2]),
                        comment: ''
                    });
                }
                
                // 匹配总价（示例：合 計    ¥2,896）
                if (line.includes('合計')) {
                    const totalMatch = line.match(/¥([\d,]+)/);
                    if (totalMatch) receipt.total = parseInt(totalMatch[1].replace(/,/g, ''));
                }
            });

            return receipt;
        }

        // 渲染商品列表
        function renderItems(filter = '') {
            const list = document.getElementById('itemList');
            list.innerHTML = '';
            
            receipts.forEach(receipt => {
                receipt.items.filter(item => 
                    item.name.includes(filter) || 
                    item.comment.includes(filter)
                ).forEach(item => {
                    const template = document.getElementById('detailTemplate').innerHTML;
                    const html = template
                        .replace(/{{name}}/g, item.name)
                        .replace(/{{price}}/g, item.price)
                        .replace(/{{comment}}/g, item.comment)
                        .replace(/{{id}}/g, item.id);
                    
                    const div = document.createElement('div');
                    div.innerHTML = html;
                    list.appendChild(div.firstElementChild);
                });
            });
        }

        // 保存评价
        function saveComment(itemId, comment) {
            receipts.forEach(receipt => {
                const item = receipt.items.find(i => i.id === itemId);
                if (item) item.comment = comment.substring(0, 30);
            });
            saveData();
        }

        // 搜索功能
        function searchItems(keyword) {
            renderItems(keyword);
        }

        // 数据持久化
        function saveData() {
            localStorage.setItem('receipts', JSON.stringify(receipts));
        }

        // 初始化加载
        renderItems();
    </script>
</body>
</html>
