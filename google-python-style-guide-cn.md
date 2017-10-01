# Python Language Rules

## Lint

    使用`pylint`检查代码的bug和编码规范等问题。

`pylint`可以检测一些常见的错误，比如拼写错误、用`var`声明变量等，但是`pylint`并不完全准确，经常会有一些误报的warning，这些warning可以忽略。

## Imports

    `import`仅用于package和module。

`import x`: import一个package或module；
`from x import y`: x是package名，y是不带前缀的module名；
`from x import y as z`: 如果有两个module的名称都是y，或者y的名字比较长时，使用这种形式；

在import中不要使用相对名称，应该使用package的全名。例如：

```python
from sound.effects import echo
...
echo.EchoFilter(input, output, delay=0.7, atten=4)
```

## Packages

    在import一个module的时候，使用module的全路径名。

可以避免module名冲突。方便module的查找。

例如：

```python
# Reference in code with complete name.
import sound.effects.echo

# Reference in code with just module name (preferred).
from sound.effects import echo
```

## Exceptions

    Exception可以使用，但是必须谨慎。

使用Exception，需遵循以下原则：

+ 使用`raise MyException('Error message')`或`raise MyException`，不要使用两个参数的形式(`raise MyException, 'Error message'`)，也不要使用过时的String形式(`raise 'Error message'`)。

+ module或package应该定义自己的exception基类，该基类应该继承`Exception`类。
例如：

```python
class Error(Exception):
    pass;
```

+ 不要使用捕获所有异常(catch-all)的形式，如`except: `, 或`except Exception: `, 以及`except StandardError`等，除非将异常重新抛出，或者当前处于线程的最外层。否则所有的异常（比如拼写错误、单元测试错误、Ctrl+C中断等）都会被捕获。
+ 尽量简化`try/except`中的代码块，代码越多，发生错误的概率就越大，而真正的错误很可能被忽略了。
+ 无论是否发生Exception，使用`finally`执行一些代码，比如关闭文件；
+ 当捕获到Exception时，使用`as`而不是逗号。

例如：

```python
try:
    raise Error
except Error as error:
    pass
```    

## Global Variables

    尽量避免使用全局变量。

当moduel被import后，module变量（全局变量）可以通过赋值修改。

尽量使用class变量，而不是全局变量。以下是例外情况：

- 脚本的默认选项；
- module的常量，例如：`PI = 3.14159`;
- 通过全局变量缓存值；

## Nested/Local/Inner Classes and Functions

    嵌套定义class和function是允许的

class可以定义在method/function/class中，function可以定义在method/function中。嵌套的函数对于上层的变量是只读的；适用于在一个作用域内定义工具类和函数。

## List Comprehensions

    适用于简单的情形

例如：

Yes:

```python
result = []
    for x in range(10):
        for y in range(5):
            if x * y > 10:
                result.append((x, y))

    for x in xrange(5):
        for y in xrange(5):
            if x != y:
                for z in xrange(5):
                    if y != z:
                        yield (x, y, z)

    return ((x, complicated_transform(x))
            for x in long_generator_function(parameter)
            if x is not None)

    squares = [x * x for x in range(10)]

    eat(jelly_bean for jelly_bean in jelly_beans
        if jelly_bean.color == 'black')
```

No:

```python
result = [(x, y) for x in range(10) for y in range(5) if x * y > 10]

return ((x, y, z)
        for x in xrange(5)
        for y in xrange(5)
        if x != y
        for z in xrange(5)
        if y != z)
```

## Default Iterators and Operators

    如果类型支持（如list/dictionary/file)，使用默认的iterator和operator。

容器类型，如list/dictionary定义了默认的iterator和成员测试操作符(`in`和`not in`)，使用这些默认的iterator和operator，简单高效，没有额外的函数调用开销，例如：

Yes:

```python
for key in adict: ...
if key not in adict: ...
if obj in alist: ...
for line in afile: ...
for k, v in dict.iteritems(): ...
```

No:

```python
for key in adict.keys(): ...
if not adict.has_key(key): ...
for line in afile.readlines(): ...
```

## Generators

    在需要的时候使用generator.

代码更简单，相比函数一次创建整个list，generator消耗的内存更少。

## Lambda Functions

    适用于`一行代码`的情形。

Lambda表达式就是匿名函数，一般作为`map()`/`filter()`等高阶函数的回调或操作符；
如果lambda表达式中的代码比较长（超过60-80字符），最好定义成函数；
常用的操作，比如乘法运算，推荐使用`operator`模块(`operator.mul`)，而不是lambda函数(`lambda x, y: x * y`)。

## Conditional Expressions

    适用于`一行代码`的情形。

条件表达式就是if语句的简写形式，代码更短也更方便，但是如果表达式语句较长，则条件可能不容易定位。
适用于只有一行代码的情形，其它情况使用if语句。

## Default Argument Values

    大部分情况下都是可以的。

函数的参数可以使用默认值，但是不要将可变对象作为函数的默认值：

Yes:

```python
def foo(a, b=None):
            if b is None:
                b = []
```

No:

```python
No:  def foo(a, b=[]):
            ...
No:  def foo(a, b=time.time()):  # The time the module was loaded???
            ...
No:  def foo(a, b=FLAGS.my_thing):  # sys.argv has not yet been parsed...
            ...
```

## True/False evaluations

    尽量使用`隐式`的false判断。

所有含义为空的值，如`0`/`None`/`[]`/`{}`/`''`都是false；

+ 使用`if foo:`，而不是`if foo != []`;
+ 不要使用`==`或`!=`去比较`None`，应该用`is`或`is not`；
+ 不要使用`==`去比较`False`，应该使用`if not x:`; 如果要区分`False`和`None`，使用
`if not x and x is not None:`；
+ 对于列表类型(string/list/tuple)，空值表示false，使用`if not seq:`或`if seq:`，比使用`if len(seq):`或
`if not len(seq):`更好；
+ 注意，'0'是true；
+ 对于整数，必须额外小心，不要将`None`当作0处理：

Yes:

```python
if not users:
        print 'no users'

if foo == 0:
        self.handle_zero()

if i % 10 == 0:
        self.handle_multiple_of_ten()
```

No:

```python
if len(users) == 0:
        print 'no users'

if foo is not None and not foo:
        self.handle_zero()

if not i % 10:
        self.handle_multiple_of_ten()
```

## Threading

    不要依赖内置类型的原子行(atomicity)

有一些内置类型(如dictionary)的某些操作看起来是原子性的，但是它们不可靠；
使用**Queue**模块中的`Queue`作为线程间交互的数据结构，或者使用**threading**模块。


# Python Style Rules

## Semicolons

    语句结尾不要使用分号，也不要通过分号将两条语句写在同一行上。

## Line length

    最大行宽为80个字符，除非：1. 很长的import语句；2. 注释中的URL。

不要使用反斜线(`\`)做行连接。
使用`(), [], {}`的隐式连接方式，如果有必要，可以将表达式放在额外的括号中：

<!-- more -->

Yes:

```python
foo_bar(self, width, height, color='black', design=None, x='foo',
        emphasis=None, highlight=0)

    if (width == 0 and height == 0 and
        color == 'red' and emphasis == 'strong'):
```

如果字符串太长，一行容不下，使用括号进行隐式连接：

```python
x = ('This will build a very long long '
        'long long long long long long string')
```

在注释中，URL始终在一行显示：

Yes:  

```python
# See details at
# http://www.example.com/us/developer/documentation/api/content/v2.0/csv_file_name_extension_full_specification.html
```

No:

```python
# See details at
# http://www.example.com/us/developer/documentation/api/content/\
# v2.0/csv_file_name_extension_full_specification.html
```

## Parentheses

    括号能不用则不用。

在条件判断、返回语句中不要使用括号，除非用于较长字符串的隐式连接。
tuple用括号是可以的。

Yes:

```python
if foo:
    bar()
while x:
    x = bar()
if x and y:
    bar()
if not x:
    bar()
return foo
for (x, y) in dict.items(): ...
```

No:

```python
if (x):
    bar()
if not(x):
    bar()
return (foo)
```

## Indentation

    代码缩进使用4个空格。

不要使用tab或混用tab和空格。
在隐式行连接时，上一行如果没有参数，则两行元素垂直对齐；如果上一行有参数，则下一行缩进4个空格。

Yes:   

```python
# Aligned with opening delimiter
foo = long_function_name(var_one, var_two,
                            var_three, var_four)

# Aligned with opening delimiter in a dictionary
foo = {
    long_dictionary_key: value1 +
                            value2,
    ...
}

# 4-space hanging indent; nothing on first line
foo = long_function_name(
    var_one, var_two, var_three,
    var_four)

# 4-space hanging indent in a dictionary
foo = {
    long_dictionary_key:
        long_dictionary_value,
    ...
}
```

No:

```python
# Stuff on first line forbidden
foo = long_function_name(var_one, var_two,
    var_three, var_four)

# 2-space hanging indent forbidden
foo = long_function_name(
    var_one, var_two, var_three,
    var_four)

# No hanging indent in a dictionary
foo = {
    long_dictionary_key:
        long_dictionary_value,
        ...
}
```

## Blank Lines

    顶层(top-level)的定义之间空两行，method之间空一行。

top-level定义，无论是function还是class，空两行。
method之间，以及class与第一个method之间，空一行。

## Whitespace

    按照标准排印规则使用空格。

`(), [], {}`里面不要使用空格：

Yes:

```python
spam(ham[1], {eggs: 2}, [])
```

No:  

```python
spam( ham[ 1 ], { eggs: 2 }, [ ] )
```

逗号，分号和冒号前面没有空格。除非是在一行的末尾，否则逗号、分号和冒号的后面需要使用空格：

Yes:

```python
if x == 4:
    print x, y
    x, y = y, x
```

No:  

```python
if x == 4 :
    print x , y
x , y = y , x
```

作为参数列表或下标索引的小括号()和中括号[]的前面不要使用空格：

Yes:

```python
spam(1)
```

No:  

```python
spam (1)
```

Yes:

```python
dict['key'] = list[index]
```

No:  

```python
dict ['key'] = list [index]
```

二元操作符(`==, >, < !=, <>, <=, >=, in, not in, is, is not, and, or, not`)的前后各使用一个空格：

Yes:

```python
x == 1
```

No:  

```python
x<1
```

`=`用于参数默认值时，前后不要使用空格：

Yes:

```python
def complex(real, imag=0.0): return magic(r=real, i=imag)
```

No:  

```python
def complex(real, imag = 0.0): return magic(r = real, i = imag)
```

对于连续的行，不要通过空格去垂直对齐(主要是`=`和`#`):

Yes:

```python
foo = 1000  # comment
long_name = 2  # comment that should not be aligned

dictionary = {
    'foo': 1,
    'long_name': 2,
}
```

No:

```python
foo       = 1000  # comment
long_name = 2     # comment that should not be aligned

dictionary = {
    'foo'      : 1,
    'long_name': 2,
}
```

## Shebang Line

    大多数的`.py`文件都不需要`#!`行，仅在main文件的第一行使用`#!/usr/bin/python`，版本号2/3后缀是可选的。

`#!`行被kernel用于查找Python解释器，但是在import module的时候被忽略，所以仅当文件被直接运行的时候才需要。

## Comments

    确保正确地使用各种不同形式的注释：module/function/method/inline.

+ Doc Strings

对于**doc strings**，建议总是使用三个双引号(`"""`)形式：与`"""`同一行是注释的概述，然后空一行，与`"""`缩进相同的位置开始是注释的详细说明。

+ Modules

每一个文件都应该包含合适的`licence`引用信息（比如Apache 2.0, BSD, LGPL, GPL）。

+ Functions and Methods

function必须包含docstring，除非：1. 不被外部使用；2. 非常短；3. 非常明显；

docstring中应该包含调用function的所有信息，而不需要阅读function的代码。docstring应该描述调用函数的语法，而不是函数的实现。对于复杂代码实现，代码旁的注释比docstring更合适。

function的docstring分为不同的section：`Args`, `Returns`, `Raises`，section名后面使用冒号，section描述缩进显示。

`Args`: 依次列出参数名，后跟一个冒号和空格，参数的描述应该包含需要的类型和参数的含义；参数名之间缩进对齐。如果函数接收可变参数列表(*foo)，或任意关键字(**bar)，应该以*foo和**bar列出。

`Returns`: （对于generator，是`Yields`），返回值的含义，如果返回None，则该section可以省略。

`Raises`: 列出所有的异常；

```python
def fetch_bigtable_rows(big_table, keys, other_silly_variable=None):
    """Fetches rows from a Bigtable.

    Retrieves rows pertaining to the given keys from the Table instance
    represented by big_table.  Silly things may happen if
    other_silly_variable is not None.

    Args:
        big_table: An open Bigtable Table instance.
        keys: A sequence of strings representing the key of each table row
            to fetch.
        other_silly_variable: Another optional variable, that has a much
            longer name than the other args, and which does nothing.

    Returns:
        A dict mapping keys to the corresponding table row data
        fetched. Each row is represented as a tuple of strings. For
        example:

        {'Serak': ('Rigel VII', 'Preparer'),
            'Zim': ('Irk', 'Invader'),
            'Lrrr': ('Omicron Persei 8', 'Emperor')}

        If a key from the keys argument is missing from the dictionary,
        then that row was not found in the table.

    Raises:
        IOError: An error occurred accessing the bigtable.Table object.
    """
    pass
```

+ Classes

docstring应该位于class的定义下面，如果class包含public的属性，使用`Attributes`依次列出，格式和函数参数相同。

```python
class SampleClass(object):
    """Summary of class here.

    Longer class information....
    Longer class information....

    Attributes:
        likes_spam: A boolean indicating if we like SPAM or not.
        eggs: An integer count of the eggs we have laid.
    """

    def __init__(self, likes_spam=False):
        """Inits SampleClass with blah."""
        self.likes_spam = likes_spam
        self.eggs = 0

    def public_method(self):
        """Performs operation blah."""
```

+ Block and Inline Comments

对于复杂的逻辑，注释应该在逻辑的上面单独说明，对于简单但不明显的逻辑，注释放在代码的行末，但是与代码至少有2个空格的间距。

```python
# We use a weighted dictionary search to find out where i is in
# the array.  We extrapolate position based on the largest num
# in the array and the array size and then do binary search to
# get the exact number.

if i & (i-1) == 0:        # true iff i is a power of 2
```

永远不要试图描述你的代码。要假设阅读代码的人比你更懂Python：

```python
# BAD COMMENT: Now go through the b array and make sure whenever i occurs
# the next element is i+1
```

## Classes

    如果class没有继承别的class，则显式继承`object`，嵌套的class也是如此。

继承`object`可以使**properties**正常工作，同时也避免了与Python 3的`new style class`的不兼容。而且，继承`object`，预定义了很多默认的属性和method，如`__new__, __init__, __delattr__, __getattribute__, __setattr__, __hash__, __repr__, __str__`。

Yes:

```python
class SampleClass(object):
        pass


class OuterClass(object):

    class InnerClass(object):
        pass


class ChildClass(ParentClass):
"""Explicitly inherits from another class already."""
```

No:

```python
class SampleClass:
    pass


class OuterClass:

    class InnerClass:
        pass
```

## Strings

    使用`format()`或`%`格式化字符串，即使所有的参数都是string。

Yes:    

```python
x = a + b
x = '%s, %s!' % (imperative, expletive)
x = '{}, {}!'.format(imperative, expletive)
x = 'name: %s; score: %d' % (name, n)
x = 'name: {}; score: {}'.format(name, n)
```

No:

```python
x = '%s%s' % (a, b)  # use + in this case
x = '{}{}'.format(a, b)  # use + in this case
x = imperative + ', ' + expletive + '!'
x = 'name: ' + name + '; score: ' + str(n)
```

不要使用`+`和`+=`在循环中拼接字符串。因为字符串是不可变的，这样会创建很多不必要的临时对象，导致运行时间是乘方级的，而不是线性的。更好地做法应该是，循环将各个字串放到list中，循环结束后通过`''.join()`连接：

Yes:

```python
items = ['<table>']
for last_name, first_name in employee_list:
    items.append('<tr><td>%s, %s</td></tr>' % (last_name, first_name))
    items.append('</table>')
employee_table = ''.join(items)
```

No:

```python
employee_table = '<table>'
for last_name, first_name in employee_list:
    employee_table += '<tr><td>%s, %s</td></tr>' % (last_name, first_name)
employee_table += '</table>'
```

在同一个文件中，对于字符串引号的使用要保持一致。使用`''`或`""`都可以，保持一致即可。两者可以同时使用，避免通过`\`转义。

Yes:

```python
Python('Why are you hiding your eyes?')
Gollum("I'm scared of lint errors.")
Narrator('"Good!" thought a happy Python reviewer.')
```

No:

```python
Python("Why are you hiding your eyes?")
Gollum('The lint. It burns. It burns us.')
Gollum("Always the great lint. Watching. Watching.")
```

如果string占多行，建议使用`"""`，而不是`'''`。
当且仅当字符串使用`''`表示，多行字符串可以使用`'''`表示。
docstring总是使用`"""`，无论什么情况下。
通常，对于多行字符串使用隐式连接(`()`)更清晰易读：

Yes:

```python
print ("This is much nicer.\n"
        "Do it this way.\n")
```

No:

```python
    print """This is pretty ugly.
Don't do this.
"""
```

## Files and Sockets

    使用完之后，要显式关闭file和socket。

没有合理地关闭file或socket会带来很多问题：

- 会消耗很多系统资源，比如文件描述符。如果这类的对象很多，可能会耗尽系统资源。
- file处于未关闭状态，可能会导致其它的操作不可用，比如移动或删除。
- 被共享的file和socket，可能被意外地读写，如果被显式关闭，则读写会立即产生异常。

当file对象被销毁的时候，file和socket会被自动关闭，但是将file对象的存活期与其状态绑定并不是好的实践：

- 无法保证runtime一定会执行file对象的析构函数。
- 对file对象意外的引用可能会延长其存活期。

推荐使用`with`语句操作file：

```python
with open("hello.txt") as hello_file:
    for line in hello_file:
        print line
```

对于不支持`with`的类似对象，可以使用`contextlib.closing()`:

```python
import contextlib

with contextlib.closing(urllib.urlopen("http://www.python.org/")) as
    front_page:
    for line in front_page:
        print line
```

## TODO Comments

    `TODO`注释用于临时的、短期的或不够优化的解决方法。

`TODO`的格式：TODO大写，后跟的小括号内是用户名/邮箱等，表示应该关注该TODO的人，接下来的冒号是可选的，后面的注
释表示TODO的内容。
TODO的用户并不一定是fix这个问题的人，所以这里的用户几乎都是自己。

```python
# TODO(kl@gmail.com): Use a "*" here for string repetition.
# TODO(Zeke) Change this to use relations.
```

如果`TODO`表示的是“在将来的某个时间点修复”，则务必包含具体的日期(`2009年11月前修复`)或事件
(`当所有的客户端都可以处理XML结果时删除这段代码`)。

## Imports formatting

    import语句应该独占一行。

Yes:

```python
import os
import sys
```

No:

```python
import os, sys
```

import总是位于文件的顶部，在module的注释和docstring的后面，而在module的全局变量和常量的前面。
import应该根据通用性进行分组：

- import标准库
- import第三方库
- import应用特定的库

在每一个分组中，import应该根据module的全包名按照字母序排列：

```python
import foo
from foo import bar
from foo.bar import baz
from foo.bar import Quux
from Foob import ar
```

## Statements

    通常，一行仅允许一条语句。

当if没有else分支，且一行可以容纳if的结果语句时，可以将if的结果语句和if的判断语句写在同一行。
不能将`try/except`的语句放在同一行：

Yes:

```python
if foo: bar(foo)
```

No:

```python
if foo: bar(foo)
else:   baz(foo)

try:               bar(foo)
except ValueError: baz(foo)

try:
    bar(foo)
except ValueError: baz(foo)
```

## Naming

    命名规范：module_name, package_name, ClassName, method_name, ExceptionName,
    function_name, GLOBAL_CONSTANT_NAME, global_var_name, instance_var_name,
    function_parameter_name, local_var_name.

避免使用的命名：

- 除了计数(counter)和迭代(iterator)，不要使用单字符变量名。
- 不要在package/module名中使用连字符(-)。
- 不要使用双下划线开头和双下划线结尾的变量(__double_leading_and_trailing__)。

命名规范：

- **Internal**表示module内部的，或者class的`private`或`protected`。
- 以`_`开头表示module内部的变量或函数(不包含在`import * from`)，使用`__`开头表示class的private变量或函数。
- 将相关的class和顶层的function放在一个module里。与Java不同的是，一个module中可以包含多个class。
- class使用首字母大写(CapWord)的命名方式，而module名使用小写和下划线(lower_with_uder.py)形式。

Python之父Guido推荐的命名规范：

| Type | Public | Internal |
| :---:  |  :---: |  :---: |
| Packages | lower_with_under |
| Modules | lower_with_under | _lower_with_under |
| Classes | CapWords | _CapWords |
| Exceptions | CapWords |
| Functions | lower_with_under() | _lower_with_under() |
| Global/Class Constants | CAPS_WITH_UNDER | _CAPS_WITH_UNDER |
| Global/Class Variables | lower_with_under | _lower_with_under |
| Instance Variables | lower_with_under | _lower_with_under (protected) or __lower_with_under (private) |
| Method Names | lower_with_under() | _lower_with_under() (protected) or __lower_with_under() (private) |
| Function/Method Parameters | lower_with_under |
| Local Variables | lower_with_under |

## Main

    即使是脚本文件，也应该是可以被其它文件import的，因此在import时，不要产生副作用，即
    不要执行脚本的主功能。主功能应该被放在`main()`函数里。

`pydoc`和单元测试都需要import文件，所以文件中应该总是添加`if __name__ == '__main__'`确保当module被import的时候，不会执行其主功能。

```python
def main():
    ...

if __name__ == '__main__':
    main()
```

在module被import的时候，所有顶层(top-level)的代码都会被执行。


## 参考

+ [Google Python Style Guide](https://google.github.io/styleguide/pyguide.html)
