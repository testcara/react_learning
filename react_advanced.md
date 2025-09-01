# React advanced feature

## 1. `map, filter` is better than `forEach`

`forEach`：只是执行遍历，没有返回值，更多是“副作用”操作。

`map`：返回一个新数组，常用来“渲染列表 UI”。

`filter`：返回满足条件的子数组。

👉 在`React`里，推荐`map/filter/reduce`而不是`forEach`，因为`React`需要“返回一个新的`UI`元素树”，而`map`可以直接生成新数组，更符合“函数式编程”的思路。

Examples:

```go
// 不推荐
const listItems = [];
items.forEach((item) => {
  listItems.push(<li key={item.id}>{item.name}</li>);
});

// 推荐
const listItems = items.map((item) => (
  <li key={item.id}>{item.name}</li>
));

```

## 2. `useCallback`

`useCallback(fn, deps)`：缓存一个函数，避免每次`render`都生成新函数，常用于传给子组件的回调。

Examples:

```go
import React, { useState, useCallback } from "react";

function Child({ onClick }: { onClick: () => void }) {
  console.log("Child render");
  return <button onClick={onClick}>Click Me</button>;
}

export default function App() {
  const [count, setCount] = useState(0);

  // 如果不用 useCallback，每次父组件渲染都会生成一个新函数，Child 会重复渲染
  const handleClick = useCallback(() => {
    setCount((prev) => prev + 1);
  }, []);

  return (
    <div>
      <p>Count: {count}</p>
      <Child onClick={handleClick} />
    </div>
  );
}
```

👉 用`useCallback`确保传给子组件的函数引用不变，避免无意义的`re-render`。

## 3. `reduce`

万能聚合器，可以累积、转换、构建对象/数组、统计等等。

- Example 1: 数组求和/聚合计算的场景。

  ```typescript
  const numbers = [1, 2, 3, 4, 5];

  const sum = numbers.reduce((acc, n) => acc + n, 0);
  ```

  reduce 的第一个参数是一个函数 `(acc, n) => acc + n`, 第二个参数`0`是累加器的初始值。

  `acc`是累加器，用来存储每一步的累积结果。`n`是当前数组元素。

  执行过程：

  ```none

  acc=0, n=1 → acc=1

  acc=1, n=2 → acc=3

  acc=3, n=3 → acc=6 …
  ```

  最终返回`15`。

- Example 2: 统计元素出现次数/适合统计/分类的场景

  ```typescript
  const fruits = ["apple", "banana", "apple", "orange", "banana", "apple"];
  const count = fruits.reduce((acc, fruit) => {
    acc[fruit] = (acc[fruit] || 0) + 1;
    return acc;
  }, {} as Record<string, number>);
  ```

  累加器`acc`初始化为空对象`{}`。

  执行过程：

  ```none
  每次遍历一个元素`fruit`：

  acc[fruit] || 0 → 如果对象里已有这个水果数量就取它，否则取 0
  加 1 并更新对象
  ```

  执行完后，得到每个水果出现次数的对象：`{ apple: 3, banana: 2, orange: 1 }`

- Example 3:数组转对象（按 id 映射）/适合从数组生成对象映射的场景，比如快速通过 id 查找数据。
  ```typescript
  const users = [
    { id: 1, name: "Tom" },
    { id: 2, name: "Jerry" },
    { id: 3, name: "Spike" },
  ];
  const userMap = users.reduce((acc, user) => {
    acc[user.id] = user.name;
    return acc;
  }, {} as Record<number, string>);
  ```
  累加器 acc 初始化为空对象 `{}`。
  执行过程：
  ```none
  每次遍历一个`user`：
  用`user.id`作为键，`user.name`作为值，存入对象
  ```
  最终得到：`{ 1: "Tom", 2: "Jerry", 3: "Spike" }`

## 4. useRef

获取 DOM 节点或在组件渲染间保持可变值而不触发重渲染

```typescript
import React, { useRef } from "react";

export default function App() {
  const inputRef = useRef<HTMLInputElement>(null);

  // 点击按钮调用 focusInput
  const focusInput = () => {
    inputRef.current?.focus(); // 聚焦 input
  };

  return (
    <div>
      <input ref={inputRef} placeholder="Type here..." />
      <button onClick={focusInput}>Focus Input</button>
    </div>
  );
}
```

## 5. useReducer

复杂状态逻辑和动作类型集中管理

```typescript
import React, { useReducer } from "react";

interface Todo {
  id: number;
  text: string;
  done: boolean;
}

type Action =
  | { type: "add"; text: string }
  | { type: "toggle"; id: number }
  | { type: "remove"; id: number };

function reducer(state: Todo[], action: Action): Todo[] {
  switch (action.type) {
    case "add":
      return [...state, { id: Date.now(), text: action.text, done: false }];
    case "toggle":
      return state.map((todo) =>
        todo.id === action.id ? { ...todo, done: !todo.done } : todo
      );
    case "remove":
      return state.filter((todo) => todo.id !== action.id);
    default:
      return state;
  }
}

export default function TodoApp() {
  const [todos, dispatch] = useReducer(reducer, []);

  return (
    <div>
      <button
        onClick={() => dispatch({ type: "add", text: "Learn useReducer" })}
      >
        Add Todo
      </button>
      {todos.map((todo) => (
        <div key={todo.id}>
          <input
            type="checkbox"
            checked={todo.done}
            onChange={() => dispatch({ type: "toggle", id: todo.id })}
          />
          {todo.text}
          <button onClick={() => dispatch({ type: "remove", id: todo.id })}>
            Remove
          </button>
        </div>
      ))}
    </div>
  );
}
```

## 6. useEffect

`React`函数组件处理副作用的标准方式。

它取代了类组件里的`componentDidMount` / `componentDidUpdate` / `componentWillUnmount`。

配合依赖数组和清理函数，可以实现高效、可控的副作用管理。

`useEffect`生命周期流程:

```none
组件挂载
   │
   ▼
第一次渲染完成（DOM 已生成）
   │
   ├── 执行 useEffect 中的副作用函数
   │      例如：fetch 数据、设置定时器、操作 DOM
   │
   ▼
组件更新（state/props 改变）
   │
   ├── 检查依赖数组
   │      ├─ 如果依赖未变化 → 不执行副作用
   │      └─ 如果依赖变化 →
   │           1. 先执行上一次副作用的清理函数（如果有）
   │           2. 再执行新的副作用函数
   │
   ▼
组件卸载
   │
   └── 执行清理函数（如果有）
```

## 7. useState

`useState` 工作流程

```none
组件挂载
   │
   ▼
初始化状态
   └─ useState(initialValue)
         └─ state = initialValue
   │
   ▼
第一次渲染完成
   │
   ▼
用户交互或事件触发 setState
   │
   ├── setState(newValue) 被调用
   │       └─ React 将 newValue 标记为最新状态
   │
   ▼
组件重新渲染
   │
   ├── useState 返回最新 state
   │
   ▼
组件 DOM 更新完成
   │
   ▼
显示新的状态值

```

## 8. clean codes

### 语法

1. 解构赋值

   ```typescript
   const person = { name: "Flora", age: 25 };
   const { name, age } = person;
   ```

   可以直接从对象或数组中取值，减少`person.name`的重复写法

2. 箭头函数-简化函数写法
   ```typescript
   const add = (a, b) => a + b;
   ```

3.模板字符串 - 替代繁琐的字符串拼接
`` typescript
    const message = `Hello ${name}, your age is ${age}`;
     ``

4. 默认参数 - 减少条件判断

   ```typescript
   function greet(name = "Guest") {
     console.log(`Hello ${name}`);
   }
   ```

5. 可选链 ?. 和空值合并 ?? - 避免多层 && 判断

   ```typescript
   console.log(user?.profile?.name ?? "Anonymous");
   ```

6. 扩展运算符 ...
   ```typescript
   const newArr = [...oldArr, 4];
   const newObj = { ...oldObj, name: "Flora" };
   ```

### React 自身特性

1. Hooks

   - useState, useReducer → 管理状态

   - useEffect → 副作用

   - useMemo → 缓存计算结果

   - useCallback → 缓存函数，避免子组件重复渲染

   - useRef → 保持引用、操作 DOM

   - useLayoutEffect, useImperativeHandle → 高级场景

2. 条件渲染简化
   ```typescript
   {
     isLoggedIn && <Dashboard />;
   }
   {
     status === "loading" ? <Spinner /> : <Content />;
   }
   ```
3. 列表渲染简化

   ```typescript
   {
     todos.map((todo) => <li key={todo.id}>{todo.text}</li>);
   }
   ```

4. 短路运算

   ```typescript
   const message = user && user.name;
   ```

5. 数组方法
   - map, filter, reduce → 替代 for / foreach，提高可读性
   - some, every → 条件判断更直观
   - find → 查找特定元素
