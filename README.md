<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>小说数据大屏</title>
    <script src="https://cdn.jsdelivr.net/npm/echarts@5.5.0/dist/echarts.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/echarts-wordcloud@2.1.0/dist/echarts-wordcloud.min.js"></script>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body {
            font-family: 'Microsoft YaHei', 'PingFang SC', sans-serif;
            background: linear-gradient(135deg, #0a0e27 0%, #1a1f4e 50%, #0d1137 100%);
            color: #e0e6ff;
            min-height: 100vh;
            overflow-x: hidden;
        }
        .header {
            text-align: center;
            padding: 24px 20px 12px;
            position: relative;
        }
        .header h1 {
            font-size: 32px;
            background: linear-gradient(90deg, #00d4ff, #7b68ee, #ff6b9d);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            letter-spacing: 6px;
            text-shadow: 0 0 30px rgba(0,212,255,0.3);
        }
        .header::after {
            content: '';
            display: block;
            width: 60%;
            height: 2px;
            margin: 12px auto 0;
            background: linear-gradient(90deg, transparent, #00d4ff, #7b68ee, #ff6b9d, transparent);
        }
        .stats-row {
            display: grid;
            grid-template-columns: repeat(6, 1fr);
            gap: 16px;
            padding: 16px 24px;
            max-width: 1600px;
            margin: 0 auto;
        }
        .stat-card {
            background: rgba(255,255,255,0.05);
            border: 1px solid rgba(255,255,255,0.1);
            border-radius: 12px;
            padding: 20px 16px;
            text-align: center;
            backdrop-filter: blur(10px);
            transition: transform 0.3s, box-shadow 0.3s;
        }
        .stat-card:hover {
            transform: translateY(-4px);
            box-shadow: 0 8px 32px rgba(0,212,255,0.2);
        }
        .stat-card .value {
            font-size: 28px;
            font-weight: bold;
            background: linear-gradient(180deg, #00d4ff, #7b68ee);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
        }
        .stat-card .unit {
            font-size: 14px;
            color: #7b8ec2;
            margin-left: 2px;
        }
        .stat-card .label {
            font-size: 13px;
            color: #8892b0;
            margin-top: 6px;
        }
        .charts-row {
            display: grid;
            grid-template-columns: 1fr 1fr 1fr;
            gap: 16px;
            padding: 8px 24px 16px;
            max-width: 1600px;
            margin: 0 auto;
        }
        .chart-card {
            background: rgba(255,255,255,0.04);
            border: 1px solid rgba(255,255,255,0.08);
            border-radius: 12px;
            padding: 20px;
            backdrop-filter: blur(10px);
        }
        .chart-card h3 {
            font-size: 16px;
            margin-bottom: 12px;
            color: #a0b0ff;
            padding-left: 10px;
            border-left: 3px solid #7b68ee;
        }
        .chart-container {
            width: 100%;
            height: 320px;
        }
        .table-section {
            padding: 8px 24px 32px;
            max-width: 1600px;
            margin: 0 auto;
        }
        .table-card {
            background: rgba(255,255,255,0.04);
            border: 1px solid rgba(255,255,255,0.08);
            border-radius: 12px;
            padding: 20px;
            backdrop-filter: blur(10px);
        }
        .table-card h3 {
            font-size: 16px;
            margin-bottom: 16px;
            color: #a0b0ff;
            padding-left: 10px;
            border-left: 3px solid #7b68ee;
        }
        .novel-table {
            width: 100%;
            border-collapse: collapse;
            font-size: 13px;
        }
        .novel-table thead th {
            background: rgba(123,104,238,0.15);
            padding: 10px 12px;
            text-align: left;
            color: #a0b0ff;
            border-bottom: 1px solid rgba(255,255,255,0.1);
            position: sticky;
            top: 0;
        }
        .novel-table tbody tr {
            transition: background 0.2s;
        }
        .novel-table tbody tr:hover {
            background: rgba(0,212,255,0.06);
        }
        .novel-table td {
            padding: 9px 12px;
            border-bottom: 1px solid rgba(255,255,255,0.04);
        }
        .tag {
            display: inline-block;
            padding: 2px 8px;
            border-radius: 10px;
            font-size: 11px;
            margin: 1px 2px;
            background: rgba(123,104,238,0.2);
            color: #b8c4ff;
        }
        .status-done {
            color: #91cc75;
        }
        .status-ongoing {
            color: #fac858;
        }
        .table-scroll {
            max-height: 420px;
            overflow-y: auto;
        }
        .table-scroll::-webkit-scrollbar { width: 6px; }
        .table-scroll::-webkit-scrollbar-track { background: transparent; }
        .table-scroll::-webkit-scrollbar-thumb { background: rgba(123,104,238,0.3); border-radius: 3px; }
        @keyframes fadeInUp {
            from { opacity: 0; transform: translateY(20px); }
            to { opacity: 1; transform: translateY(0); }
        }
        .stat-card, .chart-card, .table-card {
            animation: fadeInUp 0.6s ease-out backwards;
        }
        .stat-card:nth-child(1) { animation-delay: 0.1s; }
        .stat-card:nth-child(2) { animation-delay: 0.15s; }
        .stat-card:nth-child(3) { animation-delay: 0.2s; }
        .stat-card:nth-child(4) { animation-delay: 0.25s; }
        .stat-card:nth-child(5) { animation-delay: 0.3s; }
        .stat-card:nth-child(6) { animation-delay: 0.35s; }
    </style>
</head>
<body>

<div class="header">
    <h1>📊 小说数据可视化大屏</h1>
</div>

<div class="stats-row" id="statsRow"></div>

<div class="charts-row">
    <div class="chart-card">
        <h3>分类分布</h3>
        <div id="categoryChart" class="chart-container"></div>
    </div>
    <div class="chart-card">
        <h3>热门关键词</h3>
        <div id="wordCloudChart" class="chart-container"></div>
    </div>
    <div class="chart-card">
        <h3>字数排行榜 TOP10</h3>
        <div id="wordRankChart" class="chart-container"></div>
    </div>
</div>

<div class="charts-row">
    <div class="chart-card" style="grid-column: span 2;">
        <h3>月度更新频率</h3>
        <div id="updateChart" class="chart-container"></div>
    </div>
    <div class="chart-card">
        <h3>连载状态占比</h3>
        <div id="statusChart" class="chart-container"></div>
    </div>
</div>

<div class="table-section">
    <div class="table-card">
        <h3>小说列表</h3>
        <div class="table-scroll">
            <table class="novel-table">
                <thead>
                    <tr>
                        <th>#</th>
                        <th>书名</th>
                        <th>作者</th>
                        <th>分类</th>
                        <th>字数</th>
                        <th>状态</th>
                        <th>标签</th>
                        <th>更新时间</th>
                    </tr>
                </thead>
                <tbody id="novelTableBody"></tbody>
            </table>
        </div>
    </div>
</div>

<script>
const DATA = {
    "stats": {
        "totalNovels": 50,
        "totalAuthors": 35,
        "totalCategories": 8,
        "totalWords": 258600000,
        "avgWords": 5172000,
        "avgWordsPerAuthor": 7388571
    },
    "categories": [
        {"name":"玄幻","value":12,"color":"#5470c6"},
        {"name":"都市","value":10,"color":"#91cc75"},
        {"name":"科幻","value":8,"color":"#fac858"},
        {"name":"仙侠","value":7,"color":"#ee6666"},
        {"name":"游戏","value":5,"color":"#73c0de"},
        {"name":"历史","value":4,"color":"#3ba272"},
        {"name":"悬疑","value":3,"color":"#fc8452"},
        {"name":"轻小说","value":1,"color":"#9a60b4"}
    ],
    "wordCloud": [
        {"name":"穿越","value":95},{"name":"系统","value":88},{"name":"修仙","value":82},
        {"name":"都市","value":78},{"name":"玄幻","value":75},{"name":"热血","value":70},
        {"name":"无敌","value":68},{"name":"搞笑","value":65},{"name":"校花","value":62},
        {"name":"重生","value":58},{"name":"科幻","value":55},{"name":"异世","value":52},
        {"name":"升级","value":50},{"name":"奇幻","value":48},{"name":"冒险","value":45},
        {"name":"魔法","value":42},{"name":"战斗","value":40},{"name":"阴谋","value":38},
        {"name":"逆袭","value":35},{"name":"天才","value":32},{"name":"废柴","value":30},
        {"name":"养成","value":28},{"name":"种田","value":25},{"name":"末世","value":22},
        {"name":"星际","value":20}
    ],
    "wordRank": [
        {"name":"星河大帝","words":8920000},{"name":"完美世界","words":8540000},
        {"name":"全职高手","words":7890000},{"name":"斗破苍穹","words":7650000},
        {"name":"凡人修仙传","words":7450000},{"name":"遮天","words":7230000},
        {"name":"盘龙","words":6980000},{"name":"武动乾坤","words":6850000},
        {"name":"神墓","words":6540000},{"name":"雪中悍刀行","words":6320000}
    ],
    "updateFrequency": [
        {"month":"1月","count":45},{"month":"2月","count":38},{"month":"3月","count":52},
        {"month":"4月","count":48},{"month":"5月","count":61},{"month":"6月","count":55},
        {"month":"7月","count":68},{"month":"8月","count":72},{"month":"9月","count":58},
        {"month":"10月","count":63},{"month":"11月","count":57},{"month":"12月","count":49}
    ],
    "novels": [
        {"id":1,"title":"星河大帝","author":"梦入神机","category":"玄幻","words":8920000,"status":"完结","tags":["穿越","系统","热血"],"updateTime":"2024-03-15"},
        {"id":2,"title":"完美世界","author":"辰东","category":"玄幻","words":8540000,"status":"完结","tags":["修仙","热血","逆袭"],"updateTime":"2024-02-28"},
        {"id":3,"title":"全职高手","author":"蝴蝶蓝","category":"游戏","words":7890000,"status":"完结","tags":["电竞","热血","团队"],"updateTime":"2024-01-20"},
        {"id":4,"title":"斗破苍穹","author":"天蚕土豆","category":"玄幻","words":7650000,"status":"完结","tags":["穿越","修炼","热血"],"updateTime":"2023-12-10"},
        {"id":5,"title":"凡人修仙传","author":"忘语","category":"仙侠","words":7450000,"status":"完结","tags":["修仙","成长","凡人流"],"updateTime":"2024-04-05"},
        {"id":6,"title":"遮天","author":"辰东","category":"玄幻","words":7230000,"status":"完结","tags":["修仙","热血","冒险"],"updateTime":"2024-03-22"},
        {"id":7,"title":"盘龙","author":"我吃西红柿","category":"玄幻","words":6980000,"status":"完结","tags":["穿越","魔法","热血"],"updateTime":"2024-02-15"},
        {"id":8,"title":"武动乾坤","author":"天蚕土豆","category":"玄幻","words":6850000,"status":"完结","tags":["修炼","热血","逆袭"],"updateTime":"2024-01-08"},
        {"id":9,"title":"神墓","author":"辰东","category":"玄幻","words":6540000,"status":"完结","tags":["穿越","热血","阴谋"],"updateTime":"2023-11-25"},
        {"id":10,"title":"雪中悍刀行","author":"烽火戏诸侯","category":"武侠","words":6320000,"status":"完结","tags":["武侠","热血","权谋"],"updateTime":"2024-04-12"},
        {"id":11,"title":"都市最强装逼系统","author":"南鹤","category":"都市","words":5890000,"status":"完结","tags":["系统","都市","搞笑"],"updateTime":"2024-03-01"},
        {"id":12,"title":"三体","author":"刘慈欣","category":"科幻","words":5650000,"status":"完结","tags":["科幻","硬科幻","星际"],"updateTime":"2023-10-15"},
        {"id":13,"title":"我欲封天","author":"耳根","category":"仙侠","words":5420000,"status":"完结","tags":["修仙","热血","成长"],"updateTime":"2024-02-20"},
        {"id":14,"title":"一念永恒","author":"耳根","category":"仙侠","words":5180000,"status":"完结","tags":["修仙","搞笑","热血"],"updateTime":"2024-04-08"},
        {"id":15,"title":"星辰变","author":"我吃西红柿","category":"玄幻","words":4980000,"status":"完结","tags":["穿越","修炼","热血"],"updateTime":"2023-12-28"},
        {"id":16,"title":"校花的贴身高手","author":"鱼人二代","category":"都市","words":4750000,"status":"连载","tags":["都市","校花","保镖"],"updateTime":"2024-04-15"},
        {"id":17,"title":"庆余年","author":"猫腻","category":"历史","words":4650000,"status":"完结","tags":["权谋","历史","穿越"],"updateTime":"2024-01-30"},
        {"id":18,"title":"仙逆","author":"耳根","category":"仙侠","words":4520000,"status":"完结","tags":["修仙","逆袭","热血"],"updateTime":"2024-03-10"},
        {"id":19,"title":"飞剑问道","author":"我吃西红柿","category":"仙侠","words":4380000,"status":"完结","tags":["修仙","神话","热血"],"updateTime":"2024-02-05"},
        {"id":20,"title":"吞噬星空","author":"我吃西红柿","category":"科幻","words":4250000,"status":"完结","tags":["科幻","进化","热血"],"updateTime":"2024-04-01"},
        {"id":21,"title":"宰执天下","author":"cuslaa","category":"历史","words":4120000,"status":"完结","tags":["穿越","历史","权谋"],"updateTime":"2023-11-12"},
        {"id":22,"title":"重生之都市修仙","author":"十里剑神","category":"都市","words":3980000,"status":"完结","tags":["重生","修仙","都市"],"updateTime":"2024-03-25"},
        {"id":23,"title":"琅琊榜","author":"海宴","category":"历史","words":3850000,"status":"完结","tags":["权谋","历史","复仇"],"updateTime":"2023-09-18"},
        {"id":24,"title":"从前有座灵剑山","author":"国王陛下","category":"仙侠","words":3720000,"status":"完结","tags":["修仙","搞笑","热血"],"updateTime":"2024-01-15"},
        {"id":25,"title":"全职法师","author":"乱","category":"玄幻","words":3580000,"status":"完结","tags":["魔法","热血","冒险"],"updateTime":"2024-04-10"},
        {"id":26,"title":"惊悚乐园","author":"三天两觉","category":"游戏","words":3450000,"status":"完结","tags":["游戏","悬疑","搞笑"],"updateTime":"2024-02-28"},
        {"id":27,"title":"末日蟑螂","author":"伟岸蟑螂","category":"科幻","words":3320000,"status":"完结","tags":["末世","生存","进化"],"updateTime":"2024-03-05"},
        {"id":28,"title":"我师兄实在太稳健了","author":"言归正传","category":"仙侠","words":3180000,"status":"完结","tags":["修仙","搞笑","稳健"],"updateTime":"2024-04-12"},
        {"id":29,"title":"全球高武","author":"老鹰吃小鸡","category":"都市","words":3050000,"status":"完结","tags":["系统","都市","热血"],"updateTime":"2024-01-22"},
        {"id":30,"title":"烂柯棋缘","author":"真费事","category":"仙侠","words":2920000,"status":"完结","tags":["修仙","古典","神秘"],"updateTime":"2024-03-18"},
        {"id":31,"title":"大奉打更人","author":"卖报小郎君","category":"仙侠","words":2780000,"status":"完结","tags":["穿越","探案","搞笑"],"updateTime":"2024-04-05"},
        {"id":32,"title":"亏成首富从游戏开始","author":"青衫取醉","category":"游戏","words":2650000,"status":"完结","tags":["游戏","经营","搞笑"],"updateTime":"2024-02-10"},
        {"id":33,"title":"夜的命名术","author":"会说话的肘子","category":"都市","words":2520000,"status":"连载","tags":["穿越","都市","热血"],"updateTime":"2024-04-15"},
        {"id":34,"title":"道君","author":"跃千愁","category":"仙侠","words":2380000,"status":"完结","tags":["修仙","权谋","热血"],"updateTime":"2024-01-28"},
        {"id":35,"title":"超神机械师","author":"齐佩甲","category":"游戏","words":2250000,"status":"完结","tags":["游戏","系统","科幻"],"updateTime":"2024-03-12"},
        {"id":36,"title":"诡秘之主","author":"爱潜水的乌贼","category":"奇幻","words":2120000,"status":"完结","tags":["悬疑","奇幻","克苏鲁"],"updateTime":"2024-04-02"},
        {"id":37,"title":"修真聊天群","author":"圣骑士的传说","category":"都市","words":1980000,"status":"完结","tags":["修仙","搞笑","都市"],"updateTime":"2024-02-18"},
        {"id":38,"title":"轮回乐园","author":"那一只蚊子","category":"游戏","words":1850000,"status":"连载","tags":["无限流","热血","战斗"],"updateTime":"2024-04-14"},
        {"id":39,"title":"九星毒奶","author":"育","category":"科幻","words":1720000,"status":"完结","tags":["科幻","搞笑","成长"],"updateTime":"2024-01-05"},
        {"id":40,"title":"大江大河","author":"阿耐","category":"都市","words":1580000,"status":"完结","tags":["现实","都市","励志"],"updateTime":"2023-12-20"},
        {"id":41,"title":"我有一座恐怖屋","author":"我会修空调","category":"悬疑","words":1450000,"status":"完结","tags":["恐怖","悬疑","经营"],"updateTime":"2024-03-28"},
        {"id":42,"title":"长安十二时辰","author":"马伯庸","category":"悬疑","words":1320000,"status":"完结","tags":["悬疑","历史","推理"],"updateTime":"2024-02-25"},
        {"id":43,"title":"上海堡垒","author":"江南","category":"科幻","words":1180000,"status":"完结","tags":["科幻","爱情","战争"],"updateTime":"2023-10-30"},
        {"id":44,"title":"明克街13号","author":"纯洁滴小龙","category":"悬疑","words":1050000,"status":"连载","tags":["悬疑","都市","神秘"],"updateTime":"2024-04-13"},
        {"id":45,"title":"神话版三国","author":"坟土荒草","category":"历史","words":920000,"status":"连载","tags":["穿越","历史","三国"],"updateTime":"2024-04-15"},
        {"id":46,"title":"夜的钢琴曲","author":"默然","category":"轻小说","words":780000,"status":"完结","tags":["治愈","音乐","青春"],"updateTime":"2024-01-12"},
        {"id":47,"title":"变成血族是什么体验","author":"神行汉堡","category":"都市","words":650000,"status":"连载","tags":["都市","异能","日常"],"updateTime":"2024-04-11"},
        {"id":48,"title":"我真没想重生啊","author":"柳暗花又明","category":"都市","words":520000,"status":"完结","tags":["都市","重生","商战"],"updateTime":"2024-03-20"},
        {"id":49,"title":"术师手册","author":"听日","category":"轻小说","words":380000,"status":"完结","tags":["异世界","冒险","魔法"],"updateTime":"2024-02-08"},
        {"id":50,"title":"这只妖怪不想营业","author":"饮雪","category":"轻小说","words":250000,"status":"连载","tags":["妖怪","治愈","日常"],"updateTime":"2024-04-09"}
    ]
};

function formatWords(n) {
    if (n >= 10000) return (n / 10000).toFixed(0) + '万';
    return n.toLocaleString();
}

const statsConfig = [
    { key: 'totalNovels', label: '小说总数', unit: '部' },
    { key: 'totalAuthors', label: '作者总数', unit: '位' },
    { key: 'totalCategories', label: '分类总数', unit: '类' },
    { key: 'totalWords', label: '总字数', unit: '', format: v => (v/100000000).toFixed(2) + '亿' },
    { key: 'avgWords', label: '平均字数', unit: '', format: v => (v/10000).toFixed(0) + '万' },
    { key: 'avgWordsPerAuthor', label: '人均字数', unit: '', format: v => (v/10000).toFixed(0) + '万' }
];
const statsRow = document.getElementById('statsRow');
statsConfig.forEach(s => {
    const val = s.format ? s.format(DATA.stats[s.key]) : DATA.stats[s.key];
    statsRow.innerHTML += '<div class="stat-card"><div class="value">' + val + '<span class="unit">' + s.unit + '</span></div><div class="label">' + s.label + '</div></div>';
});

const catChart = echarts.init(document.getElementById('categoryChart'));
catChart.setOption({
    tooltip: { trigger: 'item', formatter: '{b}: {c}部 ({d}%)' },
    series: [{
        type: 'pie',
        radius: ['40%', '70%'],
        itemStyle: { borderRadius: 6, borderColor: '#1a1f4e', borderWidth: 2 },
        label: { color: '#a0b0ff', fontSize: 12 },
        labelLine: { lineStyle: { color: '#556' } },
        data: DATA.categories.map(c => ({ name: c.name, value: c.value, itemStyle: { color: c.color } }))
    }]
});

const wcChart = echarts.init(document.getElementById('wordCloudChart'));
wcChart.setOption({
    tooltip: { show: true },
    series: [{
        type: 'wordCloud',
        gridSize: 12,
        sizeRange: [16, 50],
        rotationRange: [-30, 30],
        rotationStep: 15,
        shape: 'circle',
        textStyle: {
            fontFamily: 'Microsoft YaHei',
            fontWeight: 'bold',
            color: function () {
                const colors = ['#00d4ff','#7b68ee','#ff6b9d','#91cc75','#fac858','#73c0de','#fc8452','#ee6666'];
                return colors[Math.floor(Math.random() * colors.length)];
            }
        },
        data: DATA.wordCloud.map(w => ({ name: w.name, value: w.value }))
    }]
});

const rankChart = echarts.init(document.getElementById('wordRankChart'));
rankChart.setOption({
    tooltip: { trigger: 'axis', formatter: function(p) { return p[0].name + '<br/>' + (p[0].value/10000).toFixed(0) + '万字'; } },
    grid: { left: 100, right: 30, top: 10, bottom: 20 },
    xAxis: { type: 'value', axisLabel: { color: '#7b8ec2', formatter: function(v) { return (v/10000)+'万'; } }, splitLine: { lineStyle: { color: 'rgba(255,255,255,0.05)' } } },
    yAxis: { type: 'category', data: DATA.wordRank.map(w => w.name).reverse(), axisLabel: { color: '#a0b0ff' }, axisLine: { lineStyle: { color: '#334' } } },
    series: [{
        type: 'bar',
        data: DATA.wordRank.map(w => w.words).reverse(),
        itemStyle: {
            borderRadius: [0, 4, 4, 0],
            color: new echarts.graphic.LinearGradient(0, 0, 1, 0, [
                { offset: 0, color: '#5470c6' },
                { offset: 1, color: '#00d4ff' }
            ])
        },
        barWidth: 16
    }]
});

const upChart = echarts.init(document.getElementById('updateChart'));
upChart.setOption({
    tooltip: { trigger: 'axis' },
    grid: { left: 50, right: 30, top: 20, bottom: 30 },
    xAxis: { type: 'category', data: DATA.updateFrequency.map(m => m.month), axisLabel: { color: '#a0b0ff' }, axisLine: { lineStyle: { color: '#334' } } },
    yAxis: { type: 'value', axisLabel: { color: '#7b8ec2' }, splitLine: { lineStyle: { color: 'rgba(255,255,255,0.05)' } } },
    series: [{
        type: 'line',
        data: DATA.updateFrequency.map(m => m.count),
        smooth: true,
        symbol: 'circle',
        symbolSize: 8,
        lineStyle: { width: 3, color: '#00d4ff' },
        itemStyle: { color: '#00d4ff', borderWidth: 2 },
        areaStyle: {
            color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
                { offset: 0, color: 'rgba(0,212,255,0.3)' },
                { offset: 1, color: 'rgba(0,212,255,0.02)' }
            ])
        }
    }]
});

const done = DATA.novels.filter(n => n.status === '完结').length;
const ongoing = DATA.novels.filter(n => n.status === '连载').length;
const stChart = echarts.init(document.getElementById('statusChart'));
stChart.setOption({
    tooltip: { trigger: 'item', formatter: '{b}: {c}部 ({d}%)' },
    series: [{
        type: 'pie',
        radius: ['45%', '70%'],
        itemStyle: { borderRadius: 6, borderColor: '#1a1f4e', borderWidth: 2 },
        label: { color: '#a0b0ff' },
        data: [
            { name: '完结', value: done, itemStyle: { color: '#91cc75' } },
            { name: '连载', value: ongoing, itemStyle: { color: '#fac858' } }
        ]
    }]
});

const tbody = document.getElementById('novelTableBody');
DATA.novels.forEach(n => {
    const statusClass = n.status === '完结' ? 'status-done' : 'status-ongoing';
    const tags = n.tags.map(t => '<span class="tag">' + t + '</span>').join('');
    tbody.innerHTML += '<tr><td>' + n.id + '</td><td style="color:#00d4ff;font-weight:500">' + n.title + '</td><td>' + n.author + '</td><td>' + n.category + '</td><td>' + formatWords(n.words) + '</td><td class="' + statusClass + '">' + n.status + '</td><td>' + tags + '</td><td>' + n.updateTime + '</td></tr>';
});

window.addEventListener('resize', function() {
    [catChart, wcChart, rankChart, upChart, stChart].forEach(function(c) { c.resize(); });
});
</script>
</body>
</html>
