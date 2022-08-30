# Clean C++
# C++17代码整洁之道

---

# 第一章

## 整洁的代码
```
整洁的代码是容易被任何团队的成员理解和维护
整洁的代码是高效工作的基础
整洁的代码是软件持续发展的基础
整洁的代码使开发者快乐
整洁的代码能够节省金钱
```

## 为何选择C++
```
在一些领域，编译型语言占据主导地位
高性能
如果其他语言可以让你更轻松的工作，你就可以用其他语言
C++11开启的新时代
```

## 风格使用
```c++
//作者使用类似Java和K&R风格的自身风格
class Clazz {
public:
    Clazz();
    virtual ~Clazz();
    void doSomething();

private:
    int _attribute;

    void function();
}
```
---

# 第二章 构建安全体系

```
Plain Old Unit Test
普通的单元测试

好处：减少故障bug错误error缺陷flaw

测试金字塔
Manual (Session Based) Exploratory Testring
Automated GUI Tests
Automated Integration and System Tests, cov ~10% resp:QA&business
Automated Component Tests, cov ~>=50% resp:developers
Automated Unit Tests, cov ~100% resp:developers

自动化UI测试很难编写，通常比较脆弱，相对较慢。
因此这些测试通常是手动进行的，适合于客户审核(验收测试)和QA定期的探索性测试
日常开发过程中使用它们太耗费时间，而且代价昂贵
此外，大型系统测试和UI驱动测试完全不适合检查整个系统中所有可能的执行情况。
软件系统中有太多：处理各种可能情况的代码，异常和错误处理，交叉相关问题（安全性，事务处理，日志记录）
但是通常无法通过普通用户接口去触发。

非常重要的一点，如果系统级别测试失败了，可能难以找到错误的确切原因。
```

```
单元测试

没有测试的重构不能称为重构，它仅仅是到处移动垃圾代码

优点：
代价低
快速即时反馈
重构的信心
防止陷入耗时的调试
是可以被执行的产品文档
快速检查更改代码后的异常
促进实现整洁良好的接口
促进开发，还是节省大型系统的调试时间，重构时间等

c++ 单元测试框架 CppUnit Boost.Test CUTE Google Test
一般几个单元测试框架的集合称为 xUnit
```

```
QA

开发组的目标应该是QA没有发现任何缺陷
将一个已知的有缺陷的软件移交给QA是非常不专业的行为
专业的开发人员永远不会把保证系统质量的责任推给其他部门
应该与QA建立富有成效的合作伙伴关系，紧密合作，相互补充，QA是第二道防线

通过修复QA发现的缺陷来立刻补救这些质量问题，并通过编写自动化单元测试在未来捕获这些异常
```

```
单元测试的基本原则

产品代码和测试代码级别相同，不要区别对待

单元测试的命名
如果失败了需要知道
1 测试单元的名称，谁的单元测试失败了
2 单元测试测试了什么，单元测试的环境是什么样的 （测试场景）
3 预期的单元测试结果是什么 单元测试失败的结果又是什么
因此，命名需要具备直观性和描述性

<Unit_under_Test>Test
例如：被测试系统（SUT）是Money单位，
与该测试单元对应的单元测试夹具，
以及所有的单元测试用例都应该命名为MoneyTest

名称应当包括
1 单元测试的前置条件 也就是执行单元测试之前的SUT的状态
2 被单元测试测试的部分，通常是被测试的过程、函数、或方法（API）的名称
3 单元测试预期的测试结果

<PreconditionAndStateOfUnitUnderTest>_<TestedPartOfAPI>_<ExpectedBehavior>
示例：
void CustomerCacheTest::cacheIsEmpty_addElement_sizeIsOne();
void CustomerCacheTest::cacheContainsOneElement_removeElement_sizeIsZero();
void ComplexNumberCalculatorTest::givenTwoComplexNumbers_add_Works();
void MoneyTest::givenTwoMoneyObjectsWithDifferentBalance_theInequalityComparison_Works();
void MoneyTest::createMoneyObjectWithParameter_getBalanceAsString_returnsCorrectString();
void InvoiceTest::invoiceIsReadyForAccounting_getInvoiceData_returnsToday();

另一个构建直观容易理解的单元测试名称的方法，就是在单元测试中显示特定需求
示例：
void UserAccountTest::creatingNewAccountWithExistingEmailAddressThrowsException();
void ChessEngineTest::aPawnCanNotMoveBackwards();
void ChessEngineTest::aCastlingIsNotAllowedIfInvolvedKingHasBeenMovedBefore();
void ChessEngineTest::aCastlingIsNotAllowedIfInvolvedRookHasBeenMovedBefore();
void HeaterControlTest::ifWaterTemperatureIsGreaterThan92DegTurnHeaterOff();
void BookInventoryTest::aBookThatIsInTheInventoryCanBeBorrowedByAuthorizedPeople();
void BookInventoryTest::aBookThatIsAlreadyBorrowedCanNotBeBorrowedTwice();


独立性
每个单元测试和其他单元测试都必须是独立的
如果单元测试以特定的顺序执行，将是致命的

主要的问题可能由全局状态引起，例如:在单元测试中使用单例或静态成员
单例不仅增加了单元测试之间的耦合度，还经常会保持一个全局的状态
单元测试之间因为全局状态而变得互相依赖

尤其是在遗留系统中，经常杂乱无章地使用单例模式，这就引出一个问题:
如何才能摆脱这些杂乱无章的对单例的依赖关系，让我们的代码更易于测试呢？
第六章 依赖注入 会讨论这个问题


一个测试一个断言
这是个有争议的话题，作者建议 限制一个单元测试只使用一个断言

void MoneyTest::givenTwoMoneyObjectsWithDifferentBalance_theInqualityComparison_Works() {
    const Money m1(-4000.0);
    const Money m2(2000.0);
    ASSERT_TRUE(m1 != m2);
}

有人说：还可以检查其他比较运算符，例如 Money::operator==()
在该单元测试中是否正常工作，只需要添加更多断言就可以轻松实现

void MoneyTest::givenTwoMoneyObjectsWithDifferentBalance_testAllComparisonOperators() {
    const Money m1(-4000.0);
    const Money m2(2000.0);
    ASSERT_TRUE(m1 != m2);
    ASSERT_FALSE(m1 == m2);
    ASSERT_TRUE(m1 < m2);
    ASSERT_FALSE(m1 > m2);
}

这样写单元测试存在的问题：
1 如果测试失败，很难快速找到错误原因，前面的断言掩盖了后面的错误。断言造成中断
2 命名不容易提现单元测试的意义
```

```
单元测试环境的独立初始化

在运行所有单元测试时，每个单元测试都必须是应用程序的一个可独立运行的实例，
每个单元测试都必须完全自行设置和初始化其所需的环境，这同样适用于执行单元测试后的清理工作


不对 getters 和 setters 做单元测试
1 非常简单，不需要编写
2 其他单元测试中，已经隐式地测试了它们

不对第三方代码做单元测试
1 不必验证库或者框架是否按预期的那样工作
2 预测第三方代码都有自己的单元测试
3 不使用那些 没有自己的单元测试 和 质量可疑 的库或者框架，这是一种明智的选择

不对外部系统做单元测试
道理和第三方代码一样，无视这些东西


如何处理数据库的访问？
能不使用数据库进行单元测试，就不使用
1 很难保证每个单元测试所需的前提条件
2 数据库存储速度缓慢
建议：使用模拟数据库 （测试替身）
只在内存中执行所有单元测试

不用担心数据库的测试，系统集成和系统测试级别，会测试数据库
```

```c++
// 不要混淆测试代码和产品代码

// 开发人员产生了一个想法，用测试代码来装备生产代码

#include <memory>
#include "DataAccessObject.h"
#include "CustomerDAO.h"
#include "FakeDAOForTest.h"

using DataAccessObjectPtr = std::unique_ptr<DataAccessObject>;
class Customer {
public:
    Customer() {}
    explicit Customer(bool testMode) : inTestMode(testMode) {}

    void save() {
        DataAccessObjectPtr dataAccessObject = getDataAccessObject();
        // ...use dataAccessObject to save this customer...
    };

    // ...
private:
    DataAccessObjectPtr getDataAccessObject() const {
        if (inTestMode) {
            return std::make_unique<FakeDAOForTest>();
        } else {
            return std::make_unique<CustomerDAO>();
        }
    }
    // ...more operations here...

    bool inTestMode { false };
    // ...more attributes here...
};

// FakeDAOForTest 就是测试替身，目的是替换真正的DAO

// 这段代码虽然可行，但是又几个缺点
// 1 生产代码会混淆测试代码 增加产品复杂度 降低代码可读性
// 2 Customer依赖于 CustomerDAO 和 FakeDAOForTest，
// 我们寄托希望于测试代码在生产中永远不会被调用，但它确实被编译链接并部署在生产中

// 更优雅的方法

// 第一种，在Customer::save() 中注入特定的DAO作为一个参考参数
class DataAccessObject;

class Customer {
public:
    void save(DataAccessObject& dataAccessObject) {
        // ...use dataAccessObject to save this customer...
    }
    // ...
}

// 第二种，在构造Customer类型的实例期间完成，将DAO的一个引用作为类的成员属性
// 此外 必须通过编译器禁止自动生成默认构造函数，因为我们不希望Customer的任何用户可以创建
// 一个未正确初始化的实例

class DataAccessObject;

class Customer {
public:
    Customer() = delete;
    Customer(DataAccessObject& dataAccessObject) : dataAccessObject(dataAccessObject) {}
    void save() {
        // ...use member dataAccessObject to save this customer...
    }
    // ...
private:
    DataAccessObject& dataAccessObject;
    // ...
}

// 第三种替代方案，是特定DAO可以由Customer类已知的一个工厂 (第九章设计模式Factory模式部分) 来创建
// 我们可以从外部配置Factory以创建所需的DAO
```

```c++
//c++ 11 delete 函数

// 阻止创建默认构造函数
class Clazz{
public:
    Clazz() = delete;
}

// 删除new运算符，防止在堆上动态分配一个类
class Clazz{
public:
    void* operator new(std::size_t) = delete;
}
```

```
测试替身

因为：被测试单元 不依赖 其他单元 或 外部系统
所以：
要测试的单元 与 其他模块 或 外部系统 的 依赖性 应该被 测试替身 Test Doubles 替换
测试替身被称为 伪对象 Fake Objects 或 假模型 Mock-Ups

为了以一种优雅的方式使用测试替身，尽量达到被测试单元之间的松耦合
我们必须在代码种引入一个可变点，可以用一个测试替身替换真实模块
通常可以使用一个接口达到这个目的，该接口在c++中是一个仅仅包含纯虚成员函数的抽象类

// 一个货币转换的抽象接口
class CurrencyConverter {
public:
    virtual ~CurrencyConverter() {}
    virtual long double getConversionFactor() const = 0;
}

// 访问实际的货币转换服务的类
class RealtimeCurrencyConversionService : public CurrencyConverter {
public:
    virtual long double getConversionFactor() const override;
    // ... more
}

// 测试替身
class CurrencyConversionServiceMock : public CurrencyConverter {
public:
    virtual long double getConversionFactor() const override {
        return conversionFactor;
    }

    void setConversionFactor(const long double value) {
        conversionFactor = value;
    }
private:
    long double conversionFactor{0.5};
}

// 使用这个服务的类的头文件
#include <memory>

class CurrencyConverter;

class UserOfConversionService {
public:
    UserOfConversionService() = delete;
    UserOfConversionservice(const std::shared_ptr<CurrencyConverter>& conversionService);
    void doSomething();
    // More of the public class interface follows here...

private:
    std::shared_ptr<CurrencyConverter> conversionService;
    // ...internal implementation...
};

UserOfConversionService::UserOfConversionService(const std::shared_ptr<CurrencyConverter>& conversionService) :
    conversionService(conversionService) {}

void UserOfConversionService::doSomething(){
    long double conversionFactor = conversionService->getConversionFactor();
    // ...
}

//  这种技术被称为 依赖注入模式

// 通过UserOfConversionService的构造哈桑农户传递它使用的CurrencyConverter对象
str::shared_ptr<CurrencyConverter> serviceToUser = std::make_shared<name_of_the_desired_class>();
UserOfConversionService user(serviceToUser);
// The instance of UserOfConversionService is ready for use...
user.doSomething();
```
---

# 第三章 原则

```
把更多精力放在学习基本思想，而不是新技术上

KISS Keep it simple, stupid 保持简单和直接
DTSTTCPW Do the simplest thing that could possibly work 简单到只要能正常工作

避免做一些没有必要的复杂性工作
程序员偏向以精心设计的方式编写代码，这样导致的结果是他们往往将问题复杂化
必须记住任何软件系统都有内在的复杂性，复杂问题通常需要复杂的代码
不要因为你会用，就把一些花哨技巧或一些很酷炫和奇特的东西

任何软件系统都有内在的复杂性，不可避免
为这种内在的复杂性添加不必要的复杂性是致命的
不要过分追求简单
如果switch-case中有十个条件是必须的，那它就应该有十个条件
注：可以考虑使用 表驱动法 来解决N个分支条件的问题

如果对灵活性和可拓展性有很高的质量要求，则必须增加软件的复杂性以满足这些需求
例如：使用 策略模式
只添加那些使事情整体变得更简单的复杂性的东西

对程序员来说，关注简单性可能是最困难的事情之一，并且这是一个终生的学习经验。
```

```
总是在你真正需要的时候再再现它们，而不是在你只是预见到你需要它们的时候实现它们。

YAGNI You Ain't Gonna Need It! 不要写目前用不上，但将来也许需要的代码
```

```
复制和粘贴是一个设计错误

DRY Don't repeat yourself! 尽可能地避免复制
OAOO Once And Only Once

在一个系统内部，任何一个知识点都必须有一个单一的、明确的、权威的陈述
```

```
信息隐藏原则

一段代码调用了另外一段代码，调用者不应该知道被调用者的内部实现
否则，调用者就有可能通过修改被调用者的内部实现而完成某个功能，而不是强制性地要求调用者修改自己的代码

信息隐藏是把系统分解为模块的基本原则，降低耦合

优点：
限制了模块变更的范围
如果需要修复缺陷，对其他模块的影响最小
显著提高了模型的可复用性
模块具有更好的可测试性

例子：隐藏信息较差的封装类
class AutomaticDoor {
    public :
        enum class State{
            closed = 1,
            opening,
            open,
            closing
        };
    private:
        State state;
    public:
        State getState() const;
        // ... more member functions here...
}
这不是信息隐藏 因为类内部的实现部分暴露给了外部环境，尽管该类看起来封装得很好
注意getState返回值的类型 客户端用到的枚举类State用到了这个类
#include "AutomaticDoor.h"

int main() {
    AutomaticDoor automaticDoor;
    AutomaticDoor::State doorsState = automaticDoor.getState();
    if (doorsState == AutomaticDoor::State::closed) {
        // do something
        return 0;
    }
}
```

```c++
// c++ enum

// 旧的 名称冲突
const std::string bear;
// ...and elsewhere in the same namespace...
enum Animal {dog, deer, cat, bird, bear};//error: 'bear' redeclared as different kind of symbol

// 旧的 隐式转换为int
enum Animal {dog, deer, cat, bird, bear};
Animal animal = dog;
int aNumber = animal;// Implicit conversion: works

// 新枚举
const std::string bear;
// ...and elsewhere in the same namespace...
enum class Animal {dog, deer, cat, bird, bear };// No conflict with the string named 'bear'
Animal animal = Animal::dog;
int aNumber = animal; // Compiler error!

// 强烈建议使用  枚举类 ， 旧的枚举还兼容
```

```
信息隐藏原则

良好的自动门转向设计类

class AutomaticDoor {
    public:
        bool isColosed() const;
        bool isOpening() const;
        bool isOpen() const;
        bool isClosing() const;
        // ... more operations here...
    private:
        enum class State {
            closed = 1,
            opening,
            open,
            closing
        };

        State state;
        // ... more attributes here...
}

#include "AutomaticDoor.h"

int main()
{
    AutomaticDoor automaticDoor;
    if (automaticDoor.isClosed()) {
        // do something...
    }
}
```

```
高内聚原则

任何软件实体 如 模块 组件 单元 类 函数 等
应该具有很高的 强的 内聚性

1 模块随意划分 业务领域不同的功能放在同一个模块内
应该分离不同的功能
2 散弹枪反模式 单个逻辑思想高度碎片化 分布在许多模块中
应该把与功能A相关的所有代码都拿出来，把相同逻辑的代码放到一个高内聚模块内

SRP 面向对象设计的单一职责原则
```

```c++
// 松耦合原则

// 紧耦合
class Lamp {
    public:
        void on() }{
            // ...
        }

        void off() {
            // ...
        }
};

class Switch {
    private:
        Lamp& lamp;
        bool state {false};
    
    public:
        Switch(Lamp& lamp) : lamp(lamp) {}

        void toggle() {
            if (state) {
                state = false;
                lamp.off();
            } else {
                state = true;
                lamp.on();
            }
        }
};


//  松耦合

class Switchable {
    public:
        virtual void on() = 0;
        virtual void off() = 0;
};

class Switch {
    private:
        Switchable& switchable;
        bool state {false};
    public:
        Switch(Switchable& switchable) : switchable(switchable) {}

        void toggle() {
            if (state) 
            {
                state = false;
                switchable.off();
            }
            else
            {
                state = true;
                switchable.on();
            }
        }
};

class Lamp : public Switchable {
    public:
        void on() override{

        }

        void off() override{

        }
};
```

```
小心优化原则

不成熟的优化是编程中的所有问题的根源 Knuth

有一个模糊的想法就进行程序的优化
经常摆弄个别的指示，尝试优化小的，局部的循环结构以挤出最后一点性能
其实很浪费时间

建议：只要没有明确的性能要求 就避免优化
```

```
最少惊讶原则

POLA
PLA
POLS

在 用户界面设计 和 人因工程学设计 中知名
指：不应该让用户对用户界面的意外响应而感到惊讶，也不应该对
出现或消失的控件、混乱的错误消息、公认的案件序列的异常响应或其他意外行为而感到困惑

这个原则也很好地应用到软件开发中的API设计中
调用函数不应该让调用者感到异常行为或一些隐藏的副作用
函数应该完全按照函数名称所暗示的意义执行
例如：getter函数不应该修改该对象的内部状态
```

```
童子军原则

在离开露营地的时候，应该让露营地比你来之前还要干净

一旦发现不好的东西，立即清理
1 重命名那些命名不佳的类 变量 函数 方法
2 将大型函数 分解为 更小的 函数
3 让需要注释的代码不言自明，以比避免注释
4 清理复杂而令人费解的if-else组合
5 删除一小部分重复的代码

这些都是代码重构，没有单元测试，你根本无法确定你是否破坏了某些东西

特殊文化： 代码所有权集体化 Collective Code Ownership
每个成员在任何时候都可以对任何代码进行拓展或更改
```
---

# 第四章 c++代码整洁的基本规范

```
使用c++11以上版本
```

```
良好的命名

一段openOffice老代码的问题：
注释掉的代码 德语注释 魔法数字 主要问题是命名不佳

源代码文件 命名空间 类 模板 函数 参数 变量 常量
都应该具有有意义且富有表现力的名字

起名字并不容易，而且浪费时间，但这是值得的
如果给变量，函数，或类相处合适的名字很难或几乎不可能，那可能存在某些问题
代码或者设计上的问题


名称应该自解释

非常不好
unsigned int num;
bool flag;
std::vector<Customer> list;
Product data;

好的
unsigned int numberOfArticles;
bool isChanged;
std::vector<Customer> customers;
Product orderedProduct;

不好
unsigned int totalNumberOfCustomerEntriesWithMangledAddressInformation;

利用类提供上下文信息
class CustomerRepository {
    private:
        unsigned int numberOfMangledEntries;
}


使用领域中的名称

领域驱动设计 Domain-Driven Design DDD 
接近于复杂面向对象软件开发中的一种方法，主要集中在核心领域和领域逻辑上。
换句话说，DDD试图通过将业务领域的事务和概念映射到代码中，使软件成为一个真实系统的模型

例如：汽车租赁
class ReserveCaruseCaseController {
    public:
        Customer identifyCustomer(const UniqueIdentifier& customerId);
        CarList getListOfAvailableCars(const Station& atStation, const RentalPeriod& desiredRentalPeriod) const;
        ConfirmationOfReservation reserveCar(const UniqueIdentifier& carId, const RentalPeriod& rentalPeriod) const;
    private:
        Customer& inquiringCustomer;
}


选择适当抽象层次的名称

为了控制当今软件系统的复杂性，这些系统通常是分层的。
软件系统的分层意味着将整个问题分解为较小的部分作为子任务
进行这种分解有不同的方法和标准
领域驱动设计 DDD
面向对象分析和设计 OOAD
是其中的两种方法

例子
创建单据 Billing
    计算折扣 DiscountCalculator
    创建条目 LineItemFactory
        详细方法名称 calculateReducedValueAddedTax()


避免冗余的名称

class Movie {
private:
    std::string movieTitle;

    // 或者
    std::string strignTitle;
}


避免晦涩难懂的缩写

std::size_t idx; //差
std::size_t index; //好
std::size_t customerIndex; // 首选，尤其有多个index的情况

Car ctw; // 差
Car carToWash; //好

Polygon ply1; // 差
Polygon firstPolygon; // 好

unsigned int nBottles; // 差
unsigned int bottleAmount; // 好点
unsigned int bottlesPerHour; // 用变量保存工作值，而不是使用纯数字，好

const double GOR = 9.80665; // 差
const double gravityOfEarth = 9.80665; // 更具表现力，但有误导性。常数不是引力，而是物理学中的力。

const double gravitationalAccelerationOnEarth = 9.80665; // 好
constexpr Acceleration gravitationalAccelerationOnEarth = 9.80665_ms2; // 很好


避免匈牙利命名和命名前缀

不要使用任何把类型信息加入变量名称中的命名法
匈牙利命名法在像C语言这样的弱类型语言中可能有用，在开发人员使用简单编辑器进行编程时，可能有用
但对于具有 IntelliSense 智能提示 这类功能的IDE中没用任何用途，反而带来危害


避免相同的名称用于不同的目的
```

```
注释

代码应该能讲述一个故事并且能自解释，必须尽可能避免注释

不要为易懂的代码写注释

不要通过注释禁用代码
除了快速进行测试外，不要通过注释禁用代码，同时还要用一个版本控制系统

不要写块注释
版权头注释，不需要，应该写在单独文件中，比如license.txt copyright.txt

不要用注释代替版本控制

特殊情况需要注释
比如复杂公式，为何自己违反DRY原则

建议：
确保注释为代码增加了价值
应该解释为什么这样，而不是怎样去做
尽量做到注释尽可能短且富有表现力 因为需要维护注释

Doxygen风格注释 注释生成文档 略
```

```
函数

圈复杂度 如果函数不包含if或switch，并且没有for或while，则只有一条
通过函数的路径，圈复杂度为1。如果函数包含一条表示单个决策点的if语句
则有两条通过函数的路径，圈复杂度为2

代码异味code smell
函数太长 圈复杂度较高 混合了不同的信息 参数太多
包含死亡代码dead code 函数名称误导 变量名难以理解

只做一件事情
函数应该只做一件事情 应该做好这件事情 应该仅做好这一件事情
函数做了太多事情的标志：
1 函数体量比较大，一个函数包含了很多代码
2 试图给函数起名时，不可避免使用 和 或 这样的连词
3 函数体用空行垂直分隔成代表后续步骤的几个片段
4 圈复杂度高
5 函数的入参比较多，特别是一个或多个bool类型

让函数尽可能小
理想情况下45行，最多12到15行
例外：包含单个switch语句的函数 译者认为如果太多分支，最好还是用 表驱动法

函数命名
void CustomerAccount::grantDiscount(DiscountValue discount);
void Subject::attachObserver(const Observer& observer);
void Subject::notifyAllObservers() const;
int Bottling::getTotalAmountOfFilledBOttles() const;
bool AutomaticDoor::isOpen() const;
bool CardReader::isEnabled() const;
bool DoubleLinkedList::hasMoreElements() const;

使用容易理解的名称
std::string head = html.substr(startOfHeader, lengthOfHeader);
很好，但是不如
std::string ReportRenderer::extractHtmlHeader(const std::string& html) {
    return html.substr(startOfHeader, lengthOfHeader);
}
std::string head = extractHtmlHeader(html);
函数的名称应该表达其意图和目的，而不是解释它的工作原理。
如何完成工作，应该看函数体内的代码，不要在函数名称中解释函数如何工作。
相反，要从业务的角度表达函数的目的。

函数的参数和返回值
参数：Clean Code，使用Java 这里谈论的参数是类的方法。
理想参数是零个 niladic 接下来是一个 monadic 然后是两个 dyadic 尽可能避免三个 triadic
超过三个则需要非常特别的证明 polyadic 或者无论如何不要使用
为什么参数多了不好：
可能导致依赖关系，标准内置类型除外。
必须在函数内部处理每个参数，三个参数就可以让函数相对复杂。
Java中，所有内容都必须嵌入到类中，这有时会导致很多样板式代码 boiler-plate code
C++中，这不是必须的，可以在C++中实现一个独立的函数，不属于任何类，这非常好。
对C++来说，一个参数比较类型，类的成员函数(方法)没有参数。
通常这些函数用于操作对象内部状态，或者被用于从对象中查询某些内容。

避免使用标志参数 bool 或者 enum
这是一个低内聚的案例，违反了单一职责原则
Invoice Billing::createInvoice(const BookingItems& items, const bool withDetails) {
    if (withDetails) {
        //...
    } else {
        //...
    }
}
一种方法
Invoice Billing::createSimpleInvoice(const BookingItems& items) {
    //...
}
Invoice Billing::createInvoiceWithDetails(const BookingItems& items) {
    //...
}
另一种方法
class Billing {
public:
    virtual Invoice createInvoice(const BookingItems& items) = 0;
};
class SimpleBilling : public Billing {
public:
    virtual Invoice createInvoice(const BookingItems& items) override;
};
class DetailsBilling : public Billing {
public:
    virtual Invoice createInvoice(const BookingItems& items) override;
private:
    SimpleBilling simpleBilling;//这是必须的，避免了重复代码，能够执行简单的票据创建工作
}

避免使用输出参数
输出参数的一个常见好处是，可以一次传回多个值：
bool ScriptInterpreter::executeCommand(const std::string& name,
                                const std::vector<std::string>& arguments,
                                Result& result);
ScriptInterperter interperter;
Result result;
if (interperter.executeCommand(commandName, argumentList, result)) {
    // Continue normally
} else {
    // Handle failed execution of command...
}
建议：不惜一切代价，避免使用输出参数
很不直观，可能导致混淆。被函数改变
使表达式的简单组合变得复杂，代码很快就会混乱
不可变对象无法被作为输出参数传递
如果某个方法需要返回值，用一个返回值。如果必须返回多个值，请重新设计。
返回一个包含这些值的对象的单个实例，或者可以用，std::tuple或std::pair
（
tuple:
using Customer = std::tuple<std::string, str::string, std::string, Money, unsigned int>;
Customer aCustomer = std::make_tuple("Stephan", "Roth", "Bad Schwartau", outstandingBalance, timeForPayentInDays);
auto aCustomer = std::make_tuple("Stephan", "Roth", "Bad Schwartau", outstandingBalance, timeForPayentInDays);
auto city = std::get<2>(aCustomer);//很不直观
建议仅在特殊情况下使用tuple
）
如果返回值必须区分成功和失败，可以使用特例模式，第九章

不要传递或者返回0 NULL nullptr
代码调用者必须检查，且必须处理空指针
描述软件异常，错误，和特殊情况的预期行为要困难得多。
最糟糕的结果：如果忘记了任何空检查，可能会导致严重的运行时错误，
解引用空指针将会导致段错误和应用程序崩溃。
c++中还需要考虑对象的所有权。
对调用者来说，使用之后如何处理指针所指向的资源时模糊的。
谁是该资源的拥有者，是否需要删除该对象。如何删除。
根据信息隐藏原则，以上这些都与调用者无关，但实际上我们已经将
资源的责任强加给了调用者。如果调用者没有没有正确处理，可能导致严重错误，
例如内存泄漏，二次删除，未定义行为，有时还会引发安全漏洞。
策略：
首选在栈上构造对象而不是堆上
#include "Customer.h"
//...
Customer customer;
离开作用域，实例会自动销毁，这通常发生在从函数或方法返回时。
但是
如果必须将在函数或方法中创建的对象返回给调用者，该怎么办？
旧的c++
Customer* createDefaultCustomer() {
    Customer* customer = new Customer();
    //...
    return customer;
}
这种方式可以避免昂贵的拷贝构造，但上面问题都还在。
从c++11开始，可以将大型对象直接作为返回值返回，不必担心
昂贵的拷贝构造。
Customer cerateDefaultCustomer() {
    Customer customer;
    //...
    return customer;
}
这种情况我们不再需要担心资源管理，因为 Move语义
从c++11开始支持使用。
Move语义允许资源从一个对象move到另一个对象，而不是复制他们。
并且可以非常快速的执行。
c++11标准中，所有标准库容器类都已扩展为支持move语义
#include <vector>
#include <string>

using StringVector = std::vector<std::string>;
const StringVector::size_type AMOUNT_OF_STRINGS = 10000;

StringVector createLargeVectorOfStrings() {
    StringVector theVector(AMOUNT_OF_STRINGS, "Test");
    return theVector; // Guaranteed no copy construction here!
}
Move语义是拜托大量常规指针的一种很好的方法，但我们能做的还有很多

在函数参数列表中，用const引用代替指针
void function(Type* argument);
替换为
void function(Type& argument);
优点：不需要检查引用是否为nullptr，因为引用永远不会是NULL。注：有可能造成空引用
不需要解操作符* 有利于写出清晰的代码 使用const引用

不可避免地处理指向资源的指针，使用智能指针
在堆上分配资源，使用RAII惯用法
如果API返回原始指针，那么我们就有了“依赖问题” 
自定义智能指针包装常规指针。

正确使用const
尽可能使用const，并始终为变量或对象选择适当的声明以区分可变和不可变
一个用法是常量定义
cosnt long double PI = 3.141592653589794;
另一个用法是防止参数被改变
unsigned int determineWeightOfCar(Car const* car);//指针car指向Car类型的 常量对象 Car对象不能被修改
void lacquerCar(Car* const car);//指针car是一个Car类型的指针常量，可以改Car对象，但不能修改指针
unsigned int determineWeightOfCar(Car const* const car);//指针和指针指向的对象都无法修改
void printMessage(const std::string& message);//message通过const引用传递给函数，不允许更改被引用的字符串变量
void printMessage(std::string const& message);//和上一行一样
提示：const修饰它左边的内容 例外情况：如果左侧没有内容 例如在声明的开头使用const，则const修饰右侧的内容 函数很常见啊
另一个用途是将类的非静态成员函数声明为cosnt
#include <string>
class Car {
public:
    std::string getRegistrationCode() const;
    void setRegistrationCode(const std::string& registrationCode);
    //...
private:
    std::string _registrationCode;
}
getter函数是cosnt，setter函数入参是const
std::string Car::getRegistrationCode() {
    std::string toBeReturned = registrationCode;
    registrationCode = "foo"; // 报错
    return toBeReturned;
}
```

```
C++中的C风格代码

建议：用C++代码替代旧的且容易出错的C代码结构
现代C++中几乎已经完全没用C风格的编程习惯了

使用C++的string和stream代替C风格的char*
C++的string是C++标准库的一部分
std::string std::wstring 都是模板 std::basic_string<T>上的类型定义
typedef basic_string<char> string;
typedef basic_string<wchar_t> wstring;
要创建string，必须实例化对象，例如
std::string name("Stephan");
与此相比，C风格字符串只是一个字符数组，以0结束符结束。0结束符是一个特殊字符\0，ASCII码为0
char name[] = "Stephan";
0结束符会自动添加到字符串末尾，字符串长度为8个字符。
要用char* 传递
char* pointerToName = name;
void function(char* pointerToCharacterArray) {
    //...
}
好处：
C++ string对象自己管理字符串的内存，可以轻松复制创建销毁
可变的，可以轻松添加字符串或单个字符，连接字符串，替换字符串的某些部分
提供了方便的迭代器接口，允许遍历，使用<algorithm>
C++ string 与 C++ I/O stream 例如 ostream stringstream fstream能完美配合
从C++11开始，标准库广泛使用move语义。许多算法和容器针对移动优化。
使用C风格字符串的情况：
字符串常量 不可变字符串
const char* const PUBLISHER = "Apress Media LLC";
与C风格的API兼容

避免使用printf() sprintf() gets()等
不要觉得printf()比std::cout快几微秒就用，与I/O的耗时相比完全没用 小心过早优化
printf()类型不安全，容易出错 需要一些了无类型参数，填充格式表示符的C风格字符串，很不安全
可能导致微秒的错误，未定义行为，安全漏洞
提示：
数字类型的所有变量都可以用安全方便的std::to_string()和std::to_wstring()转换为C++ string对象
int value {42};
std::string valueAsString = std::to_string(value);
适用于所有整数或浮点类型，如int long longlong unsignedint float double
缺点是某些情况下结果不准确
double d { 1e-9 };
std::cout << std::to_string(d) << "\n"; // Caution! Output 0.000000
无法配置to_string()字符串的格式化输出，例如小数位数的控制。这意味着该函数，适用范围很小。
需要更精确和自定义的转换， 必须自己提供。可以利用字符串流 包含<sstream> 和 <iomanip> 中定义的
I/O控制符 Manipulator的配置功能，而不是使用sprintf()
#include <iomanip>
#include <sstream>
#include <string>

std::string convertDoubleToString(const long double valueToConvert, const int precision) {
    std::stringstream stream {};
    stream << std::fixed << std::setprecision(precision) << valueToConvert;
    return stream.str();
}
与printf不同，C++ I/O stream允许通过提供自定义插入操作符operator <<轻松将复杂对象流化
#ifndef INVOICE_H
#define INVOICE_H

#include <chrono>
#include <memory>
#include <ostream>
#include <string>
#include <vector>

#include "Customer.h"
#include "InvoiceLineItem.h"
#include "Money.h"
#include "UniqueIdentifier.h"

using InvoiceLineItemPtr = std::shared_ptr<InvoiceLineItem>;
using InvoiceLineItems = std::vector<InvoiceLineItemPtr>;

using InvoiceRecipient = Customer;
using InvoiceRecipientPtr = std::shared_ptr<InvoiceRecipient>;

using DateTime = std::chrono::system_clock::time_point;

class Invoice {
public:
    explicit Invoice(const UniqueIdentifier& invoiceNumber);
    Invoice() = delete;
    void setRecipient(const InvoiceRecipientPtr& recipient);
    void setDateTimeOfInvoicing(const DateTime& dateTimeOfInvoicing);
    Money getSum() const;
    Money getSumWithoutTax() const;
    void addLineItem(const InvoiceLineItemPtr& lineItem);
    // ...possibly more member functions here...
private:
    friend std::ostream& operator<<(std::ostream& outstream, const Invoice& invoice);
    std::string getDateTimeOfINvoicingAsString() const;

    UniqueIdentifier invoiceNumber;
    DateTime dateTimeOfInvoicing;
    InvoiceRecipientPtr recipient;
    InvoiceLineItems invoiceLineItems;
};

std::ostream& operator<<(std::ostream& outstream, const Invoice& invoice) {
    outstream << "Invoice No.: " << invoice.invoiceNumber << "\n";
    outstream << "Recipient: " << *(invoice.recipient) << "\n";
    outstream << "Date/time: " << invoice.getDateTimeOfInvoicingAsString() << "\n";
    outstream << "Items:" << "\n";
    for (const auto& item : invoice.invoiceLineItems) {
        outstream << "    " << *item << "\n";
    }
    outstram << "Amount incoived: " << invoice.getSum() << std::endl;
    return outstream;
}

使用标准库的容器而不是C风格的数组
使用C++11标准中的std::array<TYPE,N>模板
const std::size_t arraySize = 10;
MyArrayType array[arraySize];

void function(MyArrayType const* array, const std::size_t arraySize) {
    //...
}
这样不会形成聚合单元
#include <array>

using MyTypeArray = std::array<MyType, 10>;

void funtion(const MyTypeArray& array) {
    const std::size_t arraySize = array.size();
    //...
}
具有STL兼容接口，可以使用std::array::begin()和std::array::end() <algorithm>
#include <array>
#include <algorithm>

using MyTypeArray = std::array<MyType, 10>;
MyTypeArray array;

void doSomethingWithEachElement(const MyType& element) {
    //...
}

std::for_each(std::cbegin(array), std::cend(array), doSomethingWithEachElement);
提示:
非成员函数std::begin()和std::end()，不建议使用成员函数
C++14中补充std::cbegin() std::cend() std::rbegin() std::rend()
#include <vector>

std::vector<AnyType> aVector;
auto iter = std::begin(aVector); // ...instead of 'auto iter = aVector.begin();'
std::array::at(size_type n)访问数组元素

用C++类型转换代替C风格的强制转换
类型转换都是不好的，应该尽可能避免类型转换
double d {3.1415};
int i = (int)d;
应该用
int i = static_cast<int>(d);
C++风格的类型转换会在编译器编译期间进行检查，而C风格的强制转换不会，可能运行时出错
int32_t i {200}; //Reserves and uses 4 byte memory
int64_t pointerToI = (int64_t*)&i; // Pointer points to 8 byte

*pointerToI = 9223372036854775807;// Can cause run-time error though stack corruption
不会报错
int64_t* pointerToI = static_cast<int64_t*>(&i); // Pointer points to 8 byte
编辑器会发现问题并报错
static_cast<> const_cast<> dynamic_cast<>
建议：
1 在任何情况下尽量避免类型转换 （强制转换）
2 如果无法避免 仅使用C++风格的类型转换 永远不要用C风格的类型转换
3 不要使用dynamic_cast<> 它被认为是一个糟糕的设计，它表明当前的特殊化层次结构出现了问题
4 在任何情况下，永远不要用reinterpret_cast<> 这种类型转换被打上了不安全，不可移植和依赖于实现等标记
它漫长而复杂的名称是一种暗示，让你思考目前你正在做什么

避免使用宏
C语言中最重要的遗产之一也许就是宏
宏是一段可以通过名称进行标识的代码
如果预处理器在编译时在源代码找到了宏的名称
则该名称将被其相关的代码片段替换
#define BUFFER_SIZE 1024
#define PI 3.14159265358979
#define MIN(a,b) ((a)<(b))?(a):(b);
#define MAX(a,b) ((a)>(b))?(a):(b);
宏的问题
#define DANGEROUS 1024+1024
int value = DANGEROUS * 2;
int maximum = MAX(12, value++);
int maximum = (((12)>(value++))?(12):(value++));
不要再使用宏了，除非一些非常罕见的例外情况，宏已经不再是必须的了
constexpr int HARMLESS = 1024 + 1024;
#include <alogrithm>
//...
int maximum = std::max(12, value++);
```

---

# 第五章 现代C++的高级概念

```
资源管理

主要资源包括：
内存 栈内存和堆内存
硬盘或网络文件读写所需的文件句柄
网络连接 如 连接服务器 数据库等
线程 锁 定时器 事务
其他操作系统资源 如 windows的GDI句柄

不好的例子
void doSomething() {
    ResourceType * resource = new ResoucrceType();
    try {
        // ...
        resource->foo();
    } catch (...) {
        delete resource;
        throw;
    }
    delete resource;
}
分支过多，任何一个分支忘记delete，就会导致资源泄露
资源泄露是很严重的问题，操作系统缺乏资源将直接导致临界系统状态，
可能是一个安全问题,攻击者可能利用它们进行 拒绝服务 Denial of Service DOS攻击

更简单的解决方法
void doSomething() {
    ResourceType resource;
    // ...
    resource.foo();
}
不是在任何情况下，我们都能在栈上分配内存，那么
如何保证分配的资源总是被释放

资源申请即初始化
Resource Acquisition is INitialization RAII
构造时获得，析构时释放
基于范围的资源管理
template <typename RESTYPE>
class ScopedResource final {
public:
    ScopedResource() { managedResource = new RESTYPE(); }
    ~ScopedResource() { delete managedReource; }

    RESTYPE* operator->() const { return managedReource; }
private:
    RESTYPE* managedResource;
};

#include "ScopedResource.h"
#inculde "ResourceType.h"

void doSomething() {
    ScrpedResource<ResourceType> resource;
    try {
        // ...
        resource->foo();
    } catch (...) {
        throw;
    }
}
这里不需要new 和 delete，如果这个资源超过了它的使用范围
被包装的ResourceType类的实例化资源就会通过析构函数自动释放
不需要实现这种包装器，这其实就是智能指针

智能指针
独占所有权
#include <memory>
class ResourceType {
    //...
};
std::unique_ptr<ResourceType> resource1 { std::make_unique<ResourceType>() };
auto resource2 { std::make_unique<ResourceType>() };
resource->foo();
放入std::vector中
#include "ResourceType.h"
#include <memory>
#include <vector>

using ResourceTypePtr = std::unique_ptr<ResourceType>;
using ResourceVecotr = std::vector<ResourceTypePtr>;

ResourceTypePtr resource { std::make_unique<ResourceType>() };
ResourceVecotr aCollectionOfResources;
aCollectionOfResources.push_back(std::move(resource));
// 注意：运行到这里的时候，resource实例变成了空
警告：
不要在代码中使用std::auto_ptr<T> c++17中已经删除了
不支持rvalue和Move语义
std::unique_ptr<ResourceType> pointer1 = std::make_unique<ResourceType>();
std::unique_ptr<ResourceType> pointer2;
pointer2 = std::move(pointer1);
共享所有权
std::shared_ptr<T>可以拷贝，也可以用move移动
std::shared_ptr<ResourceType> pointer1 = std::make_shared<ResourceType>();
std::shared_ptr<ResourceType> pointer2;
pointer2 = std::move(pointer1);
无所有权但是能够安全访问
从std::shared_ptr<T>的实例中获取原始指针
std::shared_ptr<ResourceType> resource = std::make_shared<ResourceType>();
//...
ResourceType* rawPointerToResource = resource.get();//原始指针
小心！
这么做可能非常危险，shared_ptr释放了，weak_ptr会指向野指针
weak_ptr仅仅观察它指向的资源
#include <memory>
void doSomething(const std::weak_ptr<ResourceType>& weakResource) {
    if (!weakReource.expired()) {
        // 有效
        std::shared_ptr<ResourceType> sharedResource = weakResource.lock();
        // 使用 shared_ptr
    }
}
int main() {
    auto sharedResource(std::make_shared<ResourceType>());
    std::weak_ptr<ResourceType> weakResource(sharedResource);

    doSomething(weakResource);
    sharedResource.reset();//删除shared_ptr指向的ResourceType实例
    doSomething(weakResource);

    return 0;
}
使用shared_ptr和weak_ptr区分资源持有者和使用者
例子：在内存中保留最近访问的对象一段时间， 缓存对象以及shared_ptr指针，定期运行垃圾清理
在使用缓存的地方用weak_ptr指向对象，获取shared_ptr。垃圾清理需要判断引用计数，不影响使用者
例子：循环依赖，
#include <memory>
class B;
class A {
public:
    void setB(std::shared_ptr<B>& pointerToB) {
        myPointerToB = pointerToB;
    }
private:
    std::shared_ptr<B> myPointerToB;
};
class B {
public:
    void SetA(std::shared_ptr<A>& pointerToA) {
        myPointerToA = pointerToA;
    }
private:
    std::shared_ptr<A> myPointerToA;
};
int main() {
    {
        // 花括号建立作用域范围
        auto pointerToA = std::make_shared<A>();
        auto pointerToB = std::make_shared<B>();
        pointerToA->setB(pointerToB);
        pointerToB->setA(pointerToA);
    }
    // 此时，A和B实例已经无法访问了，但是A和B的实例又没有被释放，内存泄漏
    return 0;
}
使用weak_ptr就能解决
class B;
class A {
public:
    void setB(std::shared_ptr<B>& pointerToB) {
        myPointerToB = pointerToB;
    }
private:
    std::weak_ptr<B> myPointerToB;
};
class B {
public:
    void setA(std::shared_ptr<A>& pointerToA) {
        myPointerToA = pointerToA;
    }
private:
    std::weak_ptr<A> myPointerToA;
};
循环依赖是糟糕的设计，应该尽可能避免循环依赖

避免显式的new和delete
new和delete会增加代码的复杂度
尽可能使用栈内存
用make functions在堆上分配资源，用std::make_unique<T>或std::make_shared<T>
尽量使用容器
如果有特殊的内存管理，利用特有的第三方库封装资源

管理特有资源
比如，在文件系统中打开文件，动态链接模块DLL，图形界面特殊平台对象(窗口，按钮，文本框)
通常这些资源是通过所谓句柄handle来管理的，句柄是操作系统资源的一个抽象唯一引用
在windows平台上，用HANDLE这种类型定义这些句柄，定义在WinNT.h中，C风格
typedef void *HANDLE;
如果想用一个合理的进程id访问Windows进程，可以使用Win32 API函数OpenProcess()检索该进程的句柄
#include <windows.h>

const DWORD processId = 4711;
HANDLE processHandle = OpenProcess(PROCESS_ALL_ACCESS, FALSE, processId);
用完后，必须用CloseHandle()函数释放句柄
BOOL success = CloseHandle(processHandle);
因此这种用法与new运算符和delete运算符有类似的对称性，所以也应利用RAII术语来管理
#include <windows.h>
class Win32HandleCloser {
public:
    void operator()(HANDLE handle) const {
        if (handle != INVALID_HANDLE_VALUE) {
            CloseHandle(handle);
        }
    }
};
注意：如果定义别名，std::shared<T> 管理的类型表即为void**
因为HANDLE已经被定义为void*
using Win32SharedHandle = std::shared_ptr<Handle>;//注意
Win32中Handle的智能指针必须按照如下定义
using Win32SharedHandle = std::shared_ptr<void>;
using Win32WeakHandle = std::weak_ptr<void>;
注意：在C++中不允许定义std::unique_ptr<void>类型！因为std::shared_ptr<T>实现了
类型删除，但是std::unique_ptr<T>没有。如果一个类支持类型删除，也就意味着它可以存储任意类型的对象，
而且会正确地释放对象占用的内存。
如果你想用共享的句柄，你必须注意在对象构造时应当传一个自定义的句柄删除器作为构造函数的参数
const DWORD processId = 4711;
Win32SharedHandle processHandle { OpenProcess(PROCESS_ALL_ACCESS, FALSE, processId), Win32HandleCloser()};
```

```
Move 语义
在以前的许多情况下，旧的C++语言强迫我们使用复制构造函数，实际上我们没有真正想要对象的深拷贝
我们只是想移动对象的负载，即对象的数据。
以前我们必须使用复制而不是move，比如：
局部变量作为函数或方法返回值时。C++11前为了防止拷贝构造，经常使用指针解决这类问题
向std::vector或者其他容器插入一个对象时
std::swap<T>模板函数的实现
前面提到的许多情况下，没有必要保持源对象的完整性，即深度复制，以便保持源对象可用
C++11除了复制构造函数和拷贝构造赋值运算符
类的开发人员现在可以实现移动构造函数，和移动赋值操作符（其实不应该这样做）
通常来说，move的效率比拷贝操作符效率高
示例：显式实现了两种类型的语义
#include <string>

class Clazz {
public:
    Clazz() noexcept;                               // 默认构造函数
    Clazz(const Clazz& other);                      // 复制构造函数
    Clazz(Clazz&& other) noexcept;                  // move构造函数
    Clazz& operator=(const Clazz& other)            // 拷贝构造函数
    Clazz& operator=(Clazz&& other) noexcept;       // move赋值运算符
    virtual ~Clazz() noexcept;                      // 析构函数
};
右值运算符用&&进行标识，&的引用被称为左值引用
左值的更好的解释时，一个在内存有位置的对象
Type var1;
Type* pointer;
Type& reference;
Type& function();
这些都是左值
int theAnswerToAllQuestions = 42;
int& function() {
    return theAnswerToAllQuestions;
}
int main() {
    function() = 23;
    return 0;
}

右值引用
例子：临时的内存分配给右值引用后，内存将变成持久的。
int&& rvalueReference = 25 + 17;
int* pointerToRvalueReference = &rvalueReference;
*pointerToRvalueReference = 23;
函数或者方法的参数
void function(Type param)   左右都可以
void X::method(Type param)
void function(Type& param)  只接受左值
void function(const Type& param)
void X::method(Type& param)
void X::method(const Type& param)
void function(Type&& param) 只接受右值
void X::method(Type&& param)
函数或者方法可能接受的返回值类型
int function()              [const] int, [const] int&, [const] int&&
int X::method()
int& function()             Non-const int, int&
int& X::method()
int&& function()            字面值，对象右值引用(通过std::move获取)，对象的生命周期比函数长
int&& X::method()
例子：显式定义copy和move语义的类
#include <utility> // std::move<T>
class Clazz {
public:
    Clazz() = default;
    Clazz(const Clazz& other) {
        // Classical copy construction for lvalues
    }
    Clazz(Clazz&& other) noexcept {
        // Move constructor for rvalues: moves content from 'other' to this
    }
    Clazz& operator=(const Clazz& other) {
        // Classical copy assignment for lvalues
        return *this;
    }
    Clazz& operator=(Clazz&& other) noexcept {
        // Move assignment for rvalues: moves content from 'other' to this
        return *this;
    }
};
int main() {
    Clazz anObject;
    Clazz anotherObject1(anObject);                 //调用拷贝构造函数
    Clazz anotherObject2(std::move(anObject));      //调用move构造函数
    anObject = anotherObject1;                      //调用拷贝赋值函数
    anotherObject2 = std::move(anObject);           //调用move赋值函数
    return 0;
}

不要滥用move
示例：不合理使用std::move()
#include <string>
#include <utility>
#include <vector>

using StringVector = std::vector<std::string>;
StringVector createVectorOfStrings() {
    StringVector result;
    // ...do something that the vector is filled with many strings...
    return std::move(result);//Bad and unnecessary, just write "return result;"!
}
编译器会自动判断，不需要这样写
小心优化

零原则
五大规则：一个类需要显示定义 析构函数，总是定义拷贝构造函数，赋值构造函数。move构造函数，move赋值函数
示例：string类的一种不合理实现
#include <cstring>

class MyString {
public:
    explicit MyString(const std::size_t sizeOfString) : data { new char[sizeOfString] } { }
    MyString(const char* const charArray, const std::size_t sizeOfArray) {
        data = new char[sizeOfArray];
        strcpy(data, charArray);
    }
    virtual ~MyString() { delete[] data; };

    char& operator[](const std::size_t index) {
        return data[index];
    }
    const char& operator[](const std::size_t index) const {
        return data[index];
    }
    // ...
private:
    char* data;
}
问题：初始化构造函数没有检查指针charArray是否为nullptr，没有考虑string增长缩短
没有copy/move构造器以及copy/move赋值运算符
int main() {
    MyString aString("Test", 4);
    MyString anotherString { aString };// 浅拷贝，如果字符串对象被销毁，内存数据被双重删除
}
修改：
class MyString {
public:
    explicit MyString(const std::size_t sizeOfString) : data { new char[sizeOfString] } { }
    MyString(const char* const charArray, const int sizeOfArray) {
        data = new char[sizeOfArray];
        strcpy(data, charArray);
    }
    virtual ~MyString() { delete[] data; };
    MyString(const MyString&) = delete;
    MyString& operator=(const MyString&) = delete;
}
删除了拷贝构造函数和拷贝赋值函数，这样不能用于vector
类中没用理由显示定义析构函数，容易出问题
修改：
#include <vector>

class MyString {
public:
    explicit MyString(const std::size_t sizeOfString) {
        data.resize(sizeOfString, ' ');
    }
    MyString(const char* const charArray, const int sizeOfArray) : MyString(sizeOfArray) {
        if (charArray != nullptr) {
            for (int index = 0; index < sizeOfArray; index++) {
                data[index] = charArray[index];
            }
        }
    }

    char& operator[](const std::size_t index) {
        return data[index];
    }
    const char& operator[](const std::size_t index) const {
        return data[index];
    }
private:
    std::vector<char> data;
};
现在用了vector代替char*，不需要在析构函数中释放资源了
默认的copy/move构造器，赋值运算符，也是可以用的
遵从了KISS原则
这就是零原则:在实现你的类的时候，应该不需要声明/定义析构函数，也不需要声明/定义赋值运算符
用智能指针和标准库来管理资源
编译器自动生成的成员函数copy move 析构 可以正确执行
```

```
编译器是你的搭档
C++11改变了编译器的角色
能在编译阶段解决的事情，就在编译阶段解决
能在编译阶段检查的事情，就在编译阶段检查
编译器对程序所知道的一切都应该由编译器决定

自动类型推导
auto theAnswerToAllQuestions = 42;
auto iter = begin(myMap);
const auto gravitationalAccelerationOnEarth = 9.80665;
constexpr auto sum = 10 + 20 + 12;
// std::initializer_list<const char*>
auto strings = { "The", "big", "brown", "fox", "jumps", "over", "the", "lazy", "dog" };
// std::initializer_list<const char*>::size_type
auto numberOfStrings = strings.size();
依赖实参的名字查找ADL 参数相关查找，也叫做Koeing查找
std::map<unsigned int, std::strign> words;
auto wordIterator = begin(words);
注意：如果相对简单的C风格数组使用std::begin()，必须显式写出命名空间。
不要害怕过多使用auto，代码仍然是静态类型的，变量的类型也是明确定义的。
初始化STL容器
C++11前
std::vector<int> integerSequence;
integerSequence.push_back(14);
integerSequence.push_back(33);
integerSequence.push_back(69);
C++11后
std::vector<int> integerSequence { 14, 33, 69 };
因为std::vecotr<T> 重载了构造函数，能够接受初始化表达式列表
初始化列表的类型是std::initializer_list<T> 定义在头文件<initializer_list>
被大括号包含，以逗号分隔的值列表，被称为braced-init-list
例子：
#include <string>
#include <vector>

using WordList = std::vector<std::string>;

class LexicalRepository {
public:
    explicit LexicalRepository(const std::initializer_list<const char*>& words) {
        wordList.insert(begin(wordList), begin(words), end(words));
    }
    //...
private:
    WordList wordList;
};
int main() {
    LexicalRepository repo { "The", "big", "brown", "fox", "jumps", "over", "the", "lazy", "dog" };
    //...
    return 0;
}
C++14支持函数的返回值自动类型推导
auto function() {
    std::vector<std::map<std::pair<int, double>, int>> returnValue;
    // ...fill returnValue with data
    return returnValue;
}
匿名函数，允许将lambda表达式赋值给变量
auto square = [](int x) {return x * x;};
auto关键字提高可读性
std::shared_ptr<controller::CreateMonthlyInvoicesController> createMonthlyInvoicesController = 
    std::make_shared<controller::CreateMonthlyInvoicesController>();
auto createMonthlyInvoicesController = 
    std::make_shared<controller::CreateMonthlyInvoicesController>();
没有必要显式地重复类型，重复显而易见违反了DRY原则


编译时计算
高性能计算 HPC High Performance Computing爱好者
嵌入式开发者，喜欢使用静态、恒定的表来分隔数据和代码的程序员
都希望在编译阶段尽可能多地进行计算
在编译时进行运算是提高程序运行效率最简单的手段。
缺点是，编译代码所需时间增加
constexpr (constant expression)
C++14前，constexpr类型的函数只能有一个return语句
constexpr int theAnswerToAllQuestions = 10 + 20 + 12;
是一个常量和const一样
constexpr int multiply(const int multiplier, const int multiplicand) {
    return multiplier * multiplicand;
}
这种函数编译阶段就可调用，但在运行阶段，也能像普通函数一样接受非常量参数，因此
这种函数必须进行单元测试。
无疑，constexpr函数也能递归调用
示例：
#include <iostream>

constexpr unsigned long long factorial(const unsigned short n) {
    return n > 1 ? n * factorial(n - 1) : 1;
}

int main() {
    unsigned shoart number = 6;
    auto result1 = factorial(number);       // 运行时才被调用
    constexpr auto result2 = factorial(10); // 编译时就计算完成

    std::cout << "result1: : << result1 << ", result2: " << result2 << std::endl;
    return 0;
}

模板变量
template <typename T>
constexpr T pi = T(3.1415926434897932384626433L)
这就是模板变量，是一种很好的，灵活的替代用宏定义变量的方法
模板实例化时，将根据它使用时的上下文决定数字常量pi的类型为float double或long double
例子：
利用模板变量pi在编译阶段计算圆周
template <typename T>
constpxtr T computeCircumference(const T radius) {
    return 2 * radius * pi<T>;
}

int main() {
    const long double radius { 10.0L };
    constexpr long double circumference = computeCircumference(radius);
    std::cout << circumference << std::endl;
    return 0;
}
类也可以实现编译时计算，可以将类的构造函数和成员函数定义为constexpr
#include <iostream>
#include <cmath>

class Rectangle {
public:
    constexpr Rectangle() = delete;
    constexpr Rectangle(const double width, const double height) :
        width { width }, height { height } { }
    constexpr double getWidth() const { return width; }
    constexpr double getHeight() const { return height; }
    constexpr double getArea() const { return width * height; }
    constexpr double getLengthOfDiagonal() const {
        return std::sqrt(std::pow(width, 2.0) + std::pow(height, 2.0));
    }

private:
    double width;
    double height;
}

int main() {
    constexpr Rectangle americanFootballPlayingField { 48.6, 110.0 };
    constexpr double area = americanFootballPlayingField.getArea();
    constexpr double diagonal = americanFootballPlayingField.getLengthOfDiagonal();

    std::cout << "The area of an American Football playing field is " <<
        area << "m^2 and the length of its diagonal is " << diagonal <<
        "m." << std::endl;
    return 0;
}
同样，constexpr类在运行及编译时都能用。
类中不能定义虚成员函数，因为编译时没有多态性
类的析构函数不能显式定义
某些C++编译器中，上面代码编译不能成功，因为std::sqrt() 和 std::pow() 并没有指定必须为constexpr类型
编译器自由选择支持
```

```
不允许未定义的行为

例子：
未定义行为，未正确使用智能指针
const std::size_t NUMBER_OF_STRINGS { 100 };
std::shared_ptr<std::string> arrayOfStrings(new std::string[NUMBER_OF_STRINGS]>);
析构时，会调用delete，但这是错误的，因为分配资源时用了new[] 析构时应该用delete[]
方案1：
提供自定义的类似于函数的删除器对象
也成为 仿函数
template< typename Type>
struct CustomArrayDeleter
{
    void operator() (Type const* pointer)
    {
        delete [] pointer;
    }
}
const std::size_t NUMBER_OF_STRINGS { 100 };
std::shared_ptr<std::string> arrayOfStrings(new std::string[NUMBER_OF_STRINGS], CustomArrayDeleter<std::string>());
在C++11中，头文件<memory>中定义了数组类型的默认删除器
const std::size_t NUMBER_OF_STRINGS {100};
std::shared_ptr<std::string> arrayOfStrings(new std::string[NUMBER_OF_STRINGS], std::default_delete<std::string[]>());
应该考虑需要满足的需求，使用std::vector并不总是实现对象数组的最佳方案
```

```
Type-Rich 编程

不要相信名字，应该相信类型
double 可以表示很多东西，它是一个数据类型，但并不是一个语义类型
定义语义明确的接口：
class SpacecraftTrajectoryControl {
public:
    void applyMomentumToSpacecraftBody(const Momentum& impulseValue);
};
力学中，动量的单位是 N*s 表示 1 kg m/s^2
MKS单位体系 m米长度，kg质量，s秒时间
templete <int M, int K, int S>
struct MksUnit {
    enum { metre = M, kilogram = K, second = S };
};
还需要一个表示值的类模板
template <typename MksUnit>
class Value {
private:
    long double magnitude { 0.0 };
public:
    explicit Value(const long double magnitude) : magnitude(magnitude) {}
    long double getMagnitude() const {
        return magnitude;
    }
};
使用这两个类模板来为具体的物理量定义类型别名
using DimensionlessQuantity = Value<MksUnit<0, 0, 0>>;
using Length = Value<MksUnit<1, 0, 0>>;
using Area = Value<MksUnit<2, 0, 0>>;
using Volume = Value<MksUnit<3, 0, 0>>;
using Mass = Value<MksUnit<0, 1, 0>>;
using Time = Value<MksUnit<0, 0, 1>>;
using Speed = Value<MksUnit<1, 0, -1>>;
using Acceleration = Value<MksUnit<1, 0, -1>>;
using Frequency = Value<MksUnit<0, 0, -1>>;
using Monment = Value<MksUnit<1, 1, -1>>;
using Force = Value<MksUnit<1, 1, -2>>;
using Pressure = Value<MksUnit<-1, 1, -2>>;

这些单位也可以用于常量的定义
template <typename MksUnit>
class Value {
public:
    constexpr explicit Value(const long double magnitude) noexcept : magnitude { magnitude } {}
    constexpr long double getMagnitude() const noexcept {
        return magnitude;
    }
private:
    long double magnitude { 0.0 };
};

constexpr Acceleration gravitationalAccelerationOnEarth { 9.80665 };
constexpr Pressure standardPressureOnSeaLevel { 1013.25 };
constexpr Speed speedOfLight { 299792458.0 };
constexpr Frequency concertPitchA { 440.0 };
constexpr Mass neutronMass { 1.6749286e-27 };

加减乘除运算
template <int M, int K, int S>
constexpr Value<MksUnit<M, K, S>> operator+
    (const Value<MksUnit<M, K, S>>& lhs, const Value<MksUnit<M, K, S>>& rhs) noexcept {
    return Value<MksUnit<M, K, S>>(lhs.getMagnitude() + rhs.getMagnitude());
}

template <int M, int K, int S>
constexpr Valeu<MksUnit<M, K, S>> operator-
    (const Value<MksUnit<M, K, S>>& lhs, const Value<MksUnit<M, K, S>>& rhs) noexcept {
    reutrn Value<MksUnit<M, K, S>>(lhs.getMagnitude() - rhs.getMagnitude()));
}

template <int M1, int K1, int S1, int M2, int K2, int S2>
constexpr Value<MksUnit(M1 + M2, K1 + K2, S1 + S2>> operator*
    (const Value<MksUnit<M1, K1, S1>>& rhs, const Value<MksUnit<M2, K2, S2>>& rhs) noexcept {
    return Value<MksUnit<M1 + M2, K1 + K2, S1 + S2>>(lhs.getMagnitude() * rhs.getMagnitude());
}

template <int M1, int K1, int S1, int M2, int K2, int S2>
constexpr Value<MksUnit<M1 - M2, K1 - K2, S1 - S2>> operator/
    (const Value<MksUnit<M1, K1, S1>>& lhs, const Value<MksUnit<M2, K2, S2>>& rhs) noexcept {
    return Value<MksUnit<M1 - M2, K1 - K2, S1 - S2>>(lhs.getMagnitude() / rhs.getMagnitude());
}
使用
constexpr Momentum impulseValueForCourseCorrection = Force { 30.0 } * Time { 3.0 };
SpacecraftTrajectoryControl control;
control.applyMomentumToSpacecraftBody(impulseValueForCourseCorrection);
类型安全在编译期间就得到保障，在运行时没有开销

更进一步
constexpr Acceleration gravitationalAccelerationOnEarth = 9.80665_ms2;
C++11后，可以为文字提供自定义后缀来定义特殊的函数，这是 文字操作符
constexpr Force operator"" _N(long double magnitude) {
    return Force(magnitude);
}
constexpr Acceleration operator"" _ms2(long double magnitude) {
    reutrn Acceleration(magnitude);
}
constexpr Time operator"" _s(long double magnitude) {
    return Time(magnitude);
}
constexpr Momentum operator"" _Ns(long double magnitude) {
    return Momentum(magnitude);
}
用户自定义字面值
字面值是一个编译时常量，可以自定义
constexpr Money operator"" _USD (const long double amount) {
    return Money(amount);
}
constexpr Money amount = 145.67_USD;

Force force = 30.9N;
Time time = 3.0_s;
Momentum momentum = force * time;

type-rich编程和用户自定义的字面值受到保护
Force force1 = 3.0;// 错误
Force force2 = 3.0_s;//错误
Force force3 = 3.0_N;//正确

auto force = 3.0_N;
constexpr auto acceleration = 100.0_ms2;

创建强类型的接口 APIs
避免在公共接口API中使用通用的，底层的内置类型int double 最坏的是void*
这种非语义类型在某些情况下是危险的，它几乎可以表示任何东西

提示：
已经有库实现了物理量类型 包括所有 SI units
standard international unit
Boost.Units就是一个很好的例子
```

```
了解你使用的库

Not invented here
NIH综合征
是一种组织反模式，描述了对现有知识的忽略或原生的尝试和测试的解决方案
是一种重新造轮子的形式
重新实现一些库或者框架已经在某些方面高度可用的，高质量的东西
这种态度背后的原因：
认为内部开发必须在几个方面药更好，被错误的认为，比现有的解决方案更便宜，更安全，更灵活和更可控
事实上，只有少数公司可用成功开发出等同或者更好的替代方案，以代替市场上已经存在的解决方案
通常，与成熟的方案比， 自行开发的库或框架，质量显然要差很多

合格的开发者应该知道这些库，不需要了解这些库及API的每个实现细节
最好知道有针对某个领域的，经过试验和测试的解决方案


熟练使用<algorightm>
想提高编码质量，请用一个目标替换所有的编码指南：没有原始循环
例子：反转
#include <vector>

std::vector<int> integers { 2, 5, 8, 22, 45, 67, 99 };

std::size_t leftIndex = 0;
std::size_t rightIndex = integers.size() - 1;

while (leftIndex < rightIndex) {
    int buffer = integers[rightIndex];
    integers[rightIndex] = integers[leftIndex];
    integers[leftIndex] = buffer;
    ++leftIndex;
    --rightIndex;
}
缺点：
很难立刻知道这段代码在做什么 while 前三行可用std::swap替代 <utility>库
容易出错，如果违反了std::vector的边界并视图访问
    std::vector::at()会导致std::out_of_range异常
    std::vector::operator[]会导致未定义行为
C++标准库提供了100多种有用的算法，可用于搜索，计数，操作容器或序列中的元素
#include <algorithm>
#include <vector>

std::vector<int> integers = { 2, 5, 8, 22, 45, 67, 99 };
std::reverse(std::begin(integers), std::end(integers));
紧凑，不容易出错，容易阅读

反转C风格的数组和字符串
#include <algothrim>
#include <string>

int integers[] = { 2, 5, 8, 22, 45, 67, 99 };
std::reverse(std::begin(integers), std::end(integers));

std::string text { "The big brown fox jump over the lazy dog!" };
std::reverse(std::begin(text), std::end(text));
作用于子序列
std::reverse(std::begin(text) + 13, std::end(text) - 9);


C++17中简单的并行算法
C++11以前，C++标准只支持单线程，必须使用第三方库 如Boost.Thread 或者编译器拓展Open Multi-Processing OpenMP来并行化程序
C++11开始，线程支持库支持多线程和并行编程。引入了线程，互斥，条件变量和futures
并行化一段代码需要良好的多线程知识，必须在设计时考虑。否则竞态条件可能因此小错误，非常难以调试。
特别是标准库算法，经常包含大量容器，了充分利用多核，应当简化并行操作。
C++17开始，部分标准库根据C++并行扩展技术规范进行了重新设计，也成为并行TS(技术规范)
目标是将开发人员从复杂的处理多线程任务中解放，例如std::thread std::mutex
事实上，有69种算法被重载，现在有一或多个版本可用，接受ExecutionPolicy额外参数来并行化
如：std::for_each std::transform std::copy_if std::sort
添加了7种可并行化的新算法： std::reduce std::exclusive_scan std::transform_reduce
在函数式编程中特别有用

执行策略
std::find外，还定义了另一个版本，使用额外模板参数指定执行策略
template< class InputIt, class T >
InputIt find( InputIt first, InputIt last, const T& value );

template< class ExecutionPolicy, class ForwardIt, class T>
ForwardIt find(ExecutionPolicy&& policy, ForwardIt first, ForwardIt last, const T& value);

ExecutionPolicy的三个标准策略是：
std::execution::seq 和单线程版本相同
std::execution::par 可并行化 允许在多个线程执行 并行算法不会自动保护数据的竞争或死锁，开发人员有责任确保执行时不会发生数据竞争
std::execution::par_unseq 可向量化和并行化 向量化利用了SIMD(单指令，多数据)指令集，SIMD意味着处理器可用同时对多个数据点执行相同的操作
当然对一个包含几个平行元素的向量排序是完全没有意义的，线程管理的开销远远高于性能收益。因此，可在运行时动态选择执行策略。
例如，根据向量大小选择执行策略
不幸的是，C++17还没有接受动态执行策略，C++20在计划中
例子：
容器的排序和输出
std::sort使用<比较操作符，如果要对自定义类进行排序，必须实现<操作符
#include <algorithm>
#include <iostream>
#include <string>
#include <vector>

void printCommaSeparated(const std::string& text) {
    std::cout << text << ", ";
}

int main() {
    std::vector<std::string> names = { "Peter", "Harry", "Julia", "Marc", "Antonio", "Glenn" };
    std::sort(std::begin(names), std::end(names));
    std::for_each(std::begin(names), std::end(names), printCommaSeparated);
    return 0;
}
对比两个序列
std::equal比较两个字符串序列
#include <algorithm>
#include <iostream>
#include <string>
#include <vector>

int main() {
    const std::vector<std::string> names1 { "Peter", "Harry", "Julia", "Marc", "Antonio", "Glenn" };
    const std::vector<std::string> names2 { "Peter", "Harry", "Julia", "John", "Antonio", "Glenn" };

    const bool isEqual = std::equal(std::begin(names1), std::end(names1), std::begin(names2), std::end(names2));

    if (isEqual)
        std::cout << "The contents of both sequences are equal.\n";
    else
        std::cout << "The contents of both sequences difffer.\n";
    
    return 0;
}
默认情况，std::equal使用==操作符比较元素，但是可用自定义
#include <algorithm>
#include <iostream>
#include <string>
#include <vector>

bool compareFirstThreeCharactersOnly(const std::string& string1, const std::string& string2) {
    return (string.compare(0, 3, string2, 0, 3) == 0);
}

int main() {
    const std::vector<std::string> names1 { "Peter", "Harry", "Julia", "Marc", "Antonio", "Glenn" };
    const std::vector<std::string> names2 { "Peter", "Harry", "Julia", "John", "Antonio", "Glenn" };

    const bool isEqual = std::equal(std::begin(names1), std::end(names1), std::begin(names2), std::end(names2), compareFirstThreeCharactersOnly);

    if (isEqual)
        std::cout << "The contents of both sequences are equal.\n";
    else
        std::cout << "The contents of both sequences difffer.\n";
    
    return 0;
}
可以用lambda函数
const bool isEqual = std::equal(std::begin(names1), std::end(names1), std::begin(names2), std::end(names2), 
    [](const auto& string1, const auto& string2) {
        return (string1.compare(0, 3, string2, 0, 3) == 0);
    });

lambda实现比较函数，代码看起来更紧凑，但是lambda表达式会影响代码可读性
还是建议用 显式的名称 compareFirstThreeCharactersOnly


熟练使用Boost
一定要掌握的库，标准库容器，<algorithm>，Boost

应该了解的一些库
<chrono> 日期时间库
例如：std::chrono::duration表示时间段 std::chrono::system_clock表示系统当前时间

<regex> 正则表达式库

<filesystem> 文件系统库
C++17以来，文件系统库已经成为C++标准的一部分。
操作系统独立的库，提供了对文件系统及其组件的各种工具。创建目录，复制文件，遍历目录，检索文件大小等
如果没用C++17，可以用Boost.Filesystem替代标准文件系统库

Range-v3
一个仅有头文件的库，简化了对C++标准库或其他库如Boost的容器的处理
可以更方便地编写迭代器相关代码
std::sort(std::begin(container, std::end(container));
替换为
range::sort(container);

并发数据结构 libcds
基于C++11，BSD许可，提供无锁算法和并发数据结构的实现，主要用于高性能并行计算
```

```
恰当的异常和错误处理机制

横切关注点 Cross-Cutting Concerns
指难以用模块化的概念解决的问题，通常用软件架构和设计来解决

其中一个横切关注点就是安全问题
如果你必须在你的软件中处理数据安全和访问权限问题，因为有高质量的要求。那么这将是一个贯穿该系统的敏感话题
几乎要在每个组件的每个地方处理这个事情

另一个横切关注点是事务处理
特别是数据库应用，你必须保证事务，要么成功，要么作为一个完整的单元失败。不可能只完成一部分

日志也是横切关注点，有时候domain-specific和productive被日志打乱，会降低代码的可读性和可理解性

如果软件架构不考虑横切关注点，那么这个架构就不是一个完整的方案。
例如：两个不同的日志框架可以用在同一个工程中,因为开发同一个系统的两个不同的开发团队可能选择不同的日志框架。

异常和错误是另外一个横切关注点。对错误和不可预测异常的特殊处理及相应，对于每个软件系统都是必要的。
那么，好的错误处理原则是什么？什么时候抛出异常合理？如何处理抛出的异常？什么情况下异常不能使用？有其他办法吗？


防患于未然
处理异常和错误的基本策略通常是避免他们。
异常安全是接口设计的一部分，接口API不仅包括函数的签名：也就是参数和返回值
还应该包含函数可能抛出的异常部分。此外，还有三个方面必须考虑
前置条件： 前置条件在函数或者类的方法调用前必须总为真。如果违反了前置条件，函数调用的结果就难以保证。
不变式：在函数调用的过程中必须总是真。条件在函数的开始和结束都为真，在面向对象中一种特殊的不变式是类不变式。
    如果违反了不变式，类的对象在方法调用后将会导致不正确或者不一致。
后置条件：函数执行结束后立即返回真。如果后置条件不成立，那么就说明在函数调用的过程中肯定出错了。
不变式、后置条件、抛异常或者不抛异常的保证，有四种异常安全级别

无异常安全 完全保证不了任何事情
例如：代码的一部分违反了不变式和后置条件，就可能导致崩溃。
不应该提供这个级别的异常安全。

基本异常安全 任何代码都应该至少保证异常安全级别。
稍微花点功夫就能达到。该级别的异常安全保证了以下几个方面
1 如果调用过程发生异常，保证资源无泄漏。包括内存和其他资源，可以通过RAII模式达到目标
2 如果调用过程发生异常，所有的不变式保持不变
3 如果调用过程发生异常，不会有数据或者内存损坏，而且所有的对象都是良好和一致的状态。 但不能保证调用函数后，数据的内容不变
严格的规则是这样的：
至少应该达到基本异常安全，这时默认的安全级别

强异常安全
除了保证基本类型安全，还有保证在异常情况下，数据内容完全恢复到调用函数或方法之前。即支持回滚。
需要额外工作，而且运行时开销可能比较大。 需要额外工作的一个例子时：copy-and-swap习惯用于保证拷贝赋值的强异常安全
所有代码使用强异常安全，就会违反KISS原则和YAGNI原则
建议： 只有在绝对需要的情况下，才为代码提供强异常安全保证
如果有关于数据完整性和数据正确性的质量要求，就必须满足强异常安全，必须通过强异常安全保证回滚机制。

保证不抛出异常
最高异常安全级别 也成为 故障透明性
简单的说，调用函数或者方法时，不必担心异常问题。函数或者方法调用总会成功。
有时候难以或者不可能达到。特别是C++语言。如果你用new运算符，直接或者间接比如std::make_shared<T>，在遇到异常时，这个函数调用绝对不成功
在以下情况，保证不抛出异常，要么绝对强制，要么明确建议；
在任何情况下类的析构函数必须保证不抛出异常：因为析构函数遇到异常也会在栈展开时调用。栈展开时如果遇到异常，就会发生严重错误，程序崩溃。
    因此，任何析构函数分配西元以及试图关闭资源的操作，像是打开文件，在堆上分配内存，必须是不抛出异常的。
Move操作应该保证不抛出异常：如果move抛出异常，那么move很大概率没有作用。对于C++标准库容器使用的类型，保证不抛出异常也很重要。
    如果move不提供保证，容器倾向于拷贝操作而不是move操作。
默认构造函数最好不抛出异常：半构造的对象很可能违反不变式，默认构造函数应该简单，应该试图避免在默认构造函数中抛出异常。
在任何情况下，swap函数必须保证不抛出异常：swap函数抛出异常将是致命的，因为程序会以不一致的状态退出。而且编写异常安全的operator==的最佳方法
    是用不抛出异常的swap函数实现的

NOEXCEPT 声明符和运算符 C++11
C++11前，函数的声明中可以添加throw关键字，用于列出函数直接或者间接抛出的所有异常类型
异常间用逗号隔开，称为  动态异常声明
throw(exceptionType, exceptionType, ...)
在C++11中已废弃
在C++17中彻底删除
C++11的throw()声明符没有异常参数列表，这个 throw() 和 noexcept(true)等价
noexcept声明符表示函数不能抛出任何异常
noexcept(true)和noexcept一样
noexcept(false)声明的函数签名可能会抛出异常
示例：
void noThrowingFunction() noexcept;
void anotherNonThrowing() noexcept(true);
void aPotentiallyThorwingFunction() noexcept(false);
用noexcept有两个好的理由：
1 函数或方法的异常抛出或者不抛出是函数接口的一部分，它是关于语义的，帮助开发人员了解可能发生什么，不可能发生什么
2 可以被编译器优化，noexcept允许编译器将以前需要的throw删掉，减少运行时开销。也就是说，异常没有被列出，目标代码就没有必要调用std::unexpect()函数

对于模板实现者也有noexcept操作符，允许编译器编译时检查，如果表达式声明不抛出任何异常则返回true：
constexpr auto isNotThrowing = noexcept(nonThrowingFunction());

constexpr函数在运行时求值可能会抛出异常，所以某些情况下也要使用noexcept


异常即异常，字面上的意思
示例：为未找到的消费者抛出异常
#include "Customer.h"
#inlcude <string>
#include <exception>

class CustomerNotFoundException : public std::exception {
    virtual const char* what() const noexcept override {
        return "Customer not found!";
    }
};

Custormer Customer::findCustomerByName(const std::string& name) const noexcept(false)
{
    throw CustomerNotFoundException();
}

看看这个函数的调用

Custormer customer;
try {
    customer = findCustomerByName("Non-existing name");
} catch (const CustomerNotFoundException& ex) {
}

非常糟糕
找不到消费者很正常，上面的例子在滥用异常
异常不能控制正确的程序流程，应该应用在真正需要异常的情形！

真正需要异常，意思是你对此束手无策，没有办法处理
仅仅在非常特殊的情况下抛出异常，不要滥用异常来控制正确的程序流程


如果遇到异常不能恢复，通常的做法是写日志记录异常如果可能，或者生成crash dump文件稍后分析，然后立即终止程序
Dead Program Tell No Lies
没有什么比在一个严重错误后当作没有发生继续运行更糟糕。比如生成数以万计的错误订单，等等。


用户自定义异常
在C++中虽然可以抛出任意类型的异常，像是int const char* 但是不推荐。
异常由其类型捕获，因此对于某些异常，创建自定义异常类非常好。
例子：
#include <stdexcept>

class MuCustomeException : public std::exception {
    virtual const char* what)_ const noexception override {
        return "Provide some details about what was going wrong here!"
    }
};

#include <iostream>

try {
    doSomethingThatThrows();
} catch (const std::exception& ex) {
    std::cerr<< ex.what() << std::endl;
}
自定义复杂异常类例子：
class DivisionByZeroException : public std::exception {
public:
    DivisionByZeroException() = delete;
    explicit DivisionByZeroException(const int dividend) {
        buildErrorMessage(dividend);
    }

    virtual const char* what() const noexcept override {
        return errorMessage.c_str();
    }
private:
    void buildErrorMessage(const int dividend) {
        errorMessage = "A division with dividend = ";
        errorMessage += std::to0_string(dividend);
        errorMessage += ", and divisor = 0, is not allowed (Division by Zero)!";
    }

    std::string errorMessage;
};
注意：由于实现机制，只能保证buildErrorMessage()函数是强异常安全的，以为使用了可能抛出异常的
std::string::operator+=() 因此，初始化构造函数也不能保证不抛出异常，这就是为什么异常类通常要设计的非常简洁
小示例：
int divide(const int dividend, const int divisor) {
    if (divisor == 0) {
        throw DivisionByZeroException(dividend);
    }
    return dividend / divisor;
}
int main() {
    try {
        divide(10, 0);
    } catch (cosnt DivisionByZeroException& ex) {
        std::cerr << ex.what() << std::endl;
        return 1;
    }
    return 0;
}


值类型抛出，常量引用类型捕获
例子：异常对象用new在堆上分配，然后以指针类型抛出
try {
    CFile f(_T("M_Cause_File.dat"), CFile::modeWrite);
    // 如果文件不存在，CFile构造器抛出异常 CFileException
}
catch(CFileException* e)
{
    if (e->m_cause == CFileException::fileNotFound)
        TRACE(_T("Error:: File not found\n"));
    e->Delete();
}
这种旧的C++编程风格，不要忘记调用Delete()成员函数
这是个不好的设计，不要这么做


注意catch的正确顺序
DivisionByZeroException 和 CommunicationInterruptedException 都继承自 std::exception
try{
    doSomethingThatCanThrowSeveralExceptions();
} catch (const DivisionByZeroException& ex) {
    //...
} catch (cosnt CommunicationInterruptedException& ex) {
    //...
} catch (const std::exception& ex) {
    // Handle all other exceptions here that are derived from std::exception
} catch (...) {
    // The rest...
}
```

---

# 第六章 面向对象

```
支持面向对象的语言 绝对不能保证开发人员能轻松实现面型对象的软件设计
长期使用面向过程语言的人，往往难以过渡到面向对象的编程范式
面向对象不是一个简单的概念，它要求开发人员以全新的方式看待这个世界

OOP背后的基本理念
在软件设计中，从与我们相关的领域对事物和概念进行建模
仅限于那些必须在软件系统中表示的事物，以满足利益相关者的需求
抽象是以适当的方式，对这些事物和概念建模的最重要的工具
不需要建模整个现实世界，只需要一个摘录。并简化成与实现系统用例相关的一些细节

面向对象是关于 数据抽象 责任划分 模块化 分治管理的
OOP是关于 复杂性的处理
软件系统可以分层次地分解为粗细粒度不同的模块
以应对 系统的复杂性。提供强大的灵活性
提高可重用性 可维护性 可测试性

分解的指导原则：
信息隐藏
高内聚
低耦合
单一责任原则 SRP
```

```
类的设计原则

类被视为封装了的软件模块
将结构特征 （同义词：属性，数据成员，字段）
和行为特征 （同义词：成员函数，方法，操作）
组合成一个有聚合力的单元
```

```
原则一： 让类尽可能小
大类，或多或少，只是被用作 过程程序的命名空间
开发人员通常不了解面向对象

大类的问题：很难被理解，并且可维护性 可测试性通常很差 更不用说可复用性
大类通常包含更多的缺陷

上帝反模式：
在许多系统中，存在具有许多属性和数百个方法的异常大类
通常以 Controller Manager Helpers 结尾
巨大的类，聚合力很差。被称为上帝类 上帝对象
反模式：同义词是 被认为糟糕的设计
上帝类很难维护，难以理解，不能测试，容易出错，对其他类大量依赖

类的大小上限 50行
```

```
原则二： 单一职责原则 SRP
Single Responsibility Principle
规定每个软件单元，其中包括 组件 类 函数。应该只有一个单一且明确定义的职责
单一职责使内聚性非常强
类很小，有很少的依赖性。清晰，易于理解 容易测试
```

```
原则三： 开闭原则 OCP
所有系统在其生命周期内都会发生变化 在开发 预期比第一个版本持续时间更长的系统 时，必须记住这一点
软件实体 （模块 类 函数）对扩展应该是开放的，但是对修改应该是封闭的

软件系统将随着时间推移而发展 必须满足不断增加的新需求
并且现有需求一定会随着 客户需要 技术进步 而不断改变
这些扩展 不仅应该以优雅的方式实现， 而且应该以尽可能小的代价完成。 最好在不需要更改现有代码的基础上实现
如果任何新的需求都会导致现有的，已经经过充分测试的部分发生 一连串变化 那将是致命的

支持开闭原则的一种方法就是继承 
通过继承可以在不修改类的情况下增加新功能
许多面向对象设计模式也支持OCP，如 策略模式，装饰器模式
第三章的 灯和开关的例子，很好的支持了OCP
```

```
原则四： 里氏替换原则(LSP)
里氏替换原则：你不能给一条狗增加四条假腿来创造一只章鱼

面向对象的继承和多态的概念乍一看似乎比较简单
继承是一种分类学的概念，被用于构建类型的特化层次结构
即子类型是从更通用的类型派生的
多态性通常意味着单个接口可以访问不同类型的对象
例子：正方形困境
Circle Rectangle Triangle TextLabel
抽象基类 Shape
class Point final {
public:
    Point() : x {5}, y {5} {}
    Point(const unsigned int initialX, const unsigned int initialY) :
        x {initialX}, y {initialY} {}
    void setCoordinates(const unsigned int newX, const unsigned int newY) {
        x = newX;
        y = newY;
    }
    //...more
private:
    unsigned int x;
    unsigned int y;
};

class Shape {
public:
    Shape() : isVisible {false} {}
    virtual ~Shape() = default;
    void moveTo(const Point& newCenterPoint) {
        hide();
        centerPoint = newCenterPoint;
        show();
    }
    virtual void show() = 0;
    virtual void hide() = 0;
    // ...
private:
    Point cneterPoint;
    bool isVisible;
};

void Shape::show() {
    isVisible = true;
}

void Shape::hide() {
    isVisible = false;
}

说明：final说明符
方法一：避免在派生类中重写单个虚成员函数
class AbstractBaseClass {
public:
    virtual void doSomething() = 0;
};

class Derived1 : public AbstractBaseClass {
public:
    virtual void doSomething() final {
        //...
    }
};

class Derived2 : public Derived1 {
public:
    virtual void doSomething() override { //编译错误
    }
};
方法二：将类标记为final，不可以用作继承
class NotDerivable final {
};

继续上面代码：
class Rectangle : public Shape {
public:
    Rectangle() : width { 2 }, height { 1 } {}
    Rectangle(const unsigned int initialWidth, const unsigned int initialHeight) :
        width { initialWidth }, height { initialHeight } {}

    virtual void show() override {
        Shape::show();
        // ...
    }
    virtual void hide() override {
        Shape::hide();
        // ...
    }
    void setWidth(const unsigned int newWidth) {
        width = newWidth;
    }
    void setHeight(const unsigned int newHeight) {
        height = newHeight;
    }
    void setEdges(const unsigned int newWidth, const unsigned int newHeight) {
        width = newWidth;
        height = newHeight;
    }
    unsigned long long getArea() const {
        return static_cast<unsigned long long>(width) * height;
    }
    //...
private:
    unsigned int width;
    unsigned int height;
};
客户端代码希望以类似的方式使用所有形状，无论对应哪个实例 矩形 原型
#include "Shape.h"
#include <memory>
#include <vector>

using ShapePtr = std::shared_ptr<Shape>;
using ShapeCollection = std::vector<ShapePtr>;

void showAllShapes(const ShapeCollection& shapes) {
    for (auto& shape : shapes)
        shape->show();
}

int main() {
    ShapeCollection shapes;
    shapes.push_back(std::make_shared<Circle>());
    shapes.push_back(std::make_shared<Rectangel>());
    shapes.push_back(std::make_shared<TextLable>());
    // ...etc

    showAllShapes(shapes);
    return 0;
}
现在如果要正方形，遇到了问题，如果正方形继承矩形，则必须满足width==height
这不好
如果正方形能设置两个边，那么会令人费解 违法最少惊讶原则
删除setWidth setHeight函数会有问题，违反
面向对象的基本原则：派生类不得删除其基类的继承属性

从Rectangle派生出Square违反了面向对象软件设计中的一个重要原则 里氏替换原则 Liskov Substitution Principle LSP
    如果S类型是T类型的一个子类型，并假设q(x)是T类型对象x的一个可证的属性，那么同样的，q(y)应该是S类型对象y的一个可证的属性。
    换句话
        使用基类指针或基类引用的函数，必须在不知道派生类的情况下使用它。
派生类必须完全可替代其基类型。
规则：
基类的前置条件不能在派生类中增强
基类的后置条件不能在派生类中被削弱，后置条件要比父类更严格
基类的所有不变量 都不能通过派生子类更改或违反
历史约束：对象的内部状态，只能通过公共接口封装中的方法调用来改变。由于派生类可能引入基类中不存在的新属性和方法，因此
    这些方法可能允许派生类的对象更改基类中那些不允许被修改的状态。所谓的历史约束就是禁止这一点。
    例如：如果基类被设计为不可变对象的蓝图，则派生类不应该在新引入的成员函数中改变不可变的成员。

为了解决这个问题，使用者必须知道他正在用哪种特定类型。

说明：RTTI 运行时类型信息
一般性概念被称为类型反射 type introspection
typeid运算符<typeinfo>和dynamic_cast<T>都属于RTTI
例如：要在运行时确定对象的类，可以这样
const std::type_info& typeInformationAboutObject = typeid(instance);
类型为std:type_info的const引用现在包含关于对象的类的信息
从C++11开始，hash code已经可以使用
std::type_info::hash_code()
引用相同类型的std::type_info的hash code也是相同的
重要的是，要知道RTTI只适用于那些产生多态的类，它针对至少具有一个虚函数的类，不论是直接定义的还是继承过来的
此外，RTTI可以在某些编译器上打开或关闭，GNU gcc编译器-fno-rtti可以禁用RTTI

例子：RTTI运行时区分
using ShapePtr =std::shared_ptr<Shape>;
using ShapeCollection = std::vector<ShapePtr>;

void resizeAllShape(const ShapeCollection& shapes) {
    try {
        for (const auto& shape : shapes) {
            cosnt auto rawPointerToShape = shape.get();
            if (typeid(*rawPointerToShape) == typeid(Rectangle)) {
                Rectangle* ractangle = dynamic_cast<Ractangle*>(rawPointerToShape);
                rectangle->setEdges(10, 20);
            } else if (typeif(*rawPointerToShape) == typeif(Square)) {
                Square* square = dynamic_cast<Square*>(rawPointerToShape);
                square->setEdge(10);
            } else {
                //...
            }
        }
    } catch (const std::bad_typeid& ex) {
        // Attempted a typeif of NULL pointer!
    }
}
不要这样写，这样不好
这个例子中，面向对象的许多好处例如动态多态性，都会被消除
每当你被迫使用RTTI来区分不同类型，它就是一种design smell 意味着不好的面型对象设计

分析问题：正方形并不是矩形的子类型
具有正方形的形状仅是矩形的特殊状态
这意味着只需要在Rectangle类中添加一个检查器方法来查询它的状态，就可以避免定义一个显式的Square类
根据KISS原则，这个解决方案可能完全满足要求
此外，可以设置方便的setter，设置所有边长相等

class Rectangle : public Shape {
public:
    //...
    void setEdgesToEqualLength(const unsigned int newLength) {
        setEdges(newLength, newLength);
    }

    bool isSquare() const {
        return width == height;
    }
    //...
};

使用组合而不是继承
但是，如果强制要求定义一个显式的Square类，我们该怎么做。
我们永远不应该从Rectangle继承，而应该从Square类继承
为了不违法DRY原则，我们使用Rectangle类的实例作为Square内部实现
class Square : public Shape {
public:
    Square() {
        impl.setEdges(5, 5);
    }

    explicit Square(const unsigned int edgeLength) {
        impl.setEdges(edgeLength, edgeLength);
    }

    void setEdge(const unsigned int length) {
        impl.setEdges(length, length);
    }

    virtual void moveTo(const Point& newCenterPoint) override {
        impl.moveTo(newCenterPoint);
    }

    virtual void show() override {
        impl.show();
    }

    virtual void hide() override {
        impl.hide();
    }

    unsigned long long getArea() const {
        return impl.getArea();
    }
private:
    Rectangle impl;
};
这里moveTo()方法也被重写了，必须将Shape中的方法变成虚方法
从Shape继承的moveTo方法操作基类Shape的centerPoint，而不是操作Rectangle的centerPoint
这是一个小缺点：从基类继承的某些部分是闲置的。

这个解决OO继承问题的方案背后的原理被称为 优先组合而非继承 FCoI
也被称为 优先委托而非继承
继承 白盒复用
组合或委托 黑盒复用
有时候后者更好，因为只能通过定义良好的公共接口来使用它，而不是从这种类型派生出一个字类
通过 组合 委托 而非 继承， 可以降低类与类之间的耦合度
```

```
原则五：接口隔离原则

接口是实现类之间松耦合的一种方法
接口就像契约：类可以通过此契约请求服务，这些服务由实现该契约的其他类提供
但是，如果契约变得过于广泛，即如果接口变得太宽或者肥胖
例子：
class Bird {
public:
    virtual ~Bird() = default;

    virtual void fly() = 0;
    virtual void eat() = 0;
    virtual void run() = 0;
    virtual void tweet() = 0;
};
class Sparrow : public Bird {
public:
    virtual void fly() override{
    }
    virtual void eat() override{
    }
    virtual void run() override{
    }
    virtual void tweet() override{
    }
};
class Penguin : public Bird {
public:
    virtual void fly() override {
        // ??? 企鹅无法飞翔
    }
    // ...
};
接口隔离原则 Interface Segregation Principle ISP
指出接口不应该包含那些与实现无关的成员函数，或者这些类不能以有意义的方式实现
上面例子，Penguin无法为Bird::fly()提供有意义的实现，但却被强制要求覆盖该成员函数
接口隔离原则指出，我们应该将 宽接口 分离成更小且高度内聚的接口
生成的小接口也称为角色接口
class Lifeform {
public:
    virtual void eat() = 0;
    virtual void move() = 0;
};

class Flyable {
public:
    virtual void fly() = 0;
};

class Audible {
public:
    virtual void makeSound() = 0;
};
现在，可以非常灵活地组合这些小的角色接口
class Sparrow : public Lifefrom, public Flyable, public Audible {
    //...
};
class Penguin : public Lifeform, public Audiable {
    // ...
};
```

```
原则六：无环依赖原则

有时两个类互相 认识
Customer 和 Account 相互认识 商店 客户 和 账户 类
实现：
Customer.h

#ifndef CUSTOMER_H
#define CUSTOMER_H

#include "Account.h"

class Customer {
private:
    Account customerAccout;
};

#endif

Account.h

#ifndef ACCOUNT_H
#define ACCOUNT_H

#include "Customer.h"

class Account {
private:
    Customer owner;
};

#endif

导致编译器错误
通过 引用 或者 指针 与 前置声明结合使用，可以避免这些编译器错误

Customer.h

#ifndef CUSTOMER_H
#define CUSTOMER_H

#include "Account.h"

class Customer {
public:
    // ...
    void setAccount(Account* account) {
        customerAccount = account;
    }
private:
    Account* customerAccout;
};

#endif

Account.h

#ifndef ACCOUNT_H
#define ACCOUNT_H

#include "Customer.h"

class Account {
public:
    // ...
    void setOwner(Customer* customer) {
        owner = customer;
    }
private:
    Customer* owner;
};

#endif
虽然编译器错误消失了，但是不好

#include "Account.h"
#include "Customer.h"

//...
Account* account = new Account {};
Customer* customer = new Customer {};
account->setOwner(customer);
customer->setAccount(account);

如果删除了一个，就会野指针，造成严重问题

这个设计问题就是环依赖问题。这是最糟糕的设计
这两个类不能分开，不能彼此独立使用，不能彼此独立测试

不要紧张，环依赖关系 总是 可以打破
以下将说明如何避免 如何打破
```

```
原则七：依赖倒置原则 DIP

接口 是我们处理环依赖问题的好帮手
c++中，使用抽象类 模拟 接口

第一步：不再允许两个类的其中一个直接访问另一个
只允许通过接口进行访问

Owner.h

#ifndef OWNER_H
#define OWNER_H

#include <memory>
#include <string>

class Owner {
public:
    virtual ~Owner() = default;
    virtual std::string getName() const = 0;
};

using OwnerPtr = std::shared_ptr<Owner>;

#endif

Customer.h

#ifndef CUSTOMER_H
#define CUSTOMER_H

#include "Owner.h"
#include "Account.h"

class Customer : public Owner {
public:
    void setAccount(AccountPtr account) {
        customerAccount = account;
    }

    virtual std::string getName() cosnt override {
        // return the Customer's name here...
    }
    // ...
private:
    AccountPtr customerAccount;
    // ...
};

using CustomerPtr = std::shared_ptr<Customer>;

#endif

Account.h

#ifndef ACCOUNT_H
#define ACCOUNT_H

#include "Owner.h"

class Account {
public:
    void setOwner(OwnerPtr owner) {
        this->owner = owner;
    }
    // ...
private:
    OwnerPtr owner;
};

using AccountPtr = std::shared_ptr<Account>;

#endif

现在 在类级别上不再有环依赖
但组件之间的环依赖还未打破
实现这个目标的步骤非常简单：将Owner接口重新定位到另一个组件中

从 CustomerManagement 组件中 移动到 Accounting 中
CustomerManagement 组件包含 Customer 从 Owner继承 引用 Account
Accounting 组件包含 Owner 接口 Account 引用 Owner

组件级别的环依赖就消失了，模块化质量提高，可以独立测试Accounting组件

**这里不太懂，为什么提高了

实际上，两个组件之间的不良依赖关系并没有真正消除
相反，通过引入接口Owner 在类级别上我们甚至多了一个依赖关系
我们帧中做的应该是颠倒这种依赖性

依赖倒置原则 Dependency Inversion Principle DIP
是一种面向对象的设计原则，用于解耦软件模块
该原则指出：面向对象设计的基础 不是 具体软件模块 的特殊属性
相反 它们的共性特征应该被合并在共享使用的抽象体，如接口中
Robert C. Martin 制定了如下原则
A 高级模块不应该依赖于低级模块，两者都应该依赖于抽象。
B 抽象不应该依赖于细节，细节应该依赖于抽象。
注意：术语“高级模块”和“低级模块”不一定是指分层架构中的概念位置，高级模块是
需要来自其他模块提供外部服务的软件模块，而其他模块是所谓的低级模块。
高级模块是 调用操作的模块
低级模块是 其内部功能被高级模块调用执行的模块

依赖倒置原则被认为是一种良好的面向对象设计的基础
它仅通过抽象 如 接口 定义所提供和所需的外部服务，就可以促进可复用软件模块的开发

重新设计Customer 和 Account 之间的 直接依赖关系

组件 CustomerManagement 包含 Customer
Customer 继承于 Owner 引用 Account
组件 Accounting 包含 Owner接口 AnyClass Account
AnyClass 引用 Owner
Account继承于AnyClass

两个组件中的类完全依赖于抽象，因此对于Accounting组件的使用者来说
哪个类需要Owner接口或提供Account接口
已经不再重要

例如，如果现在必须更改或替换Customer类，假设我们想把Accounting放入测试夹具中做组件测试
而在AnyClass类中，我们不需要更改任何内容，反之同样适用
```

```
原则八：迪米特法则

Law of Demeter LoD
最少知识原则

允许成员函数直接调用其所在类作用域内的其他成员函数
允许成员函数直接调用其所在类作用域内的成员变量的成员函数
如果成员函数具有参数，则允许成员函数直接调用这些参数的成员函数
如果成员函数创建了局部对象，则允许成员函数调用这些局部对象的成员函数

如果这四种类型的成员函数中的一种调用返回了一个在结构上比该类的直接相邻元素更远的对象，则应该禁止调用这些对象的成员函数。

class Driver {
public:
    void drive(Car& car) const{
        Engine& engine = car.getEngine();
        FuelPump& fuelPump = engine.getFuelPump();
        fuelPump.pump();
        Ignition& ignition = engine.getIgnition();
        ignition.powerUp();
        Starter& starter = engine.getStarter();
        starter.revolve();
    }
    // ...
};

不好

class Driver {
public:
    void drive(Car& car) const {
        car.start();
    }
};

class Car {
public:
    void start() {
        engine.start();
    }
private:
    Engine engine;
};

class Engien {
public:
    void start() {
        fuelPump.pump();
        ignition.powerUp();
        starter.revolve();
    }
private:
    FuelPump fuelPump;
    Ignition ignition;
    Starter starter;
};

显著减少依赖性
降低了耦合程度，遵从信息隐藏原则 开闭原则
```

```
原则九：避免贫血类

没有功能的类，仅用作一堆数据的存储区
class Customer {
public:
    void setId(const unsigned int id);
    unsigned int getId() const;
    void setForename(const std::string& forename);
    std::string getForename() const;
    void setSurname(const std::string& surname);
    std::string getSurname() const;
    // ... move setters getters
private:
    unsigned int id;
    std::string forename;
    std::string surname;
};

这是只具有数据结构的面向过程编程，与面向对象的思想完全相反
面向对象的思想，要求将数据和操作数据的功能组合成有凝聚力的单元。
```

```
原则十：只说不问

Tell, Don't Ask

和迪米特法则有相似之处
避免公共 get 方法
加强类的封装，增强信息隐藏，首要作用是 增强类的内聚

class Engine {
public:
    void start() {
        if (!fuelPump.isRunning()) {
            fuelPump.powerUp();
            if (fuelPump.getFuelPressuer() < NORMAL_FUEL_PRESSURE) {
                fuelPump.setFuelPressure(NORMAL_FUEL_PRESSURE);
            }
        }
        if (!ignition.isPoweredUp()) {
            ignition.powerUp();
        }
        if (starter.isRotating()) {
            starter.revolve();
        }
        if (engine.hasStarted()) {
            starter.openClutchToEngine();
            starter.stop();
        }
    }
private:
    FuelPump fuelPump;
    Ignition ignition;
    Starter starter;
    start const unsigned int NORMAL_FUEL_PRESSURE { 120 };
};

分支众多 圈复杂度高

class Engine {
public:
    void start() {
        fuelPup.pump();
        ignition.powerUp();
        starter.revolve();
    }
private:
    FuelPump fuelPump;
    Ignition ignition;
    Starter starter;
};

class FuelPump {
public:
    void pump() {
        if (!isRunning) {
            powerUp();
            setNormalFuelPressure();
        }
    }
private:
    void powerUp() {
        //...
    }

    void setNormalFuelPressure() {
        if (pressure != NORMAL_FUEL_PRESSURE) {
            pressure = NORMAL_FUEL_PRESSURE;
        }
    }

    bool isRunning;
    unsigned int pressure;
    static const unsigned int NORMAL_FUEL_PRESSURE { 120 };
};

并非所有getter本质上都是不好的，有时需要从对象获取信息
例如，如果要获取的信息应该显示在图形用户界面上
```

```
原则十一：避免类的静态成员

垃圾商店反模式

工具类容易变成巨大的上帝类
这类工具类通常也包含许多静态成员函数，甚至没有例外
这是弱内聚的标志

静态成员函数偏向于面向过程编程的风格，在面向对象中使用静态调用，使面向对象显得有点荒谬。
在静态成员变量的协助下，在所有类的实例中共享相同的状态本质上不是OOP
它破坏了封装，对象不再完全控制其状态

建议避免使用静态成员变量和静态成员函数

例外：类的私有常量成员，因为是只读的，并且不表示对象的状态
另一个例外是工厂方法，即 创建对象实例的静态成员函数
```

# 第七章

## 函数式编程

```
与面向对象相比，有利的属性

通过避免全局共享可变状态消除了副作用
不可变的数据和对象
函数组合和高阶函数
更好更容易的并行化
易于测试 因为没必要考虑全局可变状态或其他副作用
```

```
函数式编程中的函数，意味着 真正的数学函数
将函数视为 一组输入参数 与 一组输出参数 之间的关系
每组输入参数 仅与 一组输出参数相关
y = f ( x )
对于相同的 x 值， y 值 也总是相同的
这称为 引用透明 referential transparency
引用透明 直接引出了 纯函数 的概念
```

```
pure函数 和 impure函数

一个纯函数
double square(const double value) noexcept {
    return value * value;
};

命令式编程范式 过程化 或 面向对象编程 不能保证无副作用

#include <iostream>

class Clazz {
public:
    int funcitonWithSideEffect(const int value) noexcept {
        return value * value + someKindOfMutualState++;
    }
private:
    int someKindOfMutualState { 0 };
};

int main() {
    Clazz instanceOfClazz {};
    std::cout << instanceOfClazz.functionWithSideEffect(3) << std::endl; // output : 9
    std::cout << instanceOfClazz.functionWithSideEffect(3) << std::endl; // output : 10
    reutrn 0;
}

多线程下 全局状态 或 对象的实例状态 通常很容易出问题

函数式编程一直是C++的一部分 归因于 模板元编程 Template Metaprogramming TMP
非常复杂 是个挑战

模板生成C++代码，其实是函数式编程，并且图灵完备

例子 最大公约是 GCD

#include <iostream>

template< unsigned int x, unsigned int y >
struct GreatestCommonDivisor {
    static const unsigned int result = GreatestCommonDivisor< y, x % y >::result;
};

template< unsigned int x >
struct GreatestCommonDivisor< x, 0 > {
    static const unsigned int result = x;
};

int main() {
    std::cout << "The GCD of 40 and 10 is: " << GreatestCommonDivisor<40u, 10u>::result << std::endl;
    std::cout << "The GCD of 366 and 60 is: " << GreatestCommonDivisor<366u, 60u>::result << std::endl;
}

模板元编程 坏处 可读性 可理解性 差

C++11开始，可以用 constexpr 在编译时计算

constexpr unsigned int greatestCommonDivisor(const int unsigned int x,
                                             const int unsigned int y) noexcept {
    return y == 0 ? x : greatestCommonDivisor(y, x % y);
}
欧几里得算法

C++17中 std::gcd() 已经在标准库中

#include <iostream>
#include <numeric>

int main() {
    constexpr auto result = std::gcd(40, 10);
    std::cout << "The GCD of 40 and 10 is: " << result << std::endl;
    reutrn 0;
}
```

```
仿函数 Functor

从技术上讲， 仿函数 基本上 只是一个 定义了 括号运算符 的类 operator()
实例化之后 就可以像函数一样使用
根据operator()包含 0 1 2 个参数，Functor 分别被称为 生成器 Generator 一元仿函数 二元仿函数

生成器 例子

class IncreasingNumberGenerator {
public:
    int operator()() noexcept { return number++; }
private:
    int number { 0 };
};

int main() {
    IncreasingNumberGenerator numberGenerator {};
    std::cout << numberGenerator() << std::endl;
    std::cout << numberGenerator() << std::endl;
    std::cout << numberGenerator() << std::endl;
    return 0;
}

不要使用原始循环
要用一定数量的递增的值填充 std::vector<T>
可以使用头文件<algorithm> 中定义的 std::generate 这是一个函数模板
将给定Generator对象生成的值分配给某个范围内的每个元素

#include <algorithm>
#include <vector>

using Numbers = std::vector<int>;

int main() {
    const std::size_t AMOUNT_OF_NUMBERS { 100 };
    Numbers numbers(AMOUNT_OF_NUMBRES);
    std::generate(std::begin(numbers), std::end(numbers), IncreasingNumberGenerator());
    return 0;
}

这些仿函数不能满足纯函数的严格要求，调用时会产生副作用

<numberic> 已经包含了函数模板std::iota() 不是生成器仿函数 但可以填充容器，并以一种优雅的方式递增序列的值

Generator类型的仿函数 示例
随机数生成器仿函数模板
基于 Mersenne Twister 算法 (在头文件<random>中) 封装了伪随机数生成器PRNG 的初始化和使用所必须的所有内容

#include <random>

template <typename NUMTYPE>
class RandomNumberGenerator {
public:
    RandomNumberGenerator() {
        mersenneTwisterEngine.seed(randomDevice());
    }

    NUMTYPE operator()() {
        return distribution(mersenneTwisterEngine);
    }

private:
    std::random_device randomDevice;
    std::uniform_int_distribution<NUMTYPE> distribution;
    std::mt19937_64 mersenneTwisterEngine;
};

使用

#include "RandomGenerator.h"
#include <algorithm>
#include <functional>
#include <iostream>
#include <vector>

using Numbers = std::vector<short>;
const std::size_t AMOUNT_OF_NUBMERS { 100 };

Numbers createVectorFilledWithRandomNumbers() {
    RandomNumberGenerator<short> randomNumberGenerator {};
    Numbers randomNumbers(AMOUNT_OF_NUMBERS);
    std::generate(begin(randomNumbers), end(randomNumbers), std::ref(randomNumberGenerator));
    return randomNumbers;
}

void printNumbersOnStdOut(const Numbers& randomNumbers) {
    for (const auto& number : randomNumbers) {
        std::cout << number << std::endl;
    }
}

int main() {
    Numbers randomNumbers = crateVectorFilledWithRandomNumbers();
    printNumbersOnStdOut(randomNumbers);
    return 0;
}


一元仿函数

class ToSquare {
public:
    constexpr int operator()(const int value) const noexcept { return value * value; }
};

#include <algorithm>
#include <vector>

using Numbers = std::vector<int>;

int main() {
    cosnt std::size_t AMOUNT_OF_NUMBERS = 100;
    Numbers numbers(AMOUNT_OF_NUMBERS);
    std::generate(std::begin(numbers), std::end(numbers), IncreasingNumberGenerator());
    std::transform(std::begin(numbers), std::end(numbers), std::begin(numbers), ToSquare());
    return 0;
}

std::transform 将给定的函数或函数对象应用于一个范围，前两个参数定义，并将结果存储在另一个范围，第三个参数定义。
我们的例子中，两个范围相同。


谓词

是一种特殊的仿函数。如果只有一个参数的一元仿函数返回一个布尔值用于指示某些测试的结果为true或false，则该仿函数被称为 一元谓词

class IsAnOddNumber{
public:
    constexpr bool operator()(const int value) const noexcept { return (value % 2) != 0; }
};

该谓词可以应用于我们的数字序列，使用std::remove_if算法删除所有奇数
std::remove_if事实上没有删除任何东西，与谓词不匹配的元素都会被移动到容器末尾，返回一个迭代器，指向要删除的范围的开头位置
std::erase() 真正执行删除
两个一起用，是习惯用法

#include <algorithm>
#include <vector>

using Numbers = std::vector<int>;

int main() {
    const std::size_t AMOUNT_OF_NUMBERS = 100;
    Numbers numbers(AMOUNT_OF_NUMBERS);
    std::generate(std::begin(numbers), std::end(numbers), IncreasingNumberGenerator());
    std::transform(std::begin(numbers), std::end(numbers), std::begin(numbers), ToSquare());
    numbers.erase(std::remove_if(std::begin(numbers), std::end(numbers), IsAnObbNumber()), std::end(numbers));
    return 0;
}

为了能够以 更灵活 更通用的方式 使用仿函数，通常把它们实现为类模板

#include <type_traits>

template <typename INTTYPE>
class IsAnOddNumber {
public:
    static_assert(std::is_integral<INTTYPE>::value,
        "IsAnOddNumber requires an integer type for its template parameter INTTYPE!");

    constexpr bool operator()(const INTTYPE value) const noexcept { return (value % 2) != 0; }
};

从C++11开始，语言提供了 static_assert() 这是编译时执行检查的断言

在main函数内，我们用谓词稍微改造一下
numbers.erase(std::remove_if(std::begin(numbers), std::end(numbers), IsAnOddNumber<Numbers::value_type>()), std::end(numbers));


说明：
TYPE TRAITS
模板是泛型编程的基础
来自C++标准库的容器，迭代器，算法，都是使用C++模板概念实现的，异常灵活的泛型编程的杰出示例。
但是，从技术角度看，使用模板参数实例化模板，只是简单的文本查找和替换过程。

问题是：并非每种数据类型都适合每个模板的实例化
如果将一个数学运算定义为C++仿函数，那么使用std::string便没有意义
C++标准库头文件 <type_traits> 提供了一个全面的类型检查的集合，通过这个集合我们可以检索在编译时作为模板参数传入的类型的信息。
在 type traits 的帮助下， 我们可以让编译器 验证 想要传入模板实例内的那些参数类型
例如：
可以使用type trait std::is_nothrow_copy_constructible<T> 确保用于模板实例化的类型必须是可拷贝构造，不抛出错误且异常安全的
template <typename T>
class Clazz {
    static_assert(std::is_nothrow_copy_constructible<T>::value,
        "The given type for T must be copy-constructible and may not throw!");
};

type traits 不仅可以和 static_asset() 一起使用，利用错误消息中止编译，它们还可以被称为SFINAE的习惯用法所使用 后面会讨论


最重要的 二元仿函数

如果具有布尔类型返回值 称为二元谓词

class IsGreaterOrEqual {
public:
    bool operator()(const auto& value1, const auto& value2) const noexcept {
        return value1 >= value2;
    }
};

在C++11以前，仿函数根据它们的参数数量分别来自模板 std::unary_function 和 std::binary_function 定义在 <functional>
但是C++11中已经弃用了 C++17中已经从标准库删除了
```

```
绑定和函数包装

std::bind
std::function
包含在 <functional> 中

std::bind 是函数及其参数的一个绑定包装器
你可以将实际值 绑定 到函数或函数指针或仿函数的一个或所有参数上
换句话说，可用用现有的函数或仿函数创建新的函数对象

例子： 用std::bind 包装二元仿函数 multiply()

#include <functional>
#include <iostream>

constexpr double multiply(const double multiplicand, const double multiplier) noexcept {
    return multiplicand * multiplier;
}

int main() {
    const auto result1 = multiply(10.0, 5.0);
    auto boundMultiplyFunctor = std::bind(multiply, 10.0, 5.0);
    const auto result2 = boundMultiplyFunctor();

    std::cout << "result1 = " << result1 << " , result2 = " << result2 << std::endl;
    return 0;
}

这样做的目的是什么 绑定功能模板的实际好处是什么

std::bind 允许在编程中使用局部应用 或 偏函数应用
局部应用是一个过程 其中只有一部分函数绑定到值或者变量
而另一部分尚未绑定 未绑定的参数由占位符 _1 _2 _3等替换
这些占位符在 命名空间  std::placeholders 中定义

例子： 局部应用

#include <functional>
#include <iostream>

constexpr double multiply(const double multiplicand, const double multiplier) noexcept {
    return multiplicand * multiplier;
}

int main() {
    using namespace std;:placeholders;

    auto multiplyWith10 = std::bind(multiply, _1, 10.0);
    std::cout << "result = " << multiplyWith10(5.0) << std::endl;
    return 0;
}

局部函数应用 是一种 自适应技术
它允许我们在各种情况下使用函数或者仿函数
我们需要使用它们的功能，但我们只能提供一些而非所有的参数
在占位符的帮助下，函数参数的顺序可以适应客户端代码的期望
例如：multiplicand的位置和参数列表中multiplier的位置可以通过以下方式重新映射
auto multiplyWithExchangedParameterPosition = std::bind(multiply, _2, _1);
这里auto很有用，它是复杂类型的对象
std::_Bind_helper<bool0, double (&)(double, double), const _Placeholder<int2> &, const _Placeholder<int1> &>::type

在极少数情况下，比如使用 std::function 时
这是一个通用的多态函数包装
该模板可以包装任意的可调用对象 普通函数 仿函数 函数指针
并管理用于存储该对象的内存

例如 要将乘法函数multiply() 包装到 std::function 中
std::function<double(double, double)> multiplyFunc = multiply;
auto result = multiplyFunc(10.0, 5.0);

由于C++11和lambda表达式的引入，这些模板 已经很少被用到了
```

```
Lambda 表达式

术语 lambda函数 函数字面量 function literal
有时它们也被称为 闭包 Closure 实际上它是函数式编程的通用术语 这种叫法也不完全正确

闭包
在命令式编程语言中，我们习惯于当执行程序离开变量所在的作用域时，变量便不再可用
在函数式编程中，我们可构建一个闭包 闭包允许部分或全部局部变量的作用域与函数绑定
并且只要该函数存在，该作用域对象将一直存在

在C++中，在lambda表达式捕获列表的帮助下 我们可用创建此类闭包
闭包与lambda表达式不同，正如 面向对象的对象实例 与 其类 不同一样

lambda表达式的特殊之处在于 它们通常是内联实现的  即 在应用时实现的
这有时可以提高代码的可读性，编译器可以更有效地应用其优化策略
当然，lambda函数也可以被视为数据，例如，存储在变量中，或作为函数参数传递给所谓的高阶函数

lambda 表达式的基本结构
[ capture list ](parameter list) -> return_type_declaration { lambda body }

有两个特殊的敌方
没有名字
开头的方括号 称为lambda引入符 lambda introducer
    标记了lambda表达式的开头
    包括捕获列表 capture list 列出了来自外部作用域的所有变量，可以在lambda体内使用
        不管是 值拷贝 还是 引用。 换句话说，这些是lambda表达式的闭包

例子：
[](const double multiplicand, const double multiplier) { return multiplicand * multipier; }

通过将lambda表达式赋值给变量，我们可以创建相应的运行时对象，即所谓的闭包。
实际上这是正确的：编译器从lambda表达式生成一个未指定类型的仿函数类，该表达式在运行时实例化并指定给变量。
捕获列表中捕获的内容转换为仿函数对象的构造函数参数和成员变量。
lambda参数列表中的参数转换为仿函数括号运算符 operator() 中的参数

#include <iostream>

int main() {
    auto multiply = [](const double multiplicand, const double multiplier) {
        return multiplicand * multiplier;
    };
    std::cout << multiply(10.0, 50.0) << std::endl;
    return 0;
}

可以写的更短
std::cout << [](const double multiplicand, const double multiplier) { return multiplicand * multiplier; }(50.0, 10.0) << std::endl;

例子： 将vector中的每个元素都放入尖括号，结果存入另一个vector中

#include <algorithm>
#include <iostream>
#include <string>
#include <vector>

int main() {
    std::vector<std::string> quote { "This's", "one", "small", "step", "for", "a", "man,", "one", "giant", "leap", "for", "mankind." }；
    std::vector<std::string> result;

    std::transform(begin(quote), end(quote), back_inserter(result), [](const std::string& word) { return "<" + word + ">"; };
    std::for_each(begin(result), end(result), [](const std::string& world) { std::cout << word << " "; };

    return 0;
}


通用Lambda表达式 C++14

允许使用auto作为函数或者lambda表达式的返回类型

#include <complex>
#include <iostream>

int main() {
    auto square = [](const auto& value) noexcept { return value * value; };

    const auto result1 = square(12.56);
    const auto result2 = square(25u);
    const auto result3 = square(-6);
    cosnt auto result4 = square(std::complex<double>(4.0, 2.5));

    return 0;
}

编译函数时，参数类型和结果类型 可以根据具体参数 字面值 的类型自动推导出来
非常有用
```

```
高阶函数

高阶函数是将一个或多个其他函数作为参数的函数 或者它们可以返回函数作为结果
在C++中，任何可调用对象，例如std::function 包装的实例 函数指针 从lambda表达式创建的闭包 手动编写的仿函数 以及任何其他实现了operator()的东西都可以作为参数传递给高阶函数
C++标准库中的许多算法都是这种函数，<algorithm>和<numeric>出于不同的目的提供了许多功能强大的高阶函数
也可以自定义高阶函数

例子：

#include <functional>
#include <iostream>
#include <vector>

template<typename CONTAINERTYPE, type UNARYFUNCTIONTYPE>
void myForEach(const CONTAINERTYPE& container, UNARYFUNCTIONTYPE unaryFunction) {
    for (const auto& element : container) {
        unaryFunction(element);
    }
}

templete<typename CONTAINERTYPE, typename UNARYOPERATIONTYPE>
void myTransform(CONTAINERTYPE& container, UNARYOPERATIONTYPE unaryOperator) {
    for (auto& element : container) {
        element = unaryOperator(element);
    }
}

templete<typename NUMBERTYPE>
class ToSquare {
public:
    NUMBERTYPE operator()(const UNMBERTYPE& number) const noexcept {
        return number * number;
    }
};

templete<typename TYPE>
void printOnStdOut(const TYPE& thing) {
    std::cout << thing << ", ";
}

int main() {
    std::vector<int> numbers { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
    myTransform(numbers, ToSquare<int>());
    std::function<void(int)> printNumberOnStdOut = printOnStdOut<int>;
    myForEach(numbers, printNumberOnStdOut);
    return 0;
}


Map Filter Reduce

每种严格的函数式编程语言都必须提供至少三个有用的高阶函数
Map Filter Reduce(fold)
这三个函数构成了一个非常常见的函数式编程设计模式
这些高阶函数也包含在C++标准库中 <algorithm>

Map
std::transform <algorithm>

Filter
std::remove_if <algorithm>

例子 过滤掉回文 aibohphobia

#include <algorithm>
#include <iostream>
#include <string>
#include <vector>

class IsPalindrome {
public:
    bool operator()(const std::string& word) const {
        const auto middleOfWord = begin(word) + word.size() / 2;
        return std::equal(begin(word), middleOfWord, rbegin(word));
    }
};

int main() {
    std::vector<std::string> someWords { "dad", "hello", "radar", "vector", "deleveled", "foo", "bar", "racecar", "ROTOR", "", "C++", "aibohphobia" };
    someWords.erase(std::remove_if(begin(someWords), end(someWords), IsPalindrome()), end(someWords));
    std::for_each(begin(someWords), end(someWords, [](const auto& word) {
        std::cout << word << ",";
    });
    return 0;
}

Reduce (Fold)
std::accumulate <numeric>

例子： 计算总和

#include <numeric>
#include <iostream>
#include <vector>

int main() {
    std::vector<int> numbers { 12, 45, -102, 33, 78, -8, 100, 2017, -110 };

    const int sum = std::accumulate(begin(numbers), end(numbers), 0);
    std::cout << "The sum is: " << sum << std::endl;
    return 0;
}

例子：查找最大数字
int main() {
    std::vector<int> numbers { 12, 45, -102, 33, 78, -8, 100, 2017, -110 };

    const int maxValue = std::accumulate(begin(numbers), end(numbers), 0, [](const int value1, const int value2) {
        return value1 > value2 ? value1 : value2;
    });
    std::cout << "The highest number is: " << maxValue << std::endl;
    return 0;
}


左右侧Fold
函数式编程中关于一列元素的Fold通常由两种方式：左侧Fold和右侧Fold
如果我们将第一个元素与递归组合其余元素的结果组合在一起，则称为右侧Fold
如果我们将递归组合除最后一个元素外的所有元素的结果与最后一个元素相结合，称为左侧Fold
例如，我们要用 + 运算符 对列表取一个和
左侧Fold ((A+B)+C)+D
右侧Fold A+(B+(C+D))
简单的关联 + 操作符时，没有区别，但是在非关联二元函数时，有区别
std::accumulate和普通迭代器，会得到左侧Fold
    std::accumulate(begin, end, init_value, binary_operator)
std::accumate和反向迭代器，会得到右侧Fold
    std::accumulate(rbegin, rend, init_value, init_value, binary_operator)


C++17的fold表达式
C++17中的Fold表达式被实现为可变参数模板 即可以以类型安全的方式获取任意数量的参数的模板
这些任意数量的参数保存在所谓的参数包 parameter pack中
C++17使得在二元运算符的帮助下直接减少参数包中的参数 （即执行Fold）成为可能
C++17 Fold表达式的一般语法如下
( ... operator parampack ) // 左侧Fold
( parampack operator ... ) // 右侧Fold
( initvalue operator ... operator parampack ) // 带有初始值的左侧Fold
( parampack operator ... operator initvalue ) // 带有初始值的右侧Fold

例子： 带有初始值的左侧Fold
#include <iostream>

template<typename... PACK>
int subtractFold(int minuend, PACK... subtrahends) {
    return (minuend - ... - subtrahends);
}

int main() {
    const int result = subtractFold(1000, 55, 12, 333, 1, 12);
    std::cout << "The result is: " << result << std::endl;
    return 0;
}

注意，由于 - 符号缺乏相关性，在这种情况下不能使用右侧Fold，Fold表达式支持32个运算符，包括 == && || 等逻辑运算符

例子：至少包含一个偶数的测试参数包：
#include <iostream>

template <typename... TYPE>
bool containsEvenValue(const TYPE&... argument) {
    return ((argument % 2 == 0) || ...);
}

int main() {
    const bool result1 = containsEventValue(10, 7, 11, 9, 33, 14);
    cosnt bool result2 = containsEventValue(17, 7, 11, 9, 33, 29);

    std::cout << std::boolalpha;
    std::cout << "result1 is " << result1 << "\n";
    std::cout << "result2 is " << result2 << "\n";
    return 0;
}
```

```
整洁的函数式编程代码

以函数式风格编写的代码不够整洁，函数式代码可以做一些不平凡的事情，但它可能很难懂。

例子： 简单的Fold操作
// Build the sum of all product prices
const Money sum = std::accumulate(begin(productPrices), end(productPrices), 0.0);

这段代码没有注释就不懂，需要改进代码，使注释变得多余

改进：

const Money totalPrice = buildSumOfAllPrices(productPrices);


基本概念：无论你使用何种编程风格，良好的软件设计原则仍然适用！

使用面向对象可以很好地解决许多设计上的挑战。多态是面向对象的一大好处，我们可以用 依赖倒置原则，反转源代码和运行时期的依赖性。
使用函数式编程风格可以更好地解决复杂的数学计算。如果必须满足较高的性能和效率要求，这将不可避免地要求某些任务之间的并行化，函数式编程可以
    发挥至关重要的作用。


```

# 第八章 测试驱动开发

```
TDD提供的好处远不止一个简单的验证代码的正确性。
```

```
普通的旧单元测试的缺点

一套单元测试基本上要比没有测试好得多
但是，在许多项目中，单元测试的编写与实现的代码是并行的，有时甚至在完成需要开发的模块之后才编写

这种广泛使用的方法有时也被称为普通旧单元测试 POUT
POUT意味着 代码优先  而不是测试优先
例如：用这种方法，单元测试的代码通常会在代码编写完成后编写。
而且许多开发人员认为这个顺序是唯一的逻辑顺序。

POUT比没有单元测试要好，但有一些缺点
之后没有必要编写单元测试
    一旦一个功能工作，或者看起来能工作，用单元测试改进代码的动机很少。这是没有乐趣的，不如继续做下一件事的诱惑力大
结果代码很难测试
    通常情况，用单元测试改造现有代码不容易。因为原始代码的可测试性不被重视，这会导致紧耦合的代码出现
通过改进的单元测试达到较高的测试覆盖率并不容易
    完成代码后编写单元测试可能会导致漏掉某些问题或者bug
```

```
测试驱动开发的颠覆性

这种方法代表者思考方法的转变
相比于POUT TDD在测试代码没有编写之前不允许编写产品代码
TDD意味着，在编写相关的产品代码之前，一定要先编写测试代码

通过单元测试确定需求
编写一个测试，从而设计公共接口
在测试中编写了一些代码之后，还必须通过编译并提供测试所需要的接口
新编写的单元测试必须失败
我们必须确保测试完全失败，甚至单元测试本身的实现都是错的

编写代码，只确保新的单元测试通过 不要编写超过需求的代码
如果代码需要调用标准库，该考虑如何集成和使用这个库，特别是如何能够使用测试替身替换它

如果现在运行单元测试，并且一切都做对了，测试就会通过。
我们达到了100%的测试覆盖率。
重复这个开发过程，直到满足所有的需求。


这个事实给了我们巨大的力量。有了这个无漏洞的安全测试网，我们可以进行无畏的重构。
代码的坏味道，设计问题现在可以修复了
我们不彼害怕破坏功能，因为定期执行的单元测试会立即给我们反馈。
如果一个或多个测试在重构阶段失败，导致这一结果的代码更改会非常小。

TDD周期 红-绿-重构
红：我们编写的一个失败的测试用例
绿：我们编写产品代码，用于保证前面编写的测试用例通过
重构：删除重复代码以及其他的代码坏味道，重构代码包括产品代码和测试代码。

BOB叔叔和TDD三大原则
除非为了使一个失败的单元测试通过，否则不允许编写任何产品代码
在一个单元测试中，只允许编写刚好能够导致失败的代码-编译错误也算失败
只允许编写刚好能够是一个失败的单元测试通过的产品代码。

Robert C. Martin 即 Uncle Bob认为，严格遵守这三条规则会迫使开发人员在非常短的工作周期内完成工作。
因此，开发人员永远处于一个忙碌的状态。而不是一个代码是正确的，一切工作正常的舒适环境。
```

```
TDD 的一个小例子 Code Kata

Dave Thomas 认为 开发人员应该在小型的，与工作无关的代码库上反复练习。这样他们就可以像音乐家一样出色地完成自己的工作。
开发人员需要不断学习提升自己，为此，他们需要反复练习来实践理论知识，在一次次反馈中提高自身的能力。

代码套路指的是编程时的小练习，就是为了实现这一目的而产生的。
为了提高编程水平，开发人员需要在小练习下实践他们的技能。套路，成为软件工艺运动的一个重要组成部分
例如：IDE的快捷键，学习新的编程语言，关注某些设计原则，实践TDD
http://codekata.com  Dave Thomas 收集的 针对不同用途的套路的目录

罗马数字编码套路：
测试驱动开发套路：将阿拉伯数字转换为罗马数字

罗马数字中包含字母。例如，V代表数字5.
你的任务是使用测试驱动开发方法编写代码。将阿拉伯数字1 - 3999 转换为对应的罗马数字。
罗马数字系统中，数字由拉丁字母组合成：
1 I
5 V
10 X
50 L
100 C
500 D
1000 M
数字是通过将字符组合在一起并将其值相加而形成的。
例如：阿拉伯数字12 用 XII 10 + 1 + 1 表示
2017用 MMXVII 表示
4 9 40 90 400 900 是例外。为了避免四个重复相等字符，采用了不同写法
4 IV
9 IX
40 XL
90 XC
400 CD
900 CM
罗马数字没有0 也没有负数


准备工作
GoogleTest安装 https://github.com/googld/googletest
    新BSD许可下，独立于平台的C++单元测试框架
Git仓库
    能够回滚
考虑如何组织源代码
    建议：首先从一个文件开始，这个文件包含未来所有的单元测试：ArabicToRomanNumeralsConverterTestCase.cpp
    由于TDD逐步指导我们完成软件单元的形成过程，因此稍后再决定是否需要其他文件

对于基本的函数检查，我们编写一个主函数来初始化GoogleTest并运行所有测试，编写一个简单的单元测试PreparationsCompleted
它总是故意失败，示例：

#include <gtest/gtst.h>

int main(int argc, char** argv) {
    testing::InitGoogleTest(&argc, argv);
    return RUN_ALL_TESTS();
}

TEST(ArabicToRomanNumeralsConverterTestCase, PreparationsCompleted) {
    GTEST_FAIL();
}

将 GTEST_FAIL() 替换成 GTEST_SUCCEED() 测试就能通过

下面开始套路：


第一个测试
第一步是决定我们想要实现的第一个小需求。然后我们将为它编写一个失败的测试。
例如，我们决定将从一个阿拉伯数字转换为罗马数字开始：我们想将阿拉伯数字1转换为I
然后，我们将以及存在的虚拟测试转换为实际的单元测试，这样能证明满足这个小需求。
因此，我们还必须考虑转换函数的接口应该是什么样的。

代码：第一个测试
TEST(ArabicToRomanNumberalsConverterTestCase, 1_isConvertedTo_I) {
    ASSERT_EQ("I", convertArabicNumberToRomanNumeral(1));
}

我们将函数设计为：接收的参数为阿拉伯数字，返回的结果为字符串类型。
但是，这段代码在编译时将会报错，因为函数 convertArabicNumberToRomanNumberal()还不存在
让我们记住Bob大叔提出的TDD的三条规则中的第二条：
    在一个单元测试中，只允许编写刚好能够导致失败的代码-编译错误也算失败。
这意味着我们需要暂停测试代码的编写，开始编写产品代码，消除编译错误。
因此，我们要创建转换函数，甚至要将其写入测试用例所在的源代码文件中。当然，我们知道它不会一直在这个源文件中。

std::string convertArabicNumberToRomanNumeral(const unsigned int arabicNumber)
{
	return "";
}

加上函数，编译通过，测试失败。
现在，需要修改函数的实现，使测试通过

std::string convertArabicNumberToRomanNumeral(const unsigned int arabicNumber)
{
	return "I";
}

现在，测试通过了。
我们应当编写通过当前测试的最简单的代码。这是一个渐进的过程，我们才刚刚起步。

提交代码到储存库


第二个测试

增加测试用例
TEST(ArabicToRomanNumeralsConverterTestCase, 2_is_ConvertedTo_II) {
	ASSERT_EQ("II", convertArabicNumberToRomanNumeral(2));
}

测试失败，修改函数

std::string convertArabicNumberToRomanNumeral(const unsigned int arabicNumber)
{
	if (arabicNumber == 2)
		return "II";
	return "I";
}

现在还不需要重构，但是也快了，继续测试


第三个测试

TEST(ArabicToRomanNumeralsConverterTestCase, 3_is_ConvertedTo_III) {
	ASSERT_EQ("III", convertArabicNumberToRomanNumeral(3));
}

测试失败，修改函数

std::string convertArabicNumberToRomanNumeral(const unsigned int arabicNumber)
{
	if (arabicNumber == 3)
		return "III";
	if (arabicNumber == 2)
		return "II";
	return "I";
}

现在是重构的时候了

std::string convertArabicNumberToRomanNumeral(unsigned int arabicNumber)
{
	std::string romanNumeral;
	while (arabicNumber >= 1)
	{
		romanNumeral += "I";
		arabicNumber--;
	}
	return romanNumeral;
}

删除了 arabicNumber 的const 声明
减少了重复代码

继续下一个测试，作者希望先测试10 X
5稍后处理


第四个测试

TEST(ArabicToRomanNumeralsConverterTestCase, 10_is_ConvertedTo_X) {
	ASSERT_EQ("X", convertArabicNumberToRomanNumeral(10));
}

测试失败，修改函数
不要做过多的猜测

std::string convertArabicNumberToRomanNumeral(unsigned int arabicNumber)
{
	if (arabicNumber == 10) {
		return "X";
    } else {
		std::string romanNumeral;
		while (arabicNumber >= 1) {
			romanNumeral += "I";
			arabicNumber--;
		}
		return romanNumeral;
	}
}


第五个测试

TEST(ArabicToRomanNumeralsConverterTestCase, 20_is_ConvertedTo_XX) {
	ASSERT_EQ("XX", convertArabicNumberToRomanNumeral(20));
}

测试失败，修改函数

std::string convertArabicNumberToRomanNumeral(unsigned int arabicNumber)
{
	if (arabicNumber == 10) {
		return "X";
    } else if (arabicNumber == 20) {
		return "XX";
    } else {
		std::string romanNumeral;
		while (arabicNumber >= 1) {
			romanNumeral += "I";
			arabicNumber--;
		}
		return romanNumeral;
	}
}


第六个测试

TEST(ArabicToRomanNumeralsConverterTestCase, 30_is_ConvertedTo_XXX) {
	ASSERT_EQ("XXX", convertArabicNumberToRomanNumeral(30));
}

测试失败，修改函数

std::string convertArabicNumberToRomanNumeral(unsigned int arabicNumber)
{
	if (arabicNumber == 10) {
		return "X";
	}
	else if (arabicNumber == 20) {
		return "XX";
	}
	else if (arabicNumber == 30) {
		return "XXX";
	}
	else {
		std::string romanNumeral;
		while (arabicNumber >= 1) {
			romanNumeral += "I";
			arabicNumber--;
		}
		return romanNumeral;
	}
}

现在代码重构已经刻不容缓了。
代码已经变得相当复杂： 冗余 较高的环复杂度

std::string convertArabicNumberToRomanNumeral(unsigned int arabicNumber)
{
	std::string romanNumeral;
	while (arabicNumber >= 10) {
		romanNumeral += "X";
		arabicNumber -= 10;
	}
	while (arabicNumber >= 1) {
		romanNumeral += "I";
		arabicNumber--;
	}
	return romanNumeral;
}

重构测试代码。
测试代码包含很多重复，因此需要重构，应该设计得更加优雅。
提高 可读性 可维护性。
上面六个测试的验证语句基本相同，解决方案提供一个专用断言 ( 自定义断言 自定义匹配器 )
assertThat(x).isConvertedToRomanNumeral("string");


使用自定义断言进行更复杂的测试

为了实现我们的自定义断言，我们先编写一个失败的单元测试，它和我们之前编写的略有不同

TEST(ArabicToRomanNumeralsConverterTestCase, 33_isConvertedTo_XXXIII) {
    assertThat(33).isConvertedToRomanNumeral("XXXII");
}

首先， 故意错误的期望着 XXXII 来强制失败
然后，编译器不能成功编译这个单元测试，因为 assertThat 没有
isConvertedToRomanNumeral也没有
TDD的第二条规则：
    在一个单元测试中只允许编写刚好能够导致失败的代码，编译错误也算失败。
因此，我们必须首先通过自定义断言，来使编译器完成它的任务
    一个assertThat(<parameter>)函数，返回一个定制断言类的实例
    自定义断言类，它包含真实的断言方法，验证被测试对象的一个或多个属性


class RomanNumeralAssert {
public:
	RomanNumeralAssert() = delete;
	explicit RomanNumeralAssert(const unsigned int arabicNumber) :
		arabicNumberToConvert(arabicNumber) { }
	void isConvertedToRomanNumeral(const std::string& expectedRomanNumeral) const {
		ASSERT_EQ(expectedRomanNumeral, convertArabicNumberToRomanNumeral(arabicNumberToConvert));
	}

private:
	const unsigned int arabicNumberToConvert;
};

RomanNumeralAssert assertThat(const unsigned int arabicNumber) {
	RomanNumeralAssert assert{ arabicNumber };
	return assert;
}

注意：
    断言类中还可以用静态公有类方法 替代非成员函数 assertThat。
    当面临命名空间冲突时，这很有必要。
    例如：相同函数名的冲突。当然，在使用类方法时，命名空间名称必须提前
    RomanNumeralAssert:: assertThat (33).isConvertedToRomanNumeral("XXXIII");


现在，代码可以编译通过，但是执行测试时失败。
更正测试用例，整合所有测试用例到一个断言中。

TEST(ArabicToRomanNumeralsConverterTestCase, conversionOfArabicNumbersToRomanNumerals_Works) {
	assertThat(1).isConvertedToRomanNumeral("I");
	assertThat(2).isConvertedToRomanNumeral("II");
	assertThat(3).isConvertedToRomanNumeral("III");
	assertThat(10).isConvertedToRomanNumeral("X");
	assertThat(20).isConvertedToRomanNumeral("XX");
	assertThat(30).isConvertedToRomanNumeral("XXX");
	assertThat(33).isConvertedToRomanNumeral("XXXIII");
}

继续增加测试用例

	assertThat(100).isConvertedToRomanNumeral("C");
	assertThat(200).isConvertedToRomanNumeral("CC");
	assertThat(300).isConvertedToRomanNumeral("CCC");

为用例增加生成代码

	while (arabicNumber >= 100) {
		romanNumeral += "C";
		arabicNumber -= 100;
	}


再次精简代码

三个while循环看起来非常相似。是时候重构了。

struct ArabicToRomanMapping {
	unsigned int arabicNumber;
	std::string romanNumeral;
};

const std::size_t numberOfMappings = 1;
using ArabicToRomanMappings = std::array<ArabicToRomanMapping, numberOfMappings>;

const ArabicToRomanMappings arabicToRomanMappings = {
	{ 100, "C" }
};


std::string convertArabicNumberToRomanNumeral(unsigned int arabicNumber)
{
	std::string romanNumeral;
	while (arabicNumber >= arabicToRomanMappings[0].arabicNumber) {
		romanNumeral += arabicToRomanMappings[0].romanNumeral;
		arabicNumber -= arabicToRomanMappings[0].arabicNumber;
	}
	while (arabicNumber >= 10) {
		romanNumeral += "X";
		arabicNumber -= 10;
	}
	while (arabicNumber >= 1) {
		romanNumeral += "I";
		arabicNumber--;
	}
	return romanNumeral;
}


一个是可行的，现在用三个

const ArabicToRomanMappings arabicToRomanMappings = { {
	{ 100, "C" },
	{ 10, "X" },
	{ 1, "I" }
} };

std::string convertArabicNumberToRomanNumeral(unsigned int arabicNumber)
{
	std::string romanNumeral;
	for (const auto& mapping : arabicToRomanMappings) {
		while (arabicNumber >= mapping.arabicNumber) {
			romanNumeral += mapping.romanNumeral;
			arabicNumber -= mapping.arabicNumber;
		}
	}
	return romanNumeral;
}


加入新的测试用例

assertThat(1000).isConvertedToRomanNumeral("M");

对应修改代码

const ArabicToRomanMappings arabicToRomanMappings = { {
	{ 1000, "M" },
	{ 100, "C" },
	{ 10, "X" },
	{ 1, "I" }
} };


更多测试用例

	assertThat(2000).isConvertedToRomanNumeral("MM");
	assertThat(3000).isConvertedToRomanNumeral("MMM");
	assertThat(3333).isConvertedToRomanNumeral("MMMCCCXXXIII");

依然生效


加入 5 V 测试用例

	assertThat(5).isConvertedToRomanNumeral("V");

不出所料，失败了
修改代码

const ArabicToRomanMappings arabicToRomanMappings = { {
	{ 1000, "M" },
	{ 100, "C" },
	{ 10, "X" },
	{ 5, "V" },
	{ 1, "I" }
} };

测试通过了


加入新的测试用例

	assertThat(6).isConvertedToRomanNumeral("VI");
	assertThat(37).isConvertedToRomanNumeral("XXXVII");

依然通过


50 L  500 D
没有不同，加入用例和修改代码


处理4 IV

增加用例
	assertThat(4).isConvertedToRomanNumeral("IV");

失败了，修改代码
const ArabicToRomanMappings arabicToRomanMappings = { {
	{ 1000, "M" },
	{ 500, "D" },
	{ 100, "C" },
	{ 50, "L" },
	{ 10, "X" },
	{ 5, "V" },
	{ 4, "IV" },
	{ 1, "I" }
} };
通过了


同样处理 9 IX 40 XL 90 XC


完成了

如果我们的代码片段中所有需求都成功实现了
并且找不到一个新的单元测试来生成新的代码
那么就完成了
https://github.com/clean-cpp/book-samples/


最后，将生产代码与测试代码分离
将生产代码移动到创建的新文件中
```

```
TDD 的优势

开发时逐步完成需求
非常快速的反馈循环
有助于思考接下来该做什么
无间隙的规范 以 可执行代码的形式出现
开发人员能更加自觉和负责地处理依赖关系
100%单元测试覆盖率

TDD不能保证良好的设计，它不是解决设计问题的灵丹妙药
但是有效果，使用TDD很难写出糟糕混乱的代码

什么时候不应该用TDD
简单，粒度小，不复杂的部分，比如
没有函数的单纯的数据类 或 只是将两个模块结合后产生的代码

在TDD中，原型设计是非常困难的任务。
你进入新的领域，并不知道应该采取什么方案。
首先编写单元测试就非常有挑战性，
这种情况下，最好快速写出第一个基本解决方案，并在随后的开发任务中通过改进单元测试来确保质量

另一个挑战：获得一个好的架构
TDD不能取代软件系统的粗粒度结果 （ 子系统， 组件 ） 的必要抉择
面临关于 框架 库 技术 架构模式 的基本决策 TDD没有帮助

推荐书籍
Jeff Langr
Modern C++ Programming with Test-Driven Development
```

# 第九章 设计模式 习惯用法

```
原则：KISS DRY YAGNI 单一职责 开闭原则 信息隐藏
原则是决策基础的基本 真理 或 规律

设计模式是在特定的环境下，为解决具体问题而设计的解决方案

决策和模式为人们提供了解决方案；原则帮助人们设计自己的原则。

不要夸大设计模式的作用。过度使用设计模式，特别是没有好的理由证明是合理的使用设计模式，
可能会带来灾难性的后果，可能遇到过度设计的痛苦
永远记住KISS YAGNI原则
```

```
依赖注入模式
Dependency Injection  DI

依赖注入是敏捷架构的关键元素
能够帮助软件开发人员显著改进软件设计


首先考虑 单例模式

经常用于 日志记录器 数据库连接 中央用户管理
工厂类 经常用单例实现
工具类 实现也经常用单例 工具类本身就是不好的习惯，因为是低内聚的表现

单例模式的使用宗旨：
确保一个类只有唯一的一个实例，并提供对该实例的全局访问。

例子：Effective C++ 中 Scott Meyers 的 单例实现

#ifndef SINGLETON_H_
#define SINGLETON_H_

class Singleton final {
public:
    static Singleton& getInstance() {
        static Single theInstance {};
        return theInstance;
    }

    int doSomething() {
        return 42;
    }
    //...
private:
    Singleton() = default;
    Singleton(const Singleton&) = delete;
    Singleton(Singleton&&) = delete;
    Singleton& operator=(const Singleton&) = delete;
    Singleton& operator=(Singleton&&) = delete;
    // ...
};

#endif

优点，C++11以后，getInstance()中使用一个静态变量构造实例的过程
默认是线程安全的
但是，并不意味着Singleton类中，其他成员函数都是线程安全的
必须由开发人员保证

例子： Singleton类的使用

#include "AnySingletonUser.h"
#include "Singleton.h"
#include <string>

void AnySingletonUser::aMemberFunction() {
    std::string result = Singleton::getInstance().doThis();
}

void AnySingletonUser::anotherMemberFunction() {
    int result = Singleton::getInstance().doThat();
    double value = Singleton::getInstance().doSomethingMore();
}

由于单例的全局可见性和可访问性。其他类可以在任何地方使用单例。这就意味着，
在软件设计中，对单例对象的所有依赖都隐藏在了代码中。
通过检查类的接口，无法看到这些依赖关系

面向对象中，单例的使用就像面向过程中全局变量的使用一样
这对项目中的依赖情况有明显的负面影响
所有使用Singleton的地方都和它紧密耦合在一起
我们失去了利用多态替换实现的可能性，单元测试也无法实现 因为测试替身无法被替换 Mock Object

另一个缺点：
如果由于新的或需求变化而不得不更改，这种更改会触发所有依赖类的一系列更改。

最后，在分布式系统中，很难保证一个类拥有唯一实例。
单例对象很难保证单实例化。并且由它们导致的紧密耦合也存在问题。

替代单例：
只创建一个实例，并且在需要的地方注入它


依赖注入
从根本上讲，依赖注入是一种技术。
在这种技术中，客户端对象需要的独立的服务对象是由外部提供的。
客户端对象不需要关心它所需要的服务对象本身，或者主动请求服务对象，
例如：从工厂或者服务定位器中请求。

依赖注入的含义可以表示为：
将组件与其需要的服务分离，这样组件就不必知道这些服务的名称，也不必知道如何获取它们。

例子：
日志记录器
Accounting CutomerRepository ShoppingCard 依赖于 Logger

#include <string_view>

class Logger final {
public:
    static Logger& getInstance() {
        static Logger theLogger {};
        return theLogger;
    }

    void writeInfoEntry(std::string_view entry) {
        //...
    }
    void writeWarnEntry(std::string_view entry) {
        //...
    }
     void writeErrorEntry(std::string_view entry) {
        //...
     }
};

----------
std::string_view C++17

高性能字符串代理
构造很廉价 因此复制也很廉价

可以作为C风格字符串，字符数组 的 适配器
甚至可以作为来自不同框架的 CString QString 的适配器

CString aStrign("I'm a string object of the MFC type CString");
std::string_view viewOnCString { (LPCTSTR)aString };

如果字符串数据已经被其他对象拥有，需要只读访问，那么它是表示字符串最理想的类
例如：在函数传递字符串常量参数时，不应该再广泛地使用std::string的常量引用。
而应该用std::string_view替换std::string的常量引用
----------

使用Logger单例写日志的多个类中的CustomerRepository类

#include "Customer.h"
#include "Identifier.h"
#include "Logger.h"

class CustomerRepository {
public:
    Customer findCustomerById(const Identifier& customerId) {
        Logger::getInstance().writeInfoEntry("Starting to search for a customer specified by a given unique identifier...");
        //...
    }
    //...
};

为了摆脱单例对象，并且能够在单元测试期间用一个测试替身替换Logger对象
首先我们必须使用依赖倒置原则DIP 这意味着我们必须首先引入一个抽象类 (一个接口)
并使CustomerRepository和具体的Logger都依赖于该接口

CustomerRepository依赖于LoggingFacility
StandardOutputLogger继承自LoggingFacility

新接口

#include <memory>
#include <string_view>

class LoggingFacility {
public:
    virtual ~LoggingFacility() = default;
    virtual void writeInfoEntry(std::string_view entry) = 0;
    virtual void writeWarnEntry(std::string_view entry) = 0;
    virtual void writeErrorEntry(std::string_view entry) = 0;
};

using Logger = std::shared_ptr<LoggeringFacility>;

新日志类

#include "LoggingFacility.h"
#include <iostream>

class StandardOutputLogger : public LoggingFacility {
public:
    virtual void writeInfoEntry(std::string_view entry) override {
        std::cout << "INFO " << entry << std::endl;
    }

    virtual void writeWarnEntry(std::string_view entry) override {
        std::cout << "WARNING " << entry << std::endl;
    }

    virtual void writeErrorEntry(std::string_view entry) override {
        std::cout << "Error " << entry << std::endl;
    }
};

接下来，需要修改CustomerRepository类
    首先，我们创建一个新的Logger类型的智能指针的成员变量
    该指针实例化通过一个初始化构造函数传递到这个类中
    换句话说，我们允许在创建期间把实现LoggingFacility接口的类的实例 注入
    CustomerRepository对象中。
    我们还删除了默认构造函数，因为我们不希望在没有Logger的情况下创建CustomerRepository实例
    此外，我们删除了实现中对单例对象的直接依赖，并且用Logger的智能指针来写日志

#include "Customer.h"
#include "Identifier.h"
#include "LoggingFacility.h"

class CustomerRepository {
public:
    CustomerRepository() = delete;
    explicit CustomerRepository(const Logger& loggingService) : logger { loggingService } {}
    //...

    Customer findCustomerById(const Identifier& customerId) {
        logger->writeInfoEntry("Starting to search for a customer specified by a given unique identifier...");
        //...
    }
    //...
private:
    Logger logger;
};

作为重构的结果，现在我们已经实现了CustomerRepository类不再依赖于特定的日志记录器。
相反，CustomerRepository类只依赖于抽象(接口)，这种抽象在类及其接口中是显式可见的，
因为他由成员变量和构造函数的参数表示。这意味着现在CustomerRepository类接受从外部传入的用于日志记录的服务对象

Logger logger = std::make_shared<StandardOutputLogger>();
CustomerRepository customerRepository { logger };

这种设计变化有积极的影响，能够促进松耦合。

客户端对象 Customerrepository现在可以配置提供日志功能的各种服务对象，
StandardOutputLogger FilesystemLogger 等等

此外，CustomerRepository类的可测试性也得到了显著改进，不再对单例有隐藏的依赖。
现在，可以很容易地用模拟对象mock object替换真正的日志服务
例如，我们可以用spy方法装备模拟对象，以检查单元测试中哪些数据通过LoggingFacility接口离开了
CustomerRepository对象

例子：一个测试替身，用于对依赖于LoggingFacility的类进行单元测试
namespace test {

#include "../src/LoggingFacility.h"
#include <string>

class LoggingFacilityMock : public LoggingFacility {
public:
    virtual void writeInfoEntry(std::string_view entry) override {
        recentlyWrittenLogEntry = entry;
    }

    virtual void writeWarnEntry(std::string_view entry) override {
        recentlyWrittenLogEntry = entry;
    }

    virtual void writeErrorEntry(std::string_view entry) override {
        recentlyWrittenLogEntry = entry;
    }

    std::string_view getRecentlyWrittenLogEntry() const {
        return recentlyWrittenLogEntry;
    }

private:
    std::string recentlyWrittenLogEntry;
};

using MockLogger = std::shared_ptr<LoggingFacilityMock>;

}

在这个单元测试示例中， 你可以看到模拟对象的活动
例子：使用模拟对象进行单元测试的例子
#include "../src/CustomerRepository.h"
#include "LoggingFacilityMock.h"
#include <gtest/gtest.h>

namespace test {
TEST(CustomerTestCase, WrittenLogEntryIsAsExpected) {
    MockLogger logger = std::make_shared<LoggingFacilityMock>();
    CustomerReposity customerRepositoryToTest { logger };
    Identifier customerId { 1234 };

    customerRepositoryToTest.findCustomerById(customerId);

    ASSET_EQ("Starting to search for a customer specified by a given unique identifier...",
        logger->getRecentlyWrittenLogEntry());}
}

上面的例子中，使用依赖注入模式 代替 单例模式。
基本上，一个好的面向对象软件设计应该尽可能地保证所涉及的模块或组件是松耦合的
而依赖注入是实现这一目标的关键
通过一致地使用这种设计模式，软件设计将具有非常灵活的插件体系结构。
对软件测试的影响是，这种技术会产生高度可测试的对象。

从对象本身删除对象创建和关联的功能，并将对象创建和关联的功能集中在基础结构组件中
即所谓的 汇编器 Assembler 或 注入器 Injector
这个组件通常在程序启动时操作，并处理正软件系统的构建计划 (，如配置文件)
也就是说，它按照正确的顺序实例化对象和服务，并将服务注入需要它们的对象中。

换句话说，在设计期间，没有类知道 Assembler类 的存在 ( 这并不完全正确，因为至少有一个软件系统中的其他元素知道
这个Assembler组件的存在，因为组装过程通常在程序的开始由某组件执行)

Assembler组件中的代码:
Logger loggingServiceToInject = std::make_shared<StandardOutputLogger>();
auto customerRepository = std::make_shared<CustomerRepository>(loggingServiceToInject);

这种依赖注入称为 构造函数注入，因为被注入的服务对象作为参数，传递给客户端对象的构造函数。
构造函数注入的优点是，客户端对象在构造过程中被完全初始化，然后就可以立即使用。

但是，如果程序在运行时将服务对象注入客户端对象，例如，如果在程序执行时偶尔创建一个客户端对象，或者在运行时
更好日志记录器Logger，那么，我们应该怎么办呢？
客户端对象必须为注入的服务对象提供setter，示例：

#include "Address.h"
#include "LoggingFacility.h"

class Customer {
public:
    Customer() = default;

    void setLoggingService(const Logger& loggingService) {
        logger = loggingService;
    }
    //...
private:
    Address address;
    Logger logger;
};

这种注入称为setter注入，当然可以将构造函数注入和setter注入结合起来

依赖注入是一种设计模式，它能够使软件松耦合，并且具有很好的可配置性。
可以根据不同客户端或产品的配置文件创建不同的对象。它极大地提高了软件系统的可测试性
因为通过依赖注入技术可以很容易地注入模拟对象。
因此，在设计软件系统时不要忽略这种模式。

在实践中，一些即可用于商业解决方案也可用于开源解决方案的框架，经常使用依赖注入技术。
```

```
Adapter模式
同义词Wrapper
最常用的设计模式之一：原因在于，不兼容接口的适配肯定是软件开发中经常遇到的情况。
例如，如果必须集成由另一个团队开发的模块，或者使用第三方库的情况。

Adapter模式的任务说明：
把一个类的接口转换为客户端期望的另一个接口。Adapter可以让因接口不兼容而无法一起工作的类一起工作。

进一步改造上一节关于依赖注入的例子。假设我们希望使用BoostLog v2进行日志记录，但是，我们也希望能够使用
其他的日志库替换BoostLog v2
解决方案很简单:我们只需要提供LoggingFacility接口的另一个实现，它将BoostLog的接口适配到我们要使用的接口

我们用BoostTrivialLogAdapter类实现接口LoggingFacility的代码：

#include "LoggingFacility.h"
#include <boost/log/trivial.hpp>

class BoostTrivialLogAdapter : public LoggingFacility {
public:
    vritual void writeInfoEntry(std::string_view entry) override {
        BOOST_LOG_TRIVIAL(info) << entry;
    }
    virtual void writeWarnEntry(std::string_view entry) override {
        BOOST_LOG_TRIVIAL(warn) << entry;
    }
    virtual void writeErrorEntry(std::string_view entry) override {
        BOOST_LOG_TRIVIAL(error) << entry;
    }
};

优点时显而易见的，通过Adapter模式，整个软件系统中只有一个类依赖于第三方日志记录系统。
这意味着，我们的代码不会受到日志所有者特有语句的污染，例如BOOST_LOG_TRIVIAL()
因为Adapter类只是LoggingFacility接口的另一个实现，所以我们也可以使用依赖注入
将实例注入想要使用它的所有客户端对象中。

Adapter可以为不兼容的接口提供广泛的适配和转换的可能性。
它的适用范围来源于简单的适配，例如操作名称和数据类型的转换，直到支持整个不同的操作集合。
在上面的例子中，我们把对带有一个字符串参数的成员函数的调用，转换成了对steam的插入操作符的调用。

如果要适配的接口很类似，那么接口适配当然更容易。但如果接口之间相差很大，Adapter的代码实现可能会非常复杂。
```

```
Strategy模式

第六章中描述的开发-封闭OCP原则作为可以扩展的面向对象设计的指导方针，
策略(Strategy)模式是这一重要原则的体现。

该模式的任务说明：
定义一组算法，然后封装每个算法，并使它们可以相互替换。策略模式允许算法独立于使用它的客户端而变化。

在软件设计中，以不同的方式做同一件事情是一个常见的需求，想想列表的排序算法。
不同的排序算法，有不同的时间复杂度和空间复杂度。
例如：冒泡排序，快速排序，归并排序，插入排序和堆排序
冒泡排序是复杂度最低的，也是内存消耗最少的，但是也是最慢的排序算法之一。
相比之下，快速排序是一种快速高效的排序算法，通过递归很容易实现，不需要额外的内存，
但在预先排好序或倒序的列表上效率非常低。
借助策略模式，使用时可以动态地选择不同的排序算法，例如，根据要排序列表的不同特性选择不同的排序算法。

另外一个例子，假设我们希望在任意一个业务IT系统中使用Customer类实例的文本表示。需求指出，
文本表示可以被格式化成多种形式：纯文本格式，XML格式和JSON格式
首先，让我们为格式化策略引入一个抽象，抽象的Formatter类：

Formatter类中包含了所有格式化类的公共的东西

#include <memory>
#include <string>
#include <string_view>
#include <sstream>

class Formatter {
public:
    virtual ~Formatter() = default;

    Formatter& withCustomerId(std::string_view custoemrId) {
        this->customerId = customerId;
        return *this;
    }

    Formatter& withForename(std::string_view forename) {
        this->forename = forename;
        return *this;
    }

    Formatter& withSurname(std::string_view surname) {
        this->surname = surname;
        return *this;
    }

    Formatter& withStreet(std::string_view street) {
        this->street = street;
        return *this;
    }

    Formatter& withZipCode(std::string_view zipCode) {
        this->zipCode = zipCode;
        return *this;
    }

    Formatter& withCity(std::string_view city) {
        this->city = city;
        return *this;
    }

    virtual std::string format() const = 0;

protected:
    std::string customerId { "000000" };
    std::string forename { "n/a" };
    std::string surname { "n/a" };
    std::string street { "n/a" };
    std::string zipCode { "n/a" };
    std::string city { "n/a" };
};

using FormatterPtr = std::unique_ptr<Formatter>;

提供利益相关者请求的格式化样式的三个特定格式化程序如下：

#include "Formatter.h"

class PlainTextFormatter : public Formatter {
public:
    virtual std::string format() const override {
        std::stringstream formattedString {};
        formattedString << "[" << customerId << "]: "
            << forename << " " << supername << ", "
            << street << ", " << zipCode << " "
            << city << ".";
        return formattedString.str();
    }
};

class XmlFormatter : public Formatter {
public:
    virtual std::string format() const override {
        std::stringstream formattedString {};
        formattedString <<
            "<customer id=\"" << customerId << "\">\n" <<
            "  <forename>" << forename << "</forename>\n" <<
            "  <surname>" << surname << "</surname>\n" <<
            "  <street>" << street << "</street>\n" <<
            "  <zipcode>" << zipcode << "</zipcode>\n" <<
            "  <city>" << city << "</city>\n" <<
            "</customer>\n";
        return formattedString.str();
    }
};

class JsonFormatter : public Formatter {
public:
    virtual std::string format() const override {
        std::stringstream formattedString {};
        formattedString <<
            "{\n" <<
            "  \"CustomerId : \"" << customerId << END_OF_PROPERTY <<
            "  \"Forename: \"" << forename << END_OF_PROPERTY <<
            "  \"Surname: \"" << surname << END_OF_PROPERTY <<
            "  \"Street: \"" << street << END_OF_PROPERTY <<
            "  \"ZIP code: \"" << zipCode << END_OF_PROPERTY <<
            "  \"City: \"" << city << "\"\n" <<
            "}\n";
        return formattedString.str();
    }
private:
    static constexpr cosnt char* const END_OF_PROPERTY { "\",\n" };
};

在这里可以清楚地看到，开发-封闭原则OCP得到了非常好的支持。
当需要一个新的输出格式化时，只需要实现Formatter抽象类的一个特殊化即可，
不需要修改现有的格式化程序

如何使用

#include "Address.h"
#include "CustomerId.h"
#include "Formatter.h"

class Customer {
public:
    //...
    std::string getAsFormattedString(const FormatterPtr& formatter) const {
        return formatter->
            withCustomerId(customerId.toString()).
            withForename(forename).
            withSurname(surname).
            withStreet(address.getStreet()).
            withZipCode(address.getZipCodeAsString()).
            withCity(address.getCity()).
            format();
    }
    //...
private:
    CustomerId customerId;
    std::string forename;
    std::string surname;
    Address address;
};

Customer::getAsFormattedString() 成员函数有一个接受指向格式化对象的
unique_ptr指针，这个参数可以用于控制该成员函数返回的字符串格式。
换句话说，Customer::getAsFormattedString()成员函数支持格式化策略

关于Formatter类的公共接口的特殊设计，有许多with...()成员函数连接的接口
这里使用了另一种设计模式，称为Fluent接口
在面向对象编程中，Fluent接口是设计API的一种风格，其代码的可读性与普通的文章类似。
在第8章测试驱动开发中，我们曾经看到过这一的接口，我们引入了一个自定义的断言，
以编写更优雅，可读性更好的测试代码。在我们的例子中，关键在于每个with...()
成员函数都是自引用的，也就是说，调用Formatter类的成员函数的后面的上下文
与前面的上下文是等效的，除非调用了最终的format()函数

Customer引用Formatter接口
PlainTextFormatter JsonFormatter XmlFormatter继承自Formatter接口

策略模式能够保证本例中的Customer::getAsFormattedString()成员函数的调用者可以根据需要配置输出格式
很容易添加另一种输出格式
```

```
Command模式

软件系统接收到指令后，通常需要执行各种各样的操作。
例如：文本处理软件的用户通过用户界面发出各种指令，
他们想打开一个文档，保存一个文档，打印一个文档，复制一段文本，粘贴一段复制的文本等
这种通用的模式在其他领域中也存在，例如，在金融领域中，客户可以向证券交易商发出
购买股票，出售股票等请求。在制造业这样的技术领域，命令被用来控制工业设备和机器。

在实现由命令控制的软件系统时，重要的是保证操作的请求者与实际执行操作的对象分离。
这背后的指导原则是 松耦合原则 和 关注点分离 的原则。

餐馆就是一个很好的类比。
在餐馆中，服务员接收顾客点的菜，但服务员不负责做饭，做饭是厨房的事情。
事实上，对于顾客来说，事物的制作过程是透明的，也许是餐厅准备的食物，也许是食物从其他地方运送过来。

在面向对象的软件开发中，有一种名为 Command (Action) 的行为模式可以促进这种分离。
说明如下：
将请求封装为对象，从而允许你使用不同的请求，队列或日志的请求参数化客户端，或支持可撤销操作。

命令模式的一个很好的例子是Client/Server架构体系，其中client即调用者
发送命令给server即接收者或被调用者 接收并执行命令。

抽象Command类

#include <memory>

class Command {
public:
    virtual ~Command() = default;
    virtual void execute() = 0;
};

using CommandPtr = std::shared_ptr<Command>;

简单的命令 输出字符串

#include <iostream>

class HelloWorldOutputCommand : public Command {
public:
    virtual void execute() override {
        std::cout << "Hello World!" << "\n";
    }
};

接收并执行命令的元素，被称为Receiver
这里，扮演接收者角色的类是Server

#include "Command.h"

class Server {
public:
    void acceptCommand(const CommandPtr& command) {
        command->execute();
    }
};

目前，该类只包含一个可以接收和执行命令的简单公共成员函数。
最后我们需要所谓的Invoker，即在Client/Server架构中的Client类

class Client {
public:
    void run() {
        Server theServer{};
        CommandPtr helloWorldOutputCommand = std::make_shared<HelloWorldOutputCommand>();
        theServer.acceptCommand(helloWorldOutputCommand);
    }
};

main函数

#include "Clinet.h"

int main() {
    Client client {};
    client.run();
    return 0;
}

如果现在编译和执行这个程序，在标准输出控制台就会输出 "Hello World!" 字符串
乍一看，很无趣，但我们通过Command模式实现的是，命令的初始化和发送与命令的执行是分离的。

由于这种设计模式支持开发-封闭原则OCP
添加新的命令非常容易，只需堆现有的代码进行微小的修改即可实现。例如，如果想要强制服务器等待一段时间，
我们可以添加以下代码

#include "Command.h"
#include <chrono>
#include <thread>

class WaitCommand : public Command {
public:
    explicit WaitCommand(const unsigned int durationInMilliseconds) noexcept :
        durationInMilliseconds{durationInMilliseconds}{};
    virtual void execute() override {
        std::chrono::milliseconds dur(durationInMilliseconds);
        std::this_thread::sleep_for(dur);
    }
private:
    unsigned int durationInMilliseconds { 1000 };
};

使用新的WaitCommand类

class Client {
public:
    void run() {
        Server theServer {};
        const unsigned int SERVER_DRLAY_TIMESPAN { 3000 };

        CommandPtr waitCommand = std::make_shared<WaitCommand>(SERVER_DELAY_TIMESPAN);
        theServer.acceptCommand(waitCommand);

        CommandPtr helloWorldOutputCommand = std::make_shared<HelloWorldOutputCommand>();
        theServer.acceptCommand(helloWorldOutputCommand);
    }
};

我们可以使用值参数化命令，由于纯虚execute()成员函数的签名是由Command接口指定为无参数的，因此参数化是在初始化构造函数的帮助下完成的。
此外，我们不需要Server类，因为它可以立即处理和执行新拓展的命令。
Command模式提供了应用程序的多种可能性。
例如：命令可以排队，也支持命令的异步执行:Invoker发送命令然后立即执行其他的操作，发送的命令稍后由Receiver执行。
然而，缺少了一些东西！在上面引用的Command模式的任务声明中，有 支持可撤销操作 的内容
下面专门讨论这个问题
```

```
Command处理器模式

上面的Client/Server体系结构中，作弊了。事实上，服务器不会像那样执行命令，到达服务器的命令对象将被分布到
负责执行命令的服务器的内部。
例如：可以在另一种称为 职责链 的设计模式 的帮助下完成。

考虑稍微复杂的例子，假设我们有一个绘图程序，用户可以用该程序绘制许多不同的形状，例如，圆形和矩形。
为此，可以调用用户界面相应的菜单进行操作。
熟悉的软件开发人员通过 Command设计模式 执行这些绘图操作。
然而，利益相关者指出 用户也可以撤销绘图操作。

为了满足这个需求，首先我们需要有可撤销的命令。

UndoableCommand接口通过组合 Command和Revertable实现

#include <memory>

class Command {
public:
    virtual ~Command() = default;
    virtual voi execute() = 0;
};

class Revertable {
public:
    virtual ~Revertable() = default;
    virtual void undo() = 0;
};

class UndoableCommand : public Command, public Revertable { };

using CommandPtr = std::shared_ptr<UndoableCommand>;

根据接口隔离原则 ISP
我们添加了一个支持撤销功能的Revertable接口
UndoableCommand类同时继承现有的 Command接口 和 新增加的 Revertable接口

具体例子： 画圆

#include "Command.h"
#include "DrawingProcessor.h"
#include "Point.h"

class DrawCircleCommand : public UndoableCommand {
public:
    DrawCircleCommand(DrawingProcessor& receiver, const Point& centerPoint,
        const double radius) noexcept :
        receiver { receiver }, centerPoint { centerPoint }, radius { radius } {}

        virtual void execute() override {
            receiver.drawCircle(centerPoint, radius);
        }

        virtual void undo() override {
            receiver.eraseCircle(centerPoint, radius);
        }

    private:
        DrawingProcessor& receiver;
        const Point centerPoint;
        const double radius;
};

处理绘图操作的DrawingProcessor类
class DrawingProcessor {
public:
    void drawCircle(const Point& centerPoint, const double radius) {
        //...
    }

    void eraseCircle(const Point& centerPoint, const double radius) {
        //...
    }
};

核心部分 CommandProcessor

#include <stack>

class CommandProcessor {
public:
    void execute(const CommandPtr& command) {
        command->execute();
        commandHistory.push(command);
    }

    void undoLastCommand() {
        if (commandHistory.empty()) {
            return;
        }
        commandHistory.top()->undo();
        commandHistory.pop();
    }
private:
    std::stack<std::shared_ptr<Revertable>> commandHistory;
};

CommandProcessor类不是线程安全的，包含了std::stack<T>后进先出 LIFO 数据类型
执行CommandProcessor::execute()后，命令对象存储到stack
调用CommandProcessor::undoLastCommand()时，命令出栈。

现在将可撤销操作建模为命令对象，在这种情况下，命令接收者就是CommandProcessor本身

UndoCommand类为 CommandProcessor提供撤销操作

#include "Command.h"
#include "CommandProcessor.h"

class UndoCommand : public UndoableCommand {
public:
    explicit UndoCommand(CommandProcessor& receiver) noexcept :
        receiver { receiver } {}

    virtual void execute() override {
        receiver.undoLastCommand();
    }

    virtual void undo() override {
        // Intentionally left blank, because an undo should not be undone.
    }

private:
    CommandProcessor& receiver;
};

在实际使用Command模式时，常常需要能够从几个简单的命令组合成一个更复杂的命令，
或者记录和回放命令(脚本) 为了能够更方便地实现这些需求，下面的设计模式比较合适
```

```
Composite模式

计算机结构中广泛使用的数据结构是树结构，到处都可以找到树结构。
例如，数据媒体，比如硬盘 上的文件系统，它的分层组织就符合树的结构。
集成开发环境IDE的项目浏览器通常也具有树结构。在编译器设计中，
用到一种叫做 抽象语法树 AST 的方法，顾名思义，它是指以树状结构表示源代码的抽象语法结构。
抽象语法树通常是编译器在语法分析阶段的结果。

对树状数据结构的面向对象蓝图被称为组合模式。
该模式的任务说明如下：
将对象组合成树结构来表示 “部分-整体” 的层次结构。
组合允许客户端统一处理单个对象和对象的组合。

我们在Command和Command处理器中的示例可以扩展为符合Command，
并且Command还可以记录和重放。所以我们在之前的设计中添加了一个新类，
一个CompositeCommand:

#include "Command.h"
#include <vector>

class CompositeCommand : public UndoableCommand {
public:
    void addCommand(CommandPtr& command) {
        commands.push_back(command);
    }

    virtual void execute() override {
        for (const auto& command : commands) {
            command->execute();
        }
    }

    virtual void undo() override {
        for (const auto& command : commands) {
            command->undo();
        }
    }

private:
    std::vector<CommandPtr> commands;
};

CompositeCommand有一个成员函数addCommand() 它允许你将命令添加到
CompositeCommand的实例。由于CompositeCommand类也实现了UndoableCommand接口
因此可以将其实例视为普通的command
换句话说，我们可以以其他的CompositeCommand来分层地组合出一个新的CompositeCommand
通过Composite模式的递归结构，你可以生成command树

现在可以使用新添加的类 CompositeCommand 作为宏录制器
以便记录和重放command序列

int main() {
    CommandProcessor commandProcessor {};
    DrawingProcessor drawingProcessor {};

    auto macroRecorder = std::make_shared<CompositeCommand>();

    Point circleCenterPoint { 20, 20 };
    CommandPtr drawCircleCommand = std::make_shared<DrawCircleCommand>(drawingProcessor, circleCenterPoint, 10);
    commandProcessor.execute(drawCircleCommand);
    macroRecorder->addCommand(drawCircleCommand);

    Point rectangleCenterPoint { 30, 10 };
    CommandPtr drawRectangleCommand = std::make_shared<DrawRectangleCommand>(drawingProcessor, rectangleCenterPoint, 5, 8);
    commandProcessor.execute(drawRectangleCommand);
    macroRecorder->addCommand(drawRectangleCommand);
    commandProcessor.execute(macroRecorder);

    CommandPtr undoCommand = std::make_shared<UndoCommand>(commandProcessor);
    commandProcessor.execute(undoCommand);

    return 0;
}

在Composite模式的帮助下，现在我们很容易把简单的command组装成复杂的command序列 command被称为叶子
由于CompositeCommand还实现了UndoableCommand接口，因此它们可以像简单的command一样被使用。
这极大地简化了客户端的代码。

小缺点，只有在使用类CompositeCommand的具体实例 macroRecorder时才能访问成员函数
CompositeCommand::addCommand() 而通过UndoableCommand接口无法使用该成员函数。
换句话说，这里的组合类和叶子的地位并不平等。

如果看通用的 Composite模式，会看到管理子元素的函数是在抽象中声明的。
于是，在我们的例子中，这意味着我们必须在接口 UndoableCommand中声明一个 addCommand()
这将违反 ISP
这一致命的后果是，叶子元素必须覆盖addCommand() 并且必须为这个成员函数提供有意义的实现
但这是不可能的！
如果我们向 DrawCircleCommand的实例添加一个command 假使不违反 最少惊讶原则 会出什么问题呢？
如果这样做，将违反 里氏替换原则 LSP 因此，对案例进行权衡 并且区别对待组合类和叶子是比较好的选择。
```

```
Observer模式

观察者模式
一种众所周知的，用于构建软件系统体系结构的模式是 模型-视图-控制器
Model-View-Controller MVC 借助于这种体系结构模式，通常应用程序的部分是结构化的。
它背后的原理是关注点分离 Separation of Concerns SoC
除此之外，在模型中，持有要显式数据的模型与这些数据的显示(即所谓的视图)被分隔开了。

在MVC中，视图和模型之间的耦合应该尽可能松散。这种松耦合通常用Observer模式实现。
描述：
定义对象之间一对多的依赖关系，以便在一个对象更改状态时，自动通知并更新其所有的依赖关系。

像往常一样，我们可以使用示例来更好地解释模式。考虑一个电子表格应用程序，它是许多办公软件套件的
自然组成部分。在这样的应用程序中，数据可以显示在工作表，饼状图图形和许多其他表示形式中，即所谓的视图。
我们可以创建关于数据的不同视图，也可以关闭它们。

首先，我们需要视图的一个名为Observer的抽象元素。

#include <memory>

class Observer {
public:
    virtual ~Observer() = default;
    virtual int getId() = 0;
    virtual void update() = 0;
};

using ObserverPtr = std::shared_ptr<Observer>;

Observer用于观察所谓的Subject 为此，它们可以在Subject上注册，也可以注销。

代码：可以在Subject中添加和删除Observer

#include "Observer.h"
#include <algorithm>
#include <vector>

class IsEqualTo final {
public:
    explicit IsEqualTo(const ObserverPtr& observer) :
        observer { observer } {}
    bool operator()(const ObserverPtr& observerToCompare) {
        return observerToCompare->getId() == observer->getId();
    }

private:
    ObserverPtr observer;
};


class Subject {
public:
    void addObserver(ObserverPtr& observerToAdd) {
        auto iter = std::find_if(begin(observers), end(observers),
            IsEqualTo(observerToAdd));
        if (iter == end(observers)) {
            observers.push_back(observerToAdd);
        }
    }

    void removeObserver(ObserverPtr& observerToRemove) {
        observer.erase(std::remove_if(begin(observers), end(observers),
            IsEqualTo(observerToRemove)), end(observers));
    }

protected:
    void notifyAllObservers() const {
        for (const auto& observer : observers) {
            observer->update();
        }
    }

private:
    std::vector<ObserverPtr> observers;
};

除了 Subject类， 还定义了一个 IsEqualTo 仿函数，
该模式的核心是 notifyAllObservers() 成员函数，
因为是被继承自 Subject 的具体字类调用的，因此被设置为 protected成员
该函数迭代所有 已注册的Observer实例，并调用其 update()成员函数

具体的Subject类

#include "Subject.h"
#include <iostream>
#include <string_view>

class SpreadsheetModel : public Subject {
public:
    void changeCellValue(std::string_view column, const int row, const double value) {
        std::cout << "Cell [" << column << ", " << row << "] = " << value << std::endl;
        // Change value of a spreadsheet cell, and then...
        notifyAllObservers();
    }
};

这是最小化的实现
下面是实现了抽象Observer接口的三个具体的视图

#include "Observer.h"
#include "SpreadsheetModel.h"

class TableView : public Observer {
public:
    explicit TableView(SpreadsheetModel& theModel) :
        model { theModel } {}
    virtual int getId() override {
        return 1;
    }
    virtual void update() override {
        std::cout << "Update of TableView." << std::endl;
    }
private:
    SpreadsheetModel& model;
};

class BarChartView : public Observer {
public:
    explicit BarChartView(SpreadsheetModel& theModel) : 
        model { theModel } {}
    virtual int getId() override {
        return 2;
    }

    virtual void Update() override {
        std::cout << "Update of BarChartView." << std::endl;
    }
private:
    SpreadsheetModel& model;
};

class PieChartView : public Observer {
public:
    explicit PieChartView(SpreadsheetModel& theModel) :
        model { theModel } {}
    virtual int getId() override {
        return 3;
    }

    virtual void update() override {
        std::cout << "Update of PieChartView." << std::endl;
    }
private:
    SpreadsheetModel& model;
};

使用

#include "SpreadsheetModel.h"
#include "SpreadsheetViews.h"

int main() {
    SpreadsheetModel spreadsheetModel {};

    ObserverPtr observer1 = std::make_shared<TableView>(spreadsheetModel);
    spreadsheetModel.addObserver(observer1);

    ObserverPtr observer2 = std::make_shared<BarChartView>(spreadsheetModel);
    spreadsheetModel.addObserver(observer2);

    spreadsheetModel.changeCellValue("A", 1, 42);

    spreadsheetModel.removeObserver(observer1);

    spreaadsheetModel.changeCellValue("B", 2, 23.1);

    ObserverPtr observer3 = std::make_shared<PieChartView>(spreadsheetModel);
    spreadsheetModel.addObserver(observer3);

    spreadsheetModel.changeCellValue("C", 3, 3.1415926);

    return 0;
}

除了松耦合这一 积极特征 具体的Subject对Observer一无所知
该模式还很好地支持了开闭原则。
在无须调整或更改现有类的任何内容的前提下，我们可以非常轻松地添加新的具体的Observer
```

```
Factory模式

根据关注点分离原则，对象创建应该与对象在特定领域内的任务分开。
上面讨论的依赖注入模式就是以最直接的方式遵循了这一原则，因为整个对象的创建过程集中在
基础元素中，并且对象不必关心这些。

但是，如果需要在运行时的某个时刻动态创建对象，我们该怎么办？这个任务可以由对象工厂来接管
Factory设计模式基本上相对简单，并且以很多不同的形式和种类出现在代码库中。
除了遵循SoC原则外，它还严格遵循信息隐藏原则，因为实例的创建过程被隐藏在其用户之外。

正如前面说过的那样，Factory模式可以有无数的形式和变体。
我们只讨论一个简单的变体。

简单Factory

#include "LoggingFacility.h"
#include "StandardOutputLogger.h"

class LoggerFactory {
public:
    static Logger crate() {
        return std::make_shared<StandardOutputLogger>();
    }
};

使用

#include "LoggerFactory.h"

int main() {
    Logger logger = LoggerFactory::create();
    // ...
    return 0;
}

更复杂的LoggerFactory 读取配置文件内容，根据配置创建特定Logger

#include "LoggingFacility.h"
#include "StandardOutputLogger.h"
#include "FilesystemLogger.h"

#include <fstream>
#include <string>
#include <string_view>

class LoggerFactory {
private:
    enum class OutputTarget : int {
        STDOUT,
        FILE
    };

public:
    explicit LoggerFactory(std::string_view configurationFileName) :
        configurationFileName { configurationFileName } {}

    Logger create() const {
        const std::string configurationFileContent = readConfigurationFile();
        OutputTarget outputTarget = evaluateConfiguration(configurationFileContent);
        return createLogger(outputTarget);
    }

private:
    std::string readConfigurationFile() const {
        std::ifstream filesystem(configurationFileName);
        return std::string(std::istreambuf_iterator<char>(filestream),
            std::istreambuf_iterator<char>()); }
    OutputTarget evaluateConfiguration(std::string_view configurationFileContent) const {
        //...
        return OutputTarget::STDOUT;
    }

    Logger createLogger(OutputTarget outputTarget) const {
        switch(outputTarget) {
            case OutputTarget::FILE:
                return std::make_shared<FilesystemLogger>();
            case OutputTarget::STDOUT:
            default:
                return std::make_shared<StandardOutputLogger>();
        }
    }

    const std::string configurationFileName;
};

工厂模式与依赖注入的区别：尽管CustomerRepository类和Assembler没有依赖关系，但Customer在使用
Factory模式时，知道factory类的存在。
这种依赖性并不是一个严重的问题，但它再次清楚地表明，使用依赖注入可以最大地实现松散耦合。
```

```
Facade模式

Facade模式是一种结构型设计模式，它通常被用到架构级别
其模式意图如下：
为子系统中的一组接口提供统一的接口。Facade定义了一个更高级的接口，使得子系统更容易使用。

根据关注点分离原则 单一职责原则 信息隐藏原则
构建大型软件系统通常会产生一些更大的组件或模块
通常这些组件或模块有时可称为 子系统
即使在分层体系结构中，我们也可以将各个层视为子系统

为了增强软件的封装性，软件的组件或子系统的内部结构对客户来说应该是隐藏的。
应该尽量减少子系统之间的通信，以及它们之间的依赖关系。如果子系统的客户必须知道其内部结构及其
各部分之间相互作用的详细信息，那么软件系统的设计问题将是致命的。

Facade模式通过为客户端提供定义明确且简单的接口来规范对复杂子系统的访问。
任何对子系统的访问都必须通过Facade完成

例子：名为Billing的子系统，用于处理账单。它内部结构由几个相互关联的部分组成。子系统的客户端无法直接访问这些部件。
它们必须使用Facade的BillingService作为客户端的访问点

Facade并不特别，它通常只是一个简单的类，在公共接口上接收请求，并将请求转发到子系统的内部结构中。
有时Facade只是简单地转发一个调用子系统内部结构元素的请求，但偶尔它也会执行数据转换，并且它也是一个Adapter

Facade类BillingService实现了两个接口 接口配置 和 账单生成
根据接口隔离原则 ISP
```

```
Money Class 模式

尽管高精度的数值有时候非常重要，你还是应该避免使用浮点数
float double或者long double类型的浮点变量在简单加法中都会出问题

例子：加上十个浮点数，结果不准确

#include <assert.h>
#include <iostream>

int main() {
    double sum = 0.0;
    double addend = 0.3;

    for (int i = 0; i < 10; i++) {
        sum = sum + addend;
    };

    assert(sum == 3.0);
    return 0;
}

显而易见的断言失败。浮点数在计算机内部以二进制格式保存，不可能将0.3或者其他的值精确地存储在float
double long double类型的变量中。因为没有二进制的有限长度的精确表示。
十进制中，也有类似的问题，我们不能仅用十进制表示1/3，0.333333333又不完全准确。

这一问题有几种解决方案。对于货币，我们可以用合适的方法将货币值存储在具有所需精度的整数中。
例如：12.45美元 存储为1245 如果要求不是很高，则整数是可行的解决方案，请注意，C++标准没有指定
整型的大小，因此，必须小心非常大的数字，因为可能发生溢出。如果有顾虑，应该使用64位的整数， 
因为它可以容纳非常大的金额数目。

--------------------------------------------------
在头文件 <limits> 中 我们可以找到表示算数类型 整数或
浮点数 实际范围具体大小的类模板
例如：int的最大表示范围

#include <limits>
constexpr auto INT_LOWER_BOUND = std::numeric_limits<int>::min();
constexpr auto INT_UPPER_BOUND = std::numeric_limits<int>::max();
--------------------------------------------------

另一种流行的解决方案是提供一个特殊的类，即所谓的Money类
描述：
提供一个类来表示确切的金额。Money类处理不同的货币和它们之间的转换。

------------------
Money
------------------
-amount:Integer
-currency:Currency
------------------
+Money()
+Money(other:Money)
+Money(amount:Integer,currency:Currency)
+operator=(other:Money):Money
+operator=(newAmount:Integer):Money
+operator==(other:Money):Boolean
+operator+(addend:Money):Money
+operator-(subtrahend:Money):Money
+operator*(multiplier:Integer):Money
+getAsFloatingPointValue():double
+getAsPrintableString():String
------------------

Money Class模式基本上是一个封装金融金额以及货币的类，而处理金钱只是这类问题中的一个例子。
还有许多其他必须被准备表示的属性或度量，例如，物理中的精确测量
时间 电压 电流 距离 质量 频率 物质量等

虽然在许多商业IT系统中准确处理金额是一种非常常见的情况，但在大多数主流C++
基类库中你是找不到Money类的。但是不要重新造轮子！
有很多不同的C++ Money类实现。通常，一种实现并不能满足所有的要求，关键是你要了解问题所在。
在选择或设计Money类时，你可以考虑几个约束和要求：
要处理的全部值的范围 最小值 最大值 是多少
哪些舍入规则是适用的？某些国家、地区有针对舍入的法规或惯例
是否有准确性的法律要求
必须考虑哪些标准，例如ISO4217国际货币代码标准
如何将值显示给用户
转换的频率如何

对Money类进行100%的单元测试覆盖是绝对必要的，以检查该类所在的所有情况下是否都按照预期正常工作
当然，与用整数表示纯数字相比，Money类有一个小缺点：性能差 这可能在某些系统中是一个问题
但是在大多数情况下，它的优势占主导。 不要过早优化
```

```
特例模式

第4章说过，返回nullptr是不好的，应该避免。
第5章 异常即异常 异常之应该用于真正的异常情况，而不是应该用于控制正常程序流程

现在一个开发且有趣的问题：如何在不使用nullptr或其他特殊值的情况下，处理那些并不是真正出现了
异常 如内存分配失败 的特殊情况

示例 按名称查找客户
Customer CustomerService::findCustomerByName(const std::string& name) {
    // Code that searches the customer by name...
    // ...but what shall we do, if a customer with the given name does not exist?!
}

一种可能的情况是始终返回列表而不是单个实例，如果返回的列表是空，则表示要查询的业务对象不存在：

nullptr的替代法，如果查找失败，则返回空列表

#include "Customer.h"
#include <vector>

using CustomerList = std::vector<Customer>;

CustomerList CustomerService::findCustomerByName(const std::string& name) {
    // Code that searches the customer by name...
    // ...and if a customer with the given name does not exist:
    return CustomerList();
}

现在，可以在程序序列中查询返回的列表是否为空。但是什么情况会产生一个空列表？是否有错误导致列表为空？
成员函数std::vector<T>::empty()并不能回答这个问题。
列表为空是列表的一种状态，但该状态没有针对特定域的语义。

毫无疑问，这个解决方案比返回nullptr要好得多，但在某些情况下它可能还不够好。
更人性化的设计是程序返回一个可以查询内部问题的返回值，以及可以用这一结果做些什么。答案是 特例模式

描述：
为特定情况提供特殊行为的字类。

特例模式背后的思想是利用多态的优势，并且提供表示特殊情况的类，而不是返回nullptr或其他一些特殊的值。
这些特殊的类具有与调用者期望的 普通 类相同的接口。 Customer 被 NotFoundCustomer 继承

在C++源代码中，Customer类的实现和表示特殊情况的NotFoundCustomer类看起来像下面这样

#ifndef CUSTOMER_H_
#define CUSTOMER_H_

#include "Address.h"
#include "CustomerId.h"
#include <memory>
#include <string>

class Customer {
public:
    //...more member functions here...
    virtual ~Customer() = default;

    virtual bool isPersistable() const noexcept {
        return (customerId.isValid() && ! forename.empty() && ! surname.empty() &&
            billingAddress->isValid() && shippingAddress->isValid());
    }

private:
    CustomerId customerId;
    std::string forename;
    std::string surname;
    std::shared_ptr<Address> billingAddress;
    std::shared_ptr<Address> shippingAddress;
};

class NotFoundCustomer final : public Customer {
public:
    virtual bool isPresistable() const noexcept override {
        return false;
    }
};

using CustomerPtr = std::unique_ptr<Customer>;

#endif /* CUSTOMER_H_ */

现在我们可以使用表示特殊情况的对象，就好像它们是Customer类的普通实例一样。
即使在程序的不同部分之间传递对象时，空检查也是多余的，因为总有一个有效的对象。
就好像NotFoundCustomer对象是Customer的一个实例，使用它可以完成许多事情。
if (customer.isPersistable()) {
    //...持久化
}
虽然一直会返回false，但是比空检查更有意义。

--------------------------------------------------
std::optional<T> C++17

从C++17开始， 还有另一个有趣的替代方法被用于可能缺少的结果或值
std::optional<T> 定义在 <optional>
此类模板的实例表示 可选的包含值
即 可能存在也可能不存在的值

#include "Customer.h"
#include <optional>
using OptionalCustomer = std::optional<Customer>;

现在我们的搜索函数 CustomerService::findCustomerByName()可以按照如下方式实现
class CustomerRepository {
public:
    OptionalCustomer findCustomerByName(const std::stirng& name) {
        if (/*the search was successful*/) {
        return Customer();
        } else {
            return {};
        }
    }
};

在函数被调用的地方，现在你有两种方式来处理返回值

int main() {
    CustomerRepository repository {};
    auto optionalCustomer = repository.findCustomerByName("John Doe");

    // Option 1: Catch an exception, if 'optionalCustomer' is empty
    try {
        auto customer = optionalCustomer.value();
    } catch (std::bad_optional_access& ex) {
        std::cerr << ex.what() << std::endl;
    }

    // Option2: Provide a substitute for a possibly missing object
    auto customer = optionalCustomer.value_or(NotFuoundCustomer());

    return 0;
}

如果对象的缺少是非预期的情况，使用 Option1
确定是发生了严重错误

如果缺少对象是正常的，推荐使用Option2
--------------------------------------------------
```

```
什么是习惯用法

在特定的编程语言 或技术中 解决问题的一种特殊的模式
与一般的设计模式不同，习惯用法的适用性有限
仅限于特定的编程语言或特定的技术 如框架

如果必须在较低的抽象级别上解决编程问题，则通常在详细设计和实现阶段
使用习惯用法。在C和C++领域，有一个众所周知的习惯用法 Include Guard
有时也被称为 Macro Guard 或者 Header Guard
用于避免同一头文件的重复包含

#ifndef FILENAME_H_
#define FILENAME_H_

// ...content of header file...

#endif

这个习惯的一个缺点，必须确保文件名和 Include-Guard宏的命名一致，
因此，现在大多数C和C++编译器都支持非标准的
#pragma once预处理指令，把该指令放在头文件的顶部
编译器就会确定在预处理阶段只包含这个头文件一次

顺便说一下，我们已经知道了一些习惯用法， 申请资源即初始化 RAII
Erase-Remove


一些有用的C++习惯用法
可以在网上找到很多C++的习惯用法，但是不是所有习惯用法都有利于现代的，整洁的代码
它们有时非常复杂且难以理解，比如 Algebraic Hierarchy
即使熟练的C++开发人员也是如此。
随着C++11以及后续标准的发布，一些习惯用法以及过时了
```

```
不可变的类

一旦创建就不能改变其状态的对象，也就是不变类
例如：不可变对象可以用作哈希数据结构中 键值对的 键
因为键值对中的键一旦被创建就不会再改变
另一个例子：在其他几种编程语言中的字符串，如C#和Java中的字符串，也是不可改变的

好处：
不可变对象默认是线程安全的。
不变性使编写，使用，理解代码变得更加容易

要在C++中创建不可变类，必须采取以下措施
类的成员变量必须是不可变的，
    必须是const，只能使用构造函数的初始化列表，在构造函数中初始化一次。
操作方法不改变调用者的状态，而是返回改变状态后的类的新实例，原始对象没有发生改变。
    不应该有setter方法，不可变对象没有任何设置的方法。
这个类应该被标记为final
    这不是一个严格的规则，但是，如果一个新的类可以从一个不可变的类继承，那么有可能会改变基类的不可变性

例子：

#include "Identifier.h"
#include "Money.h"
#include <string>
#include <string_view>

class Employee final {
public:
    Employee(std::string_view forename,
        std::string_view surname,
        const Identifier& staffNumber,
        const Money& salary) noexcept :
        forename { forename },
        surname { surname },
        staffNumber { staffNumber },
        salary { salary } {}

    Identifier getStaffNumber() const noexcpt {
        return staffNumber;
    }

    Money getSalary() const noexcept {
        return salary;
    }

    Employee changeSalary(const Money& newSalary) const noexcept {
        return Employee(forename, surname, staffNumber, newSalary);
    }

private:
    const std::string forename;
    const std::string surname;
    const Identifier staffNumber;
    const Money salary;
};
```

```
匹配失败不是错误

实际上，匹配失败不是错误 Substitution Failure is not an Error  SFINAE
不是真正的习惯用法，而是C++编译器的一个特性

它已经成为C++98标准的一部分，在C++11中又增加了几个新特性，但它经常被以非常惯用的方式使用。
特别是在模板库中，例如C++标准库或者Boost，所以它仍被称为习惯用法

声明：
如果替换导致无效的类型或表达式，则类型推导失败。如果用替换的参数来编写无效的类型或表达式，则其格式是不正确的。
只有函数类型及其模板参数类型的上下文中的无效类型和表达式才会导致推导失败。

完全读不懂!!

在C++模板实例化错误的情况下，例如，使用错误的模板参数，错误消息可能非常冗长且含糊不清
SFINAE是一种编程技术，可确保模板参数的匹配失败不会产生烦人的编译错误。

简单地说，这意味着如果模板参数的匹配失败了，编译器将继续搜索合适的模板，而不是出错中断。

下面是一个带有两个重载函数模板的非常简单的示例：

#include <iostream>

template <typename T>
void print(typename T::type) {
    std::cout << "Calling prinet(typename T::type)" << std::endl;
}

template <typename T>
void print(T) {
    std::cout << "Calling print(T)" << std::endl;
}

struct AStruct {
    using type = int;
};

int mian() {
    print<AStruct>(42);
    print<int>(42);
    print(42);

    return 0;
}

输出：
Calling print(typename T::type)
Calling print(T)
Calling print(T)


在C++11之前，SFINAE有几个缺点，导致非常冗长和棘手的代码，这些代码难以理解。标准化程度很高，有时甚至针对特定编译器。
C++11引入了 Type Traits库， 特别是 元函数 std::enable_if() <type_traits> 现在在SFINAE中发挥着核心作用
使用该函数，我们可以根据类型特征，从重载的候选函数中有条件地 筛选函数功能
换句话说， 我们可以根据参数的类型选择一个函数的重载版本

#include <iostream>
#include <type_traits>

template <typename  T>
void print(T var, typename std::enable_if<std::is_enum<T>::value, T>::type* = 0) {
    std::cout << "Caling overloaded print() for enumerations." << std::endl;
}

template <typename T>
void print(T var, typename std::enable_if(std::is_integral<T>::value, T>::type = 0) {
    std::cout << "Calling overloaded print() for integral types." << std::endl;

template <typename T>
void print(T var, typename std::enable_if<std::is_floating_point<T>::value, T>::type = 0) {
    std::cout << "Calling overloaded print() for floating point types." << std::endl;
}

template <typename T>
void print(cosnt T& var, typename std::enable_if<std::is_class<T>::value, T>::type* = 0) {
    std::cout << "Calling overloaded print() for classes." << std::endl;
}

我们可以通过使用不同类型的参数使用重载的函数模板

enum Enumeration1 {
    Literal1,
    Literal2
};

enum class Enumeration2 : int {
    Literal1,
    Literal2,
};

class Clazz {};

int main() {
    Enumeration1 enumVar1 {};
    print(enumVar1);

    Enumeration2 enumVar2 {};
    print(enumVar2);

    print(42);

    Clazz instance {};
    print(instance);

    print(42.0f);

    print(42.0);

    return 0;
}

输出：
Calling overloaded print() for enumerations.
Calling overloaded print() for enumerations.
Calling overloaded print() for integral types.
Calling overloaded print() for classes.
Calling overloaded print() for floating point types.
Calling overloaded print() for floating point types.

由于C++11版本的 std::enable_if 有点冗长
C++14新增加了一个名为 std::enable_if_t 的别名

？？不是更长了吗
```

```
Copy-and-Swap习惯用法

在5.7.1节 防患于未然 我们学到了四个异常安全的保障
无异常安全 基本异常安全 强异常安全 保证不抛出异常
类的成员函数应该始终保证是基本异常安全的，因为这种级别通常比较容易实现

在5.2.5节 零原则 中，我们了解到应该总以某种方式设计类，
以便那些编译器自动生成的特殊成员函数 拷贝构造函数 拷贝赋值运算符 能执行正确的操作。
换句话说，当我们被迫提供一个并非默认的析构函数时，一般就是处理一些特殊情况，我们需要在对象销毁期间对它们进行特殊的处理。

然而，零原则几乎是不可能实现的，即开发者必须自己实现所有的特殊成员的功能。
在这种情况下，对一个重载赋值运算符的异常安全性的实现可能是一项很具挑战性的任务。
在这种情况下，Copy-and-Swap是优雅地解决此问题的一种方式。

意图：
实现具有强异常安全行动 拷贝赋值运算符。

例子：管理在堆上分配了资源的类

#include <cstddef>

class Clazz final {
public:
    Clazz(const std::size_t size) : resourceToManage { new char[size] }, size { size } {}
    ~Clazz() {
        delete [] resourceToManage;
    }

private:
    char* resourceToManage;
    std::size_t size;
};

当然，本类仅仅用作演示，不应该成为真实类的一部分。

假设我们想要使用Clazz类来执行以下操作
int main() {
    Clazz instance1 { 1000 };
    Clazz instance2 { instance1 };
    return 0;
}

从第5章我们就知道，编译器生成的拷贝构造函数版本在这里出错了：
它只创建了指针 resourceToManage自身的副本！
因此，我们必须提供我们自己的拷贝构造函数，就像这样：

#include <algorithm>

class Clazz final {
public:
    //...
    Clazz(const Clazz& other) : Clazz { other.size } {
        std::copy(other.resourceToManage, other.resourceTomanage + other.size, resourceToManage);
    }
    //...
};

到现在为止还很好。现在拷贝构造函数将正常工作，但是我们还需要一个拷贝赋值运算符。
如果你不熟悉Copy-and-Swap习惯用法，你可以实现赋值运算符如下：

#include <algorithm>

class Clazz final {
public:
    //...
    Clazz& operator=(const Clazz& other) {
        if (&other == this) {
            return *this;
        }
        delete [] resourceToManage;
        resourceToManage = new char[other.size];
        std::copy(other.resourceToManage, other.resourceToManage + other.size,
            resourceToManage);
        size = other.size;
        return *this;
    }
};

基本上，这个赋值运算符可以工作，但它有几个缺点。
例如，构造函数和析构函数中的代码在这里重复了，这违反了 DRY 原则
此外，在开头有一个自我分配检查。
但最大的缺点是我们不能保证异常安全。
例如，如果new语句导致异常，则会将对象置于不可预知的状态

现在，Copy-and-Swap习惯用法开始发挥作用，它也被称为
Create-Temporary-and-Swap

为了更好理解，现在介绍整个Clazz类
使用Copy-and-Swap用法更好地实现赋值运算符

#include <algorithm>
#include <cstddef>

class Clazz final {
public:
    Clazz(const std::size_t size) : resourceToManage { new char[size] }, size { size } {}
    ~Clazz() {
        delete [] resourceToManage;
    }

    Clazz(const Clazz& other) : Clazz { other.size } {
        std::copy(other.resourceToManage, other.resourceToManage + other.size,
            resourceToManage);
    }

    Clazz& operator=(Clazz other) {
        swap(other);
        return *this;
    }
private:
    void swap(Clazz& other) noexcept {
        using std::swap;
        swap(resourceToManage, other.rsourceToManage);
        swap(size, other.size);
    }

    char* resourceToManage;
    std::size_t size;
};

这里的诀窍是什么呢？ 看看完全不同的赋值运算符
它不再以const引用 (const Clazz& other)作为参数
而是以普通值 (Clazz other)作为参数
这意味着当调用该运算符时，首先调用Clazz的拷贝构造函数。
其次，拷贝构造函数调用为资源分配内存的默认构造函数。
这正是我们想要的：我们需要other的一个临时副本！

现在我们来到这一习惯用法的核心：调用私有成员函数Clazz::swap()
在这个函数中，other临时实例的内容，即其成员变量，与我们当前类上下文
相同的成员变量的内容 this 进行了交换
这是通过保证不抛出异常的 std::swap() 函数 <utility> 来完成的
在swap操作之后，临时对象现在拥有当前对象先前拥有的资源，反之亦然。

另外，Clazz::swap() 成员函数现在可以很容易地实现移动构造函数：

class Clazz {
public:
    Clazz(Clazz&& other) noexcept {
        swap(other);
    }
};

当然，良好的类设计的主要目标，应该是根本不需要显示实现拷贝构造函数和赋值运算符 零原则
但是当你被迫这样做时，你应该记住 Copy-and-Swap习惯用法
```

```
指向实现的指针

Pointer to Implementation PIMPL
这个习惯用法也被称为 Handle Body
Compilation Firewall
Cheshire Cat technique
他和 Bridge 模式有一些相似之处

意图：
通过将内部类的实现细节重新定位到隐藏的实现类中，消除对实现类的编译依赖，从而提高编译时间。

Customer类，在很多例子用过

#ifndef CUSTOMER_H_
#define CUSTOMER_H_

#include "Address.h"
#include "Identifier.h"
#include <string>

class Customer {
public:
    Customer();
    virtual ~Customer() = default;
    std::string getFullName() const;
    void setShippingAddress(const Address& address);

private:
    Identifier customerId;
    std::string forename;
    std::string surname;
    Address shippingAddress;
};

#endif /* CUSTOMER_H_ */

假设这是我们的商业软件系统中的一个中心业务实体，并且它被许多其他类使用
当该头文件更改时，即只添加，重命名一个私有成员变量，我们也需要重新编译使用该文件的所有文件
为了将这些重新编译的文件个数减少到最少，可以使用 PIMPL 习惯用法

首先，我们重建Customer类的类接口

#ifndef CUSTOMER_H_
#define CUSTOMER_H_

#include <memory>
#include <string>

class Address;

class Customer {
public:
    Customer();
    virtual ~Customer();
    std::string getFullName() const;
    void setShippingAddress(const Address& address);

private:
    Class Impl;
    std::unique_ptr<Impl> impl;
};

#endif /* CUSTOMER_H_ */

显而易见的是，所有先前的私有成员变量以及相关的 #include指令现在以及消失，
相反，存在一个名为Impl的类的 前向声明
以及一个指向该前向声明类的 std::unique_ptr<T>

现在看一下 coerssponding的实现

#include "Customer.h"

#include "Address.h"
#include "Identifier.h"

class Customer::Impl final {
public:
    std::string getFullName() const;
    void setShippingAddress(const Address& address);

private:
    Identifier customerId;
    std::string forename;
    std::string surname;
    Address shippingAddresss;
};

std::string Customer::Impl::getFullName() const {
    return forename + " " + surname;
}

void Customer::Impl::setShippingAddress(const Address& address) {
    shippingAddress = address;
}

// Implementation of class Customer starts here...

Customer::Customer() : impl { std::make_unique<Customer::Impl>() } {}

Customer::~Customer() = default;

std::string Customer::getFullName() const {
    return impl->getFullName();
}

void Customer::setShippingAddress(const Address& address) {
    impl->setShippingAddress(address);
}

现在，如果修改了 Customer::Impl的内部实现，
编译器只需编译 Customer.h/Customer.cpp
这种变化对外界没有任何影响，并且避免了几乎整个项目的耗时编译。
```

---

# 附录 UML  Unified Modeling Language

```
类图
通常被用于描述面向对象软件设计的结构

》普通箭头 黑箭头
> 三角形空心箭头

实现关系
Class—— —— —— ——>Interface

泛化关系
Rectangle——————————>Shape

关联关系
Employee——————————》TimeCard

聚合
Department<>——————》Employee

组合
Employee<实心>——————》TimeCard

依赖关系
Dependent —— —— —— —— —— —— 》 Independent
```

```
类
类图中的核心元素是类
类描述了一组共享相同规范的特性，约束，和语义的对象

类的实例通常被称为对象。因此，可以将类视为对象的蓝图。

UML符号是一个 矩形
类名
属性
操作
静态属性要带有下划线
可见性类型
+ public
# protected
~ package
- private
抽象类用斜体类名

--------
Customer
--------
-forename:String
-surname:String
-id:CustomerIdentifier
--------
+getFullName():String
+getBirthday():DateTime
+getPrintableIdentifier():String
--------

实例：
Customer* customer = new Customer();

--------
customer:Customer
--------


#include <string>
#include "DateTime.h"
#include "CustomerIdentifier.h"

class Customer {
public:
    Customer();
    virtual ~Customer();
    std::string getFullName() const;
    DateTime getBirthday() const;
    std::string getPrintableIdentifier() const;
private:
    std::string forename;
    std::string surname;
    CustomerIdentifier id;
};
```

```
接口
接口定义了一种契约：实现接口的类必须履行该契约。

接口是一系列相关的公共契约的声明。

接口始终是抽象的，默认情况下无法被实例化。


--------
<<interface>>
Person
--------
getFullName():String
getBirthday():DataTime
--------

--------
Customer
--------
-forename:String
-surname:String
-id:CustomerIdentifier
--------
+getFullName():String
+getBirthday():DateTime
+getPrintableIdentifier():String
--------

Customer—— —— —— —— ——>Person

上面是：类Customer实现了在接口Person中声明的操作

虚线箭头是接口实现

接口
#include <string>
#include "DateTime.h"

class Person {
public:
    virtual ~Person() {}
    virtual std::string getFullName() const = 0;
    virtual DateTime getBirthday() const = 0;
};

实现接口
#include "Person.h"
#include "CustomerIdentifier.h"

class Customer : public Person {
public:
    Customer();
    virtual ~Customer();

    virtual std::string getFullName() const override;
    virtual DateTime getBirthday() const override;
    std::string getPrintableIdentifier() const;
private:
    std::string forename;
    std::string surname;
    CustomerIdentifier id;
};
```

```
关联
类通常与其他类有静态关系，UML的关联可以指定这种关系

关联关系允许某一类别，例如，类或组件 的一个实例访问另一个实例


A————B

最简单的关联是  一条直线
这种关联通常不足以正确指出两个类的关系
通常解释为双向


A————》B

类A能够导向类B
在另一个方向是未指定的，也就是说，类B可能能够导向类A
建议：在项目中定义该导向的解释，设置为不可导向的


  has>
A——————》B

和上一个完全相同



1 roleOfA          0..* roleOfB
A————————————————————》B

两个关联端都具有标签name和多重性multiplicity
标签通常用于指定关联中类的角色
多重性指定关联中涉及的类的实例的允许数量
它是一个非负整数区间，以下限开始，以上限结束
任何A都有零到某一数量的B，而任何B只有一个A

1 只有一个
1..10 1到10
0..* 0到多个
* 同上
1..* 1到多个


          containsOf>       *
Whole<>——————————————————————》Part

这是一种被称为聚合的特殊关联
代表了一个整体的关系，也就是说，一个类在层次上从属于另一个类
空心菱形是这种关联的标志，用于识别整体
另外，适用于关联的所有内容也适用于聚合


      0..1        owns>       *
Whole<实心>————————————————————》Part

这是一个组合型关联关系，它是一种更强的聚合形式。
它表示整体是部分的所有者，因此对部分负责。
如果删除整个实例，则通常会删除其所有的部分实例

编程语言中，可以用各种方式实现从一个类到另一个类的关联和导向机制。
在C++中，关联通常表现为自身成员的类型是其他类，例如成员为其他类类型的引用或指针

class B;

class A {
private:
    B* b;
};

class B {
};
```

```
泛化

面向对象软件开发的核心概念就是所谓的继承。这表示对各个特定类的泛化。

泛化是一般类和具体类之间的一种分类关系。

泛化关系用于表示继承的概念：特定的类 子类 继承了更通用的类 基类 的属性和操作。
泛化关系的UML语法是一个实心箭头，带有一个封闭但未填充的箭头

               <——————————————————Rectangle
Shape{abstract}<——————————————————Trigagle
               <——————————————————Circle

一个抽象基类和三个具体的类，它们是抽象基类的特化

关系： Rectangle 是一种  Shape
```

```
依赖

除了已经提到的关联外，类和组件 还可以与其他类和组件 建立进一步的关系。
例如，如果一个类被用作成员函数的参数类型，那么这不是一种关联关系，它是
对该使用的类的一种依赖。

依赖关系是一种关系，表示单个或一组元素需要其他元素用于其规约或实现。

Dependent —— —— —— —— —— —— 》 Independent

        <<use>>
List—— —— —— —— —— —— —— —— 》 ListImpl

<<Factory>>          <<Create>>
CustomerFactory—— —— —— —— —— —— —— —— —— ——》Customer

除了最简单的形式，还有两种特殊的关系

use 使用依赖性，其中一个元素需要另一个元素或一组元素来完全实现或操作

create 创建依赖性 表示箭头尾部的元素在箭头处创建类型的实例
```

```
组件

---------------------
<<component>>
Billing
---------------------

组件表示系统的模块化部分
通常在比单个类更高的抽象级别上
组件用作一组共同实现某些功能的类的包装。

由于组件封装其内容，因此它根据已提供的和所需要的接口定义其行为。
只有这些接口可供软件环境使用才能使用组件。
```

```
Stereotypes

UML的词汇可以借助 stereotypes拓展
这种轻量级机制允许引入标准UML元素的特定平台或域扩展

<<Factory>>
    工厂类，创建对象而不将实例化逻辑暴露给客户端
<<Facade>>
    Facade类，为复杂组件或子系统中的一组接口提供统一接口
<<SUT>>
    被测系统。具有该造型的类或组件是要测试的实体，比如，在单元测试的帮助下
<<TestContext>>
    测试上下文，是一个软件实体。例如作为一组测试用例的分组机制的类
<<TestCase>>
    测试用例是与SUT交互以验证其正确性的操作
```

---