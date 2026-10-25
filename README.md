# project-

<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Calculator</title>
<style>
  :root {
    --bg: #e9ecef;
    --body: #2b2f36;
    --screen: #c9d6c3;
    --screen-ink: #1f2a1c;
    --key: #f6f7f9;
    --key-ink: #22262c;
    --op: #3d6fd8;
    --op-ink: #ffffff;
    --fn: #d5d9df;
    --eq: #e0702b;
  }
  @media (prefers-color-scheme: dark) {
    :root {
      --bg: #14161a;
      --body: #23272e;
      --screen: #9fb39a;
      --screen-ink: #14200f;
      --key: #343a43;
      --key-ink: #f1f3f5;
      --fn: #444b56;
    }
  }
  * { box-sizing: border-box; }
  html, body { height: 100%; margin: 0; }
  body {
    background: var(--bg);
    display: grid;
    place-items: center;
    font-family: "Segoe UI", system-ui, -apple-system, Roboto, sans-serif;
  }
  .calc {
    width: min(92vw, 340px);
    background: var(--body);
    border-radius: 22px;
    padding: 18px;
    box-shadow: 0 18px 40px rgba(0,0,0,.25);
  }
  .screen {
    background: var(--screen);
    color: var(--screen-ink);
    border-radius: 12px;
    padding: 12px 14px;
    margin-bottom: 16px;
    text-align: right;
    min-height: 96px;
    display: flex;
    flex-direction: column;
    justify-content: flex-end;
    overflow: hidden;
  }
  .history { font-size: 14px; opacity: .65; min-height: 20px; word-break: break-all; }
  .current { font-size: 38px; font-weight: 600; line-height: 1.15; word-break: break-all; }
  .keys { display: grid; grid-template-columns: repeat(4, 1fr); gap: 10px; }
  button {
    font: inherit;
    font-size: 20px;
    font-weight: 600;
    border: 0;
    border-radius: 12px;
    padding: 16px 0;
    cursor: pointer;
    background: var(--key);
    color: var(--key-ink);
    box-shadow: 0 2px 0 rgba(0,0,0,.25);
    transition: transform .06s ease, filter .12s ease;
  }
  button:hover { filter: brightness(.96); }
  button:active { transform: translateY(2px); box-shadow: none; }
  button:focus-visible { outline: 3px solid var(--op); outline-offset: 2px; }
  .fn { background: var(--fn); }
  .op { background: var(--op); color: var(--op-ink); }
  .eq { background: var(--eq); color: #fff; grid-row: span 2; }
  .zero { grid-column: span 2; }
</style>
</head>
<body>
  <main class="calc" aria-label="Calculator">
    <div class="screen" aria-live="polite">
      <div class="history" id="history"></div>
      <div class="current" id="current">0</div>
    </div>
    <div class="keys" id="keys">
      <button class="fn" data-action="clear">AC</button>
      <button class="fn" data-action="delete" aria-label="Delete">⌫</button>
      <button class="fn" data-action="percent">%</button>
      <button class="op" data-op="÷">÷</button>

      <button data-num="7">7</button>
      <button data-num="8">8</button>
      <button data-num="9">9</button>
      <button class="op" data-op="×">×</button>

      <button data-num="4">4</button>
      <button data-num="5">5</button>
      <button data-num="6">6</button>
      <button class="op" data-op="−">−</button>

      <button data-num="1">1</button>
      <button data-num="2">2</button>
      <button data-num="3">3</button>
      <button class="op" data-op="+">+</button>

      <button class="zero" data-num="0">0</button>
      <button data-action="decimal">.</button>
      <button class="eq" data-action="equals">=</button>
    </div>
  </main>

<script>
  const currentEl = document.getElementById('current');
  const historyEl = document.getElementById('history');

  let current = '0';
  let previous = null;
  let operator = null;
  let justEvaluated = false;

  function format(value) {
    if (value === 'Error') return value;
    const [int, dec] = value.split('.');
    const sign = int.startsWith('-') ? '-' : '';
    const digits = int.replace('-', '');
    const grouped = digits.replace(/\B(?=(\d{3})+(?!\d))/g, ',');
    return sign + grouped + (dec !== undefined ? '.' + dec : '');
  }

  function render() {
    currentEl.textContent = format(current);
    historyEl.textContent = operator ? format(previous) + ' ' + operator : '';
    currentEl.style.fontSize = current.length > 12 ? '24px' : current.length > 8 ? '30px' : '38px';
  }

  function inputNumber(n) {
    if (current === 'Error' || justEvaluated) { current = '0'; justEvaluated = false; }
    if (current.replace(/[-.]/g, '').length >= 15) return;
    current = current === '0' ? n : current + n;
  }

  function inputDecimal() {
    if (current === 'Error' || justEvaluated) { current = '0'; justEvaluated = false; }
    if (!current.includes('.')) current += '.';
  }

  function compute(a, b, op) {
    a = parseFloat(a); b = parseFloat(b);
    let r;
    switch (op) {
      case '+': r = a + b; break;
      case '−': r = a - b; break;
      case '×': r = a * b; break;
      case '÷': if (b === 0) return 'Error'; r = a / b; break;
    }
    return String(parseFloat(r.toPrecision(12)));
  }

  function chooseOperator(op) {
    if (current === 'Error') return;
    if (operator && !justEvaluated) {
      current = compute(previous, current, operator);
      if (current === 'Error') { operator = null; previous = null; return; }
    }
    previous = current;
    operator = op;
    justEvaluated = false;
    current = '0';
  }

  function equals() {
    if (!operator || current === 'Error') return;
    const result = compute(previous, current, operator);
    historyEl.textContent = format(previous) + ' ' + operator + ' ' + format(current) + ' =';
    current = result;
    previous = null;
    operator = null;
    justEvaluated = true;
    currentEl.textContent = format(current);
    currentEl.style.fontSize = current.length > 12 ? '24px' : current.length > 8 ? '30px' : '38px';
  }

  function clearAll() { current = '0'; previous = null; operator = null; justEvaluated = false; }

  function del() {
    if (current === 'Error' || justEvaluated) { clearAll(); return; }
    current = current.length > 1 && !(current.length === 2 && current.startsWith('-'))
      ? current.slice(0, -1) : '0';
  }

  function percent() {
    if (current === 'Error') return;
    current = String(parseFloat((parseFloat(current) / 100).toPrecision(12)));
  }

  function handle(kind, value) {
    if (kind === 'num') inputNumber(value);
    else if (kind === 'op') chooseOperator(value);
    else if (value === 'decimal') inputDecimal();
    else if (value === 'equals') { equals(); return; }
    else if (value === 'clear') clearAll();
    else if (value === 'delete') del();
    else if (value === 'percent') percent();
    render();
  }

  document.getElementById('keys').addEventListener('click', e => {
    const b = e.target.closest('button');
    if (!b) return;
    if (b.dataset.num !== undefined) handle('num', b.dataset.num);
    else if (b.dataset.op) handle('op', b.dataset.op);
    else handle('action', b.dataset.action);
  });

  document.addEventListener('keydown', e => {
    const k = e.key;
    if (/^[0-9]$/.test(k)) handle('num', k);
    else if (k === '.') handle('action', 'decimal');
    else if (k === '+') handle('op', '+');
    else if (k === '-') handle('op', '−');
    else if (k === '*' || k === 'x') handle('op', '×');
    else if (k === '/') { e.preventDefault(); handle('op', '÷'); }
    else if (k === '%') handle('action', 'percent');
    else if (k === 'Enter' || k === '=') { e.preventDefault(); handle('action', 'equals'); }
    else if (k === 'Backspace') handle('action', 'delete');
    else if (k === 'Escape' || k === 'Delete') handle('action', 'clear');
  });

  render();
</script>
</body>
</html>
