# Class

  首先类是一种设计模式，比不是必须的。

  类本质上是一个复制的过程。

### ES6的Class解决了什么问题？

  - 不再引用杂乱的prototype
  - 通过extends来指定原型
  - super实现相对多态
  - Class字面语法不能声明属性，避免出错
  - extends有助于我们继承内置对象

  Class只是现有[[prototype]]上的一种语法糖

  所以在定义子类时并不是复制行为，而是委托行为（所以子类都会收到父类的后续影响）

  同时Class也不能定义类成员属性

  super静态绑定