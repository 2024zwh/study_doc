## 一、异常处理

> 通过 try except 结构处理异常

### 1.1 try except几种结构

- **一个except**

```python
try:
    a = 1
except Exception:
    pass
```

- **多个except**

  > 一个 [`except`](https://docs.python.org/zh-cn/3.13/reference/compound_stmts.html#except) 子句中的类匹配的异常将是该类本身的实例或其所派生的类的实例（但反过来则不可以 --- 列出派生类的 *except 子句* 不会匹配其基类的实例）

  ```python
  class B(Exception):
      pass
  
  class C(B):
      pass
  
  class D(C):
      pass
  
  for i in [B, C, D]:
      try:
          raise i
      except D:
          print("D")
      except C:
          print("C")
      except B:
          print("B")
         
         
  # 输出顺序 B C D
  
  
  for i in [B, C, D]:
      try:
          raise i
      except B:
          print("B")
      except D:
          print("D")
      except C:
          print("C")
          
  # 输出顺序 B B B
  ```

- **加上 else**

  > else放在所有Exception之后，没有出现异常的时候执行

  ```python
  try:
      raise Exception
  except Exception:
      print('error')
  else:
      print('no error')
  # 输出error
  ```

- **加上 finally**

  > 有没有异常都会最后执行
  >
  > 特殊情况 如果except没有获取到异常，则会在finally 执行后，触发异常

  ```python
  try:
      raise Exception
  except Exception:
      print('error')
  finally:
      print('finally')
  ```

  **特殊说明**

  > 如果except没有获取到异常，则会在finally 执行后，触发异常

  ```python
  >>> def device(x, y):
  ...      try:
  ...          result = x / y       
  ...      except ZeroDivisionError:
  ...          print('y is zero') 
  ...      else:
  ...          print('result is ', result)
  ...      finally:
  ...          print('finally end')
  ... 
  >>> device(2,1)                    
  result is  2.0
  finally end   
  >>> device(2, 0)
  y is zero  
  finally end
  >>> device('2', '1')
  finally end
  Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
    File "<stdin>", line 3, in device
  TypeError: unsupported operand type(s) for /: 'str' and 'str'
  ```

  

### 1.2 raise

> 主动抛出异常

```python
>>> raise NameError('HiThere')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
    raise NameError('HiThere')
NameError: HiThere
```

**主动触发多个异常**

```python
>>> from exceptiongroup import ExceptionGroup
>>> def f():
...     excs = [OSError('error 1'), SystemError('error 2')]
...     raise ExceptionGroup('there were problems', excs)
...
>>> f()
  + Exception Group Traceback (most recent call last):
  |   File "<stdin>", line 1, in <module>
  |   File "<stdin>", line 3, in f
  | exceptiongroup.ExceptionGroup: there were problems (2 sub-exceptions)
  +-+---------------- 1 ----------------
    | OSError: error 1
    +---------------- 2 ----------------
    | SystemError: error 2
    +------------------------------------
>>> try:
...     f()
... except Exception as e:
...     print(f'caught {type(e)}: e')
... 
caught <class 'exceptiongroup.ExceptionGroup'>: e

```



### 1.3 异常链

> 一个异常发生在except内，它会将有处理的异常附加到它上面

```python
try:
    open("database.sqlite")
except OSError:
    raise RuntimeError("unable to handle error")

"""
Traceback (most recent call last):
  File "D:\pycharm\ai-fastapi\test2.py", line 2, in <module>
    open("database.sqlite")
FileNotFoundError: [Errno 2] No such file or directory: 'database.sqlite'

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "D:\pycharm\ai-fastapi\test2.py", line 4, in <module>
    raise RuntimeError("unable to handle error")
RuntimeError: unable to handle error
"""
```

**使用raise from**:

> 为了表明一个异常是另一个异常的直接后果，raise语句允许一个可选的 from子句
>
> from 必须为**异常实例或者None**，为None的时候表示**禁用异常链**

```python
def func():
    raise ConnectionError

try:
    func()
except ConnectionError as exc:
    raise RuntimeError('Failed to open database') from exc

"""
Traceback (most recent call last):
  File "D:\pycharm\ai-fastapi\test2.py", line 5, in <module>
    func()
  File "D:\pycharm\ai-fastapi\test2.py", line 2, in func
    raise ConnectionError
ConnectionError

The above exception was the direct cause of the following exception:

Traceback (most recent call last):
  File "D:\pycharm\ai-fastapi\test2.py", line 7, in <module>
    raise RuntimeError('Failed to open database') from exc
RuntimeError: Failed to open database
"""
```

**禁用自动异常链**

```python
def func():
    raise ConnectionError

try:
    func()
except ConnectionError as exc:
    raise RuntimeError('Failed to open database') from None

"""
Traceback (most recent call last):
  File "D:\pycharm\ai-fastapi\test2.py", line 7, in <module>
    raise RuntimeError('Failed to open database') from None
RuntimeError: Failed to open database
"""
```

## 二、类

> 面向对象的核心思想，映射现实社会的中的个体，每个对象都是不同的，有相同的共同点，比如都有头发，但是头发的颜色，卷直又有不同的值

### 2.1 命名空间和作用域

#### 2.1.1 命名空间

> 命名空间是名称和对象的映射，命名空间大部分都是通过字典实现的，主要分为下面三类

- **内置名称的命名空间**

  > 解释器自带的命名空间，放一些自定义的功能函数，比如print，dict，
  >
  > python解释器启动的时候创建，且永远不会被删除

- **模块全局命名空间**

  > 程序初始化该模块时产生的，通常也是随程序退出被删除，不通模块全局命名空间名称互不影响，z.name, c.name

- **函数局部命名空间**

  > 函数被调用的时候产生，函数返回或者产生异常后退出

#### 2.1.2 作用域

> 作用域是代码中可以访问的变量的区域，它决定了变量的可见性和访问顺序

##### 1.查找顺序

> 在访问变量的时候，最先会从同级别的作用查找，然后依次向外部查找，下面是从最内层作用域开始

- **最内层作用域**，主要是函数内部作用域，最先被查找
- **嵌套函数的外层作用域**（闭包作用域），内层没有找到，如果有闭包作用域，在从该作用域查找变量
- **倒数第二层作用域**，模块的全局变量
- **内置作用域**，python自带的作用域

##### 2.示例

- **最内层作用域**

```
def test():
    x = 1
    print(x)
```

- **嵌套函数的外层作用域**

  > 可以通过使用 nonlocal 修改变量值

  ```
  def waper():
      x = 1
      def inner():
          nonlocal x
          x += 1
          print(x)
      return inner
  ```

- **倒数第二层作用域**

  > 可以通global 声明，在函数内部可以被修改

  ```
  x = 1
  def test():
      global x
      x += 1
      print(x)
  ```

  

- **内置作用域**

  ```
  print("1111")
  ```

### 2.2 类的属性

#### 2.2.1 正确定义类

> 定义类规则：实例变量是用于对象的唯一属性，类变量为对象提供相同的属性和方法，要合理使用，最好不要把可变数据类型定义到类变量中

**正确的**

```python
class DogTu:
    type = "tugou"  # 类变量

    def __init__(self, name):
        self.name = name  # 实例变量
        self.like_foods = []
    
    def add_like_food(self, food):
        self.like_foods.append(food)


dog_dh = DogTu("大黄")
dog_dh.add_like_food("骨头")
dog_dh.add_like_food("狗粮")
print(dog_dh.name,"喜欢吃：", dog_dh.like_foods)
dog_lbz = DogTu("老班长")
dog_lbz.add_like_food("骨头")
dog_lbz.add_like_food("苹果")
print(dog_lbz.name,"喜欢吃：",dog_lbz.like_foods)

"""
输出结果
大黄 喜欢吃： ['骨头', '狗粮']
老班长 喜欢吃： ['骨头', '苹果']
"""

```

**错误的**

> 共享数据可能在涉及 mutable（具名元祖）对象例如列表和字典的时候导致不想要的结果

```python
class DogTu:
    type = "tugou"  # 类变量
    like_foods = []

    def __init__(self, name):
        self.name = name  # 实例变量
    
    def add_like_food(self, food):
        self.like_foods.append(food)


dog_dh = DogTu("大黄")
dog_dh.add_like_food("骨头")
dog_dh.add_like_food("狗粮")
print(dog_dh.name,"喜欢吃：", dog_dh.like_foods)
dog_lbz = DogTu("老班长")
dog_lbz.add_like_food("骨头")
dog_lbz.add_like_food("苹果")
print(dog_lbz.name,"喜欢吃：",dog_lbz.like_foods)

"""
大黄 喜欢吃： ['骨头', '狗粮']
老班长 喜欢吃： ['骨头', '狗粮', '骨头', '苹果']
"""
```

#### 2.2.2 类的封装

> 封装是指将对象的数据（属性）和方法（行为）绑定在类中，并对外隐藏实现细节。通过类的封装，类内部的转态被保护起来，外部只能访问提供统一方法来和内部属性进行交互

##### 1.好处

- **保护数据完整性**：只能通过类提供的方法进行访问和修改
- **简化外部调用**：用户值关心输入和输出，不用关心内部实现逻辑，方便相互合作分工

##### 2.三种成员

- **公有成员**：直接访问
- **受保护成员**：以 _ 单下划线开头的，_age, 建议子类和内部访问，不建议外部直接访问
- **私有成员**：以 __ 双下划线开头的，__age, 只可以被内部访问

**示例**：

```python
class BankAccount:
    def __init__(self, owner, balance=0):
        self.owner = owner                # 公有属性：账户持有人（公开访问）
        self._account_number = "123456"   # 受保护属性（单下划线）：内部使用，不建议外部直接访问
        self.__balance = balance          # 私有属性（双下划线）：严格隐藏，仅通过方法访问

    # 获取余额（接口方法）
    def get_balance(self):
        return self.__balance

    # 存款（接口方法）
    def deposit(self, amount):
        if amount > 0:
            self.__balance += amount
            print(f"{amount} 元已存入，当前余额：{self.__balance} 元")
        else:
            print("存款金额必须大于 0")

    # 取款（接口方法）
    def withdraw(self, amount):
        if 0 < amount <= self.__balance:
            self.__balance -= amount
            print(f"{amount} 元已取出，当前余额：{self.__balance} 元")
        else:
            print("余额不足或金额无效")

    # 获取账户号（受保护属性的访问控制）
    def _get_account_number(self):
        return self._account_number
    
account = BankAccount("张三", 1000)
print(account.owner)  # 输出：张三（直接访问公有属性）
print(account._account_number)       # 输出：123456（虽然可以访问，但不符合规范）
print(account._get_account_number()) # 输出：123456（推荐通过方法访问）
account.deposit(500)      # 输出：500 元已存入，当前余额：1500 元
account.withdraw(200)     # 输出：200 元已取出，当前余额：1300 元
# 直接访问私有属性会报错
# print(account.__balance)  # 报错：AttributeError
print(account.get_balance())
```

#### 2.2.3 类的继承

> 继承是指子类继承父类的属性和方法的机制。子类可以复用父类的方法，或者对父类的属性进行改写。如果不支持继承，语言特性就不值得称为“类”

##### 1.好处

- **代码复用**：减少重复代码
- **层次分明**：有利于代码的管理
- **扩展功能**：可以不修改原代码的情况下修改或者添加新功能

##### 2.普通继承

>- 子类可以通过 `super()` 调用父类的方法
>- 子类可以命名一个和父类一样的函数，重写父类的方法

```python
class Animal:
    def __init__(self, name):
        self.name = name

    def speak(self):
        raise NotImplementedError("子类必须实现此方法")

class Dog(Animal):

    def __init__(self, name, age):
        super().__init__(name)
        self.age = age

    def speak(self):
        return f"{self.name} says Woof!"

class Cat(Animal):
    def speak(self):
        return f"{self.name} says Meow!"

# 使用
dog = Dog("Buddy", 3)
cat = Cat("Whiskers")
print(dog.speak())  # 输出: Buddy says Woof!
print(cat.speak())  # 输出: Whiskers says Meow!
```

##### 3.super说明

> 返回一个代理对象，它会将方法调用委托给 *type* 的父类或兄弟类
>
> 比如`__mro__`为 `D -> B -> C -> A -> object` 并且 *type* 的值为 `B`，则 `super(B, self)`将会搜索 `C -> A -> object`



##### 4.多重继承

> 当多个父类中存在同名方法时，Python 需要确定调用哪个类的方法。MRO（Method Resolution Order）定义了方法和属性的查找顺序，遵循 **C3 线性化算法**，确保基类仅调用一次

**C3 线性化算法**

- **子类优先于父类**
- **继承顺序从左到右**（即先继承的第一个父类优先）
- **单调性**（Monotonicity）：如果类 D 在类 E 之前，那么在所有派生类中 D 也必须在 E 之前

**示例：**

> 使用`__mro__`或者mro()，查看调用顺序，

```python
class A:
    def __init__(self):
        print('A')

class A2:
    def __init__(self):
        print('A2')

class B2(A2):
    def __init__(self):
        super().__init__()
        print('B2')


class B(A):
    def __init__(self):
        super().__init__()
        print('B')

class C(A):
    def __init__(self):
        super().__init__()
        print('C')


class D(B2, B, C):
    pass
  

print(D.mro())
D()

"""
(<class '__main__.D'>, <class '__main__.B2'>, <class '__main__.A2'>, <class '__main__.B'>, <class '__main__.C'>, <class '__main__.A'>, <class 'object'>)
A2
B2
"""
```

**调用父类的方法**

> 多重继承父类方法冲突的时候，可以显示的调用，也可以按MRO调用， 
>
> 下面是示例

```python
class A:
    def __init__(self):
        print('A')

class B(A):
    def __init__(self):
        print('B')

class C(A):
    def __init__(self):
        print('C')

class D(B, C):
    def __init__(self):
        super().__init__() # 按MRO 顺序调用
        C.__init__(self) # 指定调用
        print('D')

D()
"""
输出结果
B
C
D
"""
```

**建议：**

- **优先使用单一继承**：80% 的场景应使用单一继承，避免复杂性

  ```python
  class Fly:
      def fly(self):
          print("Flying")
  
  class Bird(Fly):
      pass
  
  class Bat(Fly):
      pass
  ```

  

- **使用 Mixin 模式**：将功能模块化，通过 Mixin 类组合功能，而不是直接多重继承复杂类

  ```python
  class Loggable:
      def log(self, msg):
          print(f"Log: {msg}")
  
  class Connectable:
      def connect(self):
          print("Connecting...")
  
  class Server(Connectable, Loggable):
      def start(self):
          self.log("Starting server")
          self.connect()
  
  s = Server()
  s.start()
  # 输出:
  # Log: Starting server
  # Connecting...
  ```

- **避免命名冲突**：为方法和属性选择独特的名称，减少冲突的可能性

- **合理设计类层次结构**：确保继承关系清晰，避免过度嵌套

##### 5.判断是否属于一个类的方法

- **isinstance(object, B)**：用于判断一个实例对象的类型
- **issubclass(D, B)**：用于判断一个类的继承关系

#### 2.2.4 多态

> 多态是指同一个类型的不同对象，操作相同的方法，但是返回的状态不同。多态允许不同类的对象对同一消息做出不同的响应

##### 1.好处

- **灵活性**：同一接口可以处理多种类型的数据
- **可扩展性**：新增子类时无需修改现有代码。
- **代码简洁性**：通过统一的接口调用不同实现。

##### 2.实现:

> 多态的核心是 **方法重写**（子类重写父类的方法）

```python
class Animal:
    def __init__(self, name):
        self.name = name

    def speak(self):
        return f"{self.name}  动物叫"

class Dog(Animal):
    def speak(self):
        return f"{self.name} 汪汪"

class Cat(Animal):
    def speak(self):
        return f"{self.name} 喵喵"

# 使用
dw = Animal("Dog")
dog = Dog("Buddy")
cat = Cat("Whiskers")
print(dw.speak())
print(dog.speak())
print(cat.speak())  

"""
Dog  动物叫
Buddy 汪汪
Whiskers 喵喵
"""
```



### 2.3 类的魔法方法

- **`__new__`**：实例对象创建的时候调用

- **`__init__`**：实例初始化属性的时候调用

- **`__call__`**：实例对象被当函数调用的时候触发

  - **保持状态的函数对象**

    > 与普通函数不同，`__call__` 方法可以通过实例属性保存状态，适合需要“有状态”的场景

    ```python
    class Counter:
        def __init__(self):
            self.count = 0
    
        def __call__(self):
            self.count += 1
            return self.count
    
    counter = Counter()
    print(counter())  # 输出: 1
    print(counter())  # 输出: 2
    ```

    

  - **实现装饰器**

    > 通过 `__call__` 方法，可以自定义装饰器类，实现更复杂的装饰逻辑

    ```python
    class Logger:
        def __init__(self, func):
            self.func = func
    
        def __call__(self, *args, **kwargs):
            print(f"调用函数: {self.func.__name__}")
            return self.func(*args, **kwargs)
    
    @Logger
    def greet(name):
        return f"Hello, {name}!"
    
    print(greet("Alice"))  # 输出: 调用函数: greet\nHello, Alice!
    ```

    

  - **策略模式或工厂模式**

    > 根据不同的初始化参数，动态改变对象的行为

    ```python
    # 策略
    class Power:
        def __init__(self, exponent):
            self.exponent = exponent
    
        def __call__(self, base):
            return base ** self.exponent
    
    square = Power(2)
    cube = Power(3)
    
    print(square(4))  # 输出: 16 (4^2)
    print(cube(3))    # 输出: 27 (3^3)
    
    # 工厂模式
    class AnimalFactory:
        def __call__(self, kind):
            if kind == "dog":
                return Dog()
            elif kind == "cat":
                return Cat()
            else:
                raise ValueError("Unknown animal")
    
    factory = AnimalFactory()
    dog = factory("dog")
    cat = factory("cat")
    ```

    

- **`__del__`**：实例对象删除的时候触发

- **`__exit__` 和`__enter__`** ：上下文管理器

**示例**：

```python

class A:
    def __new__(cls):
        print("new")
        return super().__new__(cls)
    
    def __init__(self):
        print("init")
    
    def __del__(self):
        print("del")
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        print("exit")
    
    def __enter__(self):
        print("enter")
        return self
    
    def __call__(self):
        print('call')


with A() as a:
    a()
    
"""
new
init
enter
call
exit
del
"""
```



## 三、多线程，协程，多进程

### 3.1 多线程

> Python多线程是一种实现并发编程的重要方式，适用于处理**I/O密集型任务**（如网络请求、文件读写等）。但是python 中有GIL线程锁（同一个进程下，同一时刻只能有一个地方可以执行python代码），所以多线程不能利用多核CPU并行执行CPU密集型任务。多线程更适合I/O密集型任务（如等待网络响应、文件读写），此时线程在等待时会释放GIL，其他线程可继续执行

#### 1.常使用的线程控制

##### Lock：线程锁，保护共享资源有序切换

> 循环控制，循环打印A B A B

```python
from threading import Thread, Lock

is_print_a = True

local_lock = Lock()

def task1():
    global is_print_a
    count = 0
    while count < 50:
        if is_print_a :
            print("A")
            local_lock.acquire()
            is_print_a = False
            local_lock.release()
            count += 1
        

def task2():
    global is_print_a
    count = 0
    while count < 50:
        if is_print_a is False:
            print("B")
            local_lock.acquire()
            is_print_a = True
            local_lock.release()
            count += 1


if __name__ == "__main__":
    t1 = Thread(target=task1)
    t2 = Thread(target=task2)
    t1.start()
    t2.start()
    t1.join()
    t2.join()



```



##### `Event`: 事件驱动通信

> `Event` 是一种简单的线程间通信机制，通过一个布尔标志（`flag`）来通知其他线程某个事件是否发生

- 核心方法：
  - `set()`：将标志设为 `True`，通知所有等待的线程。
  - `clear()`：将标志重置为 `False`。
  - `wait(timeout=None)`：阻塞线程直到标志为 `True`。
  - `is_set()`：检查标志是否为 `True`。
- 使用场景：
  - **简单的一对多通知**：例如，主线程等待所有子线程完成任务后再继续执行。
  - **红绿灯模型**：某个线程等待外部信号（如“绿灯”）后才开始执行。

**示例**

```python
import threading
import time

# 创建一个事件对象
event = threading.Event()

def wait_for_event():
    print("等待事件触发...")
    event.wait()  # 等待事件被触发
    print("事件已触发，继续执行！")

def trigger_event():
    time.sleep(2)  # 模拟耗时操作
    print("触发事件...")
    event.set()  # 设置事件标志为True

# 创建并启动线程
t1 = threading.Thread(target=wait_for_event)
t2 = threading.Thread(target=trigger_event)
t1.start()
t2.start()

t1.join()
t2.join()

"""
等待事件触发...
触发事件...
事件已触发，继续执行！
"""
```



##### ``Condition`: **条件变量与复杂同步**

> `Condition` 是基于锁（`Lock` 或 `RLock`）的高级同步工具，允许线程在满足特定条件时等待或唤醒其他线程

- **核心方法**：
  - `acquire()` / `release()`：获取/释放底层锁。
  - `wait(timeout=None)`：释放锁并等待，直到被 `notify()` 或 `notify_all()` 唤醒。
  - `notify(n=1)`：唤醒一个等待的线程。
  - `notify_all()`：唤醒所有等待的线程
- **适用场景**
  - **生产者-消费者模型**：当缓冲区为空或满时，线程需要等待或通知其他线程。
  - **线程协作**：多个线程需要按特定顺序执行（如轮询）

 **示例**

> 生产者消费者

```python
import threading
import time

# 共享资源
data = []
condition = threading.Condition()

def producer():
    with condition:  # 自动获取锁
        print("生产者准备生产数据")
        data.append(1)  # 生产数据
        print("生产者通知消费者")
        condition.notify()  # 唤醒等待的消费者

def consumer():
    with condition:  # 自动获取锁
        print("消费者等待数据")
        condition.wait()  # 释放锁并等待
        print("消费者收到通知，消费数据:", data.pop())

# 创建并启动线程
t1 = threading.Thread(target=consumer)
t2 = threading.Thread(target=producer)
t1.start()
t2.start()

t1.join()
t2.join()
```

> 循环控制，循环打印ABABAB

```PYTHON
import threading

# 条件变量用于线程间的同步
condition = threading.Condition()
# 控制打印顺序的计数器
counter = 0

def print_a():
    global counter
    for _ in range(50):
        with condition:
            # 等待直到轮到打印A
            while counter % 2 != 0:
                condition.wait()
            print('A')
            # 更新计数器并通知其他线程
            counter += 1
            condition.notify_all()

def print_b():
    global counter
    for _ in range(50):
        with condition:
            # 等待直到轮到打印B
            while counter % 2 == 0:
                condition.wait()
            print('B')
            # 更新计数器并通知其他线程
            counter += 1
            condition.notify_all()

thread_a = threading.Thread(target=print_a)
thread_b = threading.Thread(target=print_b)
thread_a.start()
thread_b.start()

thread_a.join()
thread_b.join()
```



##### Semaphore`: **资源访问限流**

> `Semaphore` 是一个计数器，用于控制同时访问某个资源的线程数量

- 核心方法：
  - `acquire()`：获取信号量（计数减1），若计数为0则阻塞。
  - `release()`：释放信号量（计数加1），唤醒等待的线程。
- **适用场景**
  - **资源池**：如数据库连接池、线程池。
  - **限流**：限制同时执行的线程数量，防止资源耗尽（如高并发下的限流）

**示例**

> 限制资源访问

```python
import time
from threading import Thread, Semaphore

sem = Semaphore(3)

def task(name):
    sem.acquire() # 获取锁
    print(f"{name} start")
    time.sleep(2)
    sem.release() # 释放锁
    print(f"{name} end")

if __name__ == "__main__":
    threads = [Thread(target=task, args=(f"Thread {i}",)) for i in range(10)]
    for t in threads:
        t.start()
    for t in threads:
        t.join()

"""
Thread 0 start
Thread 1 start
Thread 2 start
Thread 2 end
Thread 0 end  
Thread 1 end  
Thread 3 start
Thread 5 start
Thread 4 start
Thread 3 end
Thread 5 end  
Thread 7 start
Thread 8 start
Thread 6 start
Thread 4 end  
Thread 6 end
Thread 8 end
Thread 7 end
Thread 9 start
Thread 9 end
"""          
 
```

#### 2.线程使用案例

##### 简单使用

```python
import threading
import time

def worker(name):
    print(f"线程 {threading.current_thread().name} 正在运行...")
    print(f"处理任务: {name}")
    time.sleep(2)
    print(f"线程 {threading.current_thread().name} 结束")

if __name__ == "__main__":
    print(f"主线程 {threading.current_thread().name} 正在运行...")
    t = threading.Thread(target=worker, args=("数据处理",), name="WorkerThread")
    t.start()
    t.join()
    print(f"主线程 {threading.current_thread().name} 结束")
```

##### 对Thread进行封装

> 把逻辑封装到run中，会默认直接调用

```python
import threading
import time

class MyThread(threading.Thread):
    def __init__(self, name):
        super().__init__(name=name)

    def run(self):
        print(f"线程 {self.name} 正在运行...")
        time.sleep(2)
        print(f"线程 {self.name} 结束")

if __name__ == "__main__":
    t = MyThread(name="CustomThread")
    t.start()
    t.join()
```

##### **线程池**

> 通过`concurrent.futures.ThreadPoolExecutor` 实现

- **future对象**

  > `Future` 对象用于跟踪任务状态和结果

  - **`result(timeout=None)`**: 阻塞等待任务完成并返回结果（可设置超时）
  - **`done()`**：用于检查任务是否完成，Ture：已经完成
  - **`cancel()`**：取消任务（仅在任务未开始时有效），Ture：取消成功
  - **`add_done_callback(fn)`**：注册回调函数，任务完成后自动调用

- **核心函数**

  - **`submit(fn, *args, **kwargs)`**

    > 提交单个任务，返回一个future对象

  - **`map(fn, *iterables, timeout=None, chunksize=1)`**

    > 批量提交任务，返回结果的生成器（按任务提交顺序）

  - **`shutdown(wait=True, cancel_futures=False)`**

    > `wait=True`：阻塞直到所有任务完成。
    >
    > `cancel_futures=True`：取消未开始的任务（Python 3.9+）

- **示例** 

  ```python
  from concurrent.futures import ThreadPoolExecutor
  import time
  
  def task(n):
      print(f"任务 {n} 开始...")
      time.sleep(1)
      print(f"任务 {n} 结束")
      return n * n
  
  if __name__ == "__main__":
      with ThreadPoolExecutor(max_workers=3) as executor:
          res = executor.submit(task, 0)
          res.result() # 阻塞
          results = list(executor.map(task, [1, 2, 3, 4, 5]))
      print("结果:", results)
      
  """
  任务 0 开始...
  任务 0 结束
  任务 1 开始...
  任务 2 开始...
  任务 3 开始...
  任务 3 结束
  任务 1 结束
  任务 2 结束
  任务 5 开始...
  任务 4 开始...
  任务 5 结束
  任务 4 结束
  结果: [1, 4, 9, 16, 25]
  """
  ```

##### **定时任务**

> 通过threading.Timer实现

```python
import threading
import time

def periodic_task():
    print("执行定时任务...")
    threading.Timer(5, periodic_task).start()  # 每隔5秒执行一次

periodic_task()
while True:
    time.sleep(1)  # 主线程保持运行， 让出执行劝
```



### 3.2 协程

> 协程是一个微线程，一个进程下，同一时刻只有一个线程在执行任务，也是用来处理IO任务，和线程不同的是，可以自己控制，减少了锁的切换开销

#### 3.2.1 原理

>Python 协程的原理主要基于 **协作式调度** 和 **事件循环机制**，其核心思想是通过程序主动让出执行权（而非操作系统强制切换），从而在单线程内实现高并发

##### 1.协作调度

> 协程的执行控制权又用户自己控制，通过await让出执行权，然后事件循环会调用其它的程序

##### 2.事件循环（event-loop）

> 是协程调度的核心，负责管理所有异步任务的执行顺序

**工作原理**

- event-loop会循环从任务队列中获取任务
- 执行任务，当遇到await，事件循环会重新从任务队列中获取任务
- 当await 部分完成后，事件循环自动唤醒程序继续执行

#### 3.2.2 核心对象

##### 1.asyncio

> `asyncio` 提供了多种方法来管理异步任务和事件循环

- asyncio.run()

  > 启动主事件循环并运行协程

  ```python
  import asyncio
  
  async def main():
      print("Hello")
      await asyncio.sleep(1)
      print("World")
  
  asyncio.run(main())
  ```

- `asyncio.create_task(coro, *, name=None)`

  > 提交协程任务，coro必须是协程对象，并且返回Task对象

  ```python
  import asyncio
  
  
  async def test():
      print('hello world')
      await asyncio.sleep(1)
      print('hello again')
      return 'done'
  
  
  async def main():
      task = asyncio.create_task(test())
      reust = await task
      print(reust)
  
  
  asyncio.run(main())
  
  ```

  

- `asyncio.ensure_future(arg, *, loop=None)`

  > 它不仅可以接受协程对象，还可以接受一个已经存在的 *Future* 对象，并将其包装为任务，用来提交任务
  >
  > 即使没有显式 *await*，任务也会被调度并执行

  ```python
  import asyncio
  
  
  async def test(name):
      print(f'{name} says hello world')
      await asyncio.sleep(1)
      print(f'{name} says hello again')
      return f'{name} done'
  
  async def main():
      task = asyncio.ensure_future(test('test-0'))
      task = asyncio.create_task(test('test-1'))
      reust = await task
      print(reust)
  
  asyncio.run(main())
  """
  test-0 says hello world
  test-1 says hello world
  test-0 says hello again
  test-1 says hello again
  test-1 done
  """
  ```

  

- `asyncio.gather(*aws, return_exceptions=False)`

  > 并发运行协程列表
  >
  > `aws`：协程或任务的集合。
  >
  > `return_exceptions`：若为 `True`，异常会被作为结果返回，否则会引发异常

  ```python
  import asyncio
  
  
  async def test(name):
      print(f'{name} says hello world')
      await asyncio.sleep(1)
      print(f'{name} says hello again')
      return f'{name} done'
  
  async def main():
      task0 = asyncio.create_task(test('test-0'))
      task1 = asyncio.create_task(test('test-1'))
      results = await asyncio.gather(task0, task1)
      for result in results:
          print(result)
  
  asyncio.run(main())
  
  ```

- `asyncio.wait(aws, *, loop=None, timeout=None, return_when=ALL_COMPLETED)`

  > 并发执行结果，并且设置等待时间，分别返回已经执行完成和未完成的TASK
  >
  > return_when`：指定何时返回（如 `FIRST_COMPLETED`、`ALL_COMPLETED)

  ```python
  import asyncio
  
  
  async def test(name):
      print(f'{name} says hello world')
      await asyncio.sleep(1)
      print(f'{name} says hello again')
      return f'{name} done'
  
  async def main():
      task0 = asyncio.create_task(test('test-0'))
      task1 = asyncio.create_task(test('test-1'))
      done, pending = await asyncio.wait([task0, task1])
      for i in done:
          print(i.result())
  
  asyncio.run(main())
  
  ```

##### 2.Task

> Task` 是 `Future` 的子类，用于调度协程的执行。`Task` 本质上是一个封装了协程的 `Future

- **`task.cancel()`**

  > 取消任务

- **`task.cancelled()`**

  > 查看任务是否取消

- **`task.get_name()` / `task.set_name(name)`**

  > 获取任务名称/设置任务名称

- **`task.result()` **

  > 获取任务结果

- **`task.exception()`**

  > 获取任务异常

- **`task.add_done_callback(callback)`**

  > 添加任务执行的回调函数

##### 3.Future

> `Future`是一种特殊的 **低层级** 可等待对象，表示一个异步操作的 **最终结果**。
>
> 当一个 Future 对象 *被等待*，这意味着协程将保持等待直到该 Future 对象在其他地方操作完毕。
>
> 在 asyncio 中需要 Future 对象以便允许通过 async/await 使用基于回调的代码。
>
> 通常情况下 **没有必要** 在应用层级的代码中创建 Future 对象

```python
async def main():
    await function_that_returns_a_future_object()

    # 这样也可以：
    await asyncio.gather(
        function_that_returns_a_future_object(),
        some_python_coroutine()
    )
```

#### 3.2.3 使用案例

##### 1.异步请求加并发控制

> 使用 asyncio.Semaphore控制并发和aiohttp请求IO

```python
import asyncio
import aiohttp

async def fetch(session, url, semaphore):
    async with semaphore:
        print(f"开始请求 {url}")
        async with session.get(url) as response:
            text = await response.text()
            print(f"请求完成 {url}, 状态码: {response.status}")
            return text

async def main():
    urls = [
        "https://example.com",
        "https://example.org",
        "https://example.net",
    ]
    semaphore = asyncio.Semaphore(2)
    async with aiohttp.ClientSession() as session:
        tasks = [fetch(session, url, semaphore) for url in urls]
        await asyncio.gather(*tasks)

if __name__ == "__main__":
    asyncio.run(main())
```

##### 2.生产者消费者模式

> 使用asyncio.Queue()

```python
import asyncio

async def producer(queue):
    for i in range(5):
        await asyncio.sleep(0.5)  # 模拟生产延迟
        item = f"Item-{i}"
        print(f"生产者生成 {item}")
        await queue.put(item)

async def consumer(queue):
    while True:
        item = await queue.get()
        if item is None:  # 退出信号
            break
        await asyncio.sleep(1)  # 模拟消费延迟
        print(f"消费者处理 {item}")
        queue.task_done()

async def main():
    queue = asyncio.Queue()
    # 启动生产者和消费者
    producer_task = asyncio.create_task(producer(queue))
    consumer_task = asyncio.create_task(consumer(queue))

    await producer_task  # 等待生产者完成
    await queue.join()   # 等待队列中所有任务被消费
    queue.put_nowait(None)  # 发送退出信号
    await consumer_task

if __name__ == "__main__":
    asyncio.run(main())
```

### 3.3 多进程

> Python的多进程（Multiprocessing）是利用多核CPU资源的一种强大技术，尤其适合**CPU密集型任务**（如大规模计算、图像处理等）。通过多进程，可以绕过Python的全局解释器锁（GIL），实现真正的并行执行。

#### 3.3.1 核心对象

##### 1.Process

> 创建进程

```python
import multiprocessing
import os
def work(idx):
    print(f"子进程 {idx}，PID: {os.getpid()}，启动！")

if __name__=="__main__":
    task = multiprocessing.Process(target=work, args=(1,))
    task.start()
    task.join()
```



##### 2.Queue` / `Pipe

> 进程间通信

**Queue**

```python
import multiprocessing
import time
import os
def work(queue):
    while True:
        data = queue.get()
        if data == None: break
        print(f"消费者 {os.getpid()}，消费了 {data}")
        time.sleep(1)

def producer(queue):
    for i in range(10):
        queue.put(i)
        print(f"生产者 {os.getpid()}，生产了 {i}")
        time.sleep(1)
    queue.put(None)




if __name__=="__main__":
    queue = multiprocessing.Queue()
    work_task = multiprocessing.Process(target=work, args=(queue,))
    producer_task = multiprocessing.Process(target=producer, args=(queue,))
    producer_task.start()
    work_task.start()
    producer_task.join()
    work_task.join()
```

**Pipe**

> 一对一通信，双向传输数据，conn1, conn2 = Pipe(duplex=True)默认为True

```python
from multiprocessing import Process, Pipe
import time

def worker(conn):
    # 子进程接收数据
    msg = conn.recv()
    print(f"子进程收到: {msg}")
    
    # 子进程发送数据
    time.sleep(1)  # 模拟延迟
    conn.send("子进程的回复")

if __name__ == "__main__":
    # 创建双向管道
    parent_conn, child_conn = Pipe()
    
    # 创建子进程
    p = Process(target=worker, args=(child_conn,))
    p.start()
    
    # 父进程发送数据
    parent_conn.send("父进程的消息")
    
    # 父进程接收数据
    reply = parent_conn.recv()
    print(f"父进程收到: {reply}")
    
    # 等待子进程结束
    p.join()
    
    # 关闭连接
    parent_conn.close()
    child_conn.close()
```



##### 3.pool

> 进程池
>
> **`map`**：同步执行，等待所有任务完成后再继续。
>
> **`apply_async`**：异步执行，适合需要非阻塞调用的场景

```python
from multiprocessing import Pool
import time

def compute_square(n):
    print(f"计算 {n} 的平方...")
    time.sleep(2)  # 模拟耗时操作
    return n ** 2

if __name__ == '__main__':
    with Pool(4) as pool:  # 创建4个进程的进程池
        results = pool.map(compute_square, [1, 2, 3, 4])  # 并行执行
        print("结果:", results)
```

```python
from multiprocessing import Pool

def async_task(n):
    return n * 2

if __name__ == '__main__':
    with Pool(2) as pool:
        result = pool.apply_async(async_task, (5,))  # 异步执行
        print("异步结果:", result.get())  # 获取结果
```



##### 4.Manager

> 共享数据结构， 支持dict 和list

```python
from multiprocessing import Manager, Process, Lock

def modify_data(shared_dict, lock, key, value):
    with lock:
        shared_dict[key] = value
        print(f"进程 {key} 修改了数据: {shared_dict}")

if __name__ == "__main__":
    with Manager() as manager:
        shared_dict = manager.dict()  # 创建共享字典
        lock = manager.Lock()         # 创建共享锁
        processes = []

        # 启动多个进程
        for i in range(3):
            p = Process(target=modify_data, args=(shared_dict, lock, f'key_{i}', f'value_{i}'))
            processes.append(p)
            p.start()

        # 等待所有进程结束
        for p in processes:
            p.join()

        print("最终共享字典内容:", shared_dict)
```

#### 3.3.2 注意事项

- **Windows系统下的限制：**
  - 在Windows上运行多进程时，必须将主代码放在`if __name__ == '__main__':`块中，否则会因递归创建进程而崩溃。
- **进程池的关闭：**
  - 使用`Pool`时，需调用`pool.close()`和`pool.join()`，确保所有任务完成后再退出。
- **数据共享的代价：**
  - 进程间共享数据需要序列化（如`pickle`），频繁传输大数据可能影响性能。
- **异常处理：**
  - 子进程中的异常无法直接捕获，需通过日志或`Queue`传递错误信息。







## 面试题

### 1.**闭包**

> 闭包是一种的特殊函数，它不仅能够记住并访问其定义时的作用域中的变量，而且即使在其外部函数执行完毕后，这些变量的值仍然可以被保持在内存中。
>

**示例：**

```
def out(x):
    def inner(y):
        return x + y
    return inner

inner = out(1)
print(inner(2))
# 输出3
```

### 2.装饰器

> 装饰器是利用闭包原理实现的，主要用来再不休修改原代码的情况下，给类或者函数添加新的功能

**函数的装饰器**

> 获取函数的运行时间

```
import time
def warp(func):
    def inner(*args, **kw):
        start = time.time()
        res = func(*args,**kw)
        print(f"{func.__name__}运行时间{time.time() - start}")
        return res
    return inner

@warp
def test(a, b):
    time.sleep(1)
    return a + b

test(1,1)

"""
test运行时间1.0014243125915527
"""

```

**类的装饰器**

> 用装饰器实现单例模式

```python
import time
from concurrent.futures.thread import ThreadPoolExecutor
from threading import Lock

def single(cls):
    tmp_data = {}
    lock = Lock()
    def inner(*arg, **kw):
        with lock:
            if cls not in tmp_data:
                tmp_data[cls] = cls(*arg, **kw)
            return tmp_data[cls]
    return inner



@single
class TestObject:
    pass


def thread_test(name):
    time.sleep(0.2)
    print(name)
    print(id(TestObject()))


with ThreadPoolExecutor(5) as pool:
    results = pool.map(thread_test, [i for i in range(10)])
    for res in results:
        pass



"""
2501798317968
2501798317968
"""
```



### 3.垃圾回收

> python垃圾回收机制，是通过引用计数，分代回收和标记清除这三种机制处理的

#### 3.1 引用计数

> 会给每个对象进行引用计数，当引用计数为0的时候，会自动回收对象， 但是无法处理循环引用的对象

**工作原理**

- 每个对象内部维护一个 `ob_refcnt` 字段，记录当前有多少个引用指向该对象。
- 引用增加的情况：
  - 对象被赋值给变量（如 `a = [1,2,3]`）。
  - 对象作为参数传递给函数。
  - 对象被添加到容器（如列表、字典）中。
- 引用减少的情况：
  - 变量被重新赋值（如 `a = None`）。
  - 变量被删除（如 `del a`）。
  - 容器被销毁或元素被移除。

**示例**

```python
import sys
a = [1, 2, 3]
b = a
c = a

print('当前引用数量:',sys.getrefcount(a))
del c
print('删除c后的数量:',sys.getrefcount(a))
del b
print('删除b后的数量:',sys.getrefcount(a))
del a # 最后引用计数数量

"""
当前引用数量: 4
删除c后的数量: 3
删除b后的数量: 2
"""

```

#### 3.2 分代回收

> 把对象分为三代，每次触发回收机制，没有回收掉就往后移动，这样做的好处是减少扫描次数，减少性能影响

##### **1. 核心思想**

- **弱代假说**：大多数对象的生命周期较短，而存活时间长的对象通常会持续存活。
- 将对象按存活时间划分为三代：
  - **第 0 代（新生代）**：新创建的对象。
  - **第 1 代（中年代）**：经过一次 GC 后仍存活的对象。
  - **第 2 代（老年代）**：经过多次 GC 后仍存活的对象。

##### **2. 触发条件**

- 每代有独立的计数器，记录对象分配和删除次数。
- 当某代的计数器超过阈值时，触发该代的 GC。
  - **第 0 代**：GC 频率最高（默认每 700 次分配触发一次）。
  - **第 1 代**：GC 频率较低。
  - **第 2 代**：GC 频率最低

##### 3. **优点**

- **减少 GC 开销**：优先回收年轻代对象，减少扫描的总对象数。
- **提高效率**：老年代对象通常稳定，无需频繁回收



**示例**

```python
import gc

# 查看当前分代阈值
print(gc.get_threshold())  # 默认输出 (700, 10, 10)
```

#### 3.3 标记清除

> 通过标记那些对象可达，然后对所有为被标记的对象进行回收，解决循环引用的问题

**工作原理**

- **标记阶段**（Mark）：
  - 从根对象（如全局变量、栈中的变量）出发，递归标记所有可达对象。
- **清除阶段**（Sweep）：
  - 扫描内存池，回收所有未被标记的对象（不可达对象）。

```python
import gc
import sys
print(gc.get_threshold())
l1 = [1, 2, 3]
l2 = [4, 5, 6]

l1.append(l2)
l2.append(l1)

del l1, l2

print("触发垃圾回收前的不可达对象数量:", gc.collect())
# 输出 2
print("触发垃圾回收后的不可达对象数量:", gc.collect())
# 输出 0
```



#### **3.4 gc常用方法**

##### **1. `gc` 模块的方法**

- **`gc.collect(generation=2)`**：

  - **作用**：手动触发垃圾回收，清理所有代（0、1、2）或指定代的对象。

  - 参数：

    - `generation`：0、1、2 分别表示第 0、1、2 代。

  - **返回值**：回收的不可达对象数量。

  - 示例：

    ```
    import gc
    print("回收对象数量:", gc.collect())  # 清理所有代
    ```

- **`gc.set_threshold(threshold0, threshold1, threshold2)`**：

  - **作用**：设置分代回收的阈值（触发 GC 的条件）。

  - 参数：

    - `threshold0`：第 0 代的分配次数阈值。
    - `threshold1`：第 1 代的晋升次数阈值。
    - `threshold2`：第 2 代的晋升次数阈值。

  - **默认值**：`(700, 10, 10)`。

  - 示例：

    ```
    import gc
    gc.set_threshold(1000, 5, 5)  # 调整阈值
    ```

- **`gc.get_threshold()`**：

  - **作用**：获取当前的分代回收阈值。

  - 示例：

    ```
    import gc
    print(gc.get_threshold())  # 输出: (700, 10, 10)
    ```

- **`gc.enable()` / `gc.disable()`**：

  - **作用**：启用或禁用自动垃圾回收。

  - 示例：

    ```
    import gc
    gc.disable()  # 禁用 GC
    gc.enable()   # 启用 GC
    ```

- **`gc.isenabled()`**：

  - **作用**：检查 GC 是否处于启用状态。

  - **示例**：

    ```
    import gc
    print(gc.isenabled())  # 输出 True 或 False
    ```

- **`gc.set_debug(flags)`**：

  - **作用**：启用调试模式，记录 GC 的详细行为（如对象回收、阈值触发等）。

  - 常用标志：

    - `gc.DEBUG_STATS`：输出统计信息。
    - `gc.DEBUG_COLLECTABLE`：输出可回收对象。
    - `gc.DEBUG_UNCOLLECTABLE`：输出不可回收对象。

  - **示例**：

    ```python
    import gc
    gc.set_debug(gc.DEBUG_STATS | gc.DEBUG_COLLECTABLE)
    ```

##### **2. 容器对象的 GC 支持**

- `Py_TPFLAGS_HAVE_GC` 标志：
  - **作用**：自定义容器类需实现此标志，以支持 GC 跟踪（如 `list`、`dict`）。
  - **相关函数**：
    - `PyObject_GC_Track()`：将对象加入 GC 跟踪。
    - `PyObject_GC_UnTrack()`：将对象移出 GC 跟踪。
    - `PyObject_GC_New()`：创建支持 GC 的容器对象。
  - **示例**（C 扩展中使用，Python 层无需直接调用）。



##### **3. 其他相关方法**

- **`weakref` 模块**：

  - **作用**：创建弱引用（不增加引用计数），用于观察对象是否被回收。

  - **示例**：

    ```python
    import weakref
    a = [1, 2, 3]
    wr = weakref.ref(a)
    del a
    print(wr())  # 输出 None（对象已被回收）
    ```

- **`tracemalloc` 模块**：

  - **作用**：追踪内存分配，分析内存泄漏。

  - **示例**：

    ```python
    import tracemalloc
    tracemalloc.start()
    # 执行代码...
    snapshot = tracemalloc.take_snapshot()
    top_stats = snapshot.statistics('lineno')
    for stat in top_stats:
        print(stat)
    ```





