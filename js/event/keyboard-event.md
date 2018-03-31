##  前言
最近因一个需求在```element-ui```的```Select```组件文档内找不到对应的示例，也就是```filter-method```方法的具体使用样例，尝试几次之后也使用了一种办法实现，但是感觉并不是最优，于是尝试看组件源码来寻求结果。

在探寻的过程中发现自己对键盘输入事件的理解还并不到位，以至于有一些代码看得云里雾里，赶紧查阅资料写写demo总结一下(下文所说只在chrome与ff里做了探究)。

## 浏览器的键盘事件(keyboard event)

  - 键盘事件主要有以下四类
    
    - keydown 
    - keypress
    - keyup
    - input (尽在input与textarea里触发)

  - 事件触发顺序
    一开始我以为触发顺序看起来很easy 不就应该是```keydown -> keypress -> input -> keyup```这个顺序，但实际情况里还有许多特殊情况。(如果不是在input与textarea里的则去掉input即可)

    首先说一下，键盘上的键位可以分为2种类型
      
      - 输入型键位 0-9 a-z等等一切你要输入字符的按键

      - 功能性键位 Ctrl Command Shift等
    
    这两种类型在事件触发顺序上是有所差别的

      - 输入型键位

        - 轻按一下快速弹起

          ```keydown -> keypress -> input -> keyup```

        - 按住一段时间再弹起

          在mac上测试的时候，搜狗输入法以及自带的拼音输入法顺序为 ```keydown -> (keypress -> input -> keydown -> keypress -> input ...循环) -> keyup```，输入框里输入字符一直增加

          然而切换到系统自带的英文输入法 ```keydown -> keypress -> input -> (keydown -> keydown -> ...循环) -> keyup```，输入框里输入字符只有1个

          注：仅在表单里不同输入法可能表现不一致，如果window层面上的监听则表现一直，都为```(keydown -> kewpress...循环) -> keyup```

          具体原因不详，希望有大佬可以指出，感觉是输入法底层实现的问题。但是我们可以看出，在input元素里，```只有触发keydown事件是无法成功在其中输入值的```

      - 功能性键位

        这类键位不会触发```keypress```，其实在测试功能性键位的时候，我认为还可以具体分为2种键位

        - 功能修饰性键位

          如```Ctrl, Command```等，需要组合其他键位才能有效果，这类键位的长按与短按触发顺序都一致，为```keydown -> keyup```，

        - 单功能性键位

          如```Ese 上下左右箭头```这种不需要组合的生效的键位。

          - 短按触发顺序

            ```keydown -> keyup```

          - 长按触发顺序

            ```keydown -> keydown...循环 -> keyup``` 
    
  - 事件阻止

    - 冒泡

      这4类事件都会冒泡，阻止按照常规调用Api接口就可以阻止冒泡了。

    - 默认行为

      键盘事件的默认行为就是输入，那么已经了解过了事件触发顺序，我们一定可以猜到，如果想阻止在```input输入框```里输入字符，我们可以在```input```事件之前阻止默认行为，即在```keydown和keypress```事件的时候调用```e.preventDefault()```即可阻止输入，这个技巧也常用于输入验证的时候阻止一些超过次数的输入。而在```keydown```里阻止默认行为的话，连```keypress```事件都不会触发

  - input事件

    如果你在```input,textarea```输入框里输入中文的时候，在ff与chrome有不同的事件触发流程

    - chrome

      没有keypress事件的触发

    - ff

      仅触发1次keydown，多次input和1次keyup

    所以监听input事件就足够了，参考百度搜索栏，我们可以实现一个搜索的提示性的功能，对中文还是很友好的。

  ## 何时获取得到表单input的值
  通过上述知识，我们可以猜到只有在输入框的值发生变化的时候才会触发```input```事件，那么在```input```事件里必然可以精确的获取到当前```input```的值，同理```keyup```更晚，一定可以获取到。```keydown,keypress```事件触发的时候，则无法获取到```input```表单改变的最新值。

  ##  应用

  - 可以监听```input```对输入内容作实时校验
  - 可以监听```input```对输入内容作友好的提示
  - 可以监听```keydown```对组合快捷键作相应，作一个出色的Web应用
  - 可以阻止一些输入的内容
  
  ##  后记

  感觉把input事件放到这里面不太合适，毕竟其他三个时间都是键盘事件，而input是专属于表单值变化而触发的事件。但是感觉一般的坑都在这里，所以就加进来了。
  
  ## 参考

  - [https://developer.mozilla.org/zh-CN/docs/Web/Events/keydown](https://developer.mozilla.org/zh-CN/docs/Web/Events/keydown) 
  - [https://developer.mozilla.org/zh-CN/docs/Web/Events/keyup](https://developer.mozilla.org/zh-CN/docs/Web/Events/keyup)
  - [https://developer.mozilla.org/zh-CN/docs/Web/Events/keypress](https://developer.mozilla.org/zh-CN/docs/Web/Events/keypress)
  - [https://developer.mozilla.org/zh-CN/docs/Web/Events/input](https://developer.mozilla.org/zh-CN/docs/Web/Events/input)