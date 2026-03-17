<script setup>
import { ref } from "vue";

const displayValue = ref("0");
const firstOperand = ref(null);
const currentOperator = ref(null);
const waitingForSecondOperand = ref(false);
const expressionValue = ref("");

function getOperatorSymbol(operator) {
  switch (operator) {
    case "/":
      return "÷";
    case "*":
      return "×";
    case "-":
      return "−";
    case "+":
      return "+";
    default:
      return "";
  }
}

function updateExpressionWithSecondOperand() {
  if (currentOperator.value && firstOperand.value !== null && !waitingForSecondOperand.value) {
    expressionValue.value = `${firstOperand.value} ${getOperatorSymbol(currentOperator.value)} ${displayValue.value}`;
  }
}

function formatResult(value) {
  if (!Number.isFinite(value)) {
    return "Error";
  }
  const rounded = Math.round((value + Number.EPSILON) * 1e12) / 1e12;
  return String(rounded);
}

function resetIfError() {
  if (displayValue.value === "Error") {
    clearAll();
  }
}

function inputDigit(digit) {
  resetIfError();
  if (waitingForSecondOperand.value) {
    displayValue.value = digit;
    waitingForSecondOperand.value = false;
    updateExpressionWithSecondOperand();
    return;
  }
  displayValue.value = displayValue.value === "0" ? digit : `${displayValue.value}${digit}`;
  updateExpressionWithSecondOperand();
}

function inputDot() {
  resetIfError();
  if (waitingForSecondOperand.value) {
    displayValue.value = "0.";
    waitingForSecondOperand.value = false;
    updateExpressionWithSecondOperand();
    return;
  }
  if (!displayValue.value.includes(".")) {
    displayValue.value += ".";
    updateExpressionWithSecondOperand();
  }
}

function clearAll() {
  displayValue.value = "0";
  firstOperand.value = null;
  currentOperator.value = null;
  waitingForSecondOperand.value = false;
  expressionValue.value = "";
}

function deleteOne() {
  resetIfError();
  if (waitingForSecondOperand.value) {
    return;
  }
  if (displayValue.value.length <= 1) {
    displayValue.value = "0";
    return;
  }
  displayValue.value = displayValue.value.slice(0, -1);
  updateExpressionWithSecondOperand();
}

function operate(a, b, operator) {
  switch (operator) {
    case "+":
      return a + b;
    case "-":
      return a - b;
    case "*":
      return a * b;
    case "/":
      return b === 0 ? Infinity : a / b;
    default:
      return b;
  }
}

function chooseOperator(nextOperator) {
  resetIfError();
  const inputValue = Number(displayValue.value);

  if (firstOperand.value === null) {
    firstOperand.value = inputValue;
  } else if (currentOperator.value && !waitingForSecondOperand.value) {
    const result = operate(firstOperand.value, inputValue, currentOperator.value);
    const formatted = formatResult(result);
    displayValue.value = formatted;
    firstOperand.value = formatted === "Error" ? null : Number(formatted);
  }

  if (firstOperand.value !== null) {
    expressionValue.value = `${firstOperand.value} ${getOperatorSymbol(nextOperator)}`;
  }
  currentOperator.value = nextOperator;
  waitingForSecondOperand.value = true;
}

function calculate() {
  if (!currentOperator.value || firstOperand.value === null) {
    return;
  }

  const secondOperand = Number(displayValue.value);
  const operatorSymbol = getOperatorSymbol(currentOperator.value);
  expressionValue.value = `${firstOperand.value} ${operatorSymbol} ${displayValue.value} =`;
  const result = operate(firstOperand.value, secondOperand, currentOperator.value);
  const formatted = formatResult(result);

  displayValue.value = formatted;
  firstOperand.value = formatted === "Error" ? null : Number(formatted);
  currentOperator.value = null;
  waitingForSecondOperand.value = false;
}
</script>

<template>
  <main class="app">
    <section class="calculator">
      <h1>计算器</h1>
      <div class="expression">{{ expressionValue || "\u00A0" }}</div>
      <div class="display">{{ displayValue }}</div>

      <div class="keypad">
        <button class="key function" @click="clearAll">C</button>
        <button class="key function" @click="deleteOne">⌫</button>
        <button class="key operator" @click="chooseOperator('/')">÷</button>
        <button class="key operator" @click="chooseOperator('*')">×</button>

        <button class="key" @click="inputDigit('7')">7</button>
        <button class="key" @click="inputDigit('8')">8</button>
        <button class="key" @click="inputDigit('9')">9</button>
        <button class="key operator" @click="chooseOperator('-')">−</button>

        <button class="key" @click="inputDigit('4')">4</button>
        <button class="key" @click="inputDigit('5')">5</button>
        <button class="key" @click="inputDigit('6')">6</button>
        <button class="key operator" @click="chooseOperator('+')">+</button>

        <button class="key" @click="inputDigit('1')">1</button>
        <button class="key" @click="inputDigit('2')">2</button>
        <button class="key" @click="inputDigit('3')">3</button>
        <button class="key equal" @click="calculate">=</button>

        <button class="key zero" @click="inputDigit('0')">0</button>
        <button class="key" @click="inputDot">.</button>
      </div>
    </section>

    <a class="legacy-link" href="/legacy.html">查看改动前页面</a>
  </main>
</template>

<style scoped>
.app {
  min-height: 100vh;
  display: grid;
  place-content: center;
  gap: 14px;
  font-family: Arial, sans-serif;
  color: #1f2937;
}

.calculator {
  width: min(92vw, 340px);
  background: #ffffff;
  border-radius: 16px;
  padding: 16px;
  box-shadow: 0 8px 24px rgba(15, 23, 42, 0.12);
}

h1 {
  margin: 0 0 10px;
  text-align: center;
  font-size: 20px;
}

.display {
  min-height: 56px;
  margin-bottom: 12px;
  border-radius: 10px;
  background: #f1f5f9;
  padding: 12px;
  text-align: right;
  font-size: 28px;
  line-height: 1.2;
  overflow: hidden;
  text-overflow: ellipsis;
}

.expression {
  min-height: 22px;
  margin-bottom: 6px;
  text-align: right;
  color: #64748b;
  font-size: 14px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.keypad {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 8px;
}

.key {
  border: none;
  border-radius: 10px;
  height: 48px;
  font-size: 18px;
  cursor: pointer;
  background: #e2e8f0;
  color: #0f172a;
}

.key:hover {
  filter: brightness(0.95);
}

.function {
  background: #cbd5e1;
}

.operator {
  background: #93c5fd;
}

.equal {
  background: #2563eb;
  color: #ffffff;
  grid-row: span 2;
  height: auto;
}

.zero {
  grid-column: span 2;
}

.legacy-link {
  display: inline-block;
  text-align: center;
  padding: 8px 14px;
  border-radius: 8px;
  text-decoration: none;
  color: #ffffff;
  background: #2563eb;
}

.legacy-link:hover {
  background: #1d4ed8;
}
</style>
