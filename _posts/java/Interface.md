---
layout: post
#标题配置
title:  Java基础-接口
#时间配置
date:   2018-12-13 11:41:00 +0800
#大类配置
categories: Java
#小类配置
tag: 教程
---

* content
{:toc}

#接口
软件工程中存在许多情况，当不同的程序员团队同意一份“合同”来阐明他们的软件如何相互作用时，这是很重要的。每个组都应该能够在不知道如何编写其他组代码的情况下编写代码。一般来说，接口就是这样的合同。

例如，想象一个未来主义社会，计算机控制的机器人汽车在没有人工操作员的情况下将乘客运送到城市街道。汽车制造商编写操作汽车的软件（当然是Java） - 停止，启动，加速，向左转，等等。另一个工业集团，即电子制导仪器制造商，使计算机系统接收GPS（全球定位系统）位置数据和无线传输交通状况并利用该信息来驾驶汽车。

汽车制造商必须发布一个行业标准的界面，详细说明可以调用哪些方法来使汽车移动（任何汽车，来自任何制造商）。然后，指导制造商可以编写调用界面中描述的方法的软件来命令汽车。工业集团都不需要知道其他集团的软件是如何实施的。事实上，每个小组都认为其软件具有高度专有性，并保留随时修改它的权利，只要它继续遵守已发布的界面即可。
##接口定义
在Java编程语言中，一个接口是一个引用类型，类似于类，它可以包含只常量，方法签名，默认的方法，静态方法和嵌套类型。方法体仅适用于默认方法和静态方法。接口无法实例化 - 它们只能由类实现或由其他接口扩展。

定义接口类似于创建新类：
```
public interface OperateCar {

   // constant declarations, if any

   // method signatures
   
   // An enum with values RIGHT, LEFT
   int turn(Direction direction,
            double radius,
            double startSpeed,
            double endSpeed);
   int changeLanes(Direction direction,
                   double startSpeed,
                   double endSpeed);
   int signalTurn(Direction direction,
                  boolean signalOn);
   int getRadarFront(double distanceToCar,
                     double speedOfCar);
   int getRadarRear(double distanceToCar,
                    double speedOfCar);
         ......
   // more method signatures
}

```
请注意，方法签名没有大括号，并以分号结束。

要使用接口，请编写实现该接口的类。当可实例化的类实现接口时，它为接口中声明的每个方法提供方法体。例如:
```
public class OperateBMW760i implements OperateCar {

    // the OperateCar method signatures, with implementation --
    // for example:
    int signalTurn(Direction direction, boolean signalOn) {
       // code to turn BMW's LEFT turn indicator lights on
       // code to turn BMW's LEFT turn indicator lights off
       // code to turn BMW's RIGHT turn indicator lights on
       // code to turn BMW's RIGHT turn indicator lights off
    }

    // other members, as needed -- for example, helper classes not 
    // visible to clients of the interface
}

```