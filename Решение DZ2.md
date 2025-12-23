'use strict';

// ❗ импортируемые функции НЕ типизируем по условию
const makeOrdinal = require('./makeOrdinal');
const isFinite = require('./isFinite');
const isSafeNumber = require('./isSafeNumber');

// ======================
// ЧИСЛОВЫЕ КОНСТАНТЫ
// ======================
const TEN: number = 10;
const ONE_HUNDRED: number = 100;
const ONE_THOUSAND: number = 1_000;
const ONE_MILLION: number = 1_000_000;
const ONE_BILLION: number = 1_000_000_000;
const ONE_TRILLION: number = 1_000_000_000_000;
const ONE_QUADRILLION: number = 1_000_000_000_000_000;
const MAX: number = 9_007_199_254_740_992;

// ======================
// СЛОВАРИ
// ======================
const LESS_THAN_TWENTY: string[] = [
  'zero', 'one', 'two', 'three', 'four', 'five',
  'six', 'seven', 'eight', 'nine', 'ten',
  'eleven', 'twelve', 'thirteen', 'fourteen',
  'fifteen', 'sixteen', 'seventeen', 'eighteen', 'nineteen'
];

const TENTHS_LESS_THAN_HUNDRED: string[] = [
  'zero', 'ten', 'twenty', 'thirty', 'forty',
  'fifty', 'sixty', 'seventy', 'eighty', 'ninety'
];

// ======================
// ОСНОВНАЯ ФУНКЦИЯ
// ======================
function toWords(
  number: number | string,
  asOrdinal?: boolean
): string {
  const num: number = parseInt(number as string, 10);

  if (!isFinite(num)) {
    throw new TypeError(
      'Not a finite number: ' + number + ' (' + typeof number + ')'
    );
  }

  if (!isSafeNumber(num)) {
    throw new RangeError(
      'Input is not a safe number, it’s either too large or too small.'
    );
  }

  const words: string = generateWords(num);
  return asOrdinal ? makeOrdinal(words) : words;
}

// ======================
// РЕКУРСИВНАЯ ФУНКЦИЯ
// ======================
function generateWords(
  number: number,
  words?: string[]
): string {
  let remainder: number;
  let word: string;

  // Завершение рекурсии
  if (number === 0) {
    return !words ? 'zero' : words.join(' ').replace(/,$/, '');
  }

  // Первый запуск
  if (!words) {
    words = [];
  }

  // Отрицательные числа
  if (number < 0) {
    words.push('minus');
    number = Math.abs(number);
  }

  if (number < 20) {
    remainder = 0;
    word = LESS_THAN_TWENTY[number];

  } else if (number < ONE_HUNDRED) {
    remainder = number % TEN;
    word = TENTHS_LESS_THAN_HUNDRED[Math.floor(number / TEN)];

    if (remainder) {
      word += '-' + LESS_THAN_TWENTY[remainder];
      remainder = 0;
    }

  } else if (number < ONE_THOUSAND) {
    remainder = number % ONE_HUNDRED;
    word = generateWords(Math.floor(number / ONE_HUNDRED)) + ' hundred';

  } else if (number < ONE_MILLION) {
    remainder = number % ONE_THOUSAND;
    word = generateWords(Math.floor(number / ONE_THOUSAND)) + ' thousand,';

  } else if (number < ONE_BILLION) {
    remainder = number % ONE_MILLION;
    word = generateWords(Math.floor(number / ONE_MILLION)) + ' million,';

  } else if (number < ONE_TRILLION) {
    remainder = number % ONE_BILLION;
    word = generateWords(Math.floor(number / ONE_BILLION)) + ' billion,';

  } else if (number < ONE_QUADRILLION) {
    remainder = number % ONE_TRILLION;
    word = generateWords(Math.floor(number / ONE_TRILLION)) + ' trillion,';

  } else {
    remainder = number % ONE_QUADRILLION;
    word =
      generateWords(Math.floor(number / ONE_QUADRILLION)) +
      ' quadrillion,';
  }

  words.push(word);
  return generateWords(remainder, words);
}

export = toWords;
