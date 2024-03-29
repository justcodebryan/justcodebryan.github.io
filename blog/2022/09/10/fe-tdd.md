---
slug: fe-tdd
title: 前端TDD初探
authors: [justcodebryan]
tags: [Front-end Engineering]
---

# 测试分类

按照软件工程自底而上的概念, 前端测试分为以下四类:

1. 单元测试(Unit Testing)

   指的是以原件的单元为单位，对软件进行测试。单元可以是一个函数，也可以是一个模块或一个组件，基本特征就是只要输入不变，必定返回同样的输出。一个软件越容易些单元测试，就表明它的模块化结构越好，给模块之间的耦合越弱。React 的组件化和函数式编程，天生适合进行单元测试

2. 集成测试(Integration Testing)

   在单元测试的基础上，将所有模块按照设计要求组装成子系统或者系统，进行测试

3. 端对端测试(E2E Testing)

   在正式全面的测试之前，对主要功能进行的与测试，确认主要功能是否满足需要，软件是否能正常运行

4. 功能测试

   相当于是黑盒测试，测试者不了解程序的内部情况，不需要具备编程语言的专门知识，只知道程序的输入、输出和功能，从用户的角度针对软件界面、功能和外部结构进行测试，不考虑内部的逻辑

建议的测试数量:

单元测试 > E2E 测试 > 快照测试

选型: `Jest` + `Enzyme` + `enzyme-adapter-react-16`

安装依赖, 因为是`React16`以上的版本, 所以需要安装插件

```bash
yarn add jest enzyme enzyme-adapter-react-16 @types/jest @types/enzyme @types/enzyme-adapter-react-16 ts-jest -D
```

因为需要使用`sass`, 所以需要添加`identity-obj-proxy`才能使得测试正常运行

```bash
yarn add identity-obj-proxy -D
```

快照序列化需要`enzyme-to-json`

```bash
yarn add enzyme-to-json -D
```

在`package.json`中添加

```json
{
  "scripts": {
    "test": "jest --config jest.config.js --no-cache"
  }
}
```

在根目录下添加`__mocks__`文件夹, 并且创建`js`文件命名为`fileTransformer.js`

用于 mock 其他文件, 图片之类的文件

```js
// fileTransformer.js
const path = require("path");

module.exports = {
  process(src, filename, config, options) {
    return "module.exports = " + JSON.stringify(path.basename(filename)) + ";";
  },
};
```

在根目录添加一个文件`jest.config.js`

```js
// jest.config.js文件
module.exports = {
  verbose: true,
  setupFiles: ["./tests/setup.js"],
  setupFilesAfterEnv: ["./tests/setupAfterEnv.ts"],
  moduleFileExtensions: ["ts", "tsx", "js", "jsx", "json", "md"], // 文件扩展名
  moduleNameMapper: {
    "\\.(css|scss)$": "identity-obj-proxy", // scss文件处理的依赖
  },
  transform: {
    "^.+\\.tsx?$": "ts-jest",
    "\\.(jpg|jpeg|png|gif|eot|otf|webp|svg|ttf|woff|woff2|mp4|webm|wav|mp3|m4a|aac|oga)$":
      "<rootDir>/__mocks__/fileTransformer.js",
  },
  testRegex: "(/__test__/.*|(\\.|/)(test|spec))\\.tsx?$", // 正则表达式找到需要测试的文件
  globals: {
    "ts-jest": {
      tsConfig: "tsconfig.json",
    },
  },
  testPathIgnorePatterns: ["/node_modules/", "dist"], // 忽略测试的文件夹
  collectCoverage: true, // 收集覆盖率
  coverageDirectory: "./coverage", // 覆盖率报告
  snapshotSerializers: ["enzyme-to-json/serializer"], // 快照序列化
};
```

在根目录下添加`tests`文件夹

创建`setup.js`

```js
// setup.js
const React = require("react");

// eslint-disable-next-line no-console
console.log("Current React Version:", React.version);

/* eslint-disable global-require */
if (typeof window !== "undefined") {
  global.window.resizeTo = (width, height) => {
    global.window.innerWidth = width || global.window.innerWidth;
    global.window.innerHeight = height || global.window.innerHeight;
    global.window.dispatchEvent(new Event("resize"));
  };
  global.window.scrollTo = () => {};
  if (!window.matchMedia) {
    Object.defineProperty(global.window, "matchMedia", {
      value: jest.fn((query) => ({
        matches: query.includes("max-width"),
        addListener: jest.fn(),
        removeListener: jest.fn(),
      })),
    });
  }
}

// 因为是React16版本的缘故, 需要配置adapter
// 配置全局adapter
const Enzyme = require("enzyme");

const Adapter = require("enzyme-adapter-react-16");

Enzyme.configure({ adapter: new Adapter() });

Object.assign(Enzyme.ReactWrapper.prototype, {
  findObserver() {
    return this.find("ResizeObserver");
  },
  triggerResize() {
    const ob = this.findObserver();
    ob.instance().onResize([{ target: ob.getDOMNode() }]);
  },
});
```

创建`setupAfterEnv.ts`:

```typescript
import toMatchRenderedSnapshot from "./matchers/rendered-snapshot";

expect.extend({
  toMatchRenderedSnapshot,
});
```

快照测试生成一个`UI`结构, 并用字符串的形式放在`__snapshots__`文件里, 通过, 比较两个字符串来判断`UI`是否发生改变

创建`matchers/rendered-snapshot.ts`

```typescript
import { render } from "enzyme";
import { ReactElement } from "react";

// 快照测试
export default function toMatchRenderedSnapshot(
  this: jest.MatcherUtils,
  jsx: ReactElement<unknown>
): { message(): string; pass: boolean } {
  try {
    expect(render(jsx)).toMatchSnapshot();

    return {
      message: () => "expected JSX not to match snapshot",
      pass: true,
    };
  } catch (e) {
    return {
      message: () => `expected JSX to match snapshot: ${e.message}`,
      pass: false,
    };
  }
}
```

# 单元测试

## 模范

命名规范: `***.test.tsx` / `index.spec.tsx`

```tsx
describe("测试套件标题", () => {
  it("测试标题", () => {});
});
```

## 常用`API`

此处仅仅列出比较常用的`API`

## Jest

### globals API

`afterAll(fn, timeout)`：所有测试用例跑完以后执行的方法

`beforeAll(fn, timeout)`：所有测试用例执行之前执行的方法

`afterEach(fn)`：在每个测试用例执行完后执行的方法

`beforeEach(fn)`：在每个测试用例执行之前需要执行的方法

### 常见断言

`toHaveBeenCalled()`：用来判断 mock function 是否被调用过

`toHaveBeenCalledTimes(number)`：用来判断 mock function 被调用的次数

`resolves`：用来取出 promise 为 fulfilled 时包裹的值，支持链式调用

`rejects`：用来取出 promise 为 rejected 时包裹的值，支持链式调用

## Enzyme

### 渲染方法

1. `shallow`: 浅渲染, 子组件不会渲染, 效率高, 可以用`jQuery`的方式访问组件信息
2. `mount`: 完全渲染, 将组件渲染加载成一个真实的`DOM`节点
3. `render` : 将`React`组件渲染成静态的`HTML`字符串, 可以用来分析`html`结构

因为`shallow`和`mount`返回的是`DOM`对象, 可以用`simulate`进行交互模拟, `render`不可以

`shallow` - 一般情况下都能满足

`render` - 需要对子组件进行判断

`mount` - 需要测试组件的生命周期

类的原型方法可以通过`shallow`和`mount`方法渲染组件以后, 在通过 `spyOn(instance: Prototype, MethodName: string)`来监控并且测试类的原型的方法

**注意**: 需要在`spy`完某一个函数之后对其进行`mockRestore`

### 常用方法

1. `simulate(event, mock)`: 模拟事件, 用来触发事件, `event`为事件名称, `mock`为一个`event object`
2. `find(selector)`: 根据选择器查找节点, `selector`可以是`CSS`中的选择器, 或者是组件的构造函数, 组件的`display name`等
3. `state()`: 返回根组件的状态, 但是如果使用`React Hooks`中的`useState`是无法获取到该状态
4. `text()`: 返回当前组件的文本内容
5. `html()`: 返回当前组件的`HTML`代码

## 测试覆盖率报告

`%stmts`是语句覆盖率（statement coverage）表示是不是每个语句都执行了？

`Branch`代表分支, 比如`if-else`分支中没有测试到的

`Funcs`代表函数, 没有测试到的函数也会在显示百分比

`Lines`代表覆盖的行数, 显示已经覆盖的行数的百分比

`Uncovered Line`: 表示测试没有覆盖到行数, 具体是哪几行没有被覆盖到

下面会显示通过的测试套件, 测试数量以及快照数量

# 相关资料

1. [Jest 官方文档](https://jestjs.io/zh-Hans/)
2. [对 React 组件进行单元测试](https://juejin.im/post/5a71413e5188252edb593020)
3. [如何对 react hooks 进行单元测试](https://segmentfault.com/a/1190000020058166)
4. [使用 Jest 进行 React 单元测试](https://juejin.im/post/5b6c39bde51d45195c079d62)
5. [React 测试指南](https://segmentfault.com/a/1190000018063747)
6. [使用 jest+enzyme 测试 react 组件](https://github.com/frontend9/fe9-library/issues/244)
7. [用 Enzyme 测试使用 Hooks 的 React 函数组件](https://juejin.im/post/5e933cbc6fb9a03c74137865)
