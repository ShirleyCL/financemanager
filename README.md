<html lang="zh">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>智能购物记录本</title>
    <script src="https://unpkg.com/tesseract.js@4.0.2/dist/tesseract.min.js"></script>
    <style>
        /* 响应式布局 */
        :root { 
            font-family: -apple-system, sans-serif;
            font-size: calc(14px + 0.3vw);
        }
        .container {
            padding: 12px;
            max-width: 600px;
            margin: 0 auto;
            min-height: 100vh;
            box-sizing: border-box;
        }
        
        /* 日期筛选栏 */
        .date-filter {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 8px;
            margin: 15px 0;
        }
        .filter-btn {
            padding: 8px;
            border: 1px solid #007AFF;
            border-radius: 20px;
            text-align: center;
            cursor: pointer;
            background: white;
            transition: all 0.3s;
        }
        .filter-btn.active {
            background: #007AFF;
            color: white;
        }

        /* 数据展示区 */
        .summary-card {
            background: #f8f9fa;
            border-radius: 12px;
            padding: 15px;
            margin: 15px 0;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        }
        .summary-title {
            color: #666;
            font-size: 0.9em;
            margin-bottom: 8px;
        }
        .summary-amount {
            font-size: 1.5em;
            font-weight: bold;
            color: #333;
        }

        /* 移动端优化 */
        @media (max-width: 480px) {
            .container { padding: 8px; }
            .date-filter { grid-template-columns: 1fr; }
        }
    </style>
</head>
<body>
    <div class="container">
        <!-- 日期筛选 -->
        <div class="date-filter">
            <div class="filter-btn active" data-type="day" onclick="changeDateFilter('day')">日视图</div>
            <div class="filter-btn" data-type="month" onclick="changeDateFilter('month')">月视图</div>
            <div class="filter-btn" data-type="year" onclick="changeDateFilter('year')">年视图</div>
        </div>

        <!-- 数据汇总 -->
        <div class="summary-card">
            <div class="summary-title" id="dateRangeText"></div>
            <div class="summary-amount">¥<span id="totalAmount">0</span></div>
        </div>

        <!-- 其他元素保持不变 -->
    </div>

    <script>
        // 初始化数据存储
        let receipts = JSON.parse(localStorage.getItem('receipts') || [];
        let currentFilter = 'day';

        // 初始化显示
        function initDisplay() {
            // 生成当月数据模板
            if(receipts.length === 0) {
                const currentDate = new Date();
                receipts.push({
                    date: currentDate.toISOString().split('T')[0],
                    total: 0,
                    items: []
                });
            }
            
            updateDateDisplay();
            calculateTotal();
            renderItems();
        }

        // 日期筛选功能
        function changeDateFilter(type) {
            currentFilter = type;
            document.querySelectorAll('.filter-btn').forEach(btn => 
                btn.classList.remove('active')
            );
            document.querySelector(`[data-type="${type}"]`).classList.add('active');
            updateDateDisplay();
            calculateTotal();
        }

        // 更新日期显示
        function updateDateDisplay() {
            const now = new Date();
            let text = '';
            
            switch(currentFilter) {
                case 'day':
                    text = now.toLocaleDateString();
                    break;
                case 'month':
                    text = `${now.getFullYear()}年${now.getMonth()+1}月`;
                    break;
                case 'year':
                    text = `${now.getFullYear()}年`;
                    break;
            }
            
            document.getElementById('dateRangeText').textContent = text;
        }

        // 计算总金额
        function calculateTotal() {
            const now = new Date();
            let total = 0;

            receipts.forEach(receipt => {
                const receiptDate = new Date(receipt.date);
                let match = false;
                
                switch(currentFilter) {
                    case 'day':
                        match = receiptDate.toDateString() === now.toDateString();
                        break;
                    case 'month':
                        match = receiptDate.getMonth() === now.getMonth() && 
                                receiptDate.getFullYear() === now.getFullYear();
                        break;
                    case 'year':
                        match = receiptDate.getFullYear() === now.getFullYear();
                        break;
                }

                if(match) total += receipt.total;
            });

            document.getElementById('totalAmount').textContent = total;
        }

        // 在文件上传处理完成后调用
        function handleUploadComplete() {
            saveData();
            calculateTotal();
            renderItems();
        }

        // 数据存储优化
        function saveData() {
            try {
                localStorage.setItem('receipts', JSON.stringify(receipts));
            } catch (e) {
                if(e.name === 'QuotaExceededError') {
                    alert('存储空间已满，部分数据可能无法保存');
                }
            }
        }

        // 初始化执行
        initDisplay();
    </script>

    <!-- 原有脚本内容保留，增加以下响应式处理 -->
    <script>
        // 响应式字体调整
        function adjustFontSize() {
            const baseSize = Math.max(14, Math.min(18, window.innerWidth / 30));
            document.documentElement.style.fontSize = `${baseSize}px`;
        }

        window.addEventListener('resize', adjustFontSize);
        adjustFontSize();
    </script>
</body>
</html>
