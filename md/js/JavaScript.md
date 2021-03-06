

# **渲染流程**

- **DOM 生成、样式计算**和**布局**

  - 在 HTML 页面内容被提交给渲染引擎 后，渲染引擎首先将 HTML 解析为浏览器可以理解的 DOM;
  - 然后根据 CSS 样式表，计算出 DOM 树所有节点的样式;
  - 接着又计算每个元素的几何坐标位置，并将这些信息保存在布局树中。

- **分层**

  - 因为页面中有很多复杂的效果，为了更加方便地实现这些效果，**渲染引擎还需要为特定的节点生成专用的图层，并生成一棵对应的图层树**。
    - 渲染引擎给页面分了很多图层，这些图层按照一定顺序叠加在一起，就形 成了最终的页面
    - 通常情况下，**并不是布局树的每个节点都包含一个图层，如果一个节点没有对应的层，那么 这个节点就从属于父节点的图层**。
  - 通常满足下面两点中任 意一点的元素就可以被提升为单独的一个图层。
    - **第一点，拥有层叠上下文属性的元素会被提升为单独的一层。**
      - 明确定位属性的元素、定义透明属性的元素、使用 CSS 滤镜的元素等， 都拥有层叠上下文属性。
    - **第二点，需要剪裁(clip)的地方也会被创建为图层。**
      - 把 div 的大小限定为 200 * 200 像素，而 div 里面的文字内容比较多，文字所 显示的区域肯定会超出 200 * 200 的面积，这时候就产生了剪裁，渲染引擎会把裁剪文字 内容的一部分用于显示在 div 区域。
      - 出现这种裁剪情况的时候，渲染引擎会为文字部分单独创建一个层，如果出现滚动条，滚动 条也会被提升为单独的层。

- **图层绘制**

  - 渲染引擎会把一个图层的绘制拆分成很多小的**绘制指令**，然后再 把这些指令按照顺序组成一个待绘制列表
  - 你也可以打开“开发者工具”的“Layers”标签，选择“document”层，来实际体验下绘 制列表

- **栅格化(raster)操作**

  - 绘制列表只是用来记录绘制顺序和绘制指令的列表，而实际上绘制操作是由渲染引擎中的**合成线程**来完成的。
  - 当图层的绘制列表准备好之后，主线程会把该绘制列表**提交(commit)**给合成线程
    - 通常一个页面可能很大，但是用户只能看到其中的一部分，我们把用户可以看到的这个部分 叫做**视口**(viewport)。
    - 在有些情况下，有的图层可以很大，比如有的页面你使用滚动条要滚动好久才能滚动到底 部，但是通过视口，用户只能看到页面的很小一部分，所以在这种情况下，要绘制出所有图 层内容的话，就会产生太大的开销，而且也没有必要。
    - 基于这个原因，**合成线程会将图层划分为图块(tile)**，这些图块的大小通常是 256x256 或者 512x512。
  - 然后**合成线程会按照视口附近的图块来优先生成位图，实际生成位图的操作是由栅格化来执行的。所谓栅格化，是指将图块转换为位图**。
  - 渲染进程把生成图块的指令发送给 GPU，然后在 GPU 中执行生成图块 的位图，并保存在 GPU 的内存中。

- **合成和显示**

  - 一旦所有图块都被光栅化，合成线程就会生成一个绘制图块的命令——“DrawQuad”， 然后将该命令提交给浏览器进程。
  - 浏览器进程里面有一个叫 viz 的组件，用来接收合成线程发过来的 DrawQuad 命令，然后 根据 DrawQuad 命令，将其页面内容绘制到内存中，最后再将内存显示在屏幕上。
  - 到这里，经过这一系列的阶段，编写好的 HTML、CSS、JavaScript 等文件，经过浏览器 就会显示出漂亮的页面了。

- 一个完整的渲染流程大致可总结为如下

  - 渲染进程将 HTML 内容转换为能够读懂的**DOM 树**结构。

  - 渲染引擎将 CSS 样式表转化为浏览器可以理解的**styleSheets**，计算出 DOM 节点的

    样式。

  - 创建**布局树**，并计算元素的布局信息。

  - 对布局树进行分层，并生成**分层树**。

  - 为每个图层生成**绘制列表**，并将其提交到合成线程。

  - 合成线程将图层分成**图块**，并在**光栅化线程池**中将图块转换成位图。

  - 合成线程发送绘制图块命令**DrawQuad**给浏览器进程。

  - 浏览器进程根据 DrawQuad 消息**生成页面**，并**显示**到显示器上。





# **变量提升**

- 三个结论

  - 在执行过程中，若使用了未声明的变量，那么 JavaScript 执行会报错。
  - 在一个**变量**定义之前使用它，不会出错，但是该变量的值会为 undefined，而不是定义时的值。
  - 在一个**函数**定义之前使用它，不会出错，且函数能正确执行。

- 变量提升

  - JavaScript 中的**声明** 和**赋值**。

    - **变量**的声明和赋值，**函数**的声明和赋值

    - ```javascript
       var myname // 声明部分
       myname = 'gek' // 赋值部分
       var myname = 'gek' // 变量的声明和赋值
        
      // 函数声明
      function foo(){
       console.log('foo') 
      }
      // 函数的声明和赋值
      var bar = function(){  
        console.log('bar') 
      }
      ```

  - 所谓的变量提升，**是指在 JavaScript 代码执行过程中，JavaScript 引擎把变量的声明部分和函数的声明部分提升到代码开头的“行为”。变量被提升后，会给变量设置默认值**，这个默认值就是我们熟悉的 undefined。

- JavaScript 代码的执行流程

  - **实际上变量和函数声明在代码里的位置是不会改变的，而且是在编译阶段被 JavaScript 引擎放入内存中**。
  - 一段 JavaScript 代码在执行之前需要被 JavaScript 引擎编译，**编译**完成之后，才会进入**执行**阶 段。
  - 输入一段代码，经过编译后，会生成两部分内容:**执行上下文 (Execution context)和可执行代码**。
    - **编译阶段**
      - **执行上下文是 JavaScript 执行一段代码时的运行环境**，比如调用一个函数，就会进入这个 函数的执行上下文，确定该函数在执行期间用到的诸如 this、变量、对象以及函数等。
      - 在执行上下文中存在一个**变量环境的对象** (Viriable Environment)，该对象中保存了变量提升的内容
      - 接下来 JavaScript 引擎会把声明以外的代码编译为字节码
    - **执行阶段**
      - JavaScript 引擎开始执行“可执行代码”，按照顺序一行一行地执行。

- 代码中出现相同的变量或者函数怎么办

  - 完整执行流程
    - **首先是编译阶段**，**第二个 showName 函数会将第一个 showName 函数覆 盖掉**。这样变量环境中就只存在第二个 showName 函数了。
    - **接下来是执行阶段**。先执行第一个 showName 函数，但由于是从变量环境中查找 showName 函数，而变量环境中只保存了第二个 showName 函数，所以最终调用的是第二个函数。
  - **一段代码如果定义了两个相同名字的函数，那么最终生效的是最后一个函数**







# **调用栈**

- 到底什么样的代码才算符合规范
  - 哪些情况下代码才算是“一段”代码，才会在执行之前就进行编译并创建执行上下文。一般说来，有这么三种情况:
    - 当 JavaScript 执行**全局代码**的时候，会编译全局代码并创建全局执行上下文，而且在 整个页面的生存周期内，全局执行上下文只有一份。
    - 当**调用一个函数**的时候，函数体内的代码会被编译，并创建函数执行上下文，一般情况 下，函数执行结束之后，创建的函数执行上下文会被销毁。
    - 当使用 **eval 函数**的时候，eval 的代码也会被编译，并创建执行上下文。
- **调用栈就是用来管理函数调用关系的一种数据结构**。
  - 函数调用
    - 先是创建了一个 add 函数，接着在代码的最下面又调用了该函数。
      - 在执行到函数 add() 之前，JavaScript 引擎会为上面这段代码创建全局执行上下文，包含 了声明的函数和变量
      - 执行上下文准备好之后，便开始执行全局代码，当执行到 add 这儿时，JavaScript 判断这 是一个函数调用，那么将执行以下操作
        - 首先，从**全局执行上下文**中，取出 add 函数代码。
        -  其次，对 add 函数的这段代码进行编译，并创建**该函数的执行上下文**和**可执行代码**。
        -  最后，执行代码，输出结果。
    - 就这样，当执行到 add 函数的时候，我们就有了两个执行上下文了——全局执行上下文和 add 函数的执行上下文。
  - 栈结构
    - **JavaScript 引擎正是利用栈的这种结构来管理执行上下文的**。在执行上下文创建好后， JavaScript 引擎会将执行上下文压入栈中，通常把这种用来管理执行上下文的栈称为**执行 上下文栈**，又称**调用栈**。
    - **调用栈是 JavaScript 引擎追踪函数执行的一个机制**，当一次有多 个函数被调用时，通过调用栈就能够追踪到哪个函数正在被执行以及各函数之间的调用关 系。
- **在开发中，如何利用好调用栈**
  - 点击“Source”标签，选择 JavaScript 代码的页面，加上断点，并刷新页面。你可以看到执行到 add 函数时，执行流程就暂停了，这时可以通过右边“call stack”来查看当前的调用栈的情况。栈的最底部是 anonymous，也就是全局的函数入口。
  - 除了通过断点来查看调用栈，你还可以使用 console.trace() 来输出当前的函数调用关系
-  栈溢出
  - 当入栈的执行上下文超过一定数目，JavaScript 引 擎就会报错，我们把这种错误叫做**栈溢出**。
  - 可以使用一些方法来避免或者解决栈溢出的问题，比如把递归调 用的形式改造成其他形式，或者使用加入定时器的方法来把当前任务拆分为其他很多小任 务。







# **块级作用域**

- **正是由于 JavaScript 存在变量提升这种特性， 从而导致了很多与直觉不符的代码，这也是 JavaScript 的一个重要设计缺陷**。

  - 虽然 ECMAScript6(以下简称 ES6)已经通过引入块级作用域并配合 let、const 关键字， 来避开了这种设计缺陷，但是由于 JavaScript 需要保持向下兼容，所以变量提升在相当长 一段时间内还会继续存在。

- 作用域

  - 作用域是指在程序中定义变量的区域，该位置决定了变量的生命周期。通俗地理解，**作用域 就是变量与函数的可访问范围**，即作用域控制着变量和函数的可见性和生命周期。
  - 块级作用域就是使用一对大括号包裹的一段代码，比如函数、判断语句、循环语句， 甚至单独的一个{}都可以被看作是一个块级作用域。

- **变量提升所带来的问题**

  - 变量容易在不被察觉的情况下被覆盖掉

  - 本应销毁的变量没有被销毁

    - ```javascript
      // 在创建执行上下文阶段，变量 i 就已经被提升了，所以当 for 循环结束之后，变量 i 并没有被销毁。
      function foo(){
       for (var i = 0; i < 7; i++) { 
       }
       console.log(i);
      }
       foo()
      ```

- **ES6** **是如何解决变量提升带来的缺陷**

  - **ES6 引入了 let 和const 关键字**，从而使 JavaScript 也能像其他语言一样拥有了块级作用域。
    - 使用 let 关键字声明的变量是可以被改变 的，而使用 const 声明的变量其值是不可以被改变的。但不管怎样，两者都可以生成块级 作用域。

- **JavaScript** **是如何支持块级作用域的**

  - **站在执行上下文的角度**
  - 代码的执行流程
    - **第一步是编译并创建执行上下文**
      - 函数内部通过 var 声明的变量，在编译阶段全都被存放到**变量环境**里面了。
      - 通过 let 声明的变量，在编译阶段会被存放到**词法环境(Lexical Environment)**中。 
      - 在函数的作用域内部，通过 let 声明的变量并没有被存放到词法环境中。
    - 接下来，**第二步继续执行代码**，当进入函数的作用域块时，作用域块中通过 let 声明的变量，会被存放在 词法环境的一个单独的区域中，这个区域中的变量并不影响作用域块外面的变量，**比如在作用域外面声明了变量 b，在该作用域块内部也声明了变量 b，当执行到作用域内部时，它们都是独立的存在。**
    - **再接下来，当执行到作用域块中的console.log(a)这行代码时，就需要在词法环境和变量环境中查找变量 a 的值了**，具体查找方式是:沿着词法环境的栈顶向下查询，如果在词法环境中的某个块中查找到了，就直接返回给 JavaScript 引擎，如果没有查找到，那么继 续在变量环境中查找。

  

  

  



# **作用域链和闭包**

- **作用域链**

  - ```javascript
    function bar() {
     console.log(myName) 
    }
     function foo() {
     var myName = " abc "
     bar()
    }
     var myName = " qaz " 
    foo()
    
    ```

  - 其实在每个执行上下文的变量环境中，都包含了一个外部引用，用来指向外部的执行上下文，我们把这个外部引用称为**outer**。

    - 当一段代码使用了一个变量时，JavaScript 引擎首先会在“当前的执行上下文”中查找该变量，比如上面那段代码在查找 myName 变量时，如果在当前的变量环境中没有查找到，那么 JavaScript 引擎会继续在 outer 所指向的执行上下文中查找

  - bar 函数和 foo 函数的 outer 都是指向全局上下文的，这也就意味着如 果在 bar 函数或者 foo 函数中使用了外部变量，那么 JavaScript 引擎会去全局执行上下文 中查找。我们把这个查找的链条就称为**作用域链**。

- **词法作用域**

  - 词法作用域就是指作用域是**由代码中函数声明的位置来决定的**，所以词法作用域是静态的作 用域，通过它就能够预测代码在执行过程中如何查找标识符。
  - foo 函数调用的 bar 函数，那为什么 bar 函数的外部引用是全局执行上下文，而不是 foo 函数的执行上下 文?
    - foo 和 bar 的上级作用域都是全局作用域，所以如果 foo 或者 bar 函数使用了一个它们没有定义的变量，那么它们会到全局作用域去查找。也就是说，**词法作用域是代码阶段就决定好的，和函数是怎么调用的没有关系**。

- **块级作用域中的变量查找**

  - 在编写代码的时候，如果你使用了一个在当前作用域中不存在的 变量，这时 JavaScript 引擎就需要按照作用域链在其他作用域中查找该变量。

- **闭包**

  - ```javascript
    function foo() {
    	var myName = " abc " 
    	let test1 = 1
    	const test2 = 2
    	var innerBar = { 
    		getName:function(){
    			console.log(test1)
      	  return myName
    		},
    		setName:function(newName){ 
    			myName = newName
    	}
      return innerBar
    } 
    
    var bar = foo()
    bar.setName(" qaz ")
    bar.getName()
    console.log(bar.getName())
    ```

  - foo 函数执行完成之后，其执行上下文从栈顶弹出了，但是由于返回的 setName 和 getName 方法中使用了 foo 函数内部的变量 myName 和 test1，所以这两 个变量依然保存在内存中。

  - **根据词法作用域的规则，内部函数 getName 和 setName 总是可以访问它们的外部函数 foo 中的变量**，所以当 innerBar 对象返回给全局变量 bar 时，虽然 foo 函数已经执行结 束，但是 getName 和 setName 函数依然可以使用 foo 函数中的变量 myName 和 test1。

  - 在 JavaScript 中，根据词法作用域的 规则，内部函数总是可以访问其外部函数中声明的变量，当通过调用一个外部函数返回一个 内部函数后，即使该外部函数已经执行结束了，但是内部函数引用外部函数的变量依然保存在内存中，我们就把这些变量的集合称为闭包。**比如外部函数是 foo，那么这些变量的集合就称为 foo 函数的闭包。**

- **闭包是怎么回收的**

  - 通常，如果引用闭包的函数是一个**全局变量**，那么闭包会一直存在直到页面关闭;但如果这 个闭包以后不再使用的话，就会造成内存泄漏。
  - 如果引用闭包的函数是个**局部变量**，等函数销毁后，在下次 JavaScript 引擎执行垃圾回收 时，判断闭包这块内容如果已经不再被使用了，那么 JavaScript 引擎的垃圾回收器就会回收这块内存。
  - 所以在使用闭包的时候，你要尽量注意一个原则:**如果该闭包会一直使用，那么它可以作为 全局变量而存在;但如果使用频率不高，而且占用内存又比较大的话，那就尽量让它成为一 个局部变量**。







# **this**

- **在对象内部的方法中使用对象内部的属性是一 个非常普遍的需求**——**this 机制**

- **JavaScript** **中的** **this** **是什么**

  - 执行上下文中 包含了变量环境、词法环境、外部环境，但其实还有一个 this 没有提及
  - **this 是和执行上下文绑定的**，也就是说每个执行上下文中都有一个 this

- **全局执行上下文中的** **this**

  - 全局执行上下文中的 this 是指向window 对象的。这也是 this 和作用域链的唯一交点，作用域链的最底端包含了 window 对象，全局执行上下文中的 this 也是指向 window 对象。

- **函数执行上下文中的** **this**

  - 我们在 foo 函数内部打印出来 this 值，执行这段代码，打印出来的也是 window 对象，这 说明在默认情况下调用一个函数，其执行上下文中的 this 也是指向 window 对象的。

- 能不能设置执行上下文中的 this 来指向其他对象呢?通常情况下，有下面三种方式来设置函数执行上下文中的 this 值。

  - **通过函数的** **call** **方法设置**

    - 你可以通过函数的**call**方法来设置函数执行上下文的 this 指向

    - ```javascript
      let bar = {
      	myName : " aaa ", 
         test1 : 1
      }
      function foo(){
      this.myName = " ccc " 
      }
      foo.call(bar)
      console.log(bar)
      console.log(myName)
      ```

  - **通过对象调用方法设置**

    - ```javascript
      var myObj = {
      name : " aaa ", 
        showThis: function(){
          console.log(this)
      	}
      }
      myObj.showThis()
      ```

    - 使用对象来调用其内部的一个方法，该方法的 this 是**指向对象本身**的。

  - **通过构造函数中设置**

    - ```javascript
      function CreateObj(){
      	this.name = " aaa " 
      }
      var myObj = new CreateObj()
      ```

    - 其实，当执行 new CreateObj() 的时候，JavaScript 引擎做了如下四件事:

      - 首先创建了一个空对象 tempObj;
      - 接着调用 CreateObj.call 方法，并将 tempObj 作为 call 方法的参数，这样当 CreateObj 的执行上下文创建时，它的 this 就指向了 tempObj 对象;
      - 然后执行 CreateObj 函数，此时的 CreateObj 函数执行上下文中的 this 指向了 tempObj 对象;
      - 最后返回 tempObj 对象。

- this 的设计缺陷以及应对方案

  - **嵌套函数中的** **this** **不会从外层函数中继承**

    - ```javascript
      // 函数 bar 中的 this 指向的是全局 window 对象，而函数 showThis 中的 this 指向的是 myObj 对象
      var myObj = {
      	name : " aaa ",
        showThis: function(){
      		console.log(this)
      		function bar(){console.log(this)}}
       		bar()
       }
      }
      myObj.showThis()
      ```

    - **你可以通过小技巧来解决这个问题**，比如在 showThis 函数中**声明一个变量 self 用来 保存 this**

    - **你也可以使用 ES6 中的箭头函数来解决这个问题**

      - ```javascript
        var myObj = {
        name : " aaa ", 
          showThis: function(){
        		console.log(this) 
            var bar = ()=>{
        			this.name = " bbb " 
              console.log(this)
        		bar() 
          }
        }
        myObj.showThis() 
        console.log(myObj.name) 
        console.log(window.name)
        ```

  - **普通函数中的** **this** **默认指向全局对象** **window**

    - 如果要让函数执行上下文中的 this 指向某个对象，最好的方式是通过 call 方法来显示调用。
    - 这个问题可以通过设置 JavaScript 的“严格模式”来解决。在严格模式下，默认执行一个 函数，其函数的执行上下文中的 this 值是 undefined，这就解决上面的问题了。





# **栈空间和堆空间**

- JavaScript 是什么类型的语言

  - **在使用之前 就需要确认其变量数据类型的称为静态语言**。
  - **相反地，我们把在运行过程中需要检查数据类型的语言称为动态语言**。
  - **支持隐式类型转换的语言称为弱类型语言， 不支持隐式类型转换的语言称为强类型语言**。
  - **JavaScript 是一种弱类型的、动态的语言**

- JavaScript 的数据类型

  - JavaScript 中的数据类型一种有 8 种
  - 第一点，使用 typeof 检测 Null 类型时，返回的是 Object。这是当初 JavaScript 语言的 一个 Bug，一直保留至今，之所以一直没修改过来，主要是为了兼容老的代码。
  - 第二点，Object 类型比较特殊，它是由上述 7 种类型组成的一个包含了 key-value 对的数 据类型。vaule 可以是任何类型，包括 函数，这也就意味着你可以通过 Object 来存储函数，Object 中的函数又称为方法
  - 第三点，我们把前面的 7 种数据类型称为**原始类型**，把最后一个对象类型称为**引用类型**（object）， 之所以把它们区分为两种不同的类型，是因为它们在内存中存放的位置不一样。

- **内存空间**

  - 在 JavaScript 的执行过程中， 主要有三种类型内存空间，分别是**代码空间、栈空间**和**堆空间**。	

    - 其中的代码空间主要是存储可执行代码的

  - **栈空间和堆空间**

    - **原始类型的数据值都是直接保存在“栈”中的，引用类型的值是存放在“堆”中的**。

      - ```javascript
        function foo(){
        var a = " aaa "
        var b = a
        // 当 JavaScript 需要访问该数据的时候，是通过栈中的引用地址来访问的
        var c = {name:" aaa "} 
        var d = c
        }
        foo()
        ```

      - 对象类型是存放在堆空间的，在栈空间中只是保留了对象的引用地址

    - JavaScript 引擎需要用栈来维护程序执行期间上下文的状态， 如果栈空间大了话，所有的数据都存放在栈空间里面，那么会影响到上下文切换的效率，进 而又影响到整个程序的执行效率。

      - 比如文中的 foo 函数执行结束了，JavaScript 引擎需要 离开当前的执行上下文，只需要将指针下移到上个执行上下文的地址就可以了，foo 函数执 行上下文栈区空间全部回收

    - **通常情况下，栈空间都不会设置太大，主要用来存放一些原始类型的小数据，堆空间很大，能存放很多大的数据**。

    - **原始类型的赋值会完整复制变量值，而引用类型的赋值是复制引用地址**。

- **闭包**

  - ```javascript
    // 当执行这段代码的时候，你应该有过这样的分析:由于变量 myName、test1、test2 都是原始类型数据，所以在执行 foo 函数的时候，它们会被压入到调用栈中;
    // 当 foo 函数执行 结束之后，调用栈中 foo 函数的执行上下文会被销毁，其内部变量 myName、test1、 test2 也应该一同被销毁。
    
    function foo() {
    	var myName = " abc " 
    	let test1 = 1
    	const test2 = 2
    	var innerBar = { 
    		getName:function(){
    			console.log(test1)
      	  return myName
    		},
    		setName:function(newName){ 
    			myName = newName
    	}
      return innerBar
    } 
    
    var bar = foo()
    bar.setName(" qaz ")
    bar.getName()
    console.log(bar.getName())
    ```

  - **但是当 foo 函数的执行上下文销毁时，由于 foo 函数产生了闭包，所以变量 myName 和 test1 并没有被销毁，而是保存在内存中**，那么应该如何解释这 个现象呢?

  - 要解释这个现象，我们就得站在内存模型的角度来分析这段代码的执行流程。

    - 当 JavaScript 引擎执行到 foo 函数时，首先会编译，并创建一个空执行上下文。
    - 在编译过程中，**遇到内部函数 setName**，JavaScript 引擎还要对内部函数做一次快速的词法扫描，发现该内部函数引用了 foo 函数中的 myName 变量，由于是内部函数引用了外部函数的变量，所以 JavaScript 引擎**判断这是一个闭包，于是在堆空间创建换 一个“closure(foo)”的对象(这是一个内部对象，JavaScript 是无法访问的)，用来 保存 myName 变量。**
    - **接着继续扫描到 getName 方法时，发现该函数内部还引用变量 test1，于是 JavaScript 引擎又将 test1 添加到“closure(foo)”对象中。这时候堆中 的“closure(foo)”对象中就包含了 myName 和 test1 两个变量了。**
    - 由于 test2 并没有被内部函数引用，所以 test2 依然保存在调用栈中。
    - **当执行到 foo 函数时，闭包就产生了;当 foo 函数执行结束之 后，返回的 getName 和 setName 方法都引用“clourse(foo)”对象**，所以即使 foo 函数 退出了，“clourse(foo)”依然被其内部的 getName 和 setName 方法引用。所以在下次 调用bar.setName或者bar.getName时，创建的执行上下文中就包含 了“clourse(foo)”。

  - **总的来说，产生闭包的核心有两步:第一步是需要预扫描内部函数;第二步是把内部函数引 用的外部变量保存到堆中。**







# **垃圾回收**

- 不同语言的垃圾回收策略

  - 如 C/C++ 就是使用手动回收策略，**何时分配内存、何时销毁内存都是由代码控制的**
  - 另外一种使用的是自动垃圾回收的策略，如 JavaScript、Java、Python 等语言，**产生的垃 圾数据是由垃圾回收器来释放的**，并不需要手动通过代码来释放。

- **调用栈中的数据是如何回收的**

  - ```javascript
    function foo(){ 
    	var a = 1
    	var b = {name:" zz "} 
    	function showName(){
    		var c = " aa "
    		var d = {name:" aa "}  
    	}
    	showName() 
    }
    foo()
    ```

  - 如果执行到 showName 函数时，那么 JavaScript 引 擎会创建 showName 函数的执行上下文，并将 showName 函数的执行上下文压入到调用栈中，最终执行到 showName 函数时，其调用栈就如上图所示。与此同时，还有一个**记录 当前执行状态的指针(称为 ESP)**，指向调用栈中 showName 函数的执行上下文，表示 当前正在执行 showName 函数。

  - 接着，当 showName 函数执行完成之后，函数执行流程就进入了 foo 函数，那这时就需 要销毁 showName 函数的执行上下文了。ESP 这时候就帮上忙了，JavaScript 会将 ESP 下移到 foo 函数的执行上下文，**这个下移操作就是销毁 showName 函数执行上下文的过 程**。

  - 当 showName 函数执行结束之后，ESP 向下移动到 foo 函数的执行上 下文中，上面 showName 的执行上下文虽然保存在栈内存中，但是已经是无效内存了。比 如当 foo 函数再次调用另外一个函数时，这块内容会被直接覆盖掉，用来存放另外一个函 数的执行上下文。

  - 当一个函数执行结束之后，**JavaScript 引擎会通过向下移动 ESP 来销毁该函数保 存在栈中的执行上下文**。

- **堆中的数据是如何回收的**

  - 当上面那段代码的 foo 函数执行结束之后， ESP 应该是指向全局执行上下文的，那这样的话，showName 函数和 foo 函数的执行上下 文就处于无效状态了，不过保存在堆中的两个对象依然占用着空间。
  - **要回收堆中的垃圾数据，就需要 用到 JavaScript 中的垃圾回收器了**。
  - 在 V8 中会把堆分为**新生代**和**老生代**两个区域，**新生代中存放的是生存时间短的对 象，老生代中存放的生存时间久的对象**。
  - 新生区通常只支持 1~8M 的容量，而老生区支持的容量就大很多了。对于这两块区域， V8 分别使用两个不同的垃圾回收器，以便更高效地实施垃圾回收。
    - 副垃圾回收器，主要负责新生代的垃圾回收。 
    - 主垃圾回收器，主要负责老生代的垃圾回收。

- **垃圾回收器的工作流程**

  - **不论什么类型的垃圾回收器，它们都有一套共同的执行流程**。
    - **第一步是标记空间中活动对象和非活动对象。**所谓活动对象就是还在使用的对象，非活动对 象就是可以进行垃圾回收的对象。
    - **第二步是回收非活动对象所占据的内存**。其实就是在所有的标记完成之后，统一清理内存中 所有被标记为可回收的对象。
    - **第三步是做内存整理。**一般来说，频繁回收对象后，内存中就会存在大量不连续空间，我们 把这些不连续的内存空间称为**内存碎片**。当内存中出现了大量的内存碎片之后，如果需要分 配较大连续内存的时候，就有可能出现内存不足的情况。所以最后一步需要整理这些内存碎 片，但**这步其实是可选的，因为有的垃圾回收器不会产生内存碎片，比如接下来我们要介绍 的副垃圾回收器。**
  - **副垃圾回收器**
    - 新生代中用**Scavenge 算法**来处理。所谓 Scavenge 算法，是把新生代空间对半划分为两 个区域，一半是对象区域，一半是空闲区域
    - **为了执行效率，一般新生区的空间会被设置得比较小**。也正是因为新生区的空间不大，所以很容易被存活的对象装满整个区域。为了解决这个问 题，JavaScript 引擎采用了**对象晋升策略**，也就是经过两次垃圾回收依然还存活的对象， 会被移动到老生区中。
  - **主垃圾回收器**
    - 主垃圾回收器是采用**标记 - 清除(Mark-Sweep)**的算法进行垃圾回收的
    - 不过对一块内存多次执行标记 - 清除算 法后，会产生大量不连续的内存碎片。而碎片过多会导致大对象无法分配到足够的连续内 存，于是又产生了另外一种算法——**标记 - 整理(Mark-Compact)**，这个标记过程仍然 与标记 - 清除算法里的是一样的，但后续步骤不是直接对可回收对象进行清理，而是让所 有存活的对象都向一端移动，然后直接清理掉端边界以外的内存

- **全停顿**

  - 不过由于 JavaScript 是运行在主线程之上的，一旦执行垃圾回收算法，都需要将正在执行的 JavaScript 脚本暂停下来，待垃圾回收完毕后再恢复脚本执行。我们把这种行为叫做**全停 顿(Stop-The-World)**。
  - 为了降低老生代的垃圾回收而造成的卡顿，V8 将标记过程分为一个个的子标记过程，同时 让垃圾回收标记和 JavaScript 应用逻辑交替进行，直到标记阶段完成，我们把这个算法称 为**增量标记(Incremental Marking)算法**。
  - 使用增量标记算法，可以把一个完整的垃圾回收任务拆分为很多小的任务，这些小的任务执 行时间比较短，可以穿插在其他的 JavaScript 任务中间执行，这样当执行上述动画效果 时，就不会让用户因为垃圾回收任务而感受到页面的卡顿了。







# **编译器和解释器**

- **编译器和解释器**

  - **编译型语言在程序执行之前，需要经过编译器的编译过程，并且编译之后会直接保留机器能读懂的二进制文件，这样每次运行程序时，都可以直接运行该二进制文件，而不需要再次重 新编译了**。比如 C/C++、GO 等都是编译型语言。
  - **而由解释型语言编写的程序，在每次运行时都需要通过解释器对程序进行动态解释和执行**。 比如 Python、JavaScript 等都属于解释型语言。
  - 二者的执行流程
    - 在编译型语言的编译过程中，编译器首先会依次对源代码进行词法分析、语法分析，生成抽象语法树(AST)，然后是优化代码，**最后再生成处理器能够理解的机器码**。**如果编译成功，将会生成一个可执行的文件**。但如果编译过程发生了语法或者其他的错误， 那么编译器就会抛出异常，最后的二进制文件也不会生成成功。
    - 在解释型语言的解释过程中，同样解释器也会对源代码进行词法分析、语法分析，并生 成抽象语法树(AST)，不过它会**再基于抽象语法树生成字节码，最后再根据字节码来执行程序、输出结果。**

- **V8** **是如何执行一段** **JavaScript** **代码的**

  - V8 在执行过程中既有**解释器 Ignition**，又有**编译器 TurboFan**
  - 生成抽象语法树(AST)和执行上下文
    - 高级语言是开发者可以理解的语言，但是让编译器或者解释器来理解就非常困难了。对于编 译器或者解释器来说，它们可以理解的就是 AST 了。所以无论你使用的是解释型语言还是 编译型语言，在编译过程中，它们都会生成一个 AST。这和渲染引擎将 HTML 格式文件转 换为计算机可以理解的 DOM 树的情况类似。
      - 编译器或者解释器后续的工作都需要依赖于 AST，而不是源代码。
      - AST 是非常重要的一种数据结构，在很多项目中有着广泛的应用。其中最著名的一个项目 是 Babel。**Babel 是一个被广泛使用的代码转码器**，可以将 ES6 代码转为 ES5 代码，这意 味着你可以现在就用 ES6 编写程序，而不用担心现有环境是否支持 ES6。Babel 的工作原 理就是先将 ES6 源码转换为 AST，然后再将 ES6 语法的 AST 转换为 ES5 语法的 AST，最 后利用 ES5 的 AST 生成 JavaScript 源代码。
      - 除了 Babel 外，还有 ESLint 也使用 AST。**ESLint 是一个用来检查 JavaScript 编写规范的插件**，其检测流程也是需要将源码转换为 AST，然后再利用 AST 来检查代码规范化的问题。
    - 生成 AST 需要经过两个阶段
      - **第一阶段是分词(tokenize)，又称为词法分析**，其作用是将一行行的源码拆解成一个个 token。所谓**token**，指的是语法上不可能再分的、最小的单个字符或字符串
      - **第二阶段是解析(parse)，又称为语法分析**，其作用是将上一步生成的 token 数据，根 据语法规则转为 AST。如果源码符合语法规则，这一步就会顺利完成。但如果源码存在语 法错误，这一步就会终止，并抛出一个“语法错误”。
    - 有了 AST 后，那接下来 V8 就会生成该段代码的执行上下文。
  - 生成字节码
    - 接下来的第二步，解释器 Ignition 就登场了，它会根据 AST生成字节码，并解释执行字节码。
      - **其实一开始 V8 并没有字节码，而是直接将 AST 转换为机器码**，由于执行机器码的效率是 非常高效的，所以这种方式在发布后的一段时间内运行效果是非常好的。但是**随着 Chrome 在手机上的广泛普及，特别是运行在 512M 内存的手机上，内存占用问题也暴露 出来了**，因为 V8 需要消耗大量的内存来存放转换后的机器码。为了解决内存占用问题， V8 团队大幅重构了引擎架构，引入字节码，并且抛弃了之前的编译器，最终花了将进四年 的时间，实现了现在的这套架构。
    - **字节码就是介于 AST 和机器码之间的一种代码。但是与特定类型的机器码无关，字节码需要通过解释器将其转换为机器码后才能执行。**
      - 机器码所占用的空间远远超过了字节码，所以使用字节码可以减少系统的 内存使用。
  - 执行代码
    - **通常，如果有一段第一次执行的字节码，解释器 Ignition 会逐条解释执行**。在执行字节码 的过程中，如果发现有热点代码(HotSpot)，比如一段代码被重复执行多次，这种就称为**热点代码**，那么**后台的编译器 TurboFan 就会把该段热点的字节码编译为高效的机器码**，然后当再次执行这段被优化的代码时，只需要执行编译后的机器码就可以了，这样就大 大提升了代码的执行效率。
    - 其实字节码配合解释器和编译器是最近一段时间很火的技术，比如 Java 和 Python 的虚拟 机也都是基于这种技术实现的，我们把这种技术称为**即时编译(JIT)**。具体到 V8，就是指 解释器 Ignition 在解释执行字节码的同时，收集代码信息，当它发现某一部分代码变热了 之后，TurboFan 编译器便闪亮登场，把热点的字节码转换为机器码，并把转换后的机器码 保存起来，以备下次使用。

- **JavaScript** **的性能优化**

  - 对于优化 JavaScript 执行效率，你应该将优化的中心聚焦在**单次脚本的执行时间和脚本的网络下载上**，主要关注以下三点内容:
    - 提升单次脚本的执行速度，避免 JavaScript 的长任务霸占主线程，这样可以使得页面 快速响应交互;
    - 避免大的内联脚本，因为在解析 HTML 的过程中，解析和编译也会占用主线程;
    - 减少 JavaScript 文件的容量，因为更小的文件会提升下载速度，并且占用更低的内存。

  

  





# **消息队列和事件循环**

- 每个渲染进程都有一个主线程，并且主线程非常繁忙，既要处理 DOM，又 要计算样式，还要处理布局，同时还需要处理 JavaScript 任务以及各种输入事件。要**让这么多不同类型的任务在主线程中有条不紊地执行**，这就**需要一个系统来统筹调度这些任务， 这个统筹调度系统就是消息队列和事件循环系统**。
- 在线程运行过程中处理新任务
  - 并不是所有的任务都是在执行之前统一安排好的，大部分情况下，新的任务是在线程运行 过程中产生的。比如在线程执行过程中，又接收到了一个新的任务要求计算“10+2”。
  - **要想在线程运行过程中，能接收并执行新的任务，就需要采用事件循环机制**
- 处理其他线程发送过来的任务
  - 比如渲染主线程会频繁接收到来自于 IO 线程的一些任务，接收到这些任务之 后，渲染进程就需要着手处理。
  - 一个通用模式是使用**消息队列**。
  - 我们的改造可以分为下面三个步骤:
    -  添加一个消息队列;
    -  IO 线程中产生的新任务添加进消息队列尾部;
    - 渲染主线程会循环地从消息队列头部中读取任务，执行任务。
- 处理其他进程发送过来的任务
  - **渲染进程专门有一个 IO 线程用来接收其他进程传进来的消息**，接收到消 息之后，会将这些消息组装成任务发送给渲染主线程，后续的步骤就和前面讲解的“处理其 他线程发送的任务”一样了。
- 消息队列中的任务类型
  - 你可以参考下Chromium 的官方源码，这里面包含了很多内部消息类型，如 输入事件(鼠标滚动、点击、移动)、微任务、文件读写、WebSocket、JavaScript 定时 器等等。
  - 除此之外，消息队列中还包含了很多与页面相关的事件，如 JavaScript 执行、解析 DOM、样式计算、布局计算、CSS 动画等。
  - 以上这些事件都是在主线程中执行的，所以在编写 Web 应用时，你还需要衡量这些事件所 占用的时长，并想办法解决单个任务占用主线程过久的问题。
- 页面使用单线程的缺点
  - **第一个问题是如何处理高优先级的任务。**
    - 比如一个**典型的场景是监控 DOM 节点的变化情况**(节点的插入、修改、删除等动态变 化)，然后根据这些变化来处理相应的业务逻辑。
    - 如果 DOM 发生变化，采用同步通知的方式，会影响当前任务的**执行效率**; 如果采用异步方式，又会影响到**监控的实时性**，因为在添加到消息队列的过程中，可能前面就有很多任务在排队了。
    - 针对这种情况，微任务就应用而生了。通常我们把消息队列中的任务称为**宏任务**，每个宏任务中都包含了一个**微任务队列**，在执行 宏任务的过程中，如果 DOM 有变化，那么就会将该变化添加到微任务列表中，这样就不 会影响到宏任务的继续执行，因此也就解决了执行效率的问题。
    - 等宏任务中的主要功能都直接完成之后，这时候，渲染引擎并不着急去执行下一个宏任务， 而是执行当前宏任务中的微任务，因为 DOM 变化的事件都保存在这些微任务队列中，这 样也就解决了实时性问题。
  - **第二个是如何解决单个任务执行时长过久的问题。**
    - 针 对这种情况，JavaScript 可以通过回调功能来规避这种问题，也就是让要执行的 JavaScript 任务滞后执行。
- 浏览器页面是如何运行的
  - 你可以打开开发者工具，点击“Performance”标签，选择左上角的“start porfiling and load page”来记录整个页面加载过程中的事件执行情况。
  - 灰色的就是一个个任务，每个任务下面还有子任务，其中的 Parse HTML 任务， 是把 HTML 解析为 DOM 的任务。值得注意的是，在执行 Parse HTML 的时候，如果遇到 JavaScript 脚本，那么会暂停当前的 HTML 解析而去执行 JavaScript 脚本。





# **WebAPI**

- 通过**setTimeout**和**XMLHttpRequest**这两个 WebAPI 来 介绍事件循环的应用。这两个 WebAPI 是两种不同类型的应用，比较典型，并且在 JavaScript 中的使用频率非常高。

- **setTimeout**，它就是一个**定时器，用来指定某个函数在多少毫秒之后执行**。

  - 它会返回一个整 数，表示定时器的编号，同时你还可以通过该编号来取消这个定时器。
  - 那通常情况下，当一个定时器 的任务还没有被执行的时候，也是可以取消的，具体方法是调用**clearTimeout 函数**，并传 入需要取消的定时器的 ID。
  - 其实浏览器内部实现取消定时器的操作也是非常简单的，就是直接从 delayed_incoming_queue 延迟队列中，通过 ID 查找到对应的任务，然后再将其从队列 中删除掉就可以了。

- 浏览器怎么实现 setTimeout

  - 典型的事件
    - 当接收到 HTML 文档数据，渲染引擎就会将“解析 DOM”事件添加到消息队列中，
    - 当用户改变了 Web 页面的窗口大小，渲染引擎就会将“重新布局”的事件添加到消息队 列中。
    - 当触发了 JavaScript 引擎垃圾回收机制，渲染引擎会将“垃圾回收”任务添加到消息队 列中。
    - 同样，如果要执行一段异步 JavaScript 代码，也是需要将执行任务添加到消息队列中。
  - 怎么设计才能让定时器设置的回调事件在规定时间内被执行呢
    - 在 Chrome 中除了正常使用的消息队列之外，还有另外一个消息队列，这个队列中维护了 需要延迟执行的任务列表，包括了定时器和 Chromium 内部一些需要延迟执行的任务。所 以当通过 JavaScript 创建一个定时器时，渲染进程会将该定时器的回调任务添加到**延迟队列**中。
    - 当通过 JavaScript 调用 setTimeout 设置回调函数的时候，渲染进程将会创建一个回调任 务，包含了回调函数、当前发起时间、延迟执行时间。创建好回调任务之后，再将该任务添加到延迟执行队列中
    - 消息循环系统 是怎么触发延迟队列的。
      - 我们添加了一个**ProcessDelayTask 函数**，该函数是专门用来处理延迟执行任务的。
      - 这里我们要重点关注它的执行时机，在上段代码中，处理完消息队列中 的一个任务之后，就开始执行 ProcessDelayTask 函数。ProcessDelayTask 函数会根据发 起时间和延迟时间计算出到期的任务，然后依次执行这些到期的任务。
      - 等到期的任务执行完 成之后，再继续下一个循环过程。通过这样的方式，一个完整的定时器就实现了。

- 使用 setTimeout 的一些注意事项

  - **如果当前任务执行时间过久，会影延迟到期定时器任务的执行**

  - **如果** **setTimeout** **存在嵌套调用，那么系统会设置最短时间间隔为** **4** **毫秒**

    - 也就是说在定时器函数里面嵌套调用定时器，也会延长定时器的执行时间
    - 前面五次调用的时间间隔比较 小，嵌套调用超过五次以上，后面每次的调用最小时间间隔是 4 毫秒。之所以出现这样的 情况，是因为在 Chrome 中，定时器被嵌套调用 5 次以上，系统会判断该函数方法被阻塞 了，如果定时器的调用时间间隔小于 4 毫秒，那么浏览器会将每次调用的时间间隔设置为 4 毫秒。
    - **所以，一些实时性较高的需求就不太适合使用 setTimeout 了，比如你用 setTimeout 来实 现 JavaScript 动画就不是一个很好的主意。**

  - **未激活的页面，setTimeout执行最小间隔是1000 毫秒**

    - 也就是说，如果标签不是当前的激活标签，那么定时器最小的时间 间隔是 1000 毫秒，目的是为了优化后台页面的加载损耗以及降低耗电量。这一点你在使用 定时器的时候要注意。

  - **延时执行时间有最大值**

    - 除了要了解定时器的回调函数时间比实际设定值要延后之外，还有一点需要注意下，那就是 **Chrome、Safari、Firefox 都是以 32 个 bit 来存储延时值的**，32bit 最大只能存放的数字 是 2147483647 毫秒，这就意味着，如果 setTimeout 设置的延迟值大于 2147483647 毫 秒(大约 24.8 天)时就会溢出，这导致定时器会被立即执行。

  - **使用** **setTimeout** **设置的回调函数中的** **this** **不符合直觉**

    - 如果被 setTimeout 推迟执行的回调函数是某个对象的方法，那么该方法中的 this 关键字 将指向全局环境，而不是定义时所在的那个对象。

    - 解决这个问题

      - ```javascript
        var name= 1;
        var MyObj = { 
          name: 2,
        	showName: function(){ 
          	console.log(this.name);
        	}
        }
        setTimeout(MyObj.showName,1000)
        ```

      - 第一种是将MyObj.showName放在匿名函数中执行

        - ```javascript
          // 箭头函数
          setTimeout(() => {
          MyObj.showName()
          }, 1000);
          
          // 或者 function 函数
          setTimeout(function() {
          MyObj.showName();
          }, 1000)
          ```

      - 第二种是使用 bind 方法，将 showName 绑定在 MyObj 上面

        - ```javascript
          setTimeout(MyObj.showName.bind(MyObj), 1000)
          ```

- XMLHttpRequest

  - 自从网页中引入了 JavaScript，我们就可以操作 DOM 树中任意一个节点，例如隐藏 / 显示节点、改变颜色、获得或改变文本内容、为元素添加事件响应函数等等， 几乎可以“为 所欲为”了。
  - **XMLHttpRequest 提供了从 Web 服务器获取数据的能力**，如果你想要更新某条数 据，只需要通过 XMLHttpRequest 请求服务器提供的接口，就可以获取到服务器的数据， 然后再操作 DOM 来更新页面内容，整个过程只需要更新网页的一部分就可以了，而不用像之前那样还得刷新整个页面，这样既有效率又不会打扰到用户。

- **回调函数** **VS** **系统调用栈**

  - 将一个函数作为参数传递给另外一个函数，那作为参数的这个函数就是**回调函数**。

  - ```javascript
    // 下面的回调方法有个特点，就是回调函数 callback 是在主函数 doWork 返回之前执行的， 我们把这个回调过程称为同步回调。 
    let callback = function(){
      console.log('i am do homework') 
    }
    function doWork(cb) {
      console.log('start do work')
      cb()
      console.log('end do work')
    }
    doWork(callback)
    
    // 在这个例子中，我们使用了 setTimeout 函数让 callback 在 doWork 函数执行结束后，又 延时了 1 秒再执行，这次 callback 并没有在主函数 doWork 内部被调用，我们把这种回 调函数在主函数外部执行的过程称为异步回调。
    let callback = function(){
      console.log('i am do homework') 
    }
    function doWork(cb) {
      console.log('start do work')
      setTimeout(cb,1000)
      console.log('end do work')
    }
    doWork(callback)
    ```

  - 再深入点，站在消息循环的视角来看看同步回调和异步回调的区别

    - 当循环系统在执行一个任务的时候，都要为这个任务维护一个 **系统调用栈**。这个**系统调用栈**类似于 JavaScript 的调用栈，只不过系统调用栈是 Chromium 的开发语言 C++ 来维护的。通过 Performance 来抓取它核心的调用信息：
      - 比如记录了一个 Parse HTML 的任务执行过程，其中黄色的条目表示执行 JavaScript 的过程，其他颜色的条目表示浏览器内部系统的执行过程。
      - 整个 Parse HTML 是一个完整的任务，在执行过程中的脚本解析、样式表 解析都是该任务的子过程，其下拉的长条就是执行过程中调用栈的信息。
    - **每个任务在执行过程中都有自己的调用栈，那么同步回调就是在当前主函数的上下文中执行 回调函数。异步回调是指回调函数在主函数之外执行**，一般有两种方式:
      - 第一种是把异步函数做成一个任务，添加到信息队列尾部;
      - 第二种是把异步函数添加到微任务队列中，这样就可以在当前任务的末尾处执行微任务 了。

- XMLHttpRequest 运作机制

  - 第一步:创建 XMLHttpRequest 对象。
  - 第二步:为 xhr 对象注册回调函数。
    - ontimeout，用来监控超时请求，如果后台请求超时了，该函数会被调用;
    - onerror，用来监控出错信息，如果后台请求出错了，该函数会被调用;
    - onreadystatechange，用来监控后台请求过程中的状态，比如可以监控到 HTTP 头加载 完成的消息、HTTP 响应体消息以及数据加载完成的消息等。
  - 第三步:配置基础的请求信息。
  - 第四步:发起请求。
    - 渲染进程会将请求发送给**网络进程**，然后网络进程负责资源的下载，等网 络进程接收到数据之后，就会利用 **IPC** 来通知**渲染进程**;渲染进程接收到消息之后，会将 xhr 的**回调函数封装成任务并添加到消息队列中**，等**主线程循环系统**执行到该任务的时候， 就会**根据相关的状态来调用对应的回调函数**。

- XMLHttpRequest使用过程中的坑

  - 上述过程看似简单，但由于浏览器很多安全策略的限制，所以会导致你在使用过程中踩到非常多的“坑”。
  - **跨域问题**
    - 在 A 站点中去访问不同源的 B 站点的内容。默认情况下，跨域请求 是不被允许的。
  - **HTTPS** **混合内容的问题**
    - HTTPS 混合内容是 HTTPS 页面中 包含了不符合 HTTPS 安全要求的内容，比如包含了 HTTP 资源，通过 HTTP 加载的图像、 视频、样式表、脚本等，都属于混合内容。
    - 通常，如果 HTTPS 请求页面中使用混合内容，浏览器会针对 HTTPS 混合内容显示警告， 用来向用户表明此 HTTPS 页面包含不安全的资源。
    - 通过 HTML 文件加载的混合资源，虽然给出警告，但大部分类型还是能 加载的。而使用 XMLHttpRequest 请求时，浏览器认为这种请求可能是攻击者发起的，会 阻止此类危险的请求







# **宏任务和微任务**

- **微任务可以在实时性和效率之间做一个有效的权衡**。
  - 基于微任务的技术有 MutationObserver、Promise 以及以 Promise 为基础开发出来的很多其他的技术。
- 宏任务
  - 页面中的大部分任务都是在主线程上执行的，这些任务包括了:
    - 渲染事件(如解析 DOM、计算布局、绘制); 
    - 用户交互事件(如鼠标点击、滚动页面、放大缩小等); 
    - JavaScript 脚本执行事件; 
    - 网络请求完成、文件读写完成事件
  - 为了协调这些任务有条不紊地在主线程上执行，页面进程引入了消息队列和事件循环机制， 渲染进程内部会维护多个消息队列，比如延迟执行队列和普通的消息队列。然后主线程采用 一个 for 循环，不断地从这些任务队列中取出任务并执行任务。我们把这些消息队列中的任 务称为**宏任务**。
  - 消息队列中的任务是通过事件循环系统来执行的，这里我们可以看看在WHATWG 规范中 是怎么定义事件循环机制的。
    - WHATWG 规范定义的大致流程:
      - 先从多个消息队列中选出一个最老的任务，这个任务称为 oldestTask;
      - 然后循环系统记录任务开始执行的时间，并把这个 oldestTask 设置为当前正在执行的任 务;
      - 当任务执行完成之后，删除当前正在执行的任务，并从对应的消息队列中删除掉这个 oldestTask;
      - 最后统计执行完成的时长等信息。
  - 宏任务可以满足我们大部分的日常需求，不过**如果有对时间精度要求较高的需求，宏任务就难以胜任了**，下面我们就来分析下为什么宏任务难以满足对时间精度要求较高的任务。
    - 页面的渲染事件、各种 IO 的完成事件、执行 JavaScript 脚本的事件、用 户交互的事件等都随时有可能被添加到消息队列中，而且**添加事件是由系统操作的， JavaScript 代码不能准确掌控任务要添加到队列中的位置，控制不了任务在消息队列中的 位置，所以很难控制开始执行任务的时间。**
    - 比如我的目的是想通过 setTimeout 来设置两个回调任务，并让它们按照前后顺 序来执行，中间也不要再插入其他的任务，因为如果这两个任务的中间插入了其他的任务， 就很有可能会影响到第二个定时器的执行时间了。
- 之前了解过异步回调的概念，其主要有两种方式
  - **第一种是把异步回调函数封装成一个宏任务，添加到消息队列尾部，当循环系统执行到该任 务的时候执行回调函数**。这种比较好理解，我们前面介绍的 setTimeout 和 XMLHttpRequest 的回调函数都是通过这种方式来实现的。
  - **第二种方式的执行时机是在主函数执行结束之后、当前宏任务结束之前执行回调函数，这通 常都是以微任务形式体现的。**
- **微任务**
  - **微任务就是一个需要异步执行的函数，执行时机是在主函数执行结束之后、当前宏任务结束之前。**
  - 我们知道当 JavaScript 执行一段脚本的时候，V8 会为其创建一个全局执行上下文，在创建 全局执行上下文的同时，V8 引擎也会在内部创建一个**微任务队列**。
    - 因为在当前宏任务执行的过程中，有时候会产生多个微任务， 这时候就需要使用这个微任务队列来保存这些微任务了。不过这个微任务队列是给 V8 引擎 内部使用的，所以你是无法通过 JavaScript 直接访问的。
    - 微任务产生的时机和执行微任务队列的时机
      - 产生微任务有两种方式
        - 第一种方式是使用 **MutationObserver** 监控某个 DOM 节点，然后再通过 JavaScript 来修 改这个节点，或者为这个节点添加、删除部分子节点，当 DOM 节点发生变化时，就会产 生 DOM 变化记录的微任务。
        - 第二种方式是使用 Promise，当调用 Promise.resolve() 或者 Promise.reject() 的时候，也 会产生微任务。
      - 微任务队列是何时被执行的
        - 通常情况下，在当前宏任务中的 JavaScript 快执行完成时，也就在 JavaScript 引擎准备退出全局执行上下文并清空调用栈的时候，JavaScript 引擎会检查全局执行上下文中的微任务队列，然后按照顺序执行队列中的微任务。**WHATWG 把执行微任务的时间点称为检查 点**。
        - **如果在执行微任务的过程中，产生了新的微任务，同样会将该微任务添加到微任务队列中**， V8 引擎一直循环执行微任务队列中的任务，直到队列为空才算执行结束。也就是说**在执行微任务过程中产生的新的微任务并不会推迟到下个宏任务中执行**，而是在当前的宏任务中继 续执行。
  - **结论**
    - **微任务和宏任务是绑定的**，每个宏任务在执行时，会创建自己的微任务队列。
    - **微任务的执行时长会影响到当前宏任务的时长**。比如一个宏任务在执行过程中，产生了 100 个微任务，执行每个微任务的时间是 10 毫秒，那么执行这 100 个微任务的时间就 是 1000 毫秒，也可以说这 100 个微任务让宏任务的执行时间延长了 1000 毫秒。所以 你在写代码的时候一定要注意控制微任务的执行时长。
    - **在一个宏任务中，分别创建一个用于回调的宏任务和微任务，无论什么情况下，微任务都早于宏任务执行**。
- 监听 DOM 变化方法演变
  - MutationObserver 是用来监听 DOM 变化的一套方法
    - Mutation Event 采用了**观察者的设计模 式**，当 DOM 有变动时就会立刻触发相应的事件，这种方式属于同步回调。
      - 采用 Mutation Event 解决了实时性的问题，因为 DOM 一旦发生变化，就会立即调用 JavaScript 接口。但也正是这种实时性造成了严重的性能问题，因为每次 DOM 变动，渲 染引擎都会去调用 JavaScript，这样会产生较大的性能开销。
  - 为了解决了 Mutation Event 由于同步调用 JavaScript 而造成的性能问题，从 DOM4 开 始，推荐使用 **MutationObserver 来代替 Mutation Event。MutationObserver API 可以 用来监视 DOM 的变化**，包括属性的变化、节点的增减、内容的变化等。
    - MutationObserver 将响应函数改成异步调用，可以不用在每次 DOM 变化都触发 异步调用，而是等多次 DOM 变化后，**一次触发异步调用**，并且还会使用一个数据结构来 记录这期间所有的 DOM 变化。这样即使频繁地操纵 DOM，也不会对性能造成太大的影 响。
    - 我们通过异步调用和减少触发次数来缓解了性能问题，那么如何保持消息通知的及时性呢? 如果采用 setTimeout 创建宏任务来触发回调的话，那么实时性就会大打折扣，因为上面我 们分析过，在两个任务之间，可能会被渲染进程插入其他的事件，从而影响到响应的实时 性。
      - 这时候，**微任务**就可以上场了，在每次 DOM 节点发生变化的时候，渲染引擎将变化记录 封装成微任务，并将微任务添加进当前的微任务队列中。这样当执行到检查点的时候，V8 引擎就会按照顺序执行微任务了。
  - 综上所述， MutationObserver 采用了“**异步 + 微任务**”的策略。
    - 通过**异步**操作解决了同步操作的**性能问题**; 
    - 通过**微任务**解决了**实时性的问题**。





# **Promise**

- DOM/BOM API 中新加入的 API 大多数都是建立在 Promise 上的，而且新的前端框架也 使用了大量的 Promise。

- 异步编程的问题:代码逻辑不连续

  - 一个标准的异步编程模型
    - 页面主线程发起了一个耗时的任务，并将任务交给另外一个进程去处理，这时页面主线程会继续执行消息队列中的任务。
    - 等该进程处理完这个 任务后，会将该任务添加到渲染进程的消息队列中，并排队等待循环系统的处理。
    - 排队结束之后，循环系统会取出消息队列中的任务进行处理，并触发相关的回调操作。
  - **封装异步代码，让处理流程变得线性**
    - 由于我们重点关注的是**输入内容(请求信息)和输出内容(回复信息)**，至于中间的异步请 求过程，我们不想在代码里面体现太多，因为这会干扰核心的代码逻辑。
    - 首先，我们把输入的 HTTP 请求信息全部保存到一个 request 的结构中，包括请求地址、请求头、请求方式、引用地址、同步请求还是异步请 求、安全设置等信息。
  - **回调地狱**
    - 如果嵌套了太多的回调函数就很容 易使得自己陷入了**回调地狱**。
    - 代码之所以看上去很乱，归结其原因有两点:
      - **第一是嵌套调用**，下面的任务依赖上个任务的请求结果，并**在上个任务的回调函数内部执行新的业务逻辑。**
      - **第二是任务的不确定性**，执行每个任务都有两种可能的结果(成功或者失败)，所以体现 在代码中就需要对每个任务的执行结果做两次判断。
    - 解决思路就很清晰了:
      - 第一是消灭嵌套调用; 
      - 第二是合并多个任务的错误处理。

- **Promise:消灭嵌套调用和多次错误处理**

  - 重点关注下 Promise 的使用方式。
    - 首先我们引入了 Promise，在调用 XFetch 时，会返回一个 Promise 对象。
    - 构建 Promise 对象时，需要传入一个**executor 函数**，XFetch 的**主要业务流程都在 executor 函数中执行。**
    - **如果运行在 excutor 函数中的业务执行成功了，会调用 resolve 函数;如果执行失败 了，则调用 reject 函数。**
    - 在 excutor 函数中调用 resolve 函数时，会触发 promise.then 设置的回调函数;而调 用 reject 函数时，会触发 promise.catch 设置的回调函数。
  - Promise 主要通过下面两步解决嵌套回调问题的。
    - **首先，Promise 实现了回调函数的延时绑定**。回调函数的延时绑定在代码上体现就是先创 建 Promise 对象 x1，通过 Promise 的构造函数 executor 来执行业务逻辑;**创建好 Promise 对象 x1 之后，再使用 x1.then 来设置回调函数。**
    - **其次，需要将回调函数 onResolve 的返回值穿透到最外层**。因为我们会根据 onResolve 函数的传入值来决定创建什么类型的 Promise 任务，创建好的 Promise 对象需要返回到最 外层，这样就可以摆脱嵌套循环了。
    - ![img](https://static001.geekbang.org/resource/image/ef/7f/efcc4fcbebe75b4f6e92c89b968b4a7f.png)
  - Promise 是怎么处理异常的
    - 有四个 Promise 对象:p0~p4。无论哪个对象里面抛出异常，**都可以通过最后一 个对象 p4.catch 来捕获异常**，通过这种方式可以将所有 Promise 对象的错误合并到一个 函数来处理，这样就解决了每个任务都需要单独处理异常的问题。
    - **之所以可以使用最后一个对象来捕获所有异常，是因为 Promise 对象的错误具有“冒泡”性质，会一直向后传递**，直到被 onReject 函数处理或 catch 语句捕获为止。具备了这 样“冒泡”的特性后，就不需要在每个 Promise 对象中单独捕获异常了。

- Promise与微任务

  - 首先执行 new Promise 时，Promise 的构造函数会被执行，不过由于 Promise 是 V8 引 擎提供的，所以暂时看不到 Promise 构造函数的细节。

  - 接下来，Promise 的构造函数会调用 Promise 的参数 executor 函数。然后在 executor 中执行了 resolve，resolve 函数也是在 V8 内部实现的，那么 resolve 函数到底做了什么呢?我们知道，执行 resolve 函数，会触发 demo.then 设置的回调函数 onResolve，所 以可以推测，**resolve 函数内部调用了通过 demo.then 设置的 onResolve 函数。**

    - ```javascript
      function executor(resolve, reject) {
          resolve(100)
      }
      let demo = new Promise(executor)
      
      function onResolve(value){
          console.log(value)
      }
      demo.then(onResolve)
      ```

  - 不过这里需要注意一下**，由于 Promise 采用了回调函数延迟绑定技术，所以在执行 resolve 函数的时候，回调函数还没有绑定，那么只能推迟回调函数的执行。**

  - 可以写demo采用定时器来推迟 onResolve 的执行，不过使用定时器的效率并不是太高，好在我们有微任务，所以 Promise 又把这个定时器改造成了微任务了，**这样既可以让 onResolve_ 延时被调用，又提升了代码的执行效率。**这就是 Promise 中使用微任务的原由了。

    - ```javascript
      function Bromise(executor) {
          var onResolve_ = null
          var onReject_ = null
           //模拟实现resolve和then，暂不支持rejcet
          this.then = function (onResolve, onReject) {
              onResolve_ = onResolve
          };
          function resolve(value) {
           			 // 要让 resolve 中的 onResolve_ 函数延后执行，可以在 resolve 函数里面加上一个定时器，让其延时执行 onResolve_ 函数
                setTimeout(()=>{
                  onResolve_(value)
                },0)
          }
          executor(resolve, null);
      
      function executor(resolve, reject) {
          resolve(100)
      }
      //将Promise改成我们自己的Bromsie
      let demo = new Bromise(executor)
      
      function onResolve(value){
          console.log(value)
      }
      demo.then(onResolve)
        
      // 执行这段代码，我们发现执行出错，输出的内容是：
      
      Uncaught TypeError: onResolve_ is not a function
          at resolve (<anonymous>:10:13)
          at executor (<anonymous>:17:5)
          at new Bromise (<anonymous>:13:5)
          at <anonymous>:19:12  
        
      // 之所以出现这个错误，是由于 Bromise 的延迟绑定导致的，在调用到 onResolve_ 函数的时候，Bromise.then 还没有执行，所以执行上述代码的时候，当然会报“onResolve_ is not a function“的错误了。
      // 也正是因为此，我们要改造 Bromise 中的 resolve 方法，让 resolve 延迟调用 onResolve_。
      ```

- Promise 中是如何实现回调函数返回值穿透的？

  - 首先Promise的执行结果保存在promise的data变量中，然后是.then方法返回值为使用resolved或rejected回调方法新建的一个promise对象，即例如成功则返回new Promise（resolved），将前一个promise的data值赋给新建的promise。

- Promise 出错后，是怎么通过“冒泡”传递给最后那个捕获

  - promise内部有resolved_和rejected_变量保存成功和失败的回调，进入.then（resolved，rejected）时会判断rejected参数是否为函数，若是函数，错误时使用rejected处理错误；若不是，则错误时直接throw错误，一直传递到最后的捕获，若最后没有被捕获，则会报错。可通过监听unhandledrejection事件捕获未处理的promise错误。













































