<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <title>智能账务管理</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        .page { display: none; padding: 20px; }
        .active { display: block; }
        button { margin: 10px; padding: 12px 24px; }
        .back-btn { position: absolute; top: 10px; left: 10px; }
        #chartContainer { max-width: 800px; margin: 20px auto; }
        .record-item { border: 1px solid #ddd; padding: 10px; margin: 5px; }
    </style>
</head>
<body>
    <!-- 主页面 -->
    <div id="mainPage" class="page active">
        <h1>智能账务管理</h1>
        <button onclick="showPage('uploadPage')">拍摄/上传小票</button>
        <button onclick="showHistoryPage()">历史数据查看</button>
    </div>

    <!-- 上传页面 -->
    <div id="uploadPage" class="page">
        <button class="back-btn" onclick="showPage('mainPage')">返回</button>
        <h2>上传小票照片</h2>
        <input type="file" id="receiptInput" accept="image/*">
        <div id="preview"></div>
    </div>

    <!-- 消费单页面 -->
    <div id="billPage" class="page">
        <button class="back-btn" onclick="showPage('uploadPage')">返回</button>
        <h2>电子消费单</h2>
        <div id="billContent"></div>
        <button onclick="saveBill()">保存记录</button>
    </div>

    <!-- 历史记录页面 -->
    <div id="historyPage" class="page">
        <button class="back-btn" onclick="showPage('mainPage')">返回</button>
        <h2>历史数据</h2>
        <select id="viewMode" onchange="updateHistoryView()">
            <option value="day">日视图</option>
            <option value="month">月视图</option>
            <option value="year">年视图</option>
        </select>
        <input type="date" id="datePicker" onchange="updateHistoryView()">
        <div id="chartContainer"></div>
        <div id="recordsList"></div>
    </div>

    <script>
        // 初始化本地存储
        let records = JSON.parse(localStorage.getItem('financeRecords') || [];

        // 页面切换逻辑
        function showPage(pageId) {
            document.querySelectorAll('.page').forEach(p => p.classList.remove('active'));
            document.getElementById(pageId).classList.add('active');
        }

        // 图片上传处理
        document.getElementById('receiptInput').addEventListener('change', function(e) {
            const file = e.target.files[0];
            const reader = new FileReader();
            
            reader.onload = function() {
                // 此处应调用OCR API，这里模拟识别结果
                const mockData = {
                    date: new Date().toISOString().split('T')[0],
                    items: [
                        {name: '商品A', price: 25.5},
                        {name: '商品B', price: 35.0}
                    ],
                    total: 60.5
                };
                showBillPage(mockData);
            };
            reader.readAsDataURL(file);
        });

        function showBillPage(data) {
            showPage('billPage');
            document.getElementById('billContent').innerHTML = `
                <p>日期：${data.date}</p>
                ${data.items.map(i => `<p>${i.name} - ￥${i.price}</p>`).join('')}
                <p>总价：￥${data.total}</p>
            `;
            window.currentBill = data;
        }

        function saveBill() {
            records.push(window.currentBill);
            localStorage.setItem('financeRecords', JSON.stringify(records));
            showPage('mainPage');
        }

        // 历史记录查看
        function showHistoryPage() {
            showPage('historyPage');
            updateHistoryView();
        }

        function updateHistoryView() {
            const mode = document.getElementById('viewMode').value;
            const date = new Date(document.getElementById('datePicker').value);
            
            let filteredData = [];
            let chartData = {};
            let htmlContent = '';

            // 数据筛选逻辑
            if (mode === 'day') {
                filteredData = records.filter(r => r.date === date.toISOString().split('T')[0]);
                htmlContent = `<h3>当日总金额：￥${filteredData.reduce((sum, r) => sum + r.total, 0)}</h3>`;
                filteredData.forEach(r => {
                    htmlContent += r.items.map(i => `<div class="record-item">${i.name} - ￥${i.price}</div>`).join('');
                });
            } else {
                // 月/年视图数据聚合
                const groupKey = mode === 'month' ? 'date' : 'month';
                records.forEach(r => {
                    const key = r.date.substr(0, mode === 'month' ? 7 : 4);
                    chartData[key] = (chartData[key] || 0) + r.total;
                });

                // 图表绘制
                const ctx = document.createElement('canvas');
                document.getElementById('chartContainer').innerHTML = '';
                document.getElementById('chartContainer').appendChild(ctx);
                
                new Chart(ctx, {
                    type: 'bar',
                    data: {
                        labels: Object.keys(chartData),
                        datasets: [{
                            label: '消费金额',
                            data: Object.values(chartData),
                            backgroundColor: 'rgba(54, 162, 235, 0.2)'
                        }]
                    }
                });
            }

            document.getElementById('recordsList').innerHTML = htmlContent;
        }

        // 初始化日期选择器
        document.getElementById('datePicker').value = new Date().toISOString().split('T')[0];
    </script>
</body>
</html>
