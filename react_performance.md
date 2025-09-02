## REACT UI 常见性能问题

### 渲染相关

不必要的重新渲染（Re-renders）

1. 父组件状态变化导致子组件重复渲染。

   优化方式：React.memo、useMemo、useCallback，减少重复渲染。

2. 大量列表渲染

   渲染成百上千条列表时性能下降。

   优化方式：虚拟列表（react-window / react-virtualized）。

3. 复杂组件树

   多层嵌套组件，每次状态更新可能触发整个树渲染。

### 状态管理相关

频繁更新全局状态

1. Redux 或 Context 频繁更新会导致大量组件刷新。

   优化方式：局部状态优先，避免全局状态过度使用。

2. 不合理的数据传递

   通过 props 层层传递，导致深层组件也重渲染。

   优化方式：useContextSelector、状态分片。

### 网络请求 & 数据处理

重复请求

1. 组件每次渲染都触发相同请求。

   优化方式：缓存请求结果（React Query、SWR），useEffect 依赖控制。

2. 大数据处理阻塞 UI

   前端直接在 UI 线程处理大量数据。

   优化方式：Web Worker、分页加载、懒加载。

### 资源 & 依赖

1. 图片 / 静态资源过大

   大图片或视频直接加载会阻塞渲染。

   优化方式：图片压缩、懒加载、CDN。

2. 第三方库过重

   导致 bundle 大，首屏加载慢。

   优化方式：按需加载、代码分割（React.lazy + Suspense）。

### 其他

1. 事件处理滥用

   高频事件（scroll, mousemove）没有节流或防抖。

   优化方式：throttle / debounce。

2. 动画性能

   CSS 动画 vs JS 动画，复杂 JS 动画可能阻塞主线程。
