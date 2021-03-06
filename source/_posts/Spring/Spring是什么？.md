---
title: Spring是什么？
author: Marlowe
tags: Spring
categories: 春招面试
abbrlink: 33751
date: 2021-03-11 15:42:30
---
轻量级的开源的J2EE框架。它是一个容器框架，用来装JavaBean，也是一个中间层框架，可以起连接作用。
<!--more-->
### Spring是什么？
Spring是一个轻量级的控制反转(IOC)和面向切面(AOP)的容器框架。

1、Spring的核心是一个轻量级（Lightweight）的容器（Container）。
2、Spring是实现IoC（Inversion of Control）容器和非入侵性（No intrusive）的框架。
3、Spring提供AOP（Aspect-oriented programming）概念的实现方式。
4、Spring提供对持久层（Persistence）、事物（Transcation）的支持。
5、Spring供MVC Web框架的实现，并对一些常用的企业服务API（Application Interface）提供一致的模型封装。
6、Spring提供了对现存的各种框架（Structs、JSF、Hibernate、Ibatis、Webwork等）相整合的方案。
总之，Spring是一个全方位的应用程序框架。


### 对AOP的理解

AOP:将程序中的交叉业务逻辑(比如安全，日志，事务等)，封装成一个切面,然后注入到目标对象(具体业务逻辑)中去。AOP可以对某个对象或某些对象的功能进行增强，比如对象中的方法进行增强，可以在执行某个方法之前额外的做一些事情，在某个方法执行之后额外的做一些事情。

### 对IoC的理解

IOC:控制反转也叫依赖注入，IOC利用java反射机制，所谓控制反转是指，本来被调用者的实例是由调用者来创建的，这样的缺点是耦合性太强，IOC则是统一交给spring来管理创建，将对象交给容器管理，你只需要在spring配置文件中配置相应的bean，以及设置相关的属性，让spring容器来生成类的实例对象以及管理对象。在spring容器启动的时候，spring会把你在配置文件中配置的bean都初始化好，然后在你需要调用的时候，就把它已经初始化好的那些bean分配给你需要调用这些bean的类。


**控制反转:**
没有引入IOC容器之前，对象A依赖于对象B,那么对象A在初始化或者运行到某一点的时候， 自己必须主动去创建对象B或者使用已经创建的对象B。无论是创建还是使用对象B,控制权都在自己手上。

引入IOC容器之后,对象A与对象B之间失去了直接联系,当对象A运行到需要对象B的时候，IOC容器 会主动创建一个对象B注入到对象A需要的地方。

通过前后的对比，不难看出来:对象A获得依赖对象B的过程,由主动行为变为了被动行为,控制权颠倒过来，这就是"控制反转"这个名称的由来。

全部对象的控制权全部上缴给"第三方"IOC容器,所以，IOC容器成了整个系统的关键核心，它起到了一种类似“粘合剂”的作用，把系统中的所有对象粘合在一起发挥作用，如果没有这个”粘合剂"，对象与对象之间会彼此失去联系,这就是有人把IOC容器比喻成"粘合剂"的由来。

**依赖注入:**
“获得依赖对象的过程被反转了”。控制被反转之后，获得依赖对象的过程由自身管理变为了由IOC容器主动注入。
依赖注入是实现IOC的方法,就是由IOC容器在运行期间，动态地将某种依赖关系注入到对象之中。

#### IOC优点

1. 资源集中管理，实现资源的可配置和易管理。
2. 降低了使用资源双方的依赖程度，也就是我们说的耦合度。

#### IOC缺点

1. 创建对象的**步骤变复杂了**，不直观，当然这是对不习惯这种方式的人来说的。
2. 因为使用反射来创建对象，所以在**效率上会有些损耗**。但相对于程序的灵活性和可维护性来说，这点损耗是微不足道的。
3. 缺少IDE重构的支持，如果修改了类名，还需到**XML文件中手动修改**，这似乎是所有XML方式的缺憾所在。

#### IOC的作用

因为采用了依赖注入，在初始化的过程中就不可避免的会写大量的new。

这里IoC容器就解决了这个问题。这个容器可以**自动对你的代码进行初始化**，你只需要维护一个Configuration（可以是xml可以是一段代码），而不用每次初始化一辆车都要亲手去写那一大段初始化的代码。这是引入IoC Container的第一个好处。

IoC Container的第二个好处是：**我们在创建实例的时候不需要了解其中的细节。**
