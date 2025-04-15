<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>智能购物记录本</title>
    <script src="https://unpkg.com/tesseract.js@4.0.2/dist/tesseract.min.js"></script>
    <style>
        /* 样式保持不变 */
        :root { font-family: -apple-system, sans-serif; }
        .container { padding: 12px; max-width: 600px; margin: 0 auto; }
        .upload-btn { background: #007AFF; color: white; padding: 12px; border-radius: 8px; text-align: center; margin: 10px 0; }
        .item-card { border: 1px solid #ddd; padding: 12px; margin: 8px 0; border-radius: 6px; }
        .comment-input { width: 100%; padding: 8px; border: 1px solid #ccc; border-radius: 4px; margin-top: 8px; }
        @media (max-width: 480px) { .container { padding: 8px; } }
    </style>
</head>
<body>
    <div class="container">
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
        // 修复1: 初始化Worker时添加错误处理
        let worker;
        (async () => {
            try {
                worker = await Tesseract.createWorker({
                    logger: m => console.log(m),
                    errorHandler: err => console.error('OCR Error:', err)
                });
                await worker.loadLanguage('jpn');
                await worker.initialize('jpn');
                console.log('OCR引擎初始化完成');
            } catch (error) {
                console.error('OCR初始化失败:', error);
            }
        })();

        // 修复2: 优化数据存储结构
        let receipts = JSON.parse(localStorage.getItem('receipts') || [];
        
        // 修复3: 添加页面渲染锁防止重复渲染
        let isRendering = false;

        document.getElementById('receiptUpload').addEventListener('change', async (e) => {
            const file = e.target.files[0];
            if (!file || !worker) return;
            
            // 添加加载提示
            const originalBtnText = document.querySelector('.upload-btn').innerHTML;
            document.querySelector('.upload-btn').innerHTML = '🔄 正在解析...';
            
            try {
                const { data: { text } } = await worker.recognize(file);
                const parsedData = parseReceipt(text);
                receipts.push(parsedData);
                saveData();
                renderItems();
            } catch (error) {
                console.error('OCR处理失败:', error);
                alert('小票解析失败，请确保图片清晰并包含日文文本');
            } finally {
                document.querySelector('.upload-btn').innerHTML = originalBtnText;
            }
        });

        // 修复4: 增强小票解析逻辑
        function parseReceipt(text) {
            const lines = text.split('\n');
            let receipt = {
                id: Date.now(),
                date: new Date().toISOString(),
                total: 0,
                items: []
            };

            lines.forEach(line => {
                // 增强日期匹配（处理空格问题）
                const dateMatch = line.match(/(\d{4})\s*\/\s*(\d{1,2})\s*\/\s*(\d{1,2})/);
                if (dateMatch) {
                    receipt.date = `${dateMatch[1]}/${dateMatch[2].padStart(2,'0')}/${dateMatch[3].padStart(2,'0')}`;
                }
                
                // 增强商品匹配（处理全角字符）
                const itemMatch = line.match(/([\u3000-¥\w\s]+?)\s+(\d{3,})[※¥]/);
                if (itemMatch) {
                    receipt.items.push({
                        id: Date.now() + Math.random(),
                        name: itemMatch[1].trim().replace(/\s+/g, ' '),
                        price: parseInt(itemMatch[2]),
                        comment: ''
                    });
                }
                
                // 增强总价匹配（处理逗号分隔）
                if (line.includes('合計')) {
                    const totalMatch = line.match(/¥\s*([\d,]+)/);
                    if (totalMatch) {
                        receipt.total = parseInt(totalMatch[1].replace(/,/g, ''));
                    }
                }
            });

            return receipt;
        }

        // 修复5: 优化渲染性能
        function renderItems(filter = '') {
            if (isRendering) return;
            isRendering = true;
            
            const list = document.getElementById('itemList');
            list.innerHTML = '';
            
            const filteredItems = receipts.flatMap(r => r.items)
                .filter(item => 
                    item.name.toLowerCase().includes(filter.toLowerCase()) || 
                    item.comment.toLowerCase().includes(filter.toLowerCase())
                );

            const fragment = document.createDocumentFragment();
            filteredItems.forEach(item => {
                const template = document.getElementById('detailTemplate').innerHTML;
                const html = template
                    .replace(/{{name}}/g, item.name)
                    .replace(/{{price}}/g, item.price)
                    .replace(/{{comment}}/g, item.comment)
                    .replace(/{{id}}/g, item.id);
                
                const div = document.createElement('div');
                div.innerHTML = html;
                fragment.appendChild(div.firstElementChild);
            });
            
            list.appendChild(fragment);
            isRendering = false;
        }

        // 其他函数保持不变（需添加错误处理）
        function saveComment(itemId, comment) {
            receipts.forEach(receipt => {
                const item = receipt.items.find(i => i.id === itemId);
                if (item) item.comment = comment.substring(0, 30);
            });
            saveData();
            renderItems(); // 修复6: 保存后强制刷新
        }

        function searchItems(keyword) {
            renderItems(keyword);
        }

        function saveData() {
            try {
                localStorage.setItem('receipts', JSON.stringify(receipts));
            } catch (error) {
                console.error('存储失败:', error);
                alert('本地存储空间已满，请清理后重试');
            }
        }

        // 初始化渲染
        renderItems();
    </script>
</body>
</html>
