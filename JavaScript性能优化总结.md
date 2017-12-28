## JavaScript性能优化的小知识总结

- ### 1、避免全局查找
  - 在一个函数中会用到全局对象存储为局部变量来减少全局查找，因为访问局部变量的速度要比访问全局变量的速度更快些。
        function search(){
           //当我要使用当前页面地址和主机域名
           alert(window.location.href + window.location.host);
        }
        //最好的方式是如下这样  先用一个简单变量保存起来
        function search() {
           var location = window.location;
           alert(location.href + location.host);
        }

- ### 2、定时器
  - 如果针对的是不断运行的代码，不应该使用setTimeout,而应该是用setInterval,因为setTimeout每一次都会初始化一个定时器,而setInterval只会在开始的时候初始化一个定时器
        var timeoutTimes = 0;
        function timeout() {
           timeoutTimes++;
           if (timeoutTimes < 10) {
              setTimeout(timeout, 10);
           }
        }
        timeout();
        //可以替换为:
        var intervalTimes = 0;
        function interval() {
           intervalTimes++;
           if(intervalTimes >= 10) {
               clearInterval(interv);
           }
        }
        var interv = setInterval(interval, 10); 

  - setTimeout和setInterval的区别
        setTimeout(表达式,延迟时间)在执行时,是在载入后延迟指定时间后，去执行一次表达式，而且只执行一次;但也可以通过创建函数重复执行。
        setInterval(表达式,交互时间)，是它从载入后,每隔指定的时间就执行一次表达式3、字符串连接

- ### 3、字符串连接
  - 如果要连接多个字符串,应该少使用 +=如
        s+=a;
        s+=b;
        s+=c;
        应该写成 s += a + b + c;
  - 如果是收集字符串,比如多次对同一个字符串进行 += 操作的话,最好使用一个缓存,使用JavaScript数组来收集,最后使用join方法连接起来
        var buf = [];
        for (var i = 0; i < 100; i++) {
          buf.push(i.toString());
         }
        var all = buf.join("");

- ### 4、避免with语句
  - 和函数类似,with语句会创建自己的作用域,因此会增加其中执行的代码的作用域链的长度,由于额外的作用域链的查找,在with语句中执行的代码肯定会比外面执行的代码要慢,所以能在不使用with语句的时候尽量不要使用with语句。
        with (a.b.c.d) {
          property1 = 1;
          property2 = 2;
        }
        //可以替换为：
        var obj = a.b.c.d;
        obj.property1 = 1;
        obj.property2 = 2;

- ### 5、数字转换字符串
  - 一般最好用 "" + 1 来将数字转换成字符串，虽然看起来比较丑一点，但事实上这个效率是最高的
        性能上来说：
        ("" +) > String() > .toString() > new String();

- ### 6、浮点数转换成整型

  - 很多人喜欢使用parseInt(),其实parseInt()是用于将字符串转换成数字,而不是浮点数和整型之间的转换，应该使用Math.floor()或者Math.round()

- ### 7、多个类型声明

  - 在JavaScript中所有变量都可以使用单个var语句来声明,这样就是组合在一起,以减少整个脚本的执行时间,

- ### 8、使用直接量
        var aTest = new Array(); //替换为
        var aTest = [];
        var aTest = new Object; //替换为
        var aTest = {};
        var reg = new RegExp(); //替换为
        var reg = /../;
        //如果要创建具有一些特性的一般对象，也可以使用字面量，如下：
        var oFruit = new O;
        oFruit.color = "red";
        oFruit.name = "apple";
        //前面的代码可用对象字面量来改写成这样：
        var oFruit = { color: "red", name: "apple" };

- ### 9、使用DocumentFragment优化多次append
   - 一旦需要更新DOM,考虑使用文档碎片来构建DOM结构,然后在将其添加到现存的文档中。
        for(var i = 0; i < 1000; i++) {
            var el = document.createElement('p');
            el.innerHTML = i;
            document.body.appendChild(el);
         }
         //可以替换为：
        var frag = document.createDocumentFragment();
        for (var i = 0; i < 1000; i++) {
            var el = document.createElement('p');
            el.innerHTML = i;
           frag.appendChild(el);
        }
        document.body.appendChild(frag);

- ### 10、使用一次innerHTML赋值代替构建dom元素
  - 对于大的DOM更改,使用innerHTML要比使用标准的DOM方法创建同样的DOM结构快得多。
        var frag = document.createDocumentFragment();
        for (var i = 0; i < 1000; i++) {
           var el = document.createElement('p');
           el.innerHTML = i;
           frag.appendChild(el);
        }
        document.body.appendChild(frag);
        //可以替换为：
        var html = [];
        for (var i = 0; i < 1000; i++) {
            html.push('<p>' + i + '</p>');
        }
        document.body.innerHTML = html.join('');

- ### 11、通过模板元素clone,替代createElement

  - 很多人喜欢在JavaScript中使用document.write来给页面生成内容。事实上这样的效率较低,如果需要直接插入HTML,可以找一个容器元素,比如指定一个div或span,并设置它们的innerHTML来将自己的HTML代码插入到页面中。通常我们可能会使用字符串直接写HTML来创建节点,其实这样做,一是无法保证代码的有效性二是字符串操作效率低,所以应该是用document.createElment()方法,而如果文档中存在现成的样板节点,应该是用cloneNode()方法,因为使用createElement()方法之后,需要设置多次元素的属性,使用cloneNode()则可以减少属性的设置次数 ——同样如果需要创建很多元素,应该先准备一个样板节点
        var frag = document.createDocumentFragment();
        for (var i = 0; i < 1000; i++) {
           var el = document.createElement('p');
           el.innerHTML = i;
           frag.appendChild(el);
        }
        document.body.appendChild(frag);
        //替换为：
        var frag = document.createDocumentFragment();
        var pEl = document.getElementsByTagName('p')[0];
        for (var i = 0; i < 1000; i++) {
            var el = pEl.cloneNode(false);
            el.innerHTML = i;
            frag.appendChild(el);
         }
        document.body.appendChild(frag);

- ### 12、使用firstChild和nextSibling代替childNodes遍历dom元素
        var nodes = element.childNodes;
        for (var i = 0, l = nodes.length; i < l; i++) {
          var node = nodes[i];
          //……
        }
        //可以替换为：
        var node = element.firstChild;
        while (node) {
         //……
         node = node.nextSibling;

- ### 13、删除DOM节点

    - 删除dom节点之前,一定要删除注册该节点上的事件,不管用observe方式还是用attachEvent方式注册的事件,否则将会产生无法回收的内存。另外,在removeChild和innerHTML=‘ ’;二者之间,尽量选择后者。因为slEve(内存泄漏检测工具)中监测的结果是用removeChild无法有效地释放dom节点 

- ### 14、使用事件代理 

    - 任何可以冒泡的事件都不仅仅可以在事件目标上进行处理，目标的任何祖先节点也能处理，使用这个知识就可以将事件处理程序附加到更高的地方负责多个目标的事件处理,同样。对于内容动态增加并且子节点都需要相同的事件处理函数的情况,可以把事件注册提到父节点上,这样就不需要为每个子节点注册事件监听了。另外，现有的js库都采用observe方式来创建事件监听。

- ### 15、重复使用的调用结果,事件保存到局部变量
        //避免多次取值的调用开销
        var h1 = element1.clientHeight + num1;
        var h4 = element1.clientHeight + num2;
        //可以替换为：
        var eleHeight = element1.clientHeight;
        var h1 = eleHeight + num1;
        var h4 = eleHeight + num2;
- ### 16、优化循环
    - #### 减值迭代
      - 大多数循环使用一个从0开始、增加到某个特定值的迭代器,在很多情况下,从最大值开始，在循环中不断减值的迭代器更加高效

    - #### 简化终止条件
      - 由于每次循环过程都会计算终止条件，所以必须保证它尽可能块，也就是说避免属性查找或者其他的操作，最好是将循环控制量保存到局部变量中,也就是说对数组或列表对象的遍历时，提前将length保存到局部变量中，避免在循环的每一步重复取值。
            var list = document.getElementsByTagName('p');
            for (var i = 0; i < list.length; i++) {
                //……
            }
             //替换为：
            var list = document.getElementsByTagName('p');
            for (var i = 0, l = list.length; i < l; i++) {
               //……
            }

    - #### 简化循环体

      - 循环体是执行最多的，所以要确保其被最大限度的优化