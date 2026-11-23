
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Calculator</title>
  <style>
    * {
      box-sizing: border-box;
      font-family: system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
    }

    body {
      margin: 0;
      min-height: 100vh;
      display: grid;
      place-items: center;
      background: #121212;
      padding: 16px;
    }

    .calculator {
      width: 100%;
      max-width: 340px;
      background: #1e1e1e;
      border-radius: 20px;
      padding: 20px;
      box-shadow: 0 20px 50px rgba(0, 0, 0, 0.5);
    }

    .display {
      background: #121212;
      color: #ffffff;
      border-radius: 14px;
      padding: 16px;
      margin-bottom: 16px;
      text-align: right;
      overflow: hidden;
    }

    .expression {
      font-size: 0.95rem;
      color: #9e9e9e;
      min-height: 1.3rem;
      word-wrap: break-word;
    }

    .result {
      font-size: 2.2rem;
      font-weight: 600;
      margin-top: 4px;
      word-wrap: break-word;
    }

    .buttons {
      display: grid;
      grid-template-columns: repeat(4, 1fr);
      gap: 12px;
    }

    button {
      height: 58px;
      border: none;
      border-radius: 14px;
      font-size: 1.2rem;
      font-weight: 500;
      cursor: pointer;
      background: #2c2c2c;
      color: #ffffff;
      transition: transform 0.08s ease, background 0.15s ease;
    }

    button:hover {
      background: #383838;
    }

    button:active {
      transform: scale(0.96);
    }

    button.operator {
      background: #ff9f0a;
      color: #121212;
      font-weight: 600;
    }

    button.operator:hover {
      background: #ffb038;
    }

    button.utility {
      background: #3a3a3c;
      color: #ffffff;
    }

    button.utility:hover {
      background: #48484a;
    }

    button.equals {
      background: #30d158;
      color: #121212;
      font-weight: 700;
    }

    button.equals:hover {
      background: #4be070;
    }

    button.zero {
      grid-column: span 2;
    }
  </style>
</head>
<body>
  <main class="calculator">
    <div class="display">
      <div class="expression" id="expression"></div>
      <div class="result" id="result">0</div>
    </div>

    <div class="buttons">
      <button class="utility" data-action="clear">C</button>
      <button class="utility" data-action="backspace">⌫</button>
      <button class="utility" data-value="%">%</button>
      <button class="operator" data-value="/">÷</button>

      <button data-value="7">7</button>
      <button data-value="8">8</button>
      <button data-value="9">9</button>
      <button class="operator" data-value="*">×</button>

      <button data-value="4">4</button>
      <button data-value="5">5</button>
      <button data-value="6">6</button>
      <button class="operator" data-value="-">−</button>

      <button data-value="1">1</button>
      <button data-value="2">2</button>
      <button data-value="3">3</button>
      <button class="operator" data-value="+">+</button>

      <button class="zero" data-value="0">0</button>
      <button data-value=".">.</button>
      <button class="equals" data-action="calculate">=</button>
    </div>
  </main>

  <script>
    const expressionEl = document.getElementById("expression");
    const resultEl = document.getElementById("result");
    const buttons = document.querySelectorAll("button");

    let currentExpr = "";

    function formatDisplay(expr) {
      return expr
        .replace(/\*/g, " × ")
        .replace(/\//g, " ÷ ")
        .replace(/\+/g, " + ")
        .replace(/-/g, " − ");
    }

    function evaluateExpression(expr) {
      const sanitized = expr.replace(/%/g, "/100");
      if (!/^[0-9+\-*/.() ]+$/.test(sanitized)) {
        throw new Error("Invalid input");
      }
      const value = Function(`"use strict"; return (${sanitized})`)();
      if (!Number.isFinite(value)) {
        throw new Error("Math error");
      }
      return Number(value.toPrecision(12));
    }

    function updateUI(previewValue = null) {
      expressionEl.textContent = formatDisplay(currentExpr);
      if (previewValue !== null) {
        resultEl.textContent = previewValue;
        return;
      }
      if (!currentExpr) {
        resultEl.textContent = "0";
        return;
      }
      try {
        const lastChar = currentExpr[currentExpr.length - 1];
        if (!["+", "-", "*", "/", "."].includes(lastChar)) {
          resultEl.textContent = evaluateExpression(currentExpr);
        }
      } catch {
        // Keep last valid display while typing
      }
    }

    function appendValue(val) {
      const operators = ["+", "-", "*", "/"];
      const lastChar = currentExpr[currentExpr.length - 1];

      if (operators.includes(val)) {
        if (!currentExpr && val !== "-") return;
        if (operators.includes(lastChar)) {
          currentExpr = currentExpr.slice(0, -1) + val;
          updateUI();
          return;
        }
      }

      if (val === ".") {
        const parts = currentExpr.split(/[+\-*/]/);
        const currentNumber = parts[parts.length - 1];
        if (currentNumber.includes(".")) return;
      }

      currentExpr += val;
      updateUI();
    }

    function clearAll() {
      currentExpr = "";
      updateUI("0");
    }

    function backspace() {
      currentExpr = currentExpr.slice(0, -1);
      updateUI(currentExpr ? undefined : "0");
    }

    function calculate() {
      if (!currentExpr) return;
      try {
        const ans = evaluateExpression(currentExpr);
        expressionEl.textContent = formatDisplay(currentExpr) + " =";
        resultEl.textContent = ans;
        currentExpr = String(ans);
      } catch {
        resultEl.textContent = "Error";
        currentExpr = "";
      }
    }

    buttons.forEach((btn) => {
      btn.addEventListener("click", () => {
        const action = btn.dataset.action;
        const value = btn.dataset.value;

        if (action === "clear") clearAll();
        else if (action === "backspace") backspace();
        else if (action === "calculate") calculate();
        else if (value) appendValue(value);
      });
    });

    document.addEventListener("keydown", (e) => {
      if ((e.key >= "0" && e.key <= "9") || ["+", "-", "*", "/", ".", "%"].includes(e.key)) {
        appendValue(e.key);
      } else if (e.key === "Enter" || e.key === "=") {
        e.preventDefault();
        calculate();
      } else if (e.key === "Backspace") {
        backspace();
      } else if (e.key === "Escape") {
        clearAll();
      }
    });
  </script>
</body>
</html>
