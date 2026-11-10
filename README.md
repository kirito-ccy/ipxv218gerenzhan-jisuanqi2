<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
<title>极速记账</title>
<style>
  :root {
    --bg-main: #121212;
    --bg-panel: #1c1c1e;
    --bg-key: #2c2c2e;
    --text-main: #ffffff;
    --text-muted: #8e8e93;
    --accent: #e85a4f;
  }
  * { box-sizing: border-box; margin: 0; padding: 0; user-select: none; -webkit-tap-highlight-color: transparent; }
  body { background: var(--bg-main); color: var(--text-main); font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif; height: 100dvh; display: flex; flex-direction: column; overflow: hidden; }
  
  /* 顶部导航栏 */
  .header { display: flex; justify-content: space-between; align-items: center; padding: 12px 16px; }
  .btn-history { background: var(--bg-key); border: none; color: #4da3ff; font-size: 13px; padding: 6px 12px; border-radius: 16px; cursor: pointer; }
  .tabs { display: flex; background: var(--bg-panel); border-radius: 20px; padding: 3px; }
  .tab { padding: 6px 16px; border-radius: 17px; font-size: 14px; color: var(--text-muted); cursor: pointer; }
  .tab.active { background: var(--bg-key); color: var(--text-main); font-weight: 600; }

  /* 分类图标网格 */
  .categories { flex: 1; display: grid; grid-template-columns: repeat(5, 1fr); gap: 14px 8px; padding: 12px 16px; overflow-y: auto; align-content: start; }
  .cat-item { display: flex; flex-direction: column; align-items: center; gap: 6px; cursor: pointer; }
  .cat-icon { width: 46px; height: 46px; border-radius: 50%; background: var(--bg-panel); display: flex; align-items: center; justify-content: center; font-size: 22px; transition: 0.15s; }
  .cat-item.active .cat-icon { background: #d0e8ff; transform: scale(1.06); }
  .cat-name { font-size: 11px; color: var(--text-muted); text-align: center; white-space: nowrap; }
  .cat-item.active .cat-name { color: var(--text-main); font-weight: 500; }

  /* 底部输入与键盘区 */
  .bottom-area { background: var(--bg-panel); border-radius: 20px 20px 0 0; display: flex; flex-direction: column; }
  .tags-bar { display: flex; gap: 8px; padding: 10px 14px; overflow-x: auto; scrollbar-width: none; }
  .tag-pill { background: var(--bg-key); padding: 5px 10px; border-radius: 14px; font-size: 12px; color: #ccc; white-space: nowrap; cursor: pointer; }
  
  .display-area { padding: 4px 16px 10px; border-bottom: 1px solid #2c2c2e; }
  .amount-line { font-size: 32px; font-weight: 700; color: var(--accent); min-height: 40px; }
  .note-line { display: flex; align-items: center; gap: 10px; margin-top: 4px; }
  .time-badge { font-size: 12px; color: var(--text-muted); }
  .note-input { flex: 1; background: transparent; border: none; color: var(--text-main); font-size: 14px; outline: none; }
  
  /* 键盘布局 */
  .keypad { display: grid; grid-template-columns: repeat(4, 1fr); gap: 8px; padding: 10px 12px 24px; background: var(--bg-main); }
  .key { background: var(--bg-panel); border: none; color: var(--text-main); font-size: 20px; height: 50px; border-radius: 12px; cursor: pointer; display: flex; align-items: center; justify-content: center; }
  .key:active { filter: brightness(1.3); }
  .key.op { font-size: 18px; }
  .key.sub-text { font-size: 14px; font-weight: 500; }
  .key.done { background: var(--accent); font-size: 16px; font-weight: 600; }

  /* 账单记录弹窗 */
  .modal { position: fixed; inset: 0; background: var(--bg-main); display: none; flex-direction: column; z-index: 99; }
  .modal.open { display: flex; }
  .modal-header { display: flex; justify-content: space-between; align-items: center; padding: 16px; border-bottom: 1px solid #2c2c2e; }
  .record-list { flex: 1; overflow-y: auto; padding: 12px 16px; }
  .record-item { display: flex; justify-content: space-between; align-items: center; padding: 12px 0; border-bottom: 1px solid #222; }
  .record-left { display: flex; align-items: center; gap: 10px; }
  .record-note { font-size: 12px; color: var(--text-muted); margin-top: 2px; }
</style>
</head>
<body>

<div class="header">
  <button class="btn-history" onclick="toggleModal(true)">📋 账单明细</button>
  <div class="tabs">
    <div class="tab active" onclick="switchTab('支出', this)">支出</div>
    <div class="tab" onclick="switchTab('收入', this)">收入</div>
    <div class="tab" onclick="switchTab('转账', this)">转账</div>
  </div>
  <button class="btn-history" onclick="exportCSV()">导出</button>
</div>

<div class="categories" id="catGrid"></div>

<div class="bottom-area">
  <div class="tags-bar">
    <div class="tag-pill" onclick="switchAccount()" id="accountPill">💳 默认账户</div>
    <div class="tag-pill" onclick="toggleReimburse()" id="reimbursePill">⭕ 报销</div>
    <div class="tag-pill" onclick="quickNote('认亲费')">🏷️ 认亲费</div>
    <div class="tag-pill" onclick="quickNote('餐饮日常')">🏷️ 餐饮日常</div>
  </div>
  <div class="display-area">
    <div class="amount-line" id="amountDisplay">¥0</div>
    <div class="note-line">
      <span class="time-badge" id="timeDisplay"></span>
      <input type="text" class="note-input" id="noteInput" placeholder="点击输入备注...">
    </div>
  </div>
  <div class="keypad">
    <button class="key" onclick="press('1')">1</button>
    <button class="key" onclick="press('2')">2</button>
    <button class="key" onclick="press('3')">3</button>
    <button class="key op" onclick="press('+')">+</button>
    <button class="key" onclick="press('4')">4</button>
    <button class="key" onclick="press('5')">5</button>
    <button class="key" onclick="press('6')">6</button>
    <button class="key op" onclick="press('-')">-</button>
    <button class="key" onclick="press('7')">7</button>
    <button class="key" onclick="press('8')">8</button>
    <button class="key" onclick="press('9')">9</button>
    <button class="key sub-text" onclick="saveRecord(true)">保存再记</button>
    <button class="key" onclick="press('.')">.</button>
    <button class="key" onclick="press('0')">0</button>
    <button class="key" onclick="press('del')">⌫</button>
    <button class="key done" onclick="saveRecord(false)">完成</button>
  </div>
</div>

<!-- 历史账单弹窗 -->
<div class="modal" id="historyModal">
  <div class="modal-header">
    <strong>账单记录</strong>
    <button class="btn-history" onclick="toggleModal(false)">关闭</button>
  </div>
  <div class="record-list" id="recordList"></div>
</div>

<script>
const categories = [
  { name: '餐饮-drinking', icon: '🧋' }, { name: '购物', icon: '🛒' }, { name: '服饰', icon: '💎' },
  { name: '洗衣服', icon: '🧼' }, { name: '数码', icon: '📱' }, { name: '美妆', icon: '🎀' },
  { name: '护肤', icon: '🧴' }, { name: '应用软件', icon: '🅰️' }, { name: '住房', icon: '🏠' },
  { name: '交通', icon: '🚕' }, { name: '娱乐', icon: '🎮' }, { name: '医疗', icon: '🩺' },
  { name: '通讯', icon: '📞' }, { name: '汽车', icon: '🚗' }, { name: '学习', icon: '📖' },
  { name: '办公', icon: '🖨️' }, { name: '运动', icon: '🏸' }, { name: '社交', icon: '👥' },
  { name: '人情', icon: '🧧' }, { name: '育儿', icon: '🍼' }
];

let currentType = '支出';
let currentCat = categories[0];
let expr = '';
let accounts = ['默认账户', '微信', '支付宝', '银行卡'];
let accIdx = 0;
let reimburse = false;

function init() {
  const grid = document.getElementById('catGrid');
  grid.innerHTML = categories.map((c, i) => `
    <div class="cat-item ${i === 0 ? 'active' : ''}" onclick="selectCat(${i}, this)">
      <div class="cat-icon">${c.icon}</div>
      <div class="cat-name">${c.name}</div>
    </div>
  `).join('');
  updateTime();
  setInterval(updateTime, 30000);
}

function updateTime() {
  const now = new Date();
  document.getElementById('timeDisplay').innerText = 
    `🕒 ${String(now.getHours()).padStart(2,'0')}:${String(now.getMinutes()).padStart(2,'0')}`;
}

function selectCat(idx, el) {
  currentCat = categories[idx];
  document.querySelectorAll('.cat-item').forEach(item => item.classList.remove('active'));
  el.classList.add('active');
}

function switchTab(type, el) {
  currentType = type;
  document.querySelectorAll('.tab').forEach(t => t.classList.remove('active'));
  el.classList.add('active');
}

function switchAccount() {
  accIdx = (accIdx + 1) % accounts.length;
  document.getElementById('accountPill').innerText = `💳 ${accounts[accIdx]}`;
}

function toggleReimburse() {
  reimburse = !reimburse;
  document.getElementById('reimbursePill').innerText = reimburse ? '✅ 报销' : '⭕ 报销';
}

function quickNote(text) {
  document.getElementById('noteInput').value = text;
}

function press(key) {
  if (key === 'del') {
    expr = expr.slice(0, -1);
  } else {
    expr += key;
  }
  document.getElementById('amountDisplay').innerText = '¥' + (expr || '0');
}

function evalAmount() {
  try {
    if (!expr) return 0;
    const sanitized = expr.replace(/[^0-9+\-.]/g, '');
    return parseFloat(Function(`"use strict"; return (${sanitized})`)().toFixed(2));
  } catch {
    return parseFloat(expr) || 0;
  }
}

function saveRecord(keepOpen) {
  const amount = evalAmount();
  if (amount <= 0) { alert('请输入有效金额'); return; }
  const records = JSON.parse(localStorage.getItem('bills') || '[]');
  records.unshift({
    id: Date.now(),
    time: new Date().toLocaleString('zh-CN', { hour12: false }),
    type: currentType,
    category: currentCat.name,
    icon: currentCat.icon,
    amount: amount,
    account: accounts[accIdx],
    reimburse: reimburse,
    note: document.getElementById('noteInput').value || ''
  });
  localStorage.setItem('bills', JSON.stringify(records));
  expr = '';
  document.getElementById('amountDisplay').innerText = '¥0';
  document.getElementById('noteInput').value = '';
  if (!keepOpen) toggleModal(true);
}

function toggleModal(show) {
  const modal = document.getElementById('historyModal');
  modal.classList.toggle('open', show);
  if (show) renderHistory();
}

function renderHistory() {
  const records = JSON.parse(localStorage.getItem('bills') || '[]');
  const list = document.getElementById('recordList');
  if (!records.length) { list.innerHTML = '<p style="color:#666;text-align:center;margin-top:40px;">暂无记账记录</p>'; return; }
  list.innerHTML = records.map(r => `
    <div class="record-item">
      <div class="record-left">
        <span style="font-size:24px">${r.icon}</span>
        <div>
          <div>${r.category} <small style="color:#888">[${r.account}]</small></div>
          <div class="record-note">${r.time} ${r.note ? '· ' + r.note : ''}</div>
        </div>
      </div>
      <div style="font-weight:700;color:${r.type==='收入'?'#4cd964':'#e85a4f'}">
        ${r.type==='收入'?'+':'-'}¥${r.amount}
      </div>
    </div>
  `).join('');
}

function exportCSV() {
  const records = JSON.parse(localStorage.getItem('bills') || '[]');
  if (!records.length) { alert('暂无数据可导出'); return; }
  let csv = '\uFEFF时间,类型,分类,金额,账户,报销,备注\n';
  records.forEach(r => {
    csv += `"${r.time}","${r.type}","${r.category}",${r.amount},"${r.account}","${r.reimburse?'是':'否'}","${r.note}"\n`;
  });
  const blob = new Blob([csv], { type: 'text/csv;charset=utf-8;' });
  const a = document.createElement('a');
  a.href = URL.createObjectURL(blob);
  a.download = `账单明细_${new Date().toISOString().slice(0,10)}.csv`;
  a.click();
}

init();
</script>
</body>
</html>
