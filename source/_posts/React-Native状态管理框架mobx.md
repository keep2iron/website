---
title: React-Native状态管理框架mobx实战
date: 2018-11-06 11:00:34
tags: react native
desc: React-Native状态管理框架mobx实战
---

<img src="https://cn.mobx.js.org/flow.png" width="680px"/>

> 文章中的代码基于 ract-native 0.57.1编写

<a href="https://cn.mobx.js.org/">MobX</a> 是一个经过战火洗礼的库，它通过透明的函数响应式编程(transparently applying functional reactive programming - TFRP)使得状态管理变得简单和可扩展。MobX背后的哲学很简单:

*任何源自应用状态的东西都应该自动地获得。*

<!-- more -->

上面提到了一个重要的关键点 <a href="https://zh.wikipedia.org/wiki/%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B">*响应式编程*</a>。

<h4>响应式编程</h4>

- 面向数据

- 计算模型会自动将变化的值通过数据流进行传播

mobx充分体现了上面提到的这两点，那么接下来的问题是....

<h4>如何上手mobx？</h4>


##### Observable(可观察的状态)
该类用于修饰类成员变量（实例）即可完成可观察的功能。
```
ES Next(装饰器)语法:

import { observable } from "mobx";

class Todo {
    id = Math.random();
    @observable title = "";
    @observable finished = false;
}

ES5
import { decorate, observable } from "mobx";

class Todo {
    id = Math.random();
    title = "";
    finished = false;
}
decorate(Todo, {
    title: observable,
    finished: observable
})
```
##### Computed values(计算值)
刚开始接触mobx的时候我并没有认为这个计算有什么意义，不过随着接触的深入发现这个概念也是一个极其重要的概念。
例如需求如下：购物车中有一个显示购物车数量的按钮，通过前面的装饰器我们很容易编写出表示购物车列表的state数据

```
	@observable shoppingCart = [];
```
那么我们还需要知道购物车的数量，你可能会说通过shoppingCart.length获取。但是list是会实时改变的，那么如何不做到利用另外一个变量的情况下获取购物车的数量呢？请看如下代码。
```
@computed
getShoppingCartCount(){
	return this.shoppingCart.length; 
}
```
当添加了一个shoppingCart或者其属性发生变化时，MobX 会确保 shoppingCart 自动更新。 像这样的计算可以类似于 MS Excel 这样电子表格程序中的公式。每当只有在需要它们的时候，它们才会自动更新。
##### observer（观察者）、Action
```
// ShoppingCartStore.js
export default class ShoppingCartStore {
    // 购物车列表
    @observable
    _shoppingCartList: Array<ShoppingCartProduct> = [];

    @computed
    get shoppingCartCount() {
        return this.shoppingCartList.length;
    }

    @action
    async loadShoppingCarts() {
        const resp = await this.respitory.loadShoppingCart();
        if (resp) {
            runInAction(() => {
                this._shoppingCartList = resp.list;
            });
        }
    }
}

// ShoppingCartPage.js
@observer
export default class ShoppingCartPage extends Component<Props, State> {
	
	componentDidMount(){
		const { shoppingCartStore } = this.props;
 
		shoppingCartStore.loadShoppingCarts().catch(() => {});
	}

	render(){
		return (
			<FlatList
				...//此处省略若干代码
                data={this.store.shoppingCartList}
            />);
	}
}
```
上面的代码算是一个完整的购物车列表显示流程
1.由componentDidiMount触发store加载函数loadShoppingCarts,通过*this.respitory.loadShoppingCart*获取返回的数据，随后执行this._shoppingCartList = resp.list;那么为什么要写runInAction呢？
因为针对所有@observable的赋值操作必须在@action注解的函数或者在runInAction中执行。
> 状态(这里就是_shoppingCartList)应该以某种方式来更新。
> 当状态更新后，MobX 会以一种高效且无障碍的方式处理好剩下的事情。像上面如此简单的语句，已经足够用来自动更新用户界面了。

