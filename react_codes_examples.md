## 条件展开数组（spread + ternary） 的写法

```javascript
const fruits = ["apple", "orange", "banana"];
const newOne = true;
const allFruits = [...fruits, ...(newOne ? ["pear"] : [])];
console.log(allFruits);

const baseColumns = [{ name: "cara" }, { name: "dailei" }, { name: "test" }];
const allColumns = [...baseColumns, ...(newOne ? [{ name: "andy" }] : [])];
console.log(allColumns);

const newfruits = ["apple"];
const addOrange = true;
const addBanana = false;
const allNewFruits = [
  ...newfruits,
  ...(addOrange ? ["orange"] : []),
  ...(addBanana ? ["banana"] : []),
];
console.log(allNewFruits);

/*
动态STRING
*/
function getMessages(showWarning) {
  const baseInfo = ["Info: All good"];
  const allMessages = [
    ...baseInfo,
    ...(showWarning ? ["Warning: Check this!"] : []),
  ];
  return allMessages;
}

console.log(getMessages(true));

/*
动态表格列
*/
function createTableColumns(showAge, showEmail) {
  const baseColumns = [{ title: "Name" }];
  const columns = [
    ...baseColumns,
    ...(showAge ? [{ title: "Age" }] : []),
    ...(showEmail ? [{ title: "Email" }] : []),
  ];
  return columns;
}
console.log(createTableColumns(true, false));
```
