<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>紧急事务处理器</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: #1a1a1a;
            color: #333;
            padding: 20px;
            min-height: 100vh;
        }

        h1 {
            text-align: center;
            color: #fff;
            margin-bottom: 20px;
            font-size: 2.5em;
            text-shadow: 2px 2px 4px rgba(0,0,0,0.3);
        }

        .matrix {
            display: grid;
            grid-template-columns: 1fr 1fr;
            grid-template-rows: 1fr 1fr;
            gap: 15px;
            max-width: 1400px;
            margin: 0 auto;
            height: calc(100vh - 120px);
        }

        .quadrant {
            border-radius: 12px;
            padding: 20px;
            display: flex;
            flex-direction: column;
            box-shadow: 0 4px 15px rgba(0,0,0,0.3);
            transition: transform 0.2s;
        }

        .quadrant:hover {
            transform: translateY(-3px);
            box-shadow: 0 8px 25px rgba(0,0,0,0.4);
        }

        .quadrant-title {
            font-size: 1.5em;
            font-weight: bold;
            margin-bottom: 15px;
            padding-bottom: 10px;
            border-bottom: 3px solid rgba(0,0,0,0.2);
            display: flex;
            align-items: center;
            gap: 10px;
        }

        .quadrant-title .icon {
            font-size: 1.3em;
        }

        /* 紧急且重要 - 深红色 */
        .urgent-important {
            background: linear-gradient(135deg, #ff4444 0%, #cc0000 100%);
            color: #fff;
        }

        /* 紧急不重要 - 中红色 */
        .urgent-not-important {
            background: linear-gradient(135deg, #ff6666 0%, #ff3333 100%);
            color: #fff;
        }

        /* 重要不紧急 - 浅红色 */
        .not-urgent-important {
            background: linear-gradient(135deg, #ff9999 0%, #ff6666 100%);
            color: #fff;
        }

        /* 不紧急不重要 - 绿色 */
        .not-urgent-not-important {
            background: linear-gradient(135deg, #66cc66 0%, #44aa44 100%);
            color: #fff;
        }

        .add-task-form {
            display: flex;
            gap: 8px;
            margin-bottom: 15px;
        }

        .add-task-form input {
            flex: 1;
            padding: 10px 15px;
            border: none;
            border-radius: 6px;
            font-size: 14px;
            outline: none;
        }

        .add-task-form input:focus {
            box-shadow: 0 0 0 3px rgba(255,255,255,0.3);
        }

        .add-task-form button {
            padding: 10px 20px;
            background: rgba(0,0,0,0.3);
            color: #fff;
            border: none;
            border-radius: 6px;
            cursor: pointer;
            font-size: 14px;
            font-weight: bold;
            transition: background 0.2s;
        }

        .add-task-form button:hover {
            background: rgba(0,0,0,0.5);
        }

        .tasks {
            flex: 1;
            overflow-y: auto;
            display: flex;
            flex-direction: column;
            gap: 8px;
            padding-right: 5px;
        }

        .tasks::-webkit-scrollbar {
            width: 6px;
        }

        .tasks::-webkit-scrollbar-track {
            background: rgba(0,0,0,0.1);
            border-radius: 3px;
        }

        .tasks::-webkit-scrollbar-thumb {
            background: rgba(0,0,0,0.3);
            border-radius: 3px;
        }

        .task-item {
            background: rgba(255,255,255,0.95);
            color: #333;
            padding: 12px 15px;
            border-radius: 8px;
            display: flex;
            align-items: flex-start;
            gap: 10px;
            transition: all 0.2s;
            cursor: pointer;
        }

        .task-item:hover {
            transform: translateX(5px);
            background: rgba(255,255,255,1);
        }

        .task-checkbox {
            width: 20px;
            height: 20px;
            border: 2px solid #666;
            border-radius: 4px;
            cursor: pointer;
            flex-shrink: 0;
            margin-top: 2px;
            transition: all 0.2s;
            position: relative;
        }

        .task-checkbox:hover {
            border-color: #333;
        }

        .task-checkbox.checked {
            background: #4CAF50;
            border-color: #4CAF50;
        }

        .task-checkbox.checked::after {
            content: '✓';
            color: #fff;
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            font-size: 14px;
            font-weight: bold;
        }

        .task-content {
            flex: 1;
            word-break: break-word;
            line-height: 1.5;
        }

        .task-content.completed {
            text-decoration: line-through;
            color: #999;
        }

        .task-delete {
            background: #ff4444;
            color: #fff;
            border: none;
            width: 24px;
            height: 24px;
            border-radius: 4px;
            cursor: pointer;
            font-size: 14px;
            line-height: 1;
            opacity: 0;
            transition: opacity 0.2s;
            flex-shrink: 0;
        }

        .task-item:hover .task-delete {
            opacity: 1;
        }

        .task-delete:hover {
            background: #cc0000;
        }

        .empty-state {
            text-align: center;
            color: rgba(255,255,255,0.7);
            font-style: italic;
            padding: 20px;
        }

        @media (max-width: 900px) {
            .matrix {
                grid-template-columns: 1fr;
                grid-template-rows: repeat(4, auto);
                height: auto;
            }

            .quadrant {
                min-height: 300px;
            }
        }
    </style>
</head>
<body>
    <h1>🚨 紧急事务处理器</h1>

    <div class="matrix">
        <!-- 紧急且重要 -->
        <div class="quadrant urgent-important">
            <div class="quadrant-title">
                <span class="icon">🔴</span>
                紧急且重要
            </div>
            <div class="add-task-form">
                <input type="text" placeholder="添加任务..." id="input-urgent-important">
                <button onclick="addTask('urgent-important')">添加</button>
            </div>
            <div class="tasks" id="tasks-urgent-important"></div>
        </div>

        <!-- 紧急不重要 -->
        <div class="quadrant urgent-not-important">
            <div class="quadrant-title">
                <span class="icon">🟠</span>
                紧急不重要
            </div>
            <div class="add-task-form">
                <input type="text" placeholder="添加任务..." id="input-urgent-not-important">
                <button onclick="addTask('urgent-not-important')">添加</button>
            </div>
            <div class="tasks" id="tasks-urgent-not-important"></div>
        </div>

        <!-- 重要不紧急 -->
        <div class="quadrant not-urgent-important">
            <div class="quadrant-title">
                <span class="icon">🟡</span>
                重要不紧急
            </div>
            <div class="add-task-form">
                <input type="text" placeholder="添加任务..." id="input-not-urgent-important">
                <button onclick="addTask('not-urgent-important')">添加</button>
            </div>
            <div class="tasks" id="tasks-not-urgent-important"></div>
        </div>

        <!-- 不紧急不重要 -->
        <div class="quadrant not-urgent-not-important">
            <div class="quadrant-title">
                <span class="icon">🟢</span>
                不紧急不重要
            </div>
            <div class="add-task-form">
                <input type="text" placeholder="添加任务..." id="input-not-urgent-not-important">
                <button onclick="addTask('not-urgent-not-important')">添加</button>
            </div>
            <div class="tasks" id="tasks-not-urgent-not-important"></div>
        </div>
    </div>

    <script>
        // 数据存储
        let tasksData = {
            'urgent-important': [],
            'urgent-not-important': [],
            'not-urgent-important': [],
            'not-urgent-not-important': []
        };

        // 从本地存储加载数据
        function loadTasks() {
            const saved = localStorage.getItem('emergencyTasks');
            if (saved) {
                tasksData = JSON.parse(saved);
            }
            renderAllTasks();
        }

        // 保存到本地存储
        function saveTasks() {
            localStorage.setItem('emergencyTasks', JSON.stringify(tasksData));
        }

        // 添加任务
        function addTask(quadrant) {
            const input = document.getElementById(`input-${quadrant}`);
            const taskText = input.value.trim();

            if (taskText === '') {
                input.focus();
                return;
            }

            const newTask = {
                id: Date.now(),
                text: taskText,
                completed: false
            };

            tasksData[quadrant].unshift(newTask);
            saveTasks();
            renderTasks(quadrant);
            input.value = '';
            input.focus();
        }

        // 切换任务完成状态
        function toggleTask(quadrant, taskId) {
            const task = tasksData[quadrant].find(t => t.id === taskId);
            if (task) {
                task.completed = !task.completed;
                saveTasks();
                renderTasks(quadrant);
            }
        }

        // 删除任务
        function deleteTask(quadrant, taskId) {
            tasksData[quadrant] = tasksData[quadrant].filter(t => t.id !== taskId);
            saveTasks();
            renderTasks(quadrant);
        }

        // 渲染任务列表
        function renderTasks(quadrant) {
            const container = document.getElementById(`tasks-${quadrant}`);
            const tasks = tasksData[quadrant];

            if (tasks.length === 0) {
                container.innerHTML = '<div class="empty-state">暂无任务</div>';
                return;
            }

            container.innerHTML = tasks.map(task => `
                <div class="task-item" onclick="toggleTask('${quadrant}', ${task.id})">
                    <div class="task-checkbox ${task.completed ? 'checked' : ''}"></div>
                    <div class="task-content ${task.completed ? 'completed' : ''}">${escapeHtml(task.text)}</div>
                    <button class="task-delete" onclick="event.stopPropagation(); deleteTask('${quadrant}', ${task.id})">×</button>
                </div>
            `).join('');
        }

        // 渲染所有任务
        function renderAllTasks() {
            Object.keys(tasksData).forEach(quadrant => {
                renderTasks(quadrant);
            });
        }

        // HTML转义防止XSS
        function escapeHtml(text) {
            const div = document.createElement('div');
            div.textContent = text;
            return div.innerHTML;
        }

        // 绑定回车键事件
        document.querySelectorAll('.add-task-form input').forEach(input => {
            input.addEventListener('keypress', function(e) {
                if (e.key === 'Enter') {
                    const quadrant = this.id.replace('input-', '');
                    addTask(quadrant);
                }
            });
        });

        // 初始化
        loadTasks();
    </script>
</body>
</html>
