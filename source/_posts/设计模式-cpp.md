---
title: 设计模式-cpp
date: 2021-02-19 09:42:49
categories: c++
tags:
---

参考：

> [图说设计模式](https://github.com/me115/design_patterns)
>
> [design-patterns-cpp](https://github.com/JakubVojvoda/design-patterns-cpp)
>
> [design-patterns-for-humans](https://github.com/kamranahmedse/design-patterns-for-humans)
>
> http://c.biancheng.net/view/1383.html

### 创建型模式（Creational  Patterns)

#### 简单工厂模式(Simple Factory Pattern)

此模式包含三中角色：

- Factory：工厂，负责所有产品实例的创建
- Product：抽象产品角色，所有产品实例的公共接口
- ConcreteProduct：具体产品角色

![../_images/SimpleFactory.jpg](https://gitee.com/yanzixian/picBed/raw/master/img202102/20210223104529.jpg)

简单工厂使用一个具体的工厂类创建所有的产品实例，因此需要包含必要的判断逻辑，指出何时创建什么产品，或者对于不同的产品使用不同的创建方法；

工厂类没有使用继承形成等级结构，增加一个新的产品需要更改工厂类的内部代码；

代码实现：

```c++
#include <iostream>
#include <string>

/*
 * Product
 * products implement the same interface so that the classes can refer
 * to the interface not the concrete product
 */
class Product
{
public:
  virtual ~Product() {}
  
  virtual std::string getName() = 0;
  // ...
};

/*
 * Concrete Product
 * define product to be created
 */
class ConcreteProductA : public Product
{
public:
  ~ConcreteProductA() {}
  
  std::string getName()
  {
    return "type A";
  }
  // ...
};

/*
 * Concrete Product
 * define product to be created
 */
class ConcreteProductB : public Product
{
public:
  ~ConcreteProductB() {}
  
  std::string getName()
  {
    return "type B";
  }
  // ...
};

/*
 * Factory
 * contains the implementation for all of the methods
 * to manipulate products except for the factory method
 */

class Factory
{
public:
  ~Factory() {}
  
  Product* createProductA()
  {
    return new ConcreteProductA();
  }
  
  Product* createProductB()
  {
    return new ConcreteProductB();
  }
  
  void removeProduct( Product *product )
  {
    delete product;
  }
  // ...
};


int main()
{
  //实例化一个工厂
  Factory *factory = new Factory();
  
  //生成产品A
  Product *p1 = factory->createProductA();
  std::cout << "Product: " << p1->getName() << std::endl;
  factory->removeProduct( p1 );
  //生成产品B
  Product *p2 = factory->createProductB();
  std::cout << "Product: " << p2->getName() << std::endl;
  factory->removeProduct( p2 );
  
  delete factory;
  return 0;
}
```

#### 工厂方法模式（Factory Method Pattern）

工厂方法模式在简单工厂的基础上进一步，不在使用一个工厂类创建所有产品实例，而是定义一个抽象工厂父类，使用继承创建不同的工厂子类，每个不同的产品均有对应具体的工厂类；

包含4个角色：

- Product：抽象产品
- ConcreteProduct：具体产品
- Factory：抽象工厂
- ConcreteFactory：具体工厂

![../_images/FactoryMethod.jpg](https://gitee.com/yanzixian/picBed/raw/master/img202102/20210223111026.jpg)

工厂模式使用一个具体的工厂创建一个具体的产品，新增产品时只需要增加一个具体的具体产品类和对应的具体工厂类即可，无需更改原有的代码；

代码实现：

```c++
#include <iostream>
#include <string>

/*
 * Product
 * products implement the same interface so that the classes can refer
 * to the interface not the concrete product
 */
class Product
{
public:
  virtual ~Product() {}
  
  virtual std::string getName() = 0;
  // ...
};

/*
 * Concrete Product
 * define product to be created
 */
class ConcreteProductA : public Product
{
public:
  ~ConcreteProductA() {}
  
  std::string getName()
  {
    return "type A";
  }
  // ...
};

/*
 * Concrete Product
 * define product to be created
 */
class ConcreteProductB : public Product
{
public:
  ~ConcreteProductB() {}
  
  std::string getName()
  {
    return "type B";
  }
  // ...
};

/*
 * Creator
 * contains the implementation for all of the methods
 * to manipulate products except for the factory method
 */
//抽象工厂
class Creator
{
public:
  virtual ~Creator() {}
  
  virtual Product* createProduct() = 0;
  
  virtual void removeProduct( Product *product ) = 0;
  
  // ...
};

/*
 * Concrete Creator
 * implements factory method that is responsible for creating
 * one or more concrete products ie. it is class that has
 * the knowledge of how to create the products
 */
//具体工厂A，负责创建具体产品A
class ConcreteCreatorA : public Creator
{
public:
  ~ConcreteCreatorA() {}
  
  Product* createProduct()
  {
    return new ConcreteProductA();
  }
  

  
  void removeProduct( Product *product )
  {
    delete product;
  }
  // ...
};

//具体工厂B，负责创建具体产品B
class ConcreteCreatorB : public Creator
{
public:
  ~ConcreteCreatorB() {}
  
  Product* createProduct()
  {
    return new ConcreteProductB();
  }
  
  void removeProduct( Product *product )
  {
    delete product;
  }
  // ...
};

int main()
{
  Creator *creatorA = new ConcreteCreatorA();
  
  Product *p1 = creatorA->createProduct();
  std::cout << "Product: " << p1->getName() << std::endl;
  creatorA->removeProduct( p1 );
  
  Creator *creatorB= new ConcreteCreatorB();
  Product *p2 = creatorB->createProduct();
  std::cout << "Product: " << p2->getName() << std::endl;
  creatorB->removeProduct( p2 );
  
  delete creatorA;
  delete creatorB;
  return 0;
}
```

#### 抽象工厂模式（Abstract Factroy Pattern)

工厂模式一个具体的工厂类只能创建一种产品的实例，而抽象工厂模式进一步扩展，使得一个具体的工厂可以创建多种产品的实例；

包含的角色有：

- AbstractFactory：抽象工厂
- ConcreteFactory：具体工厂
- AbstractProduct：抽象产品
- Product：具体产品

![../_images/AbatractFactory.jpg](https://gitee.com/yanzixian/picBed/raw/master/img202102/20210223162507.jpg)

每个具体工厂可以创建多个不同的产品，抽象工厂角色中规定了所有可能被创建的产品集合，使得难以增加新的产品种类（产品种类即抽象产品类，新增产品种类意味着对于所有子类进行修改）；

每个具体工厂可以对不同种类的具体产品进行组合创建；

代码实现：

```c++
#include <iostream>

/*
 * Product A
 * products implement the same interface so that the classes can refer
 * to the interface not the concrete product
 */
class ProductA
{
public:
  virtual ~ProductA() {}
  
  virtual const char* getName() = 0;
  // ...
};

/*
 * ConcreteProductAX and ConcreteProductAY
 * define objects to be created by concrete factory
 */
class ConcreteProductAX : public ProductA
{
public:
  ~ConcreteProductAX() {}
  
  const char* getName()
  {
    return "A-X";
  }
  // ...
};

class ConcreteProductAY : public ProductA
{
public:
  ~ConcreteProductAY() {}
  
  const char* getName()
  {
    return "A-Y";
  }
  // ...
};

/*
 * Product B
 * same as Product A, Product B declares interface for concrete products
 * where each can produce an entire set of products
 */
class ProductB
{
public:
  virtual ~ProductB() {}
  
  virtual const char* getName() = 0;
  // ...
};

/*
 * ConcreteProductBX and ConcreteProductBY
 * same as previous concrete product classes
 */
class ConcreteProductBX : public ProductB
{
public:
  ~ConcreteProductBX() {}
  
  const char* getName()
  {
    return "B-X";
  }
  // ...
};

class ConcreteProductBY : public ProductB
{
public:
  ~ConcreteProductBY() {}
  
  const char* getName()
  {
    return "B-Y";
  }
  // ...
};

/*
 * Abstract Factory
 * provides an abstract interface for creating a family of products
 */
class AbstractFactory
{
public:
  virtual ~AbstractFactory() {}
  
  virtual ProductA *createProductA() = 0;
  virtual ProductB *createProductB() = 0;
};

/*
 * Concrete Factory X and Y
 * each concrete factory create a family of products and client uses
 * one of these factories so it never has to instantiate a product object
 */
class ConcreteFactoryX : public AbstractFactory
{
public:
  ~ConcreteFactoryX() {}
  
  ProductA *createProductA()
  {
    return new ConcreteProductAX();
  }
  ProductB *createProductB()
  {
    return new ConcreteProductBX();
  }
  // ...
};

class ConcreteFactoryY : public AbstractFactory
{
public:
  ~ConcreteFactoryY() {}

  ProductA *createProductA()
  {
    return new ConcreteProductAY();
  }
  ProductB *createProductB()
  {
    return new ConcreteProductBY();
  }
  // ...
};


int main()
{
  ConcreteFactoryX *factoryX = new ConcreteFactoryX();
  ConcreteFactoryY *factoryY = new ConcreteFactoryY();

  ProductA *p1 = factoryX->createProductA();
  std::cout << "Product: " << p1->getName() << std::endl;
  
  ProductA *p2 = factoryY->createProductA();
  std::cout << "Product: " << p2->getName() << std::endl;
  
  delete p1;
  delete p2;
  
  delete factoryX;
  delete factoryY;
  
  return 0;
}

```

#### 建造者模式（Builder Pattern）

将一个复杂对象的构建与它的表示分离，一步一步创建一个复杂的对象，又称为生成器模式；

包含如下角色：

- Builder：抽象建造者
- ConcreteBuilder：具体建造者
- Director：指挥者
- Product：产品

![../_images/Builder.jpg](https://gitee.com/yanzixian/picBed/raw/master/img202102/20210223203715.jpg)

通过不同的具体建造者，指定不同的生成步骤（步骤内具体实施不同），建造出不同的产品；

代码实现：

```c++
#include <iostream>
#include <string>

/*
 * Product
 * the final object that will be created using Builder
 */
class Product
{
public:
  void makeA( const std::string &part )
  {
    partA = part;
  }
  void makeB( const std::string &part )
  {
    partB = part;
  }
  void makeC( const std::string &part )
  {
    partC = part;
  }
  std::string get()
  {
    return (partA + " " + partB + " " + partC);
  }
  // ...
  
private:
  std::string partA;
  std::string partB;
  std::string partC;
  // ...
};

/*
 * Builder
 * abstract interface for creating products
 */
//抽象建造者，指定建造步骤
class Builder
{
public:
  virtual ~Builder() {}
  
  Product get()
  {
    return product;
  }
  
  virtual void buildPartA() = 0;
  virtual void buildPartB() = 0;
  virtual void buildPartC() = 0;
  // ...

protected:
  Product product;
};

/*
 * Concrete Builder X and Y
 * create real products and stores them in the composite structure
 */
//具体建造者X
class ConcreteBuilderX : public Builder
{
public:
  void buildPartA()
  {
    product.makeA( "A-X" );
  }
  void buildPartB()
  {
    product.makeB( "B-X" );
  }
  void buildPartC()
  {
    product.makeC( "C-X" );
  }
  // ...
};
//具体建造者Y
class ConcreteBuilderY : public Builder
{
public:
  void buildPartA()
  {
    product.makeA( "A-Y" );
  }
  void buildPartB()
  {
    product.makeB( "B-Y" );
  }
  void buildPartC()
  {
    product.makeC( "C-Y" );
  }
  // ...
};

/*
 * Director
 * responsible for managing the correct sequence of object creation
 */
class Director {
public:
  Director() : builder() {}
  
  ~Director()
  {
    if ( builder )
    {
      delete builder;
    }
  }
  
  void set( Builder *b )
  {
    if ( builder )
    {
      delete builder;
    }
    builder = b;
  }
  
  Product get()
  {
    return builder->get();
  }
  
  void construct()
  {
    builder->buildPartA();
    builder->buildPartB();
    builder->buildPartC();
    // ...
  }
  // ...

private:
  Builder *builder;
};


int main()
{
  Director director;
  director.set( new ConcreteBuilderX );
  director.construct();
  
  Product product1 = director.get();
  std::cout << "1st product parts: " << product1.get() << std::endl;
  
  director.set( new ConcreteBuilderY );
  director.construct();
  
  Product product2 = director.get();
  std::cout << "2nd product parts: " << product2.get() << std::endl;
  
  return 0;
}

```

#### 单例模式（Singleton Pattern)

单例模式保证某一个类只有一个实例，而且自行实例化并向整个系统提供；

包含如下角色：

- Singleton：单例

![../_images/Singleton.jpg](https://gitee.com/yanzixian/picBed/raw/master/img202102/20210223211041.jpg)



代码实现：

```c++
#include <iostream>

/*
 * Singleton
 * has private static variable to hold one instance of the class
 * and method which gives us a way to instantiate the class
 */
class Singleton
{
public:
  // The copy constructor and assignment operator
  // are defined as deleted, which means that you
  // can't make a copy of singleton.
  //
  // Note: you can achieve the same effect by declaring
  // the constructor and the operator as private
 //禁止复制和赋值
  Singleton( Singleton const& ) = delete;
  Singleton& operator=( Singleton const& ) = delete;

  static Singleton* get()
  {
    if ( !instance )
    {
      instance = new Singleton();
    }    
    return instance;
  }
  
  static void restart()
  {
    if ( instance )
    {
      delete instance;
    }
  }
  
  void tell()
  {
    std::cout << "This is Singleton." << std::endl;
    // ...
  }
  // ...

private:
  Singleton() {}
  static Singleton *instance;
  // ...
};

Singleton* Singleton::instance = nullptr;


int main()
{
  Singleton::get()->tell();
  Singleton::restart();
  
  return 0;
}

```

#### 原型模式（Prototype Pattern）

原型模式用于从一个存在的对象克隆出一个新对象，当直接创建对象的代价比较大时，采用这种方法；

![design-pattern](https://gitee.com/yanzixian/picBed/raw/master/img202102/20210223231221.png)



代码实现：

```c++
#include <iostream>
#include <string>

/*
 * Prototype
 * declares an interface for cloning itself
 */
class Prototype
{
public:
  virtual ~Prototype() {}
  
  virtual Prototype* clone() = 0;
  virtual std::string type() = 0;
  // ...
};

/*
 * Concrete Prototype A and B
 * implement an operation for cloning itself
 */
class ConcretePrototypeA : public Prototype
{
public:
  ~ConcretePrototypeA() {}
  
  Prototype* clone()
  {
    return new ConcretePrototypeA();
  }
  std::string type()
  {
    return "type A";
  }
  // ...
};

class ConcretePrototypeB : public Prototype
{
public:
  ~ConcretePrototypeB() {}
  
  Prototype* clone()
  {
    return new ConcretePrototypeB();
  }
  std::string type()
  {
    return "type B";
  }
  // ...
};

/*
 * Client
 * creates a new object by asking a prototype to clone itself
 */
class Client
{
public:
  static void init()
  {
    types[ 0 ] = new ConcretePrototypeA();
    types[ 1 ] = new ConcretePrototypeB();
  }
  
  static void remove()
  {
    delete types[ 0 ];
    delete types[ 1 ];
  }
  
  static Prototype* make( const int index )
  {
    if ( index >= n_types )
    {
      return nullptr;
    }    
    return types[ index ]->clone();
  }
  // ...

private:
  static Prototype* types[ 2 ];
  static int n_types;
};

Prototype* Client::types[ 2 ];
int Client::n_types = 2;

int main()
{
  Client::init();
  
  Prototype *prototype1 = Client::make( 0 );
  std::cout << "Prototype: " << prototype1->type() << std::endl;
  delete prototype1;
  
  Prototype *prototype2 = Client::make( 1 );
  std::cout << "Prototype: " << prototype2->type() << std::endl;
  delete prototype2;
  
  Client::remove();
  
  return 0;
}

```



### 结构型模式（Structural Pattern）

#### 适配器模式（Adapter Pattern）

适配器模式用于对特定接口进行转换，用以兼容不匹配的接口

有对象适配器和类适配器两种实现：

包含角色：

- Target：目标抽象类
- Adapter：适配器类
- Adaptee：适配者类
- Client：客户类

对象适配器：

对象适配器采用类成员变量的形式，将适配器类和适配这者类关联在一起

![../_images/Adapter.jpg](https://gitee.com/yanzixian/picBed/raw/master/img202102/20210224141220.jpg)



类适配器：

类适配器采用多继承的方法，将适配器类和适配者类关联在一起，有些语言不支持多继承，因此，此方式存在一定的局限性

![../_images/Adapter_classModel.jpg](https://gitee.com/yanzixian/picBed/raw/master/img202102/20210224141251.jpg)



代码实现：

对象适配器：

```c++
#include <iostream>

/*
 * Target
 * defines specific interface that Client uses
 */
class Target
{
public:
  virtual ~Target() {}
  
  virtual void request() = 0;
  // ...
};

/*
 * Adaptee
 * defines an existing interface that needs adapting and thanks
 * to Adapter it will get calls that client makes on the Target
 *
 */
class Adaptee
{
public:
  void specificRequest()
  {
    std::cout << "specific request" << std::endl;
  }
  // ...
};

/*
 * Adapter
 * implements the Target interface and when it gets a method call it
 * delegates the call to a Adaptee
 */
class Adapter : public Target
{
public:
  Adapter() : adaptee() {}
  
  ~Adapter()
  {
    delete adaptee;
  }
  
  void request()
  {
    adaptee->specificRequest();
  // ...
  }
  // ...

private:
  Adaptee *adaptee;
  // ...
};


int main()
{
  Target *t = new Adapter();
  t->request();
  delete t;
  
  return 0;
}

```

类适配器：

```c++
#include <iostream>

/*
 * Target
 * defines specific interface that Client uses
 */
class Target
{
public:
  virtual ~Target() {}
  
  virtual void request() = 0;
  // ...
};

/*
 * Adaptee
 * all requests get delegated to the Adaptee which defines
 * an existing interface that needs adapting
 */
class Adaptee
{
public:
  ~Adaptee() {}
  
  void specificRequest()
  {
    std::cout << "specific request" << std::endl;
  }
  // ...
};

/*
 * Adapter
 * implements the Target interface and lets the Adaptee respond
 * to request on a Target by extending both classes
 * ie adapts the interface of Adaptee to the Target interface
 */
class Adapter : public Target, private Adaptee
{
public:
  virtual void request()
  {
    specificRequest();
  }
  // ...
};


int main()
{
  Target *t = new Adapter();
  t->request();
  delete t;
  
  return 0;
}

```

#### 桥接模式（Bridge Pattern）

将抽象部分与实现部分分离，使它们可以独立地变化

包含如下角色：

- Abstraction：抽象类
- RefinedAbstraction：扩充抽象类
- Implementor：实现类接口
- ConcreteImplementor：具体实现类

![../_images/Bridge.jpg](https://gitee.com/yanzixian/picBed/raw/master/img202102/20210224153448.jpg)

代码实现：

```c++
#include <iostream>

/*
 * Implementor
 * defines the interface for implementation classes
 */
class Implementor
{
public:
  virtual ~Implementor() {}
  
  virtual void action() = 0;
  // ...
};

/*
 * Concrete Implementors
 * implement the Implementor interface and define concrete implementations
 */
class ConcreteImplementorA : public Implementor
{
public:
  ~ConcreteImplementorA() {}
  
  void action()
  {
    std::cout << "Concrete Implementor A" << std::endl;
  }
  // ...
};

class ConcreteImplementorB : public Implementor
{
public:
  ~ConcreteImplementorB() {}
  
  void action()
  {
    std::cout << "Concrete Implementor B" << std::endl;
  }
  // ...
};

/*
 * Abstraction
 * defines the abstraction's interface
 */
class Abstraction
{
public:
  virtual ~Abstraction() {}
  
  virtual void operation() = 0;
  // ...
};

/*
 * RefinedAbstraction
 * extends the interface defined by Abstraction
 */
class RefinedAbstraction : public Abstraction
{
public:
  ~RefinedAbstraction() {}
  
  RefinedAbstraction(Implementor *impl) : implementor(impl) {}
  
  void operation()
  {
    implementor->action();
  }
  // ...

private:
  Implementor *implementor;
};


int main()
{
  Implementor *ia = new ConcreteImplementorA;
  Implementor *ib = new ConcreteImplementorB;
  
  Abstraction *abstract1 = new RefinedAbstraction(ia);
  abstract1->operation();
  
  Abstraction *abstract2 = new RefinedAbstraction(ib);
  abstract2->operation();
  
  delete abstract1;
  delete abstract2;
  
  delete ia;
  delete ib;
  
  return 0;
}

```

#### 装饰模式（Decorator Pattern）

此模式可以动态地给一个对象增加一些额外的职责、功能或行为，比生成子类更加灵活

包含一下角色：

- Component: 抽象构件
- ConcreteComponent: 具体构件
- Decorator: 抽象装饰类
- ConcreteDecorator: 具体装饰类

![../_images/Decorator.jpg](https://gitee.com/yanzixian/picBed/raw/master/img202102/20210224193329.jpg)

装饰类和具体构件类均继承自抽象构件类，但装饰类新增了一个抽象构件类型的类成员，通过这种组合方式，可以较为自由的添加新功能

代码实现：

```c++
#include <iostream>

/*
 * Component
 * defines an interface for objects that can have responsibilities
 * added to them dynamically
 */
class Component
{
public:
  virtual ~Component() {}
  
  virtual void operation() = 0;
  // ...
};

/*
 * Concrete Component
 * defines an object to which additional responsibilities
 * can be attached
 */
class ConcreteComponent : public Component
{
public:
  ~ConcreteComponent() {}
  
  void operation()
  {
    std::cout << "Concrete Component operation" << std::endl;
  }
  // ...
};

/*
 * Decorator
 * maintains a reference to a Component object and defines an interface
 * that conforms to Component's interface
 */
class Decorator : public Component
{
public:
  ~Decorator() {}
  
  Decorator( Component *c ) : component( c ) {}
  
  virtual void operation()
  {
    component->operation();
  }
  // ...

private:
  Component *component;
};

/*
 * Concrete Decorators
 * add responsibilities to the component (can extend the state
 * of the component)
 */
class ConcreteDecoratorA : public Decorator
{
public:
  ConcreteDecoratorA( Component *c ) : Decorator( c ) {}
  
  void operation()
  {
    Decorator::operation();
    this->addBehavior();
  }

  void addBehavior(){
      std::cout<<"Decorator A: add behavior"<<std::endl;
  }
  // ...
};

class ConcreteDecoratorB : public Decorator
{
public:
  ConcreteDecoratorB( Component *c ) : Decorator( c ) {}
  
  void operation()
  {
    Decorator::operation();
    this->addBehavior();
  }

   void addBehavior(){
      std::cout<<"Decorator B: add behavior"<<std::endl;
  }
  // ...
};


int main()
{
  ConcreteComponent  *cc = new ConcreteComponent();
  ConcreteDecoratorB *db = new ConcreteDecoratorB( cc );
  ConcreteDecoratorA *da = new ConcreteDecoratorA( db );
  
  Component *component = da;
  component->operation();
  
  delete da;
  delete db;
  delete cc;
  
  return 0;
}

```

#### 外观模式（Facade Pattern）

外部与子系统的通信通过一个统一的外观对象进行，为子系统的一组接口提供与一个一致的界面，通过定义一个高层接口的方式实现，使子系统更易于使用；

包含以下角色：

- Facade：外观角色
- SubSystem：子系统角色

![../_images/Facade.jpg](https://gitee.com/yanzixian/picBed/raw/master/img202102/20210308143202.jpg)

代码实现：

```c++
#include <iostream>

/*
 * Subsystems
 * implement more complex subsystem functionality
 * and have no knowledge of the facade
 */
class SubsystemA
{
public:
  void suboperation()
  {
    std::cout << "Subsystem A method" << std::endl;
    // ...
  }
  // ...
};

class SubsystemB
{
public:
  void suboperation()
  {
    std::cout << "Subsystem B method" << std::endl;
    // ...
  }
  // ...
};

class SubsystemC
{
public:
  void suboperation()
  {
    std::cout << "Subsystem C method" << std::endl;
    // ...
  }
  // ...
};

/*
 * Facade
 * delegates client requests to appropriate subsystem object
 * and unified interface that is easier to use
 */
class Facade
{
public:
  Facade() : subsystemA(), subsystemB(), subsystemC() {}
  
  void operation1()
  {
    subsystemA->suboperation();
    subsystemB->suboperation();
    // ...
  }
  
  void operation2()
  {
    subsystemC->suboperation();
    // ...
  }
  // ...
  
private:
  SubsystemA *subsystemA;
  SubsystemB *subsystemB;
  SubsystemC *subsystemC;
  // ...
};


int main()
{
  Facade *facade = new Facade();
  
  facade->operation1();
  facade->operation2();
  delete facade;
  
  return 0;
}

```



#### 享元模式（Flyweight Pattern）

面向对象技术中，很多情况下需要增加很多类和对象，对象数量太多会导致运行代价增高，性能下降；

- 享元模式通过共享技术实现相同或相似对象的重用；
- 可以共享的相同内容称为内部状态（IntrinsicState），需要外部环境设置的不能共享的内容称为外部状态（Extrinsic State）
- 通常会出现工厂模式，通过创建一个享元工厂来负责维护一个享元池（Flyweight Pool）
- 共享的是内部状态，能够共享的内部状态一般都是有限的，因此享元对象一般都设计为较小的对象

包含以下角色：

- Flyweight: 抽象享元类
- ConcreteFlyweight: 具体享元类
- UnsharedConcreteFlyweight: 非共享具体享元类
- FlyweightFactory: 享元工厂类

![../_images/Flyweight.jpg](https://gitee.com/yanzixian/picBed/raw/master/img202103/20210308145945.jpg)

代码实现：

```c++
#include <iostream>
#include <map>

/*
 * Flyweight
 * declares an interface through which flyweights can receive
 * and act on extrinsic state
 */
class Flyweight
{
public:
  virtual ~Flyweight() {}
  virtual void operation() = 0;
  // ...
};

/*
 * UnsharedConcreteFlyweight
 * not all subclasses need to be shared
 */
class UnsharedConcreteFlyweight : public Flyweight
{
public:
  UnsharedConcreteFlyweight( const int intrinsic_state ) :
    state( intrinsic_state ) {}
  
  ~UnsharedConcreteFlyweight() {}
  
  void operation()
  {
    std::cout << "Unshared Flyweight with state " << state << std::endl;
  }
  // ...
  
private:
  int state;
  // ...
};

/*
 * ConcreteFlyweight
 * implements the Flyweight interface and adds storage
 * for intrinsic state
 */
class ConcreteFlyweight : public Flyweight
{
public:
  ConcreteFlyweight( const int all_state ) :
    state( all_state ) {}
  
  ~ConcreteFlyweight() {}
  
  void operation()
  {
    std::cout << "Concrete Flyweight with state " << state << std::endl;
  }
  // ...
  
private:
  int state;
  // ...
};

/*
 * FlyweightFactory
 * creates and manages flyweight objects and ensures
 * that flyweights are shared properly
 */
class FlyweightFactory
{
public:
  ~FlyweightFactory()
  {
    for ( auto it = flies.begin(); it != flies.end(); it++ )
    {
        delete it->second;
    }
    flies.clear();
  }
  
  Flyweight *getFlyweight( const int key )
  {
    if ( flies.find( key ) != flies.end() )
    {
      return flies[ key ];
    }
    Flyweight *fly = new ConcreteFlyweight( key );
    flies.insert( std::pair<int, Flyweight *>( key, fly ) );
    return fly;
  }
  // ...

private:
  std::map<int, Flyweight*> flies;
  // ...
};


int main()
{
  FlyweightFactory *factory = new FlyweightFactory;
  factory->getFlyweight(1)->operation();
  factory->getFlyweight(2)->operation();
  delete factory;
  return 0;
}

```

使用`map`存储享元对象，内部状态`state`作为`key`，相同状态的对象只会有一个；



#### 代理模式（Proxy Pattern）

某些情况下，不直接引用一个对象，而是通过一个称之为“代理”的第三者来实现简介引用；代理对象起到中介的作用，通过代理对象可以隐藏被代理对象内容或增加某些新服务；

包含以下角色:

- Subject: 抽象主题角色
- Proxy: 代理主题角色
- RealSubject: 真实主题角色

![../_images/Proxy.jpg](https://gitee.com/yanzixian/picBed/raw/master/img202103/20210308152423.jpg)

代码实现：

```c++
#include <iostream>

/*
 * Subject
 * defines the common interface for RealSubject and Proxy
 * so that a Proxy can be used anywhere a RealSubject is expected
 */
class Subject
{
public:
  virtual ~Subject() { /* ... */ }

  virtual void request() = 0;
  // ...
};

/*
 * Real Subject
 * defines the real object that the proxy represents
 */
class RealSubject : public Subject
{
public:
  void request()
  {
    std::cout << "Real Subject request" << std::endl;
  }
  // ...
};

/*
 * Proxy
 * maintains a reference that lets the proxy access the real subject
 */
class Proxy : public Subject
{
public:
  Proxy()
  {
    subject = new RealSubject();
  }
  
  ~Proxy()
  {
    delete subject;
  }
  
  void request()
  {
    subject->request();
  }
  // ...

private:
  RealSubject *subject;
};


int main()
{
  Proxy *proxy = new Proxy();
  proxy->request();
  
  delete proxy;
  return 0;
}

```



#### 组合模式（Composite Pattern）

组合模式，又叫部分整体模式，是用于把一组相似的对象当作一个单一的对象。组合模式依据树形结构来组合对象，用来表示部分以及整体层次。这种类型的设计模式属于结构型模式，它创建了对象组的树形结构；

组合模式使客户对于单个对象和组合对象的处理具有一致性，无需进行区分；

包含以下角色：

- Component：抽象组件
- Composite：组合对象
- Leaf：单个对象

![03205131-81741e8e3eb54002b880e7f845064b8d](https://gitee.com/yanzixian/picBed/raw/master/img202103/20210308162354.jpg)

代码实现：

```c++
#include <iostream>
#include <vector>

/*
 * Component
 * defines an interface for all objects in the composition
 * both the composite and the leaf nodes
 */
class Component
{
public:
  virtual ~Component() {}
  
  virtual Component *getChild( int )
  {
    return 0;
  }
  
  virtual void add( Component * ) { /* ... */ }
  virtual void remove( int ) { /* ... */ }
  
  virtual void operation() = 0;
};

/*
 * Composite
 * defines behavior of the components having children
 * and store child components
 */
class Composite : public Component
{
public:
  ~Composite()
  {
    for ( unsigned int i = 0; i < children.size(); i++ )
    {
      delete children[ i ];
    }
  }
  
  Component *getChild( const unsigned int index )
  {
    return children[ index ];
  }
  
  void add( Component *component )
  {
    children.push_back( component );
  }
  
  void remove( const unsigned int index )
  {
    Component *child = children[ index ];
    children.erase( children.begin() + index );
    delete child;
  }
  
  void operation()
  {
    for ( unsigned int i = 0; i < children.size(); i++ )
    {
      children[ i ]->operation();
    }
  }
  
private:
  std::vector<Component*> children;
};

/*
 * Leaf
 * defines the behavior for the elements in the composition,
 * it has no children
 */
class Leaf : public Component
{
public:
  Leaf( const int i ) : id( i ) {}
  
  ~Leaf() {}
  
  void operation()
  {
    std::cout << "Leaf "<< id <<" operation" << std::endl;
  }

private:
  int id;
};


int main()
{
  Composite composite;
  
  for ( unsigned int i = 0; i < 5; i++ )
  {
    composite.add( new Leaf( i ) );
  }
  
  composite.remove( 0 );
  composite.operation();
  
  return 0;
}

```



### 行为型模式（Behavioral Pattern）



#### 模板方法模式（Tempalte-Method Pattern）

定义一个操作中的算法骨架，而将算法的一些步骤延迟到子类中，使得子类可以不改变该算法结构的情况下重定义该算法的某些特定步骤。它是一种类行为型模式。

包含以下角色：

- Abstract Class：抽象类，抽象模板
- Concrete Class：具体子类，具体实现

代码实现：

```c++
#include <iostream>

/*
 * AbstractClass
 * implements a template method defining the skeleton of an algorithm
 */
class AbstractClass
{
public:
  virtual ~AbstractClass() {}
  
  void templateMethod()
  {
    // ...
    primitiveOperation1();
    // ...
    primitiveOperation2();
    // ...
  }
  
  virtual void primitiveOperation1() = 0;
  virtual void primitiveOperation2() = 0;
  // ...
};

/*
 * Concrete Class
 * implements the primitive operations to carry out specific steps
 * of the algorithm, there may be many Concrete classes, each implementing
 * the full set of the required operation
 */
class ConcreteClass : public AbstractClass
{
public:
  ~ConcreteClass() {}
  
  void primitiveOperation1()
  {
    std::cout << "Primitive operation 1" << std::endl;
    // ...
  }
  
  void primitiveOperation2()
  {
    std::cout << "Primitive operation 2" << std::endl;
    // ...
  }
  // ...
};


int main()
{
  AbstractClass *tm = new ConcreteClass;
  tm->templateMethod();
  
  delete tm;
  return 0;
}

```



#### 责任链模式（Chain of Responsibility）

为了避免请求发送者与多个请求处理者耦合在一起，于是将所有请求的处理者通过前一对象记住其下一个对象的引用而连成一条链；当有请求发生时，可将请求沿着这条链传递，直到有对象处理它为止；

包含以下角色：

- Handler：抽象处理者
- ConcreteHandler：具体处理者
- Client：客户角色

![责任链模式的结构图](https://gitee.com/yanzixian/picBed/raw/master/img202103/20210309144100.gif)



代码实现：

```c++
#include <iostream>

/*
 * Handler
 * defines an interface for handling requests and
 * optionally implements the successor link
 */
class Handler
{
public:
  virtual ~Handler() {}
  
  virtual void setHandler( Handler *s )
  {
    successor = s;
  }
  
  virtual void handleRequest()
  {
    if (successor != 0)
    {
      successor->handleRequest();
    }
  }
  // ...

private:
  Handler *successor;
};

/*
 * Concrete Handlers
 * handle requests they are responsible for
 */
class ConcreteHandler1 : public Handler
{
public:
  ~ConcreteHandler1() {}
  
  bool canHandle()
  {
    // ...
    return false;
  }
  
  virtual void handleRequest()
  {
    if ( canHandle() )
    {
      std::cout << "Handled by Concrete Handler 1" << std::endl;
    }
    else
    {
      std::cout << "Cannot be handled by Handler 1" << std::endl;
      Handler::handleRequest();
    }
    // ...
  }
  // ...
};

class ConcreteHandler2 : public Handler
{
public:
  ~ConcreteHandler2() {}
  
  bool canHandle()
  {
    // ...
    return true;
  }
  
  virtual void handleRequest()
  {
    if ( canHandle() )
    {
      std::cout << "Handled by Handler 2" << std::endl;
    }
    else
    {
      std::cout << "Cannot be handled by Handler 2" << std::endl;
      Handler::handleRequest();
    }
    // ...
  }
  
  // ...
};


int main()
{
  ConcreteHandler1 handler1;
  ConcreteHandler2 handler2;
  
  handler1.setHandler( &handler2 );
  handler1.handleRequest();
  
  return 0;
}

```



#### 命令模式（Command Pattern）

命令模式将一个请求封装为一个对象，从而使我们可用不同的请求对客户进行参数化；对请求排队或者记录请求日志，以及支持可撤销的操作。命令模式是一种对象行为型模式，其别名为动作(Action)模式或事务(Transaction)模式；

命令模式可以对发送者和接收者完全解耦，发送者与接收者之间没有直接引用关系，发送请求的对象只需要知道如何发送请求，而不必知道如何完成请求。

包含以下角色：

- Command: 抽象命令类
- ConcreteCommand: 具体命令类
- Invoker: 调用者
- Receiver: 接收者
- Client:客户类

![../_images/Command.jpg](https://gitee.com/yanzixian/picBed/raw/master/img202103/20210308164146.jpg)

代码实现：

```c++
#include <iostream>

/*
 * Receiver
 * knows how to perform the operations associated
 * with carrying out a request
 */
class Receiver
{
public:
  void action()
  {
    std::cout << "Receiver: execute action" << std::endl;
  }
  // ...
};

/*
 * Command
 * declares an interface for all commands
 */
class Command
{
public:
  virtual ~Command() {}
  virtual void execute() = 0;
  // ...

protected:
  Command() {}
};

/*
 * Concrete Command
 * implements execute by invoking the corresponding
 * operation(s) on Receiver
 */
class ConcreteCommand : public Command
{
public:
  ConcreteCommand( Receiver *r ) : receiver( r ) {}
  
  ~ConcreteCommand()
  {
    if ( receiver )
    {
      delete receiver;
    }
  }
  
  void execute()
  {
    receiver->action();
  }
  // ...
  
private:
  Receiver *receiver;
  // ...
};

/*
 * Invoker
 * asks the command to carry out the request
 */
class Invoker
{
public:
  void set( Command *c )
  {
    command = c;
  }
  
  void confirm()
  {
    if ( command )
    {
      command->execute();  
    }
  }
  // ...

private:
  Command *command;
  // ...
};


int main()
{
  ConcreteCommand command( new Receiver() );
  
  Invoker invoker;
  invoker.set( &command );
  invoker.confirm();
  
  return 0;
}

```



#### 中介者模式（Mediator Pattern）

用一个中介对象来封装一系列的对象交互，中介者使各对象不需要显式地相互引用，从而使其耦合松散，而且可以独立地改变它们之间的交互。中介者模式又称为调停者模式，它是一种对象行为型模式；

中介者模式使得消息发送者和接收者之间的耦合较为松散，且可以随时增加或减少对象；

包含以下角色：

- Mediator: 抽象中介者
- ConcreteMediator: 具体中介者
- Colleague: 抽象同事类
- ConcreteColleague: 具体同事类

![../_images/Mediator.jpg](https://gitee.com/yanzixian/picBed/raw/master/img202103/20210309095837.jpg)

代码实现：

```c++
#include <iostream>
#include <vector>
#include <string>

class Mediator;

/*
 * Colleague classes
 * each colleague communicates with its mediator whenever
 * it would have otherwise communicated with another colleague
 */
class Colleague
{
public:
  Colleague( Mediator* const m, const unsigned int i ) : 
    mediator( m ), id( i ) {}
  
  virtual ~Colleague() {}
  
  unsigned int getID()
  {
    return id;
  }
  
  virtual void send( std::string ) = 0;
  virtual void receive( std::string ) = 0;

protected:
  Mediator *mediator;
  unsigned int id;
};

class ConcreteColleague : public Colleague
{
public:
  ConcreteColleague( Mediator* const m, const unsigned int i ) : 
    Colleague( m, i ) {}
  
  ~ConcreteColleague() {}
  
  void send( std::string msg );
  
  void receive( std::string msg )
  {
    std::cout << "Message '" << msg << "' received by Colleague " << id << std::endl;
  }
};

/*
 * Mediator
 * defines an interface for communicating with Colleague objects
 */
class Mediator
{
public:
  virtual ~Mediator() {}
  
  virtual void add( Colleague* const c ) = 0;
  virtual void distribute( Colleague* const sender, std::string msg ) = 0;

protected:
  Mediator() {}
};

/*
 * Concrete Mediator
 * implements cooperative behavior by coordinating Colleague objects
 * and knows its colleagues
 */
class ConcreteMediator : public Mediator
{
public:
  ~ConcreteMediator()
  {
    for ( unsigned int i = 0; i < colleagues.size(); i++ )
    {
      delete colleagues[ i ];
    }
    colleagues.clear();
  }
  
  void add( Colleague* const c )
  {
    colleagues.push_back( c );
  }
  
  void distribute( Colleague* const sender, std::string msg )
  {
    for ( unsigned int i = 0; i < colleagues.size(); i++ )
    {
      if ( colleagues.at( i )->getID() != sender->getID() )
      {
        colleagues.at( i )->receive( msg );
      }
    }
  }

private:
  std::vector<Colleague*> colleagues;
};

void ConcreteColleague::send( std::string msg )
{
  std::cout << "Message '"<< msg << "' sent by Colleague " << id << std::endl;
  mediator->distribute( this, msg );
}


int main()
{
  Mediator *mediator = new ConcreteMediator();
  
  Colleague *c1 = new ConcreteColleague( mediator, 1 );
  Colleague *c2 = new ConcreteColleague( mediator, 2 );
  Colleague *c3 = new ConcreteColleague( mediator, 3 );
  
  mediator->add( c1 );
  mediator->add( c2 );
  mediator->add( c3 );
  
  c1->send( "Hi!" );
  c3->send( "Hello!" );
  
  delete mediator;
  return 0;
}

```



#### 观察者模式（Observer Pattern）

定义对象间的一种一对多依赖关系，使得每当一个对象状态发生改变时，其相关依赖对象皆得到通知并被自动更新。

观察者模式又叫做发布-订阅（Publish/Subscribe）模式、模型-视图（Model/View）模式、源-监听器（Source/Listener）模式或从属者（Dependents）模式。

包含如下角色：

- Subject: 目标
- ConcreteSubject: 具体目标
- Observer: 观察者
- ConcreteObserver: 具体观察者

![../_images/Obeserver.jpg](https://gitee.com/yanzixian/picBed/raw/master/img202103/20210309103353.jpg)



代码实现：

```c++
#include <iostream>
#include <vector>

class Subject;

/*
 * Observer
 * defines an updating interface for objects that should be notified
 * of changes in a subject
 */
class Observer
{
public:
  virtual ~Observer() {}
  
  virtual int getState() = 0;
  virtual void update( Subject *subject ) = 0;
  // ...
};

/*
 * Concrete Observer
 * stores state of interest to ConcreteObserver objects and
 * sends a notification to its observers when its state changes
 */
class ConcreteObserver : public Observer
{
public:
  ConcreteObserver( const int state ) :
    observer_state( state ) {}
  
  ~ConcreteObserver() {}
  
  int getState()
  {
    return observer_state;
  }
  
  void update( Subject *subject );
  // ...

private:
  int observer_state;
  // ...
};

/*
 * Subject
 * knows its observers and provides an interface for attaching
 * and detaching observers
 */
class Subject
{
public:
  virtual ~Subject() {}
  
  void attach( Observer *observer )
  {
    observers.push_back(observer);
  }
  
  void detach( const int index )
  {
    observers.erase( observers.begin() + index );
  }
  
  void notify()
  {
    for ( unsigned int i = 0; i < observers.size(); i++ )
    {
      observers.at( i )->update( this );
    }
  }
  
  virtual int getState() = 0;
  virtual void setState( const int s ) = 0;
  // ...

private:
  std::vector<Observer*> observers;
  // ...
};

/*
 * Concrete Subject
 * stores state that should stay consistent with the subject's
 */
class ConcreteSubject : public Subject
{
public:
  ~ConcreteSubject() {}
  
  int getState()
  {
    return subject_state;
  }
  
  void setState( const int s )
  {
    subject_state = s;
  }
  // ...
  
private:
  int subject_state;
  // ...
};

void ConcreteObserver::update( Subject *subject )
{
  observer_state = subject->getState();
  std::cout << "Observer state updated." << std::endl;
}


int main()
{
  ConcreteObserver observer1( 1 );
  ConcreteObserver observer2( 2 );
  
  std::cout << "Observer 1 state: " << observer1.getState() << std::endl;
  std::cout << "Observer 2 state: " << observer2.getState() << std::endl;
  
  Subject *subject = new ConcreteSubject();
  subject->attach( &observer1 );
  subject->attach( &observer2 );
  
  subject->setState( 10 );
  subject->notify();
  
  std::cout << "Observer 1 state: " << observer1.getState() << std::endl;
  std::cout << "Observer 2 state: " << observer2.getState() << std::endl;
  
  delete subject;
  return 0;
}
```



#### 状态模式（State Pattern）

允许一个对象在其内部状态改变时改变它的行为，对象看起来似乎修改了它的类。其别名为状态对象(Objects for States)，状态模式是一种对象行为型模式。

包含如下角色：

- Context: 环境类
- State: 抽象状态类
- ConcreteState: 具体状态类

![../_images/State.jpg](https://gitee.com/yanzixian/picBed/raw/master/img202103/20210309105642.jpg)

代码实现：

```c++
#include <iostream>

/*
 * State
 * defines an interface for encapsulating the behavior associated
 * with a particular state of the Context
 */
class State
{
public:
  virtual ~State() { /* ... */ }
  virtual void handle() = 0;
  // ...
};

/*
 * Concrete States
 * each subclass implements a behavior associated with a state
 * of the Context
 */
class ConcreteStateA : public State
{
public:
  ~ConcreteStateA() { /* ... */ }
  
  void handle()
  {
    std::cout << "State A handled." << std::endl;
  }
  // ...
};

class ConcreteStateB : public State
{
public:
  ~ConcreteStateB() { /* ... */ }
  
  void handle()
  {
    std::cout << "State B handled." << std::endl;
  }
  // ...
};

/*
 * Context
 * defines the interface of interest to clients
 */
class Context
{
public:
  Context() : state() { /* ... */ }
  
  ~Context()
  {
    delete state;
  }
  
  void setState( State* const s )
  {
    if ( state )
    {
      delete state;
    }
    state = s;
  }
  
  void request()
  {
    state->handle();
  }
  // ...

private:
  State *state;
  // ...
};


int main()
{
  Context *context = new Context();
  
  context->setState( new ConcreteStateA() );
  context->request();
  
  context->setState( new ConcreteStateB() );
  context->request();
  
  delete context;
  return 0;
}

```



#### 策略模式（Strategy Pattern）

定义一系列算法，将每一个算法封装起来，并让它们可以相互替换。策略模式让算法独立于使用它的客户而变化，也称为政策模式(Policy)。

包含如下角色：

- Context: 环境类
- Strategy: 抽象策略类
- ConcreteStrategy: 具体策略类

![../_images/Strategy.jpg](https://gitee.com/yanzixian/picBed/raw/master/img202103/20210309110213.jpg)



代码实现：

```c++
#include <iostream>

/*
 * Strategy
 * declares an interface common to all supported algorithms
 */
class Strategy
{
public:
  virtual ~Strategy() { /* ... */ }
  virtual void algorithmInterface() = 0;
  // ...
};

/*
 * Concrete Strategies
 * implement the algorithm using the Strategy interface
 */
class ConcreteStrategyA : public Strategy
{
public:
  ~ConcreteStrategyA() { /* ... */ }
  
  void algorithmInterface()
  {
    std::cout << "Concrete Strategy A" << std::endl;
  }
  // ...
};

class ConcreteStrategyB : public Strategy
{
public:
  ~ConcreteStrategyB() { /* ... */ }
  
  void algorithmInterface()
  {
    std::cout << "Concrete Strategy B" << std::endl;
  }
  // ...
};

class ConcreteStrategyC : public Strategy
{
public:
  ~ConcreteStrategyC() { /* ... */ }
  
  void algorithmInterface()
  {
    std::cout << "Concrete Strategy C" << std::endl;
  }
  // ...
};

/*
 * Context
 * maintains a reference to a Strategy object
 */
class Context
{
public:
  Context( Strategy* const s ) : strategy( s ) {}
  
  ~Context()
  {
    delete strategy;
  }
  
  void contextInterface()
  {
    strategy->algorithmInterface();
  }
  // ...

private:
  Strategy *strategy;
  // ...
};


int main()
{
  Context context( new ConcreteStrategyA() );
  context.contextInterface();
  
  return 0;
}
```



#### 迭代器模式（Iterator Pattern）

提供一个对象来顺序访问聚合对象中的一系列数据，而不暴露聚合对象的内部表示。迭代器模式是一种对象行为型模式。

迭代器模式是通过将聚合对象的遍历行为分离出来，抽象成迭代器类来实现的，其目的是在不暴露聚合对象的内部结构的情况下，让外部代码透明地访问聚合的内部数据。

通常不会自己写迭代器，除非需要定制一个自己实现的数据结构对应的迭代器。

包含角色：

- Aggregate：抽象聚合角色
- ConcreteAggregate：具体聚合角色
- Iterator：抽象迭代器
- ConcreteIterator：具体迭代器

![迭代器模式的结构图](https://gitee.com/yanzixian/picBed/raw/master/img202103/20210309144921.gif)

代码实现：

```c++
#include <iostream>
#include <stdexcept>
#include <vector>

class Iterator;
class ConcreteAggregate;

/*
 * Aggregate
 * defines an interface for aggregates and it decouples your
 * client from the implementation of your collection of objects
 */
class Aggregate
{
public:
  virtual ~Aggregate() {}
  
  virtual Iterator *createIterator() = 0;
  // ...
};

/*
 * Concrete Aggregate
 * has a collection of objects and implements the method
 * that returns an Iterator for its collection
 *
 */
class ConcreteAggregate : public Aggregate
{
public:
  ConcreteAggregate( const unsigned int size )
  {
    list = new int[size]();
    count = size;
  }
  
  ~ConcreteAggregate()
  {
    delete[] list;
  }
  
  Iterator *createIterator();
  
  unsigned int size() const
  {
    return count;
  }
  
  int at( unsigned int index )
  {
    return list[ index ];
  }
  // ...

private:
  int *list;
  unsigned int count;
  // ...
};

/*
 * Iterator
 * provides the interface that all iterators must implement and
 * a set of methods for traversing over elements
 */
class Iterator
{
public:
  virtual ~Iterator() { /* ... */ }
  
  virtual void first() = 0;
  virtual void next() = 0;
  virtual bool isDone() const = 0;
  virtual int currentItem() const = 0;
  // ...
};

/*
 * Concrete Iterator
 * implements the interface and is responsible for managing
 * the current position of the iterator
 */
class ConcreteIterator : public Iterator
{
public:
  ConcreteIterator( ConcreteAggregate *l ) :
    list( l ), index( 0 ) {}
  
  ~ConcreteIterator() {}
  
  void first()
  {
    index = 0;
  }
  
  void next()
  {
    index++;
  }
  
  bool isDone() const
  {
    return ( index >= list->size() );
  }
  
  int currentItem() const
  {
    if ( isDone() )
    {
      return -1;
    }
    return list->at(index);
  }
  // ...

private:
  ConcreteAggregate *list;
  unsigned int index;
  // ...
};

Iterator *ConcreteAggregate::createIterator()
{
  return new ConcreteIterator( this );
}


int main()
{
  unsigned int size = 5;
  ConcreteAggregate list = ConcreteAggregate( size );
  
  Iterator *it = list.createIterator();
  for ( ; !it->isDone(); it->next())
  {
    std::cout << "Item value: " << it->currentItem() << std::endl;
  }
  
  delete it;
  return 0;
}
```



#### 备忘录模式（Memento Pattern）

在不破坏封装性的前提下，捕获一个对象的内部状态，并在该对象之外保存这个状态，以便以后当需要时能将该对象恢复到原先保存的状态。该模式又叫快照模式。

包含角色：

- Originator：发起者角色，记录当前内部状态信息，提供创建备忘录和恢复备忘录功能
- Mememto：备忘录角色，存储备忘录信息
- Caretaker：管理者角色，对备忘录进行管理

![备忘录模式的结构图](https://gitee.com/yanzixian/picBed/raw/master/img202103/20210309155148.gif)



代码实现：

```c++
#include <iostream>
#include <vector>

/*
 * Memento
 * stores internal state of the Originator object and protects
 * against access by objects other than the originator
 */
class Memento
{
private:
  // accessible only to Originator
  friend class Originator;
  
  Memento( const int s ) : state( s ) {}
  
  void setState( const int s )
  {
    state = s;
  }
  
  int getState()
  {
    return state;
  }
  // ...

private:
  int state;
  // ...
};

/*
 * Originator
 * creates a memento containing a snapshot of its current internal
 * state and uses the memento to restore its internal state
 */
class Originator
{
public:
  // implemented only for printing purpose
  void setState( const int s )
  {
    std::cout << "Set state to " << s << "." << std::endl;
    state = s;
  }
  
  // implemented only for printing purpose
  int getState()
  {
    return state;
  }
  
  void setMemento( Memento* const m )
  {
    state = m->getState();
  }
  
  Memento *createMemento()
  {
    return new Memento( state );
  }

private:
  int state;
  // ...
};

/*
 * CareTaker
 * is responsible for the memento's safe keeping
 */
class CareTaker
{
public:
  CareTaker( Originator* const o ) : originator( o ) {}
  
  ~CareTaker()
  {
    for ( unsigned int i = 0; i < history.size(); i++ )
    {
      delete history.at( i );
    }
    history.clear();
  }
  
  void save()
  {
    std::cout << "Save state." << std::endl;
    history.push_back( originator->createMemento() );
  }
  
  void undo()
  {
    if ( history.empty() )
    {
      std::cout << "Unable to undo state." << std::endl;
      return;
    }
    
    Memento *m = history.back();
    originator->setMemento( m );
    std::cout << "Undo state." << std::endl;
    
    history.pop_back();
    delete m;
  }
  // ...

private:
  Originator *originator;
  std::vector<Memento*> history;
  // ...
};


int main()
{
  Originator *originator = new Originator();
  CareTaker *caretaker = new CareTaker( originator );
  
  originator->setState( 1 );
  caretaker->save();
  
  originator->setState( 2 );
  caretaker->save();
  
  originator->setState( 3 );
  caretaker->undo();
  
  std::cout << "Actual state is " << originator->getState() << "." << std::endl;
  
  delete originator;
  delete caretaker;
  
  return 0;
}
```



#### 访问者模式（Visitor Pattern）

将作用于某种数据结构中的各元素的操作分离出来封装成独立的类，使其在不改变数据结构的前提下可以添加作用于这些元素的新的操作，为数据结构中的每个元素提供多种访问方式。它将对数据的操作与数据结构进行分离，是行为类模式中最复杂的一种模式。

访问者（Visitor）模式实现的关键是如何将作用于元素的操作分离出来封装成独立的类。

包含角色：

- Visitor：抽象访问者
- ConcreteVisitor：具体访问者
- Element：抽象元素
- ConcreteElement：具体元素

![访问者（Visitor）模式的结构图](https://gitee.com/yanzixian/picBed/raw/master/img202103/20210309154412.gif)



代码实现：

```c++
#include <iostream>

class Element;
class ConcreteElementA;
class ConcreteElementB;

/*
 * Visitor
 * declares a Visit operation for each class of ConcreteElement
 * in the object structure
 */
class Visitor
{
public:
  virtual ~Visitor() {}
  
  virtual void visitElementA( ConcreteElementA* const element ) = 0;
  virtual void visitElementB( ConcreteElementB* const element ) = 0;
  // ...
};

/*
 * Concrete Visitors
 * implement each operation declared by Visitor, which implement
 * a fragment of the algorithm defined for the corresponding class
 * of object in the structure
 */
class ConcreteVisitor1 : public Visitor
{
public:
  ~ConcreteVisitor1() {}
  
  void visitElementA( ConcreteElementA* const )
  {
    std::cout << "Concrete Visitor 1: Element A visited." << std::endl;
  }
  
  void visitElementB( ConcreteElementB* const )
  {
    std::cout << "Concrete Visitor 1: Element B visited." << std::endl;
  }
  // ...
};

class ConcreteVisitor2 : public Visitor
{
public:
  ~ConcreteVisitor2() {}
  
  void visitElementA( ConcreteElementA* const )
  {
    std::cout << "Concrete Visitor 2: Element A visited." << std::endl;
  }
  
  void visitElementB( ConcreteElementB* const )
  {
    std::cout << "Concrete Visitor 2: Element B visited." << std::endl;
  }
  // ...
};

/*
 * Element
 * defines an accept operation that takes a visitor as an argument
 */
class Element
{
public:
  virtual ~Element() {}
  
  virtual void accept( Visitor &visitor ) = 0;
  // ...
};

/*
 * Concrete Elements
 * implement an accept operation that takes a visitor as an argument
 */
class ConcreteElementA : public Element
{
public:
  ~ConcreteElementA() {}
  
  void accept( Visitor &visitor )
  {
    visitor.visitElementA( this );
  }
  // ...
};

class ConcreteElementB : public Element
{
public:
  ~ConcreteElementB() {}
  
  void accept( Visitor &visitor )
  {
    visitor.visitElementB( this );
  }
  // ...
};


int main()
{
  ConcreteElementA elementA;
  ConcreteElementB elementB;
  
  ConcreteVisitor1 visitor1;
  ConcreteVisitor2 visitor2;
  
  elementA.accept(visitor1);
  elementA.accept(visitor2);
  
  elementB.accept(visitor1);
  elementB.accept(visitor2);
  
  return 0;
}

```



#### 解释器模式（Interpreter Pattern）

给分析对象定义一个语言，并定义该语言的文法表示，再设计一个解析器来解释语言中的句子。也就是说，用编译语言的方式来分析应用中的实例。这种模式实现了文法表达式处理的接口，该接口解释一个特定的上下文。

包含以下角色：

- Abstract Expression：抽象表达式
- Terminal Expression：终结符表达式
- Nonterminal Expression：非终结符表达式
- Context：环境

![解释器模式的结构图](https://gitee.com/yanzixian/picBed/raw/master/img202103/20210309160512.gif)



代码实现：

```c++
#include <iostream>
#include <map>

/*
 * Context
 * contains information that's global to the interpreter
 */
class Context
{
public:
  void set( const std::string& var, const bool value)
  {
    vars.insert( std::pair<std::string, bool>( var, value ) );
  }
  
  bool get( const std::string& exp )
  {
    return vars[ exp ];
  }
  // ...

private:
  std::map<std::string, bool> vars;
  // ...
};

/*
 * Abstract Expression
 * declares an abstract Interpret operation that is common to all nodes
 * in the abstract syntax tree
 */
class AbstractExpression
{
public:
  virtual ~AbstractExpression() {}
  
  virtual bool interpret( Context* const )
  {
    return false;
  }
  // ...
};

/*
 * Terminal Expression
 * implements an Interpret operation associated with terminal symbols
 * in the grammar (an instance is required for every terminal symbol
 * in a sentence)
 */
class TerminalExpression : public AbstractExpression
{
public:
  TerminalExpression( const std::string& val ) : value( val ) {}
  
  ~TerminalExpression() {}
  
  bool interpret( Context* const context )
  {
    return context->get( value );
  }
  // ...
  
private:
  std::string value;
  // ...
};

/*
 * Nonterminal Expression
 * implements an Interpret operation for nonterminal symbols
 * in the grammar (one such class is required for every rule in the grammar)
 */
class NonterminalExpression : public AbstractExpression
{
public:
  NonterminalExpression( AbstractExpression *left, AbstractExpression *right ) : 
    lop( left ), rop( right ) {}
  
  ~NonterminalExpression()
  {
    delete lop;
    delete rop;
  }
  
  bool interpret( Context *const context )
  {
    return lop->interpret( context ) && rop->interpret( context );
  }
  // ...
  
private:
  AbstractExpression *lop;
  AbstractExpression *rop;
  // ...
};


int main()
{
  // An example of very simple expression tree
  // that corresponds to expression (A AND B)
  AbstractExpression *A = new TerminalExpression("A");
  AbstractExpression *B = new TerminalExpression("B");
  AbstractExpression *exp = new NonterminalExpression( A, B );
  
  Context context;
  context.set( "A", true );
  context.set( "B", false );
  
  std::cout << context.get( "A" ) << " AND " << context.get( "B" );
  std::cout << " = " << exp->interpret( &context ) << std::endl;
  
  delete exp;
  return 0;
}

```

