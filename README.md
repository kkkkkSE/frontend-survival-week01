# frontend-survival-week01

## 실행하기

```terminal
npm test
npm start
```

<http://localhost:8080/> (script에서 Port 8080 설정)

## 개발환경 세팅하기

### 0. 세팅에 앞서

Node.js 버전 확인 (LTS 최신버전 권장)

```terminal
node -v
```

[fnm](https://github.com/Schniz/fnm) (Fast Node Manager)로 여러 버전의 Node.js를
사용하는 것도 가능하니, 필요 시 사용하자.

아래는 LTS 최신버전을 기본으로 사용하게 해주는 커맨드다.

```terminal
fnm install --lts
fnm use lts-latest
fnm default $(fnm current)
```

현재 버전 확인은 이렇게

```terminal
fnm list
```

그리고 프로젝트 폴더가 없다면, 생성하도록 하자.

```terminal
mkdir my-react-project
cd my-react-project
```

### 1. npm 패키지 준비

`package.json`을 생성하자. 참고로 `-y`는 모든 질문에 yes로 대답하는 옵션이다.

어차피 파일 생성 후에 변경해도 상관없으니 빠르게 `-y` 옵션을 써주자.

```terminal
npm init -y
```

### 2. `.gitignore` 생성

실수로라도 업로드할 필요 없는 파일을 업로드 하지 않기 위해 미리 만들어두자.

```json
/node_modules/
/dist/
/.parcel-cache/
```

일일이 적기 귀찮다면 *gitignore* 또는 *node gitignore* 검색해서 가져오도록 하자.

만드는 김에 `.eslintignore`도 같은 내용으로 만들어두자.

### 3. 사용할 개발 프로그램 및 도구 설치, 세팅

프로그램, 도구 설치 후 설치가 잘 되었는지 `package.json`에서 꼭 확인하자.

```json
  "devDependencies": {
    "개발 도구": "^버전"
  },
  "dependencies": {
    "개발 프로그램": "^버전",
  }
```

#### TypeScript

`TypeScript`를 설치하고, `tsconfig.json` 파일을 만들자.

```terminal
npm i -D typescript
npx tsc --init
```

JSX를 사용하려면 `tsconfig.json`에서 옵션을 추가해야 한다.

```json
{
  "compilerOptions": {
    // ...
    "jsx": "react-jsx",
    // ...
  }
}
```

#### ESLint

`ESLint`를 설치하고, `.eslintrc` 파일을 만들자.

```terminal
npm i -D eslint
npx eslint --init
```

`npx eslint --init` 실행 시 몇 가지 질문을 한다. (아래 질문들은 2022년 3월 기준이다.)

읽어보고 필요한 옵션을 고르도록 하자.

```terminal
✔ How would you like to use ESLint? · style
✔ What type of modules does your project use? · esm
✔ Which framework does your project use? · react
✔ Does your project use TypeScript? · Yes
✔ Where does your code run? · browser
✔ How would you like to define a style for your project? · guide
✔ Which style guide do you want to follow? · xo-typescript
✔ What format do you want your config file to be in? · JavaScript
✔ Would you like to install them now? · Yes
✔ Which package manager do you want to use? · npm
```

`.eslintrc`에서 추가 설정을 해주자.

Jest를 사용할 거라면 `jest : true`,
JSX를 사용할 거라면 `'plugin:react/jsx-runtime'`를 추가하자.

```js
env: {
    browser: true,
    es2021: true,
    jest: true,
},
extends: [
    'plugin:react/recommended',
    'plugin:react/jsx-runtime',
    'xo',
],
// ...
```

`.eslintrc`의 `rules`에서 더 많은 옵션을 추가할 수 있다. 필요한 규칙이 있다면 구글링해서 추가하자.

또한, 위에서 `.eslintignore`를 만들지 않았다면 지금이라도 생성하자.
`.gitignore`와 동일하게 작성하면 된다.

#### React와 React Type 정보 설치

```terminal
npm i react react-dom
npm i -D @types/react @types/react-dom
```

#### Jest 설치

설치할 패키지는 매번 동일하지 않을 수 있으니 주의하자.

```terminal
npm i -D jest @types/jest @swc/core @swc/jest \
     jest-environment-jsdom \
     @testing-library/react @testing-library/jest-dom
```

Jest 설정파일 `jest.config.js`는 수동으로 생성하여 작성하자.

```js
module.exports = {
    testEnvironment: 'jsdom',
    setupFilesAfterEnv: [
        '@testing-library/jest-dom/extend-expect',
    ],
    transform: {
        '^.+\\.(t|j)sx?$': ['@swc/jest', {
            jsc: {
                parser: {
                        syntax: 'typescript',
                        jsx: true,
                        decorators: true,
                    },
                    transform: {
                        react: {
                        runtime: 'automatic',
                        },
                    },
                },
        }],
    },
    testPathIgnorePatterns: [
        '<rootDir>/node_modules/',
        '<rootDir>/dist/',
    ],
};
```

#### Parcel 설치

```terminal
npm i -D parcel
```

`package.json`에 `"main"` 대신 `"source": "./index.html"` 추가하여 메인 페이지로
설정할 수 있다.

```json
{
  "name": "frontend-survival-week01",
  "version": "1.0.0",
  "description": "",
  "source": "./index.html",
  // ...
}
```

`parcel-reporter-static-files-copy` 설치하고,
정적 파일을 업로드 할 `static` 폴더를 root에 만들어주자.

```terminal
npm install -D parcel-reporter-static-files-copy
```

필요 시 Parcel 설정파일인 `.parcelrc`도 만들어주자.

### 4. 스크립트 설정

```json
{   
  // ...
  "scripts": {
    "start": "parcel --port 8080",
    "build": "parcel build",
    "check": "tsc --noEmit",
    "lint": "eslint --fix --ext .js,.jsx,.ts,.tsx .",
    "test": "jest",
    "coverage": "jest --coverage --coverage-reporters html",
    "watch:test": "jest --watchAll"
  },
  // ...
}
```

#### 스크립트 실행하기

npm에 기본적으로 설정된 `start`, `stop`, `restart`, `test` 외에 스크립트를 추가한 경우,
실행할 때 `run`을 붙여야 한다.

```terminal
npm start
npm build
npm run check
npm run lint
npm run test
npm run coverage
npm run watch:test
```

### 5. 애플리케이션 기본 파일 생성

#### 기본 파일 생성

- `index.html`

```html
<!DOCTYPE html>
<html lang="ko">
    <head>
        <meta charset="UTF-8">
        <title>My App</title>
    </head>
    <body>
        <div id="root"></div>
        <script type="module" src="./src/main.tsx"></script>
    </body>
</html>
```

- `src/main.tsx`

```ts
import ReactDOM from 'react-dom/client';

import App from './App';

function main() {
  const container = document.getElementById('root');
  if (!container) {
    return;
  }

  const root = ReactDOM.createRoot(container);
  root.render(<App />);
}

main();
```

- `src/App.tsx`

```ts
export default function App() {
    return (
        <p>Hello, world!</p>
    );
}
```

- `src/Greeting.tsx`

```ts
export default function Greeting() {
    return (
        <p>Hello, WORLD!</p>
    );
}
```

#### 테스트 기본 파일 작성

- `src/main.test.tsx`

```ts
test('test 설명', () => {
    expect(1).toBe(1);
});
```

- `src/App.test.tsx`

```ts
import {render} from '@testing-library/react';

import App from './App';

describe('App', () => {
  it('renders greeting message', () => {
    const {container} = render(<App />);

    expect(container).toHaveTextContent('Hello, world!');
  });
});
```

- `src/Greeting.test.tsx`

```ts
import {render} from '@testing-library/react';

import Greeting from './Greeting';

describe('App', () => {
    it('renders greeting message', () => {
        const {container} = render(<Greeting />);

        expect(container).toHaveTextContent('Hello, WORLD!');
    });
});
```

### (필요 시) 6. VSCode 설정

`.vscode/settings.json` 파일 생성해서 유용한 VSCode 설정을 추가할 수 있다.

```json
{
    "editor.rulers": [
        80
    ],
    "editor.codeActionsOnSave": {
        "source.fixAll.eslint": true
    },
    "trailing-spaces.trimOnSave": true
}
```
