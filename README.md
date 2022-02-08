# -JavaScript-
《JavaScript设计模式与开发实践》读书笔记，并且将书中的一些比较老旧的代码，重新写了一下。该读书笔记只包含了常见的几种设计模式。个人感觉读了之后获益良多，于是形成此文。
# JavaScript设计模式与开发实践

封装可变化的，稳定不变的复用，意图为先

## SOLID五大原则

S 单一职责： 一个程序只做好一件事

O 开放封闭：对拓展开放，对修改封闭

L 里氏置换：子类能够覆盖父类，并能出现在父类出现的地方

I 接口独立

D 依赖导致： 使用只关注接口而不关注具体类的实现（组合、策略等）



## 单例模式

def: 保证一个类**只有一个**实例，并且提供一个**全局**的访问点来访问。实现一个单例，主要是用一个变量来标志是否已经创建过该对象，如果是的话，直接返回改对象。

应用场景： 只需要一个的对象，比如线程池、全局缓存、浏览器中的window对象

惰性单例：在需要的时候才创建对象实例。比如点击登录按钮的时候才创建登录窗口，而不是页面加载好的时候就创建。

第一次按按钮的时候就把登录弹窗给加载好，这里实现了惰性。

而当我们如果点击浮窗关闭按钮的时候，该节点会从页面上被删除掉，如果等到下一次还需要重新登录的时候，也只能重新创建节点。但是这样频繁的删除又创建节点，是不合理的。

因此在下面创建了一个变量来表示该节点是否已经被创建了，如果已经被创建，咱们把它的属性变成display:block让它出来，否则让它隐身即可。

这里的单例体现在我们用了一个闭包，将这个div变量给包起来，使得这个私有变量被封装在闭包内部作用域里面，防止全局变量污染，并且该变量也不会在函数运行完之后，被垃圾回收掉，可以很好的控制弹窗的隐身和显身。

```
<html>
<body>
    <button id="loginBtn">登录</button>
    <script>
        var createLogin = (function(){
            var div;
            return function() {
                if (!div) {
                    div = document.createElement('div');
                    div.innerHTML('我是登录浮窗');
                    div.style.display = none;
                    document.body.appendChild(div)
                }
                return div;
            }
        })()

            document.getElementById('loginBtn').onclick = function() {
                var loginLayer = createLogin();
                loginLayer.style.display = 'block'
            }
    </script>
</body>   
```

以上实现了一个惰性单例模式，这显然是违背了单一职责的原则。如果我们需要创建一个iframe呢？那我们还需要把以下判断重写一遍。

```
var obj;
if (!obj) {
	obj = xxx
}
```

所以，我们需要把**创建对象实例** 和 **管理单例**给分离出来。

```
<html>
<body>
    <button id="loginBtn">登录</button>
    <script>
        //管理单例
        var getSingle = function(fn) {
            var  result;
            return result || (result = fn.apply(this, arguments)) 
        }

        //创建login实例
        var createLogin = function(){
           
                var div = document.createElement('div');
                div.innerHTML('我是登录浮窗');
                div.style.display = none;
             
                return div;
        }

        var createSingleLogin = getSingle(createLogin)
        
        document.getElementById('loginBtn').onclick = function() {
            var loginLayer = createSingleLogin();
            loginLayer.style.display = 'block'
        }

        //创建iframe实例
        var createiframe = function(){
           var iframe = document.createElement('iframe');
           iframe.innerHTML('我是iframe');
           iframe.style.display = none;
        
           return iframe;
        }

        var createSingleiframe = getSingle(createiframe)
        
        document.getElementById('iframeBtn').onclick = function() {
            var iframeLayer = createSingleLogin();
            iframeLayer.style.display = 'block'
        }

    </script>
</body>   
```



## 策略模式

def: 定义一系列算法，把他们一个个封装起来，并且使他们可以互相替换。目的把算法使用和算法的实现分离开来。说人话就是，算法使用(不变)， 算法实现即策略(变)分开来。

最近在看了fiber，其中在比对旧树和新树在判断节点时——策略模式（宿主节点，类组件节点，函数节点diff）

举个简单🌰

```
//算法的实现
var stragies = {
            'S': function (salary){
                return salary * 4
            },
            'A': function (salary){
                return salary * 5
            },
            'B': function (salary){
                return salary * 6
            }
        }


//算法的使用
var calculateBonus = function(level, salary) {
	return stragies[level](salary)
}

console.log(calculateBonus('S', 4000));
console.log(calculateBonus('B',5000));


```

以上代码还有很大的改进空间，比如当一个实例有多个策略的时候，我们可以在算法的使用即calculateBonus改写一下，可以放置一个缓存区来进行存储策略，再一一使用这些策略。再举以下表单验证🌰

```
// 算法实现
var strategies = {
    isNonEmpty: function( value, errorMsg ){
        if ( value === '' ){ 
            return errorMsg ;
        } 
    },
    minLength: function( value, length, errorMsg ){ 
        if ( value.length < length ){
            return errorMsg;
        }
    },
    isMobile: function( value, errorMsg ){ // 手机号码格式
        if ( !/(^1[3|5|8][0-9]{9}$)/.test( value ) ){ 
            return errorMsg;
        } 
    }
};

//算法的使用
var Validator = function(){
    this.cache = []; // 保存校验规则
};
Validator.prototype.add = function( 
    var ary = rule.split( ':' ); 
    this.cache.push(function(){ 
        var strategy = ary.shift(); 
        ary.unshift( dom.value ); 
        ary.push( errorMsg ); 
        return strategies[strategy].apply(dom, ary);
    }); 
};
Validator.prototype.start = function(){
    for ( var i = 0, validatorFunc; validatorFunc = this.cache[ i++ ]; ){
        var msg = validatorFunc(); // 开始校验，并取得校验后的返回信息 
        if ( msg ){ // 如果有确切的返回值，说明校验没有通过
              return msg; 
        }
    }
};
var validataFunc = function(){
    var validator = new Validator(); // 创建一个 validator 对象
    /***************添加一些校验规则****************/
    validator.add( registerForm.userName, 'isNonEmpty', '用户名不能为空' );           
    validator.add( registerForm.password, 'minLength:6', '密码长度不能少于 6位');     
    validator.add( registerForm.phoneNumber, 'isMobile', '手机号码格式不正确' );
    var errorMsg = validator.start(); // 获得校验结果
    return errorMsg; // 返回校验结果 
}


//调用
var registerForm = document.getElementById( 'registerForm' ); registerForm.onsubmit = function(){
    var errorMsg = validataFunc(); // 如果 errorMsg 有确切的返回值，说明未通过校验 
    if ( errorMsg ){
        alert ( errorMsg );
        return false; // 阻止表单提交 
    }
};


参考链接：https://juejin.cn/post/6844903735496278029

```

## 代理模式

def: 当本体不便于访问的时候，提供一个替身对象来访问本体



这里介绍书中的三种代理模式

### 保护代理

def: 当请求带到标准的时候，则允许访问本体，否则拒绝请求。（由于比较简单这里就不代码演示了）

### 虚拟代理

def: 把开销很大的对象，延迟到真正需要他的时候再去创建。

介绍一种应用场景在开发的过程中，如果直接给img设置src的话，由于请求图片过大或者网络不佳，出现时滞问题，会出现空白，常见的做法是先用一张loading图片进行占位，等异步请求过来之后，再把它填充到img的src里面。下面介绍用虚拟代理实现图片预加载。

```
var myImage = (function() {
    var imgNode = document.createElement('img');
    document.body.appendChild(imgNode);
    return {
        setSrc: function(src) {
        imgNode.src = src;
    	}
    }
})();


//里面的this.src是
var proxyImage = (function() {
    var img = new Image;   //一个图片对象代理
    img.onload = function() { 
        //这里的this.src是下面绑定了img.src的
        myImage.setSrc(this.src)
	}
    return {	
        setSrc: function(src) {
            //真正加载好之前先加载loading图片
            myImage.setSrc('loading.gif')
            //先添加src属性
            img.src = src
        }
    }
})()

proxyImage.setSrc('https://imgcache.com/aaa.jpg')
```

这个时候有人可能会有疑问，为啥要整个proxy代理这么麻烦呢？直接把proxyImage的逻辑放到myImage还省事，这不香吗？实则不然，如果按照这么做，则违反了单一职责原则。代理负责了预加载图片，将预加载操作完成之后再把请求交给本体myImage。给img节点设置src和预加载两个功能分别隔离在两个对象里面。

**代理和本体接口的一致性**

观察以上代码就会发现proxyImage和myImage都暴露了一个setSrc的接口（闭包return出去的都是对外暴露接口，其他为内在属性）。

这样做的好处是

1、用户可以放心请求代理，只关心是否得到了想要的结果。而代理接手请求的过程中是完全透明的。

2、可能过了一段时间之后，网速快到已经不需要预加载了，不需要改本体myImage的内在逻辑

更多的应用场景有

合并http请求：为了防止点击过快，在代理中设置两秒后合并请求一起发送，这样能够减轻服务器的压力。个人认为，防抖节流效果跟这个有异曲同工之处。

### 缓存代理

def: 把开销大的运算结果放到缓存中，等到下一次再出现该缓存对象的时候，直接可以从缓存中取。举个例子

```
var mult = function() {
    var a = 1;
    for(let i=0; i < arguments.length;i++) {
    	a = a * arguments[i]
    }
    return a;
}

var cacheFactory = (function(fn){
    var cache = {};
    return function() {
        var args = Array.prototype.join.call(arguments,',');
        if (cache[args]){
            return cache[args];
        }else{
            // this是window
            return fn.apply(this,arguments);
        }
    }
})()

cacheFactory(mult(1,2,4,5))
```

## 发布订阅者模式

def: 定义对象间的一种一对多的依赖关系，当对象状态发生变化，所有依赖与它的对象都能被通知。

1、主要广泛应用于异步编程中。比如dom监听事件、ajax请求后的回调等等

2、发布和订阅对象分别互不干扰，当发布者改变时不会影响订阅者，当订阅者改变时不改变发布者

优点：1、时间上解耦  2、对象之间解耦

注意!!!!: 

1、订阅一个消息，如果该消息最终未发生，这个订阅者会在内存里面，因此会消耗内存。

2、该模式虽然可以弱化对象之间的关系，但是如果过度使用，对象之间的关系深埋在背后，会导致程序难维护



下面用es6写了一个通用的发布订阅者模式

```
class Pubsub {
    constructor() {
        this.cache = {}
    }
     //{key: [fn,fn,fn], key2: [fn,fn,fn]}

    // listen是放到cache里面
    listen(key, fn) {
        if (!this.cache[key]) {
            this.cache[key] = [fn]
        }else{
            this.cache[key].push(fn)
        }
    }

    // 根据name,调传进来的fn
    trigger(name,...args) {
        if (this.cache[name]) {
            //  // 创建副本，如果回调函数内继续注册相同事件，会造成死循环
            var fns = this.cache[name]
            for(let fn of fns) {
                fn(...args)

            }
        }
        
    }
}

// class Salesoffice extends Pubsub {
//     constructor() {
//         super() ES6在继承中强制要求，必须在子类调用super，因为子类的this是由父类得来的。
            //super等价于parent.prototype.constructor.call(sub)。
//     }
// }

var pubsub = new Pubsub()

pubsub.listen('suq88',function(price,squ) {
    console.log('价格',price);
    console.log('平方',squ);
})

pubsub.listen('suq108',function(price,squ) {
    console.log('价格',price);
    console.log('平方',squ);
})

pubsub.trigger('suq108',1000,108)
pubsub.trigger('suq88',19900,1088)
```

从需求来看，不一定需要订阅才发布。举个例子，像QQ的离线消息，离线消息放在了服务器之中，等到下一次登入上线的时候可以重新收到这条消息。再比如，获取到信息之后才渲染用户导航模块，而获取用户信息操作是一个ajax异步请求，当ajax请求成功返回之后发布事件，之前订阅的事件可以收到信息。

但是这只是一个理想的状况，有可能ajax返回地比较快，用户导航的模块还没有被加载好（也就是还没有订阅相应的事件），所以需要有先发布后订阅的能力。这个时候，我们可以采取以下方法：

（像QQ离线未读消息一样，只会被重读一次，这种操作也只能进行一次）

1、建立一个存放离线事件的堆栈

2、事件发布的时候如果还没有订阅的，暂时把发布时间这个动作包裹在一个函数里面，再放入堆栈中

3、等有对象来订阅的时候，遍历改堆栈并且执行这些包装函数，重新发布里面的时间

## 命令模式

应用场景：向某些对象发送请求，但是不知道请求者是谁，也不知道被请求操作是什么。比如，当客人点菜时，不知道做菜的人是谁以及这道菜的做法是什么。

接收者被当成command对象的属性保存起来，同时约定执行命令操作调用command.execute

```
 button1 = document.getElementById('button1')

        //设置命令
        const setCommand = (button, command) {
            button.onclick = () => {
                command.execute()
            }
        }

        //业务逻辑

        //菜的做法
        const MenuBar = {
            refresh: function() {
                console.log('refresh');
            },
        }

        //给谁做
        const RefreshMenuBarCommand = (receiver) => {
            return {
                execute: () => {
                    receiver.refresh()
                }
            }
        }

        const refreshMenuBarCommand = RefreshMenuBarCommand(MenuBar)

        //绑定按钮命令
        setCommand(button1, refreshMenuBarCommand)

```

书中还举了撤销和重做的做法：

1、棋子游戏，当悔棋到第n步的时候，可以把执行过的下棋命令存储在一个历史列表（array）中，然后倒序循环输出依次执行这些undo命令，直到执行到第n步。

2、但是当Canvas操作时，命令模式很难undo某条线，这个时候可以先清除canvas再把之前执行过的命令全部执行一遍，同样用一个历史列表可以做到。

当撤回完时候需要进行通知，我们可以用发布订阅者模式或者回调函数来通知进行下一步操作

另外还能够使用宏命令来进行一组命令的组合

```
 const colse = {
     execute: function() {
     	console.log('关门');
     }
 }

const openPC = {
	execute: function() {
		console.log('开电脑');
	}
}

const openQQ = {
    execute: function() {
        console.log('打开QQ');
        }
}

//存到缓存里面，当执行的时候再迭代出来
class MacroCommand {
    constructor() {
        this.commandList: []
    }
    
    add: function(command) {
    	this.commandList.push(command)
    }
    
    execute: function() {
        for(let i=0; i < this.commandList.length; i++) {
        	this.commandList[i].execute()
        }
    }

}

var macroCommand = MacroCommand()
macroCommand.add(close)
macroCommand.add(openPC)
macroCommand.add(openQQ)

macroCommand.execute()
```

## 组合模式

def: 用小的子对象来构建更大的对象，将对象组合成树形结构，以表示“**部分—整体”**的层次结构。**统一对待**树中所有对象。

从我们上面命令模式的宏命令场景来说也是一个组合



以上只是一个简单的树形组合。更复杂的有以下这样的组合对象和叶对象。


这个时候在宏命令直接深度递归

注意！：

1、组合模式是has-a（聚合）关系，不是is-a（继承）关系

2、组合对象和叶对象需要有一样的接口，并且对叶对象操作需要有一致性。比如公司要给每个人元旦发1000块，这可以用组合，但是要给今天生日的员工发生日邮件，就不能用，除非把今天生日的员工挑选出来。

3、双向映射关系。发钱是从公司——部门——小组——个人这样，但是某些员工如果从属两个小组那就gg，这可能收到两份钱。这种情况下，用双向映射关系，也就是把增加集合存对方的引用。这样的话修改和删除一个对象就会变得难。这种情况下不适合用组合。

4、缺点：每个对象有可能看起来跟其他对象差不多，运行才能看出区别 -> 代码难理解。创建太多对象 -> 系统负担不起

## 装饰者模式

def：不改变对象的基础上，程序运行期间，给对象动态添加职责 -> 不会影响这个类中派生的其他对象

下面介绍一种用AOP装饰函数，其主要就是在通过在函数执行前或者执行后添加新的函数来添加新功能。

```
Function.prototype.after = function(afterFn) {
    return () => {
        let ret = this( arguments);
        //绑定回原来的函数showLogin，那么arguments就能够用它的
        afterFn.apply(this,arguments);
        return ret;
    }
}
//上面可以参考箭头函数的this指向
// Function.prototype.after = function(afterFn) {
//     var _self = this;
//     return function() {
//         let ret = _self.apply(this, arguments);
//         afterFn.apply(_self,arguments);
//         return ret;
//     }
// }

Function.prototype.before = function(beforeFn) {
    return () => {
        beforeFn.apply(this,arguments);
        return this(arguments)
    }
}

let showLogin = function() {
	console.log('login');
}

const log = function() {
	console.log('after');
}

showLogin = showLogin.after(log); //这里改写了函数，所以上面要重新把原来的函数return出来

document.getElementById('loginBtn').onclick = showLogin
```

应用场景：表单验证（可以直接在beforeFn那里加判断，如果等于true则执行后面的函数，否则直接return） ， Ajax请求前带上token (个人认为这跟axios的拦截器有异曲同工之妙)，数据上报、统计函数执行时间

```
Function.prototype.before = function(beforeFn) {
    return () => {
    	beforeFn.apply(this,arguments);
   	 	return this(arguments)
    }
}

var ajax = function(type, url, param) {
	console.log(param);
}

var getToken = function() {
	return 'Token'
}

ajax.before(function(type, url, param) {
	param.Token = getToken()
})

ajax(get,'http://xxxx.com', {name:'seve'}) 

//{name:'seven', Token:'Token'}
```

## 适配器模式

def: 主要解决两个已有接口之间不匹配的问题

```
const aMap = {
	show: () =>{
		console.log('开始渲染地图A')
	}
}

const bMap = {
	display: () =>{
		console.log('开始渲染地图B')
	}
}

const bMapAdapter = {
	show: () => {
		return bMap.display()
	}
}

const renderMap = (map) =>{
	if (map.show instance of Function) {
		map.show()
	}
}

renderMap(aMap);
renderMap(bMapAdapter);

```

