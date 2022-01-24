# 一文详解React事件中this指向，面试必备

## 结论

先上结论

1. 4种处理事件处理函数中this指向问题的方案
2. 3种实现事件传参的解决方案
3. 5种this指向方案的性能比较
4. 两种最优解

## 需求

我们知道，在 `react` 中，事件处理函数中的this很容易丢失，如

```react
class App extends React.Component {
  handleClick() {
    console.log(this);// undefined 
  }
  render() {
    return <div onClick={this.handleClick} >点我</div>;
  }
}
```

## 4种this指向解决方案

结合原生 `JavaScript` 的理解，我们有如下4种解决方案

1. 箭头函数
2. 构造函数中的 `bind` 
3. 事件绑定时的 `bind`
4. 事件绑定时的匿名函数+箭头函数。

下边我们分别给出它们的实现

### 箭头函数

写法上最简单

```jsx
class App extends React.Component {
  // 修改为箭头函数
  handleClick = () => {
    console.log(this);// 正常 
  }
  render() {
    return <div onClick={this.handleClick} >点我</div>;
  }
}
```

### 构造函数中的bind 

```jsx
class App extends React.Component {
  constructor(props) {
    super();
    // 通过bind 改写this指向并返回新的函数
    this.handleClick = this.handleClick.bind(this);
  }
  handleClick() {
    console.log(this);// 正常
  }
  render() {
    // bind
    return <div onClick={this.handleClick} >点我</div>;
  }
}
```

### 事件绑定时的bind

```jsx
class App extends React.Component {
  handleClick() {
    console.log(this);// 正常
  }
  render() {
    // bind
    return <div onClick={this.handleClick.bind(this)} >点我</div>;
  }
}
```

### 事件绑定时的匿名函数+箭头函数

```jsx
class App extends React.Component {
  handleClick() {
    console.log(this);// 正常
  }
  render() {
    // bind
    return <div onClick={() => this.handleClick()} >点我</div>;
  }
}
```

虽然以上四种方案都可以解决 事件处理函数中this指向的问题，但是由于我们在开发时，往往还需要做事件传参。

因此还有以下4种事件传参写法

## 3种事件传参解决方法

1. 事件绑定时的 `bind`
2. 事件绑定时的匿名函数+箭头函数
3. 自定义属性 `dataset` 

### 事件绑定时的bind

bind 函数不但可以修改this指向返回新函数，还可以接收参数。写法相当灵活飘逸

```jsx
class App extends React.Component {
  handleClick(a, b, e) {
    console.log(this);// 正常
    console.log(a, b, e);// "过火"，"上火","事件对象"
  }
  render() {
    return <>
      <div onClick={this.handleClick.bind(this, "过火", "上火")} >点我</div>
    </>
  }
}
```



### 事件绑定时的匿名函数+箭头函数

```jsx
class App extends React.Component {
  handleClick(a, b, e) {
    console.log(this);// 正常
    console.log(a, b, e);// "过火"，"上火","事件对象"
  }
  render() {
    return <>
      <div onClick={(event) => this.handleClick("过火", "上火", event)} >点我</div>
    </>
  }
}
```

### 自定义属性 dataset 

```jsx
class App extends React.Component {
  handleClick(e) {
    console.log(this);// 正常
    console.log(e.target.dataset.msg);// "过火"
  }
  render() {
    return <>
      <div data-msg="过火" onClick={this.handleClick.bind(this)} >点我</div>
    </>
  }
}
```

虽然 `react` 事件相关代码写法看起来多种多样，但是，如果从性能角度出发，写法就剩下以下一种。 **事件绑定时的bind**

## 5种this指向解决方案的性能比较

以下比较，主要从 实例成员函数能否得到复用触发，因为 类组件也可以理解是个class，那么我们可以将其函数成员做对比，看哪种方式是共享内存。



### App

```js
class App extends React.Component {
  constructor() {
    super();
    this.ref1 = React.createRef();
    this.ref2 = React.createRef();
  }

  handleClick(e) {
    //  比较 Btn 组件中的事件处理函数
    console.log(this.ref1.current.showBtn === this.ref2.current.showBtn);
  }
  render() {
    return <>
      <div onClick={this.handleClick.bind(this)} >
        <Btn ref={this.ref1} />
        <Btn ref={this.ref2} />
      </div>
    </>
  }
}
```

### Btn 不做this处理  true

> 没有处理this指向的问题

```jsx
class Btn extends React.Component {
  showBtn() {
    console.log(this); // undefined
  }
  render() {
    return <button onClick={this.showBtn}>按钮</button>;
  }
}
```

这时 `App` 中

```jsx
  handleClick(e) {
    //  比较 Btn 组件中的事件处理函数
    console.log(this.ref1.current.showBtn === this.ref2.current.showBtn); // true
  }
```



### Btn 箭头函数  false

```jsx
class Btn extends React.Component {
  showBtn = () => {
    console.log(this);
  }
  render() {
    return <button onClick={this.showBtn}>按钮</button>;
  }
}
```

这时 `App` 中

```jsx
  handleClick(e) {
    //  比较 Btn 组件中的事件处理函数
    console.log(this.ref1.current.showBtn === this.ref2.current.showBtn);// false
  }
```

### Btn 构造函数中的 bind false

```jsx
class Btn extends React.Component {
  constructor() {
    super();
    this.showBtn = this.showBtn.bind(this);
  }
  showBtn() {
    console.log(this);
  }
  render() {
    return <button onClick={this.showBtn}>按钮</button>;
  }
}
```

这时 `App` 中

```jsx
  handleClick(e) {
    //  比较 Btn 组件中的事件处理函数
    console.log(this.ref1.current.showBtn === this.ref2.current.showBtn); // false
  }
```



### Btn 事件绑定时的 bind true

```jsx
class Btn extends React.Component {
  showBtn() {
    console.log(this);
  }
  render() {
    return <button onClick={this.showBtn.bind(this)}>按钮</button>;
  }
}
```

这时 `App` 中

```JSX
  handleClick(e) {
    //  比较 Btn 组件中的事件处理函数
    console.log(this.ref1.current.showBtn === this.ref2.current.showBtn); // true
  }
```



### Btn 事件绑定时的匿名函数+箭头函数 true

```jsx
class Btn extends React.Component {
  showBtn() {
    console.log(this);
  }
  render() {
    return <button onClick={() => this.showBtn()}>按钮</button>;
  }
}
```

这时 `App` 中

```jsx
  handleClick(e) {
    //  比较 Btn 组件中的事件处理函数
    console.log(this.ref1.current.showBtn === this.ref2.current.showBtn); // true
  }
```





## 小结

结合以上的对比分析，推荐react中，事件处理函数的写法有2种

### 方式一

```jsx
class Btn extends React.Component {
  showBtn() {
    console.log(this);
  }
  render() {
    return <button onClick={this.showBtn.bind(this)}>按钮</button>;
  }
}
```

### 方式二

```jsx
class Btn extends React.Component {
  showBtn() {
    console.log(this);
  }
  render() {
    return <button onClick={() => this.showBtn()}>按钮</button>;
  }
}
```

