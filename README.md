<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>8月考勤表</title>
    <style>
        * {
            box-sizing: border-box;
            font-family: "Microsoft YaHei", sans-serif;
        }
        body {
            margin: 0;
            padding: 15px;
            background-color: #f5f7fa;
            color: #333;
        }
        .container {
            max-width: 100%;
            overflow-x: auto;
            background: white;
            border-radius: 8px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
            padding: 15px;
        }
        h2 {
            text-align: center;
            margin-top: 0;
            color: #1e88e5;
            padding-bottom: 10px;
            border-bottom: 1px solid #e0e0e0;
        }
        .settings {
            display: flex;
            flex-wrap: wrap;
            gap: 15px;
            margin-bottom: 15px;
            padding: 10px;
            background: #f9f9f9;
            border-radius: 6px;
        }
        .setting-group {
            display: flex;
            align-items: center;
            gap: 8px;
        }
        label {
            font-size: 14px;
            white-space: nowrap;
        }
        input[type="time"] {
            padding: 6px;
            border: 1px solid #ddd;
            border-radius: 4px;
            font-size: 14px;
        }
        table {
            width: 100%;
            border-collapse: collapse;
            font-size: 13px;
            table-layout: fixed;
            min-width: 800px;
        }
        th, td {
            border: 1px solid #e0e0e0;
            padding: 8px 5px;
            text-align: center;
        }
        th {
            background-color: #1e88e5;
            color: white;
            font-weight: normal;
            position: sticky;
            top: 0;
        }
        .name-header, .name-cell {
            background-color: #e3f2fd;
            font-weight: bold;
            position: sticky;
            left: 0;
            z-index: 2;
            min-width: 80px;
        }
        .name-cell {
            background-color: #f9f9f9;
        }
        .weekend {
            background-color: #fff9e6;
        }
        input[type="text"], input[type="time"] {
            width: 100%;
            padding: 5px;
            border: 1px solid #ddd;
            border-radius: 4px;
            text-align: center;
            font-size: 12px;
        }
        .overtime-cell {
            background-color: #f5f5f5;
            font-weight: bold;
        }
        .total-row {
            background-color: #e8f5e9;
            font-weight: bold;
        }
        .instructions {
            margin-top: 15px;
            padding: 10px;
            background: #f9f9f9;
            border-radius: 6px;
            font-size: 12px;
            color: #666;
        }
        @media (max-width: 600px) {
            .settings {
                flex-direction: column;
                gap: 8px;
            }
            .setting-group {
                width: 100%;
            }
            input[type="time"] {
                width: 100%;
            }
        }
    </style>
</head>
<body>
    <div class="container">
        <h2>8月考勤表</h2>
        
        <div class="settings">
            <div class="setting-group">
                <label for="work-start">上班时间:</label>
                <input type="time" id="work-start" value="08:00">
            </div>
            <div class="setting-group">
                <label for="work-end">下班时间:</label>
                <input type="time" id="work-end" value="17:30">
            </div>
            <div class="setting-group">
                <label for="overtime-start">加班开始时间:</label>
                <input type="time" id="overtime-start" value="18:00">
            </div>
            <div class="setting-group">
                <button onclick="updateAll()">应用时间设置</button>
            </div>
        </div>
        
        <div class="table-container">
            <table id="attendance-table">
                <thead>
                    <tr>
                        <th class="name-header">姓名</th>
                        <!-- 日期列将由JavaScript生成 -->
                    </tr>
                </thead>
                <tbody>
                    <!-- 数据行将由JavaScript生成 -->
                </tbody>
            </table>
        </div>
        
        <div class="instructions">
            <p>使用说明:</p>
            <ul>
                <li>修改上方时间设置后，点击"应用时间设置"按钮更新所有行的计算规则</li>
                <li>在姓名列输入员工姓名</li>
                <li>灰色背景的日期表示周末</li>
                <li>最后两行自动计算平时加班和周末加班总时长</li>
                <li>表格支持横向滚动，适应手机屏幕</li>
            </ul>
        </div>
    </div>

    <script>
        // 初始化表格
        document.addEventListener('DOMContentLoaded', function() {
            initializeTable();
            updateAll();
        });

        // 初始化表格结构
        function initializeTable() {
            const table = document.getElementById('attendance-table');
            const thead = table.querySelector('thead tr');
            const tbody = table.querySelector('tbody');
            
            // 清空现有内容
            thead.innerHTML = '<th class="name-header">姓名</th>';
            tbody.innerHTML = '';
            
            // 创建日期表头 (8月1日到30日)
            for (let day = 1; day <= 30; day++) {
                const date = new Date(2023, 7, day); // 8月是7 (0-based)
                const isWeekend = date.getDay() === 0 || date.getDay() === 6;
                
                const th = document.createElement('th');
                th.textContent = `${day}日`;
                if (isWeekend) {
                    th.classList.add('weekend');
                }
                thead.appendChild(th);
            }
            
            // 添加平时加班和周末加班总计列
            const overtimeNormalTh = document.createElement('th');
            overtimeNormalTh.textContent = '平时加班';
            thead.appendChild(overtimeNormalTh);
            
            const overtimeWeekendTh = document.createElement('th');
            overtimeWeekendTh.textContent = '周末加班';
            thead.appendChild(overtimeWeekendTh);
            
            // 添加5个员工行
            for (let i = 0; i < 5; i++) {
                addEmployeeRow(tbody, `员工${i+1}`);
            }
            
            // 添加总计行
            addTotalRow(tbody);
        }
        
        // 添加员工行
        function addEmployeeRow(tbody, name) {
            const row = document.createElement('tr');
            
            // 姓名单元格
            const nameCell = document.createElement('td');
            nameCell.className = 'name-cell';
            const nameInput = document.createElement('input');
            nameInput.type = 'text';
            nameInput.value = name;
            nameInput.addEventListener('change', calculateOvertime);
            nameCell.appendChild(nameInput);
            row.appendChild(nameCell);
            
            // 添加日期输入单元格
            for (let day = 1; day <= 30; day++) {
                const date = new Date(2023, 7, day);
                const isWeekend = date.getDay() === 0 || date.getDay() === 6;
                
                const cell = document.createElement('td');
                if (isWeekend) {
                    cell.classList.add('weekend');
                }
                
                const timeInput = document.createElement('input');
                timeInput.type = 'time';
                timeInput.addEventListener('change', calculateOvertime);
                cell.appendChild(timeInput);
                
                row.appendChild(cell);
            }
            
            // 添加加班时间单元格
            const overtimeNormalCell = document.createElement('td');
            overtimeNormalCell.className = 'overtime-cell';
            overtimeNormalCell.textContent = '0小时';
            row.appendChild(overtimeNormalCell);
            
            const overtimeWeekendCell = document.createElement('td');
            overtimeWeekendCell.className = 'overtime-cell';
            overtimeWeekendCell.textContent = '0小时';
            row.appendChild(overtimeWeekendCell);
            
            tbody.appendChild(row);
        }
        
        // 添加总计行
        function addTotalRow(tbody) {
            const row = document.createElement('tr');
            row.className = 'total-row';
            
            const totalCell = document.createElement('td');
            totalCell.textContent = '总计';
            totalCell.style.fontWeight = 'bold';
            row.appendChild(totalCell);
            
            // 添加空单元格
            for (let i = 0; i < 32; i++) {
                const cell = document.createElement('td');
                row.appendChild(cell);
            }
            
            tbody.appendChild(row);
        }
        
        // 计算加班时间
        function calculateOvertime() {
            const workStart = document.getElementById('work-start').value;
            const workEnd = document.getElementById('work-end').value;
            const overtimeStart = document.getElementById('overtime-start').value;
            
            const rows = document.querySelectorAll('#attendance-table tbody tr:not(.total-row)');
            const totalRow = document.querySelector('#attendance-table tbody tr.total-row');
            
            let totalNormal = 0;
            let totalWeekend = 0;
            
            rows.forEach(row => {
                let normalOvertime = 0;
                let weekendOvertime = 0;
                
                const cells = row.querySelectorAll('td:not(.name-cell)');
                for (let i = 0; i < 30; i++) {
                    const timeInput = cells[i].querySelector('input');
                    if (!timeInput || !timeInput.value) continue;
                    
                    const date = new Date(2023, 7, i+1);
                    const isWeekend = date.getDay() === 0 || date.getDay() === 6;
                    
                    // 计算加班时长
                    const overtimeHours = calculateOvertimeHours(timeInput.value, overtimeStart);
                    
                    if (isWeekend) {
                        weekendOvertime += overtimeHours;
                    } else {
                        normalOvertime += overtimeHours;
                    }
                }
                
                // 更新行结果
                const normalCell = cells[30];
                const weekendCell = cells[31];
                
                normalCell.textContent = normalOvertime.toFixed(1) + '小时';
                weekendCell.textContent = weekendOvertime.toFixed(1) + '小时';
                
                totalNormal += normalOvertime;
                totalWeekend += weekendOvertime;
            });
            
            // 更新总计行
            if (totalRow) {
                const totalCells = totalRow.querySelectorAll('td');
                totalCells[31].textContent = totalNormal.toFixed(1) + '小时';
                totalCells[32].textContent = totalWeekend.toFixed(1) + '小时';
            }
        }
        
        // 计算加班小时数
        function calculateOvertimeHours(endTime, overtimeStart) {
            const [endHour, endMinute] = endTime.split(':').map(Number);
            const [startHour, startMinute] = overtimeStart.split(':').map(Number);
            
            // 如果下班时间早于加班开始时间，返回0
            if (endHour < startHour || (endHour === startHour && endMinute <= startMinute)) {
                return 0;
            }
            
            // 计算加班分钟数
            let overtimeMinutes = (endHour - startHour) * 60 + (endMinute - startMinute);
            
            // 转换为小时，保留1位小数
            return Math.round(overtimeMinutes / 6) / 10;
        }
        
        // 更新所有行的计算
        function updateAll() {
            calculateOvertime();
        }
    </script>
</body>
</html>
