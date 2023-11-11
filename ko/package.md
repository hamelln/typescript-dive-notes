# 목표

> _패키지의 타입을 읽고, 작성하는 법을 이해합니다._

&nbsp;&nbsp;&nbsp;&nbsp;_Editor's note: 미정._

### 모듈

&nbsp;&nbsp;&nbsp;&nbsp;node_modules의 react폴더에는 `cjs` 폴더와 `umd` 폴더가 있습니다. 둘 다 react.development.js가 있는데 두 코드는 거의 다 똑같되 약간의 차이가 있습니다.

```javascript
/// umd/react.development.js
(function (global, factory) {
// CommonJS인지 확인(Node.js 등)
  typeof exports === 'object' && typeof module !== 'undefined' ? factory(exports) :
// Asynchronous Module Definition인지 확인(RequireJS 등)
  typeof define === 'function' && define.amd ? define(['exports'], factory) :
// 브라우저인지 확인
  (global = global || self, factory(global.React = {}));
}(this, (function (exports) { 'use strict';
```

- CommonJS(CJS)
Node.js에서 사용합니다. 모듈을 동기적으로 로드하며 주로 백엔드에서 사용합니다.

- Asynchronous Module Definition(AMD)
주로 RequireJS에서 사용합니다. 모듈을 비동기적으로 로드하며 주로 프론트엔드에서 사용합니다.

- ECMAScript Module(ESM)
일반적인 브라우저, 근래의 Node.js에서 사용합니다. 모듈을 비동기적으로 로드하고 import 시 트리 쉐이킹을 해 번들 크기를 줄입니다. CommonJS의 `require`는 동적으로 가져오고 내보낼 수 있어 런타임에만 평가가 가능하지만 import는 정적이기 때문에 런타임 전에 분석이 가능합니다. 

- Universal Module Definition(UMD)
CJS, AMD 등 여러 케이스에 대응합니다.  
똑같이 작성하더라도 CJS와 AMD는 작성법, 작동 방식, 효율이 다릅니다. 보통은 효율적인 방식을 먼저 시도해보고, 실패할 경우엔 UMD로 대응합니다. 크로스 브라우징, polyfill처럼 범용성 대비라고 생각하면 됩니다. react 패키지의 index를 보면 cjs로 먼저 접근합니다.

```javascript
if (process.env.NODE_ENV === 'production') {
  module.exports = require('./cjs/react.production.min.js');
} else {
  module.exports = require('./cjs/react.development.js');
}
```

### tsc와 tsconfig

tsc는 TypeScript Compile입니다. 터미널에 `npx tsc`를 입력하면 타입스크립트가 자바스크립트로 컴파일되니다. tsconfig에서 타입스크립트 컴파일에 대한 옵션을 지정할 수 있습니다.

```javascript
"compilerOptions": {
    "target": "es2016", // 컴파일된 JS가 실행되는 ECMAScript 버전.
    "module": "CommonJS", // 모듈 시스템 지정.
    "strict": true, // 모든 타입 체킹 옵션 활성화.
    "esModuleInterop": true, // ESM과의 호환성을 높이는 옵션. CommonJS 모듈을 ESM처럼 사용 가능합니다.
    "forceConsistentCasingInFileNames": true, // 파일명의 대소문자가 일치하지 않으면 오류를 발생시킵니다.
    "outDir": "dist", // 컴파일 결과물 저장 폴더. dist는 distribution(배포)의 줄임말입니다. 
    "declaration": true, // 컴파일 후 d.ts(타입 정보를 담은 파일)을 생성합니다. 
    "declarationDir": "types", // d.ts를 저장할 폴더.
    "rootDir": "." 
}
```

보통 dist폴더에 컴파일 결과를 보관합니다. 개발할 땐 타입스크립트를 쓰지만 브라우저는 타입스크립트를 실행할 수 없기 때문에 dist에 담긴 자바스크립트 코드를 사용합니다.

### 로컬 패키지

간단한 실습을 해봅니다. 아래와 같은 코드를 만들고 위 tsconfig의 컴파일 옵션을 만들고, package.json에서 컴파일 옵션을 지정합니다. 

```typescript
// src/add.ts
const add = (x: number, y: number): number => x + y;
export default add;

// index.ts
import add from "./src/add";
export { add };

// package.json
{
  "name": "add-ts",
  "version": "1.0.0",
  "main": "dist/index.js",
  "types": "types/index.d.ts"
}
```

`npx tsc`를 실행하면 dist 폴더와 types 폴더가 생성됩니다. `npm pack`을 실행하면 add-ts는 pgz 패키지 파일로 변환됩니다. 이제 `npm i ../add-ts.pgz`처럼 로컬 경로로 패키지 설치가 가능합니다.

```typescript
import { add } from "add-ts"; // 해당 패키지 types폴더의 index.d.ts에서 타입을 참조합니다. add-ts의 package.json에서 명시했기 때문입니다. 
add(1, 2); // 3

// add.d.ts
declare const add: (x: number, y: number) => number; // ctrl을 누르고 add를 클릭하면 컴파일로 만들어진 add.d.ts를 보여줍니다.
export default add;
```

## 마치며

> 

### 참조

- [장호승(2022.10). CommonJS와 ESM에 모두 대응하는 라이브러리 개발하기: exports field. toss tech](https://toss.tech/article/commonjs-esm-exports-field)
- [Igor Irianto(2019.07). What the heck are CJS, AMD, UMD, and ESM in Javascript?](https://dev.to/iggredible/what-the-heck-are-cjs-amd-umd-and-esm-ikm)
