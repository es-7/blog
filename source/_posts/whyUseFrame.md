---
title: 为何前端要使用框架？
date: 2015-08-02
top: 2000
---

亲爱的各位观众看官好～最近在团队里组织了一次内容关于React的分享，然而后端有同学反映未能理解为何前端需要使用框架，框架究竟解决了什么问题。
<!--more-->  
回想刚入坑前端的时候，也接触了一些Java，当了解到其实Java可以生成HTML模板之后，对前端的信仰几乎崩溃。既然后端就可以渲染前端的话，前端价值到底在哪？感觉页面就是随便切一下就行，根本没必要用框架，当时着实迷茫了很久。


后来对前端领域接触深了，再经过大神的指点，总算是理解为何前端需要使用框架。根据自己的理解，再结合分享时后端同学的疑问，于是有了这篇文章。本文相对小白向，只是通过虚拟一个项目说明问题，还望各位大神不吝赐教。

## 开始新项目
> 产品： 来来来，新项目来啦。最近发现用户喜欢撸猫，我们来个在线云撸猫！页面要求有猫图，点一下计数加一点当撸了一次！多久能上线？？
这个简单啊！

![狂暴姿态](whyUseFrame/1.png)

厉害的我连jQuery都不用，原生上！

```html
<body>
    <img src="1.jpg" alt="">
    <p>0</p>
    <script>
        const img = document.querySelector('img'); 
        const p = document.querySelector('p'); 
        let count = 0; 
        img.addEventListener('click', function(){ 
               count += 1; 
               p.innerHTML = count; 
        })
    </script>
</body>
```
简单易懂，对不对！

## 需求变更

> 产品： 线上大受欢迎，然而要改需求了爸爸！现在要两只喵，点击要分开算哦，各自显示就好。你最棒了，我知道你肯定可以的！

看在叫了爸爸的份上，还是给他改一下吧。然而第一个版本多粗糙啊，各种污染全局变量，第二版我要写得棒棒的！


```html
<body>
<section class="catSection">
    <img src="cat1.jpg" alt="" class="catImg">
    <p class="catCount">0</p>
</section>
<section class="catSection">
    <img src="cat2.jpg" alt="" class="catImg">
    <p class="catCount">0</p>
</section>
<script>
    (function() {
        const catSection = document.querySelectorAll('.catSection');
        catSection.forEach((section) => {
            let count = 0;
            const catImg = section.querySelector('.catImg');
            const catCount = section.querySelector('.catCount');
            catImg.addEventListener('click', () => {
                count++;
                catCount.innerHTML = count;
            })
        })
    })()
</script>
```
没有污染全局变量，还可以扩展需求，后续产品就算要求再多的猫我都能hold住！

 ![6不6](whyUseFrame/2.png)
 
## 再一次需求变更

> 产品： 父皇，你听我说啊，就最后一次，真的最后一次。现在猫两只不够啊，但太多的话展示又不好看。现在需求做一个列表，列表有N只猫，点那只猫展示那只猫，每次只展示意志，而且猫要有对应的名字哦，点击也要分开统计！其实跟以前差不多？就改改就好了。明天就要哦！

 ![草泥马](whyUseFrame/3.png)
 
 然而，吐槽归吐槽，需求还是要做的。而且需求明天就要，代码写得再好也可能马上改功能，代码还是实现功能就算了吧。
 具体代码由各位看官自己实现（建议先停下来，动手去实现这个需求），这里我就不再上代码了。很可能我们这次写的代码，就不会太考虑什么全局变量污染，也不考虑封装的问题，逐渐趋向于实现功能就好了，因为需求太多了，时间和精力限制了去写“好”代码。
 当然了，产品经理说需求最后一次改，都是骗人的。下次可能会有点击到某个值时，自动切换猫，动态添加猫等等的新需求。我们现在这样组织代码的形式，是典型的”意大利面式“代码（简单说，就是各种东西整合在一起，层级不分，难以维护）。这种代码写起来直观，但日后的维护是相当难的。写出上述例子后，不妨隔两天再去看看能否轻易理解那一份代码。自己写的代码尚且如此，他人的更不在话下了。
 那么，这种随着项目越来越复杂，逻辑越来越多，我们该怎么写代码呢？

## 更好地组织代码

前端其实有一种说法：我们现在的”新“东西，都是其他领域玩过的。虽然看起来很气人，但这也是事实。当现状不知如何处理时，不妨参考下其他领域的解决方案。离前端比较近的就是后端了，那么后端是怎么管理的呢？最典型的设计就是MVC了。那么，前端能不能借鉴呢？
说干就干，以上面的需求为例，我们试着用MVC的方式组织一下代码，看下和你刚才写的有什么不同。
先来M，也就是Model，数据层，对外提供接口可以获取相关的数据。这么组织的话，是不是蛮好懂的：

```js
const model = (function() {
  //相关数据
  const _model = {
    catLists: [
      {
        src: '1.jpg',
        name: 'cat1',
        count: 0
      },
      {
        src: '2.jpg',
        name: 'cat2',
        count: 0
      }
    ],
    targetCatIndex: 0, //目前可被点击的是哪知喵在catLists中的索引
  };

  //获取getCatLists
  function getCatLists() {
    return _model.catLists;
  }

  //获取目标对象
  function getTargetCatObj() {
    return _model.catLists[_model.targetCatIndex];
  }
  //修改targetCatIndex
  function setTargetCatIndex(name) {
    _model.catLists.some((catObj, index) => {
      if (catObj.name === name) {
        _model.targetCatIndex = index;
        return true
      }
    })
  }

  //目标对象点击数+1
  function addTargetCatCount() {
    const catObj = getTargetCatObj();
    catObj.count += 1;
  }

  return {
    getCatLists,
    getTargetCatObj,
    setTargetCatIndex,
    addTargetCatCount
  }
})();
```
在自执行函数中，设计了一个对象命名为_model，通过闭包存储它。自执行函数返回一个对象，其中包含四个函数。四个函数执行后，可以返回或修改_model中对应的数据。通过注释看其实还是挺清楚的。
跟着是V，也就是view层，负责页面渲染。这个可能复杂一点，但不想把它弄得太繁琐，不如就两个方法吧。就一个初始化的init()和负责更新视图的render()方法就好啦。

先确定HTML模板：
```html
<ul class="catList"></ul>
<section class="clickArea">
    <img src="" class="catImage">
    <p class="catCount"></p>
    <p class="catName"></p>
</section>
```

再组织一下view层的代码：

```js
const view = (function() {

  //获取各个需要操作的DOM节点
  const img = document.querySelector('.catImage');
  const name = document.querySelector('.catName');
  const count = document.querySelector('.catCount');

  //初始化页面
  function init(catLists, targetObj) {
    const list = document.querySelector('.catList');
    const fragment = document.createDocumentFragment();
    //为ul添加对应的li
    for (let i = 0, len = catLists.length; i < len; i++) {
      const li = document.createElement('li');
      const name = catLists[i].name;
      li.innerHTML = name;
      li.addEventListener('click', function() {
        //之后会有controller相关的代码，其实就是换一只可点击的喵
        controller.changeTargetCat(name);
      });
      fragment.appendChild(li);
    }
    list.appendChild(fragment);
    img.addEventListener('click', function() {
      //之后会有controller相关的代码，其实就是计数+1
      controller.addCount(name);
    });
    render(targetObj);
  }

  //重新渲染页面
  function render(targetObj) {
    img.src = targetObj.src;
    name.innerHTML = targetObj.name;
    count.innerHTML = targetObj.count;
  }

  return {
    init,
    render
  }
})();
```

view层的代码其实也很简单的，和model层的套路差不多，通过自执行函数结合闭包存储之后要操作的节点，对外暴露由两个方法组成的对象，分别是init与render。init用于初始化话页面，render用于重新渲染页面。里面调用了controller，其实就是之后要介绍的controller。
最后是C层，也就是controller，主要是用于逻辑相关的处理，算是整个设计里面的大脑。不过由于这项目比较简单，所以代码反而是最简单的：

```js
const controller = {
  addCount(name) {
    //通过model的接口增加目标对象的计数
    model.addTargetCatCount(name);
    controller.renderView();
  },
  changeTargetCat(name) {
    //通过model的接口修改目标索引
    model.setTargetCatIndex(name);
    controller.renderView();
  },
  init() {
    //通过model的接口获取相关数据
    const { getCatLists, getTargetCatObj } = model;
    //传参并命令view层初始化
    view.init(getCatLists(), getTargetCatObj());
  },
  renderView() {
    //传参并命令view层重新渲染
    view.render(model.getTargetCatObj());
  }
};
```

controller的设计其实是比较简陋的，只是一个包含了四个方法的对象。其中addCount对应点击加一的操作，changeTargetCat对应换猫的操作。上述两个方法其实是改变了数据的，而只要数据发生了变化，一律调用renderView重新渲染。
至此主要代码已经写完了，之后调用一下controller.init();，就可以开心的撸猫，完成需求了。
如果之前你动手实现了上述需求的话，不妨对比一下我们设计代码上的差别。也许你写的代码还是之前那种“面条式”代码，但它也是可用的啊，而且还不用这么多代码呢，长得也还算能维护的样子，为何要用这种繁琐的方式去阻止代码呢？
然而，按照之前说的,产品可能提更多的需求，下次可能会有点击到某个值时，自动切换猫，动态添加猫等等的新需求时，你现在的代码组织形式能否很快地完成需求？当日后修改某些需求时，不小心触发了潜藏的bug（经常有的情况），又是否能快速定位出问题并快速改好呢？
“面条式”代码经常是数据、视图与处理逻辑耦合起来的，很容易牵一发而动全身，当业务相当复杂的时候，开发可能还好说，维护简直是不可想象的。而你可能已经观察到了，遵循MVC设计的代码，虽然繁琐，但各层是完全分开的，尽管数据与视图可以直接调用对方的接口进行交互，但都是必须通过控制层来做统一处理。数据、视图与处理逻辑解耦之后，代码的可阅读性与可维护性都是一个飞跃。通过牺牲空间（多写代码）来换取代码的可维护性与可拓展性，这是一笔划算的买卖。

## 小结

说了半天，好像还没有说出为何前端需要使用框架。然而在复杂的项目中，你同意我通过MVC组织代码会比“面条式”代码好吗？如果同意的话，将我刚才代码中不变的部分抽象起来（组件通讯、报错处理等），想方设法提高渲染的性能（使用Virtual Dom），如果认为前端和其他不一样，数据和视图还是可以进行受控的交互(即MVVM)，那么整合起来，不就是一个框架了吗？
其实必须要承认，人的脑力是有限的，一款产品的需求可能是无限的。当这款产品已经让你无法掌握每个细节，每位参与开发的同学只能掌握局部细节，而其他部分只管调用是必然的事实。但是，如何确定其他部分可信，调用不会出bug呢？这时候就该使用框架了。框架较大程度上能约束与规范每位开发者的行为，不按照框架的规定很可能就会报错，这样多人协作就有了基本的保证。
但是，不是说使用框架就是最佳实践。当项目不复杂的时候（比如一次性的活动页），我们有足够能力去掌握项目的细节，那么使用框架反而不是好的选择。毕竟再好的框架在性能上都会有损失，而被框架的条条框框约束着，也总是令人不喜的。
简单地说：脑子不够，框架来凑；自己组织不好代码，靠框架来给我们组织。阅读至此殊为不易，感谢各位看官，希望本文对你有一点帮助，谢谢！