<template>
  <div>
    <h2>Simple Expression Parser</h2>
    <input v-model="inputExpression" placeholder="Введите выражение" />
    <button @click="parseInputExpression">Parse</button>
    <p v-if="error" style="color: red">{{ error }}</p>
    <p v-if="parsedResult">{{ parsedResult }}</p>
  </div>
</template>

<script setup lang="ts">
import { ref } from "vue";

// Входные данные и состояния
const inputExpression = ref<string>("");
const parsedResult = ref<string>("");
const error = ref<string>("");

// Типы токенов
enum TokenType {
  DIGIT = "DIGIT",
  IDENTIFIER = "IDENTIFIER",
  OPERATOR = "OPERATOR",
  LPAREN = "LPAREN", // (
  RPAREN = "RPAREN", // )
  MULTIPLY = "MULTIPLY", // *
  END = "END", // конец выражения
}

interface Token {
  type: TokenType;
  value: string;
}

// Функция для токенизации (лексический анализ)
const tokenize = (input: string): Token[] => {
  const tokens: Token[] = [];
  let i = 0;

  while (i < input.length) {
    const char = input[i];

    if (/\d/.test(char)) {
      // Числа
      let num = "";
      while (i < input.length && /\d/.test(input[i])) {
        num += input[i];
        i++;
      }
      tokens.push({ type: TokenType.DIGIT, value: num });
    } else if (/[a-zA-Z]/.test(char)) {
      // Идентификаторы
      let id = "";
      while (i < input.length && /[a-zA-Z]/.test(input[i])) {
        id += input[i];
        i++;
      }
      tokens.push({ type: TokenType.IDENTIFIER, value: id });
    } else if (char === "+" || char === "-") {
      // Операторы + или -
      tokens.push({ type: TokenType.OPERATOR, value: char });
      i++;
    } else if (char === "*") {
      // Оператор *
      tokens.push({ type: TokenType.MULTIPLY, value: char });
      i++;
    } else if (char === "(") {
      tokens.push({ type: TokenType.LPAREN, value: char });
      i++;
    } else if (char === ")") {
      tokens.push({ type: TokenType.RPAREN, value: char });
      i++;
    } else if (char === " ") {
      // Пропускаем пробелы
      i++;
    } else {
      throw new Error(`Неподдерживаемый символ: ${char}`);
    }
  }
  tokens.push({ type: TokenType.END, value: "" });
  return tokens;
};

// Переменная для хранения текущего токена
let currentToken: Token | null = null;
let tokens: Token[] = [];
let pos = 0;

// Функция для перемещения к следующему токену
const nextToken = () => {
  if (pos < tokens.length) {
    currentToken = tokens[pos];
    pos++;
  } else {
    currentToken = null;
  }
};

// Функция для проверки текущего токена
const expect = (type: TokenType) => {
  if (currentToken?.type === type) {
    nextToken();
  } else {
    throw new Error(`Ожидался ${type}, но получен ${currentToken?.type}`);
  }
};

// Анализатор простого выражения
const parseExpression = (): string => {
  let result = parseTerm();

  while (currentToken?.type === TokenType.OPERATOR) {
    const operator = currentToken.value;
    nextToken();
    const nextTerm = parseTerm();
    result += ` ${operator} ${nextTerm}`;
  }
  return result;
};

// Анализ терма
const parseTerm = (): string => {
  let result = parseFactor();

  while (currentToken?.type === TokenType.MULTIPLY) {
    nextToken();
    const nextFactor = parseFactor();
    result += ` * ${nextFactor}`;
  }
  return result;
};

// Анализ множителя (фактор)
const parseFactor = (): string => {
  if (currentToken?.type === TokenType.DIGIT) {
    const value = currentToken.value;
    nextToken();
    return value;
  } else if (currentToken?.type === TokenType.IDENTIFIER) {
    const value = currentToken.value;
    nextToken();
    return value;
  } else if (currentToken?.type === TokenType.LPAREN) {
    nextToken();
    const expr = parseExpression();
    expect(TokenType.RPAREN);
    return `(${expr})`;
  } else {
    throw new Error("Неожиданный токен");
  }
};

// Главная функция для обработки выражения
const parseInputExpression = () => {
  error.value = "";
  parsedResult.value = "";

  // Проверка на пустое выражение
  if (inputExpression.value.trim() === "") {
    error.value = "Введите выражение!";
    return;
  }

  try {
    tokens = tokenize(inputExpression.value);
    pos = 0;
    nextToken();
    parsedResult.value = parseExpression();

    // Дополнительная проверка на конец выражения
    if (currentToken?.type !== TokenType.END) {
      throw new Error(
        "Ожидался конец выражения, но получен дополнительный токен."
      );
    }
  } catch (e) {
    error.value = (e as Error).message;
  }
};
</script>

<style scoped>
input {
  padding: 10px;
  margin: 10px 0;
  font-size: 16px;
  width: 300px;
}

button {
  padding: 10px 20px;
  background-color: #4caf50;
  color: white;
  border: none;
  cursor: pointer;
  font-size: 16px;
}

button:hover {
  background-color: #45a049;
}

p {
  font-size: 18px;
}
</style>
  