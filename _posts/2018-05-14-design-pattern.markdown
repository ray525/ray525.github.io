---
layout: post
title:  "设计模式"
date:   2018-05-14 18:04:00
categories: design 
tags: design
---

* content
{:toc}

![intro](https://design-patterns.readthedocs.io/zh_CN/latest/_images/uml_class_struct.jpg)

[作业部落记录](https://www.zybuluo.com/ranger-01/note/1121036)




## 设计模式基础与原则
1. 设计模式基础：
    - 抽象
    - 封装
    - 继承
    - 多态
2. [设计模式的原则](http://www.uml.org.cn/sjms/201211023.asp)
    - 开闭原则（Open Close Principle）
        - 对扩展开放，对修改关闭。想要达到这样的效果，我们需要使用接口和抽象类
    - 里氏代换原则（Liskov Substitution Principle）
        - 任何基类可以出现的地方，子类一定可以出现
        - LSP 是继承复用的基石，只有当派生类可以替换掉基类，且软件单位的功能不受到影响时，基类才能真正被复用，而派生类也能够在基类的基础上增加新的行为。
        - 实现开闭原则的关键步骤就是抽象化，而基类与子类的继承关系就是抽象化的具体实现，所以里氏代换原则是对实现抽象化的具体步骤的规范。
    - 依赖倒转原则（Dependence Inversion Principle）
        - 针对接口编程，依赖于抽象而不依赖于具体。
    - 接口隔离原则（Interface Segregation Principle）
        - 降低类之间的耦合度
    - 迪米特法则，又称最少知道原则（Demeter Principle）
        - 一个实体应当尽量少地与其他实体之间发生相互作用，使得系统功能模块相对独立。
    - 合成复用原则（Composite Reuse Principle）
        - 尽量使用合成/聚合的方式，而不是使用继承。

```c++
# mother can only read book, 依赖的具体Book
class Book{
	public String getContent(){
		return "很久很久以前有一个阿拉伯的故事……";
	}
}

class Mother{
	public void narrate(Book book){
		System.out.println("妈妈开始讲故事");
		System.out.println(book.getContent());
	}
}

public class Client{
	public static void main(String[] args){
		Mother mother = new Mother();
		mother.narrate(new Book());
	}
} 

# 依赖接口，方便扩展
interface IReader{
	public String getContent();
} 

class Newspaper implements IReader {
	public String getContent(){
		return "林书豪17+9助尼克斯击败老鹰……";
	}
}
class Book implements IReader{
	public String getContent(){
		return "很久很久以前有一个阿拉伯的故事……";
	}
}

class Mother{
    # 依赖IReader接口，子类（Book，Newspaper可以替换IReader）
	public void narrate(IReader reader){
		System.out.println("妈妈开始讲故事");
		System.out.println(reader.getContent());
	}
}

public class Client{
	public static void main(String[] args){
		Mother mother = new Mother();
		mother.narrate(new Book());
		mother.narrate(new Newspaper());
	}
}
```

## [设计模式分类](https://design-patterns.readthedocs.io/zh_CN/latest/read_uml.html)  
- 创建型模式：主要用于描述如何创建对象
    - 简单工厂，工厂方法，抽象工厂方法，单例，建造者
- 结构型模式：主要用于描述如何实现类或对象的组合
    - 适配，桥接，装饰，外观，享元，代理
- 行为型模式：主要用于描述类或对象怎样交互以及怎样分配职责
    - 命令，观察者，中介者，状态，策略

## [具体例子](https://github.com/yogykwan/design-patterns-py)

### 1. 工厂方法模式(Factory Method Pattern)
    
![facion_method](https://design-patterns.readthedocs.io/zh_CN/latest/_images/FactoryMethod.jpg)
    
```python
import abc


class Leifeng:
    __metaclass__ = abc.ABCMeta

    @abc.abstractmethod
    def wash(self):
        """"wash"""

    @abc.abstractmethod
    def sweep(self):
        """sweep"""

    @abc.abstractmethod
    def buy_rice(self):
        """buy rice"""


class Undergraduate(Leifeng):
    def wash(self):
        print "undergraduate wash"

    def sweep(self):
        print "undergraduate sweep"

    def buy_rice(self):
        print "undergraduate buy rice"


class Volunteer(Leifeng):
    def wash(self):
        print "volunteer wash"

    def sweep(self):
        print "volunteer sweep"

    def buy_rice(self):
        print "volunteer buy rice"


class IFactory:
    __metaclass__ = abc.ABCMeta

    @abc.abstractmethod
    def CreateLeifeng(self):
        """create class leifeng"""


class UndergraduateFactory(IFactory):
    def CreateLeifeng(self):
        return Undergraduate()


class VolunteerFactory(IFactory):
    def CreateLeifeng(self):
        return Volunteer()


if __name__ == "__main__":
    # create undergraduate to sweep
    i_factory = UndergraduateFactory()
    leifeng = i_factory.CreateLeifeng()
    leifeng.sweep()

    # create volunteer to wash
    i_factory = VolunteerFactory()  # just replace UndergraduateFactory with VolunteerFactory
    leifeng = i_factory.CreateLeifeng()
    leifeng.wash()
```

### 2. 单例模式

![singleton](https://design-patterns.readthedocs.io/zh_CN/latest/_images/Singleton.jpg)

```python
import threading


class Singleton(object):
    instance = None
    lock = threading.RLock()

    @classmethod
    def __new__(cls):
        if cls.instance is None:
            cls.lock.acquire()
            if cls.instance is None:
                cls.instance = super(Singleton, cls).__new__(cls)
            cls.lock.release()
        return cls.instance


if __name__ == "__main__":
    instance1 = Singleton()
    instance2 = Singleton()
    print id(instance1) == id(instance2)
```

### 3. 装饰模式

![decorator](https://design-patterns.readthedocs.io/zh_CN/latest/_images/Decorator.jpg)

```python
import abc


class Person:
    def show(self):
        print "person"


class Finery(Person):
    def __init__(self, c):
        self.component = c

    def show(self):
        """finery show"""


class Tie(Finery):
    def show(self):
        print "tie ",
        self.component.show()


class Suit(Finery):
    def show(self):
        print "suit ",
        self.component.show()


class Shoes(Finery):
    def show(self):
        print "shoes ",
        self.component.show()


if __name__ == "__main__":
    person = Person()
    tie = Tie(person)
    suit = Suit(tie)
    shoes = Shoes(suit)
    shoes.show()
```

### 4. 观察者模式
![observer](https://design-patterns.readthedocs.io/zh_CN/latest/_images/Obeserver.jpg)

```python
import abc


class Observer:
    __metaclass__ = abc.ABCMeta

    def __init__(self, n, nr):
        self.name = n
        self.notifier = nr

    @abc.abstractmethod
    def update(self):
        """updated by notifier"""


class NbaObserver(Observer):
    def update(self):
        print self.name + ", " + self.notifier.state + ", close NBA"


class StockObserver(Observer):
    def update(self):
        print self.name + ", " + self.notifier.state + ", close stock"


class Notifier:
    def __init__(self):
        self.observers = []
        self.state = ""

    def attach(self, o):
        self.observers.append(o)

    def detach(self, o):
        self.observers.remove(o)

    def notify(self):
        for observer in self.observers:
            observer.update()


class Boss(Notifier):
    pass


if __name__ == "__main__":
    boss = Boss()
    nba_observer = NbaObserver("Bob", boss)
    boss.attach(nba_observer)
    stock_observer = StockObserver("Alice", boss)
    boss.attach(stock_observer)
    boss.state = "boss's back himself"
    boss.notify()
```
  