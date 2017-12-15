---
title: JavaScript实现私有变量
date: 2017-9-10 08:55:29   //在此处输入你编辑这篇文章的时间。
categories: JavaScript   //在此处输入这篇文章的分类。
toc: true  //在此处设定是否开启目录，需要主题支持。
---


*ES6之前*

---
构造函数Person和它的私有方法们：

    function Person(name, age){
        // 私有变量
        this.name = name;
        this.age = age;
        function privateName(){
        // 在constructor中函数声明，实现Person的第1个私有方法
            console.log("Name is: "+this.name);
            return this.name;
        };
        var privateAge = function(){
        // 在constructor中定义函数表达式，实现Person的第2个私有方法
            console.log("Age is: "+this.age);
            return this.age;
        };
        this.publicFunc = function(){
            // 需要绑定正确的this作用域，我们使用call方法调用
            privateName.call(this);  
            privateAge.call(this);
        }

    };
    var Peter = new Person("Peter", 24);
    Peter.privateAge();  // undefined 验证了私有方法不可访问
    Peter.publicFunc();  // console will print name and age 公有方法访问成功
    
######不足之处：
  1. 内存消耗，每次new操作都会，每个函数都会有一笔新的开销。
  2. 私有方法并不存在于原型链上，因此prototype的方法不能访问该方法。

---
构造函数Person和它的私有方法们 (by Module Pattern)：
  
    var PersonMP = (function(name, age){
        this.name = name;
        this.age = age;
        function privateName(){
            return this.name;
        };
        function privateAge(){
            return this.age;
        }
        return {
            publicFunc : function(){
                privateName.call(this);
                privateAge.call(this);
            }
        }
    })();

######不足之处：
  1.  如果切换变量或方法的public/private状态，必须要去每个相关地方手动修改。
  2. debug时，对于私有变量较难侦测，且私有变量难以扩充，灵活度不够。
---
小结：

利用闭包特性，retrun一个对象字面量，其中裹挟public method以达到提供外界API的目的。此中还是用到了IIFE立即执行函数相关知识。当然，仍有一些改进空间：
  1. 譬如每次都要用到call俩切换scope，可以封装一个方法，一劳永逸。
  
    //返回一个函数，该函数的this绑定到当前对象
    _:function(fun){
        var that = this;
        return function(){
            return fun.apply(that,arguments); // 注意return
        }
    }

  2.将方法绑到constructor的原型链上，可以达到共享的目的，即讲Model Pattern中的IIFE定义给Person.prototype即可：

    function Person(name,age){
        this.name = name;
        this.age = age;    
    }  
    Person.prototype = (function(){
        function privateName(){
            return this.name;
        };
        function privateAge(){
            return this.age;
        }
        return {
            constructor : Person, // 注意定义好constructor指向， 对象字面量会覆盖原prototype
            publicFunc : function(){
                this._(privateName);
                this._(privateAge);
            }，
            _ : function(fun){
                var that = this;
                return function(){
                    return fun.apply(that, arguments); // 注意return
                }
            }
    })();

---
*ES6之前后*

---
我们有class。
  1.  静态变量

    // ES6 的类定义实现了静态方法的定义，但静态变量呢？
    // 可以用如下方式实现: 
    class Person{
        static get name(){
          return 'jelly';
        }
    };
    Person.name;  // 

  2. 私有属性（私有属性有多种实现方式，只谈及其中一种）
// 闭包
const TbFedMembers = (() => {
  const HuaChen = 'jelly';
  
  return class{
    getOneMemberName(){
      return HuaChen;
    }
  };
})();