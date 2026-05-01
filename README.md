# @catbee/cron-parser

## ⏱️ Cron expression parser for Node.js and TypeScript

A lightweight, timezone-aware cron expression parser with full support for seconds, DST transitions, iterators, randomized scheduling, and crontab file parsing. Built for production systems with clean TypeScript types and zero-config ESM/CJS support.

<div style="display: flex; flex-wrap: wrap; gap: 0.5rem; margin: 1rem 0;">
  <img src="https://github.com/catbee-technologies/cron-parser/actions/workflows/ci.yml/badge.svg" alt="Build Status" />
  <img src="https://codecov.io/gh/catbee-technologies/cron-parser/graph/badge.svg" alt="Coverage" />
  <img src="https://img.shields.io/node/v/@catbee/cron-parser" alt="Node Version" />
  <img src="https://img.shields.io/npm/v/@catbee/cron-parser" alt="NPM Version" />
  <img src="https://img.shields.io/npm/dt/@catbee/cron-parser" alt="NPM Downloads" />
  <img src="https://img.shields.io/npm/types/@catbee/cron-parser" alt="TypeScript Types" />
  <img src="https://img.shields.io/npm/l/@catbee/cron-parser" alt="License" />
</div>

---

## Installation

```bash
npm install @catbee/cron-parser
```

---

## Cron Format

```
*    *    *    *    *    *
┬    ┬    ┬    ┬    ┬    ┬
│    │    │    │    │    │
│    │    │    │    │    └─ day of week (0-7, 1L-7L) (0 or 7 is Sun)
│    │    │    │    └────── month (1-12, JAN-DEC)
│    │    │    └─────────── day of month (1-31, L)
│    │    └──────────────── hour (0-23)
│    └───────────────────── minute (0-59)
└────────────────────────── second (0-59, optional)
```

---

## Special Characters

| Character | Description               | Example                                                                |
| --------- | ------------------------- | ---------------------------------------------------------------------- |
| `*`       | Any value                 | `* * * * *` (every minute)                                             |
| `?`       | Any value (alias for `*`) | `? * * * *` (every minute)                                             |
| `,`       | Value list separator      | `1,2,3 * * * *` (1st, 2nd, and 3rd minute)                             |
| `-`       | Range of values           | `1-5 * * * *` (every minute from 1 through 5)                          |
| `/`       | Step values               | `*/5 * * * *` (every 5th minute)                                       |
| `L`       | Last day of month/week    | `0 0 L * *` (midnight on last day of month)                            |
| `#`       | Nth day of month          | `0 0 * * 1#1` (first Monday of month)                                  |
| `H`       | Randomized value          | `H * * * *` (random minute every hour)                                 |

---

## Predefined Expressions

| Expression  | Description                               | Equivalent      |
| ----------- | ----------------------------------------- | --------------- |
| `@yearly`   | Once a year at midnight of January 1      | `0 0 0 1 1 *`   |
| `@monthly`  | Once a month at midnight of first day     | `0 0 0 1 * *`   |
| `@weekly`   | Once a week at midnight on Sunday         | `0 0 0 * * 0`   |
| `@daily`    | Once a day at midnight                    | `0 0 0 * * *`   |
| `@hourly`   | Once an hour at the beginning of the hour | `0 0 * * * *`   |
| `@minutely` | Once a minute                             | `0 * * * * *`   |
| `@secondly` | Once a second                             | `* * * * * *`   |
| `@weekdays` | Every weekday at midnight                 | `0 0 0 * * 1-5` |
| `@weekends` | Every weekend at midnight                 | `0 0 0 * * 0,6` |

---

## Field Values

| Field        | Values | Special Characters              | Aliases                        |
| ------------ | ------ | ------------------------------- | ------------------------------ |
| second       | 0-59   | `*` `?` `,` `-` `/` `H`         |                                |
| minute       | 0-59   | `*` `?` `,` `-` `/` `H`         |                                |
| hour         | 0-23   | `*` `?` `,` `-` `/` `H`         |                                |
| day of month | 1-31   | `*` `?` `,` `-` `/` `H` `L`     |                                |
| month        | 1-12   | `*` `?` `,` `-` `/` `H`         | `JAN`-`DEC`                    |
| day of week  | 0-7    | `*` `?` `,` `-` `/` `H` `L` `#` | `SUN`-`SAT` (0 or 7 is Sunday) |

---

## Options

| Option      | Type                     | Description                                                                                                                 |
| ----------- | ------------------------ | --------------------------------------------------------------------------------------------------------------------------- |
| currentDate | Date \| string \| number | Current date. Defaults to current local time in UTC. If not provided but `startDate` is set, `startDate` is used instead. |
| endDate     | Date \| string \| number | End date of iteration range. Sets the iteration range end point                                                             |
| startDate   | Date \| string \| number | Start date of iteration range. Sets the iteration range start point                                                          |
| tz          | string                   | Timezone (e.g. `Europe/London`)                                                                                            |
| hashSeed    | string                   | Seed used for `H` randomized field values                                                                                   |
| strict      | boolean                  | Enable strict validation for expression format and day-of-month/day-of-week rules                                            |

Supported string date formats:

- ISO8601
- HTTP and RFC2822
- SQL

---

## ⚡ Quick Start

```ts
import CronExpressionParser, {
  CronExpressionOptions,
  CronFileParser,
  CronFileParserResult
} from '@catbee/cron-parser';

const options: CronExpressionOptions = {
  currentDate: '2023-01-01T00:00:00Z',
  tz: 'UTC',
  strict: false
};

const expression = CronExpressionParser.parse('*/5 * * * *', options);
console.log('Next:', expression.next().toString());

const nextThree = expression.take(3);
console.log('Next 3:', nextThree.map(date => date.toString()));

const includesNow = expression.includesDate(new Date());
console.log('Includes now?', includesNow);
```

---

## Crontab File Operations

```ts
import { CronFileParser } from '@catbee/cron-parser';

const result = CronFileParser.parseFileSync('./crontab.txt');
console.log('Variables:', result.variables);
console.log('Expressions:', result.expressions.length);
console.log('Errors:', result.errors);
```

```ts
import { CronFileParser } from '@catbee/cron-parser';

const result = await CronFileParser.parseFile('/path/to/crontab');
console.log('Variables:', result.variables);
console.log('Expressions:', result.expressions.length);
console.log('Errors:', result.errors);
```

---

## Advanced Features

### Strict Mode

Strict mode enforces more explicit cron expressions by rejecting ambiguous forms.

- Prevents both `dayOfMonth` and `dayOfWeek` from being used together in strict mode
- Requires the full 6-field expression if strict mode is enabled
- Rejects empty expressions

```ts
import CronExpressionParser from '@catbee/cron-parser';

try {
  CronExpressionParser.parse('0 0 12 1-31 * 1', { strict: true });
} catch (err) {
  console.error(err.message);
}
```

### Last Day of Month / Week Support

The parser supports `L` in day-of-month and day-of-week fields to represent the last occurrence.

```ts
const lastMonday = CronExpressionParser.parse('0 0 0 * * 1L');
const lastDayOfMonth = CronExpressionParser.parse('0 0 L * *');
```

### Randomized Scheduling (`H`)

The `H` character produces deterministic jitter when used with the same `hashSeed`.

```ts
const interval = CronExpressionParser.parse('H * * * *', {
  currentDate: '2023-03-26T01:00:00Z',
  hashSeed: 'job-name'
});

console.log(interval.stringify());
```

### Timezone Support

The parser handles timezone-aware scheduling and DST transitions with Luxon.

```ts
const interval = CronExpressionParser.parse('0 * * * *', {
  currentDate: '2023-03-26T01:00:00',
  tz: 'Europe/London'
});

console.log(interval.next().toString());
console.log(interval.next().toString());
```

---

## 📚 Documentation

Full documentation and examples are available at:

https://catbee.in/docs/@catbee/cron-parser/intro

---

## 🤝 Credits

Based on the original [cron-parser](https://www.npmjs.com/package/cron-parser) library and extended under Catbee.

## Contributing

Contributions are welcome! Please see `CONTRIBUTING.md` for development setup, testing, and PR guidance.

---

## 📜 License

MIT © Catbee Technologies
