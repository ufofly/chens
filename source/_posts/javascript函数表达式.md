---
title: javascript 函数表达式
date: 2023-03-27 11:36:24
tags:

---



#  window 网络配置

- 网关“在链路上”
  表示直接发送给目标，而不需要经过路由器（指路由表的网关IP和IF参数对应的接口的IP是一样的）





# javascript 函数表达式

使用函数表达式，并将 `welcome` 赋值给在 `if` 外声明的变量，并具有正确的可见性。

```javascript
let age = prompt("What is your age?", 18); 
let welcome; 
if (age < 18) {   
    welcome = function() {    
        alert("Hello!");  
    }; 
} else {   
    welcome = function() {    
        alert("Greetings!");  
    }; 
} 
welcome(); // 现在可以了
```

- 函数是值。它们可以在代码的任何地方被分配，复制或声明。
- 如果函数在主代码流中被声明为单独的语句，则称为“函数声明”。
- 如果该函数是作为表达式的一部分创建的，则称其“函数表达式”。
- 在执行代码块之前，内部算法会先处理函数声明。所以函数声明在其被声明的代码块内的任何位置都是可见的。
- 函数表达式在执行流程到达时创建。
