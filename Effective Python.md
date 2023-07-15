# Effective Python

## Chapter Ⅰ: Pythonic Thinking

### Item 1: Know Which Version of Python You're Using

### Item 2: Follow the PEP 8 Style Guide

> Pylint: 自动PEP8工具

* 文件中, 函数或类之间相隔两个空行. 单个类中, 函数之间间隔一个空行.

* 命名规则:

  * Functions, variables, and attributes: `lowercase_ underscore` 
  * Protected instance attributes: `_leading_underscore`
  * Private instance attributes: `__double_leading_ underscore`
  * Classes (including exceptions) : `CapitalizedWord`
  * Module-level constants: `ALL_CAPS`
  * Instance methods in classes: `self`
  * Class methods: `cls`

  > ```python
  > class Person:
  >     count = 0
  > 
  >     def __init__(self, name):
  >         self.name = name
  >         Person.count += 1
  > 
  >     @classmethod
  >     def get_count(cls):
  >         return cls.count
  > # 创建Person类的实例
  > person1 = Person("John")
  > person2 = Person("Alice")
  > 
  > # 调用类方法
  > print(Person.get_count())  # 输出: 2
  > ```

* 表达语句:
  * 不通过检查长度`if len(somelist) == 0`来检查空容器或字符串`(like [] or '') `, 使用 `if not somelist` 默示其不存在. 同理, 使用` if somelist` 默示其存在
  * 多行表达式使用圆括号分割, 而不是使用续行符`\`
* 引用包: 
  * 引用顺序: 标准库, 第三方库, 自己的module. 每部分引用使用字母序排列

### Item 3: Know the Differences Between bytes and str 

在写Python程序时，最好在接口的最远边界对Unicode数据进行编码和解码，这种方法通常被称为Unicode三明治。你的程序的核心应该使用包含Unicode数据的str类型，并且不应假设任何关于字符编码的事情。这种方法让你可以非常接受替代文本编码（如Latin-1，Shift JIS和Big5），同时严格控制你的输出文本编码（理想情况下，是UTF-8）

> Python 中 有两种表示字符序列的方法 `bytes` 和 `str` . 
>
> * `bytes` 包含 raw unsigned 8-bit values. (often displayed in the ASCII encoding)
> * `str` 包含 Unicode 编码 表示文本字符串
>
> ```python
> # bytes
> a = b'h\x65llo'
> print(list(a))
> print(a)
> >>>
> [104, 101, 108, 108, 111]
> b'hello'
> 
> # str
> a = 'a\u0300 propos'
> print(list(a))
> print(a)
> >>>
> ['a', '`', ' ', 'p', 'r', 'o', 'p', 'o', 's']
> à propos
> ```
>
> 实现这两种编码方式的转换, 必须调用encode和decode方法
>
> * `encode` convert Unicode data to binary data
> * `decode` convert binary data to Unicode data

>  Takes a bytes or str instance and always returns  a str
>
> ```python
> def to_str(bytes_or_str):
> 	if isinstance(bytes_or_str, bytes):
>     	value = bytes_or_str.decode('utf-8')
>     else:
>     	value = bytes_or_str
>     return value # Instance of str
> print(repr(to_str(b'foo')))
> print(repr(to_str('bar')))
> >>>
> 'foo'
> 'bar'
> ```
>
> Takes a bytes or str instance and always returns  a bytes
>
> ```python
> def to_bytes(bytes_or_str):
> 	if isinstance(bytes_or_str, str):
> 		value = bytes_or_str.encode('utf-8')
> 	else:
>         value = bytes_or_str
> 	return value # Instance of bytes
> ```

* 如果您想读取或写入二进制数据，请始终使用二进制模式（如 "rb "或 "wb"）打开文件。

  ```python
  with open('data.bin', 'wb') as f:
  	f.write(b'\xf1\xf2\xf3\xf4\xf5')
  ```

* 如果您想读取或写入 Unicode 数据，请注意系统的默认文本编码。如果您想避免意外，请向 open 明确传递编码参数。

  ` python3 -c 'import locale; print(locale.getpreferredencoding())'`

  ```python
  with open('data.bin', 'r', encoding='cp1252') as f:
  	data = f.read()
  assert data == 'ñòóôõ'
  ```

### Item 4: Prefer Interpolated F-Strings Over C-style Format Strings and str.format 

```python
# 类C方法
new_way = '%(key)-10s = %(value).2f' % {
 'key': key, 'value': value} # Original

# Format 方法
formatted = '{1} = {0}'.format(key, value)

# Interpolated F-Strings 方法
key = 'my_var'
value = 1.234
formatted = f'{key} = {value}'
```

```python
for i, (item, count) in enumerate(pantry):
	old_style = '#%d: %-10s = %d' % (
		i + 1,
		item.title(),
		round(count))
	new_style = '#{}: {:<10s} = {}'.format(
		i + 1,
		item.title(),
		round(count))
	f_string = f'#{i+1}: {item.title():<10s} = {round(count)}'
	assert old_style == new_style == f_string
```

#### C-style 问题

1. 类型转换不兼容：如果改变了格式化表达式右侧元组中数据值的类型或顺序，可能会因为类型转换不兼容而出错。
2. 可读性差：当需要对值进行微小的修改再将其格式化为字符串时，C-style格式化表达式变得难以阅读。
3. 重复使用值：如果在格式字符串中多次使用同一值，必须在右侧元组中重复这个值。
4. 冗余和冗长：虽然使用字典可以解决问题1和3，但是它引入了其他问题。比如，它在格式化表达式中引入了更多的字符（字典的键和冒号操作符），使得表达式变得更长和视觉噪音更多。此外，每个键至少要指定两次——一次在格式说明符中，一次在字典的键中，可能还需要一次作为包含字典值的变量名。这导致了格式化表达式变得过于冗长，这些表达式往往需要跨多行，格式字符串需要跨多行连接，字典赋值需要每个值一行。

#### str.format 

1. `str.format`方法为Python提供了比旧的C风格格式化字符串更强大的功能，允许使用者用`{}`作为占位符并通过`.`语法进行格式化，如小数位数、字符串长度等。并且支持在占位符中使用索引或关键字来引用特定的参数，比如`'{0} = {1}'.format(key, value)`。
2. 通过`{}`作为占位符，可以避免在格式化字符串时误解析`%`符号为占位符。例如，在C风格格式化字符串中，你需要通过双`%`来转义`%`，而在`str.format`方法中，你需要通过双大括号`{{}}`来转义大括号。
3. `str.format`方法支持在格式化字符串中重复引用相同的参数，而无需多次传入，例如`'{0} loves food. See {0} cook.'.format(name)`。

然而，尽管`str.format`方法带来了许多新的功能和便利，但它也存在一些问题：

1. 对于需要在格式化之前对值进行小改动的情况，`str.format`并没有提供便利，仍然需要在格式化字符串之前处理这些值。
2. 在使用诸如字典键和列表索引等高级特性进行格式化时，`str.format`方法的代码冗余度高，如使用字典键作为关键字参数进行格式化。

#### Interpolated F-Strings

1. **表达力**：f-string可以在格式化表达式中直接引用当前Python作用域中的所有变量，从而完全消除了在`str.format`中提供键和值的冗余性，例如`f'{key} = {value}'`。
2. **格式化选项**：f-string支持与`str.format`相同的所有格式化选项，例如在占位符中使用冒号指定格式化参数，以及将值强制转换为Unicode和repr字符串，如`f'{key!r:<10} = {value:.2f}'`。
3. **简洁性**：f-string的使用要比C风格的格式化字符串以及`str.format`方法更短、更简洁。
4. **支持Python表达式**：f-string允许在占位符中使用完整的Python表达式，这使得对格式化值进行小修改变得十分方便，例如`f'#{i+1}: {item.title():<10s} = {round(count)}'`。
5. **多行格式化**：如果需要，你也可以将f-string分割成多行，从而增加代码的清晰度。Python会将多行的f-string自动拼接起来，形成一个字符串。
6. **动态精度**：f-string甚至允许在格式化指定符选项中使用Python表达式。例如，你可以通过一个变量来动态决定小数的位数，如`f'My number is {number:.{places}f}'`。

总的来说，f-string结合了表达力、简洁性和清晰性，是Python程序员的最佳内置选择，无论何时需要将值格式化为字符串，都应优先考虑使用f-string。

### Item 5: Write Helper Functions Instead of Complex Expressions

```python
# 复杂的表达式
red = int(my_values.get('red', [''])[0] or 0)

# 优化后的代码
def get_first_int(values, key, default=0):
    found = values.get(key, [''])
    if found[0]:
        return int(found[0])
    return default

red = get_first_int(my_values, 'red')
```

### Item 6: Prefer Multiple Assignment Unpacking Over Indexing

Python中的多重赋值拆包功能，它可以更直观地处理元组或列表的元素，而不必使用索引进行访问。

1. Python具有内置的元组类型，可以创建不可变、有序的值序列。一旦创建了元组，就不能通过给索引赋新值来修改它。
2. Python还具有拆包的语法，这允许在单个语句中分配多个值。例如，如果你知道元组是一对值，可以将它赋给两个变量名，而不是使用索引来访问它们。
3. 拆包的语法比使用元组的索引产生更少的视觉噪声，且通常需要更少的代码行。这种模式匹配语法也适用于列表、序列，甚至可以在任意深度的嵌套中进行。
4. 拆包可以在不需要创建临时变量的情况下交换两个值。例如，在实现冒泡排序算法时，可以使用拆包语法在一行内交换列表中两个位置的值。
5. 拆包的另一个有价值的应用是在for循环的目标列表，以及类似的构造中，如列表解析和生成器表达式等。使用拆包和内置的enumerate函数可以更简洁地迭代列表。
6. 通过使用拆包，可以避免在可能的情况下使用索引，从而使代码更清晰、更Pythonic。

```python
# 普通索引访问
item = ('Peanut butter', 'Jelly')
first = item[0]
second = item[1]
print(first, 'and', second)

# 多重赋值拆包
item = ('Peanut butter', 'Jelly')
first, second = item # Unpacking
print(first, 'and', second)

# 在循环中使用解包
snacks = [('bacon', 350), ('donut', 240), ('muffin', 190)]
for rank, (name, calories) in enumerate(snacks, 1):
 print(f'#{rank}: {name} has {calories} calories')

# 值交换
# 传统的值交换方式
temp = a[i]
a[i] = a[i-1]
a[i-1] = temp

# 使用多重赋值拆包的值交换
a[i-1], a[i] = a[i], a[i-1]  # Swap
```

### Item 7: Prefer enumerate Over range

### Item 8: Use zip to Process Iterators in Parallel

- `zip` 内置函数可以用来**并行迭代多个迭代器**。
- `zip` 创建了一个惰性生成器，生成的是元组，所以它可以用于无限长的输入。
- 如果提供给 `zip` 的迭代器长度不一，`zip` 会静默地截断其输出到最短的迭代器长度。
- 如果你希望在不同长度的迭代器上使用 `zip` 而不进行截断，可以使用 `itertools` 内置模块的 `zip_longest` 函数。

```python
# 原始列表
names = ['Cecilia', 'Lise', 'Marie', 'Rosalind']
lengths = [7, 4, 5]

# 使用zip进行并行迭代
for name, length in zip(names, lengths):
    print(f'{name}: {length}')
```

### Item 9: Avoid else Blocks After for and while Loops

### Item 10: Prevent Repetition with Assignment Expressions

这段文字是关于在Python编程中避免重复使用赋值表达式的讨论，特别是利用“海象操作符”（walrus operator）`:=`。

海象操作符首次出现在Python 3.8中，旨在解决代码重复的问题。它的工作方式是在一个表达式中进行赋值并进行求值，这使得在原本不允许赋值语句的地方（例如if语句的条件表达式中）也可以进行变量赋值。

```python
# 模拟switch/case语句
if (count := fresh_fruit.get('banana', 0)) >= 2:
    pieces = slice_bananas(count)
    to_enjoy = make_smoothies(pieces)
elif (count := fresh_fruit.get('apple', 0)) >= 4:
    to_enjoy = make_cider(count)
elif count := fresh_fruit.get('lemon', 0):
    to_enjoy = make_lemonade(count)
else:
    to_enjoy = 'Nothing'
```

## Chapter Ⅱ: Lists and Dictionaries

### Item 11: Know How to Slice Sequences

- 在切片时避免冗长，不要为开始索引提供0，或者为结束索引提供序列的长度。

- 切片赋值将会替换原始序列中的该范围，即使长度不同。

  ```python
  a = ['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h']
  a[2:7] = [99, 22, 14]
  # a 现在是 ['a', 'b', 99, 22, 14, 'h']
  ```

### Item 12: Avoid Striding and Slicing in a Single Expression

Python有一种特殊的语法，能够在一个序列切片时设置步进，形式为somelist[start\:end:stride]。这让你可以在切片一个序列时，选择每n个项目。

然而，问题在于步进语法往往会导致意料之外的行为，从而引入bug。例如，逆转一个字节字符串的常见Python技巧是用-1的步进对字符串进行切片。这在Unicode字符串上也能正确工作。但是当Unicode数据被编码为UTF-8字节字符串时，这将会出错

```python
w = '寿司'
x = w.encode('utf-8')
y = x[::-1]
z = y.decode('utf-8')
>>>
Traceback ...
UnicodeDecodeError: 'utf-8' codec can't decode byte 0xb8 in 
position 0: invalid start byt
```

为了避免问题，建议避免在切片中同时使用步进以及开始和结束索引。如果必须使用步进，更推荐让步进保持为正值并省略开始和结束索引。如果必须使用步进和开始或结束索引，考虑使用一个赋值操作来进行步进，再用另一个赋值操作进行切片：

```python
y = x[::2]  # ['a', 'c', 'e', 'g']
z = y[1:-1]  # ['c', 'e']
```

### Item 13: Prefer Catch-All Unpacking Over Slicing

> 当将列表划分为不重叠的部分时，全面解包比切片和索引更不易出错。

首先，解包在处理序列时可以提供更简洁和更易读的代码。通常，你需要知道你要解包的序列的长度，但是，Python 提供了一个所谓的"全面解包"功能，它通过使用星号表达式（*）来获取所有未匹配其他解包模式的值。例如

```python
car_ages_descending = [20, 19, 15, 9, 8, 7, 6, 4, 1, 0]
oldest, second_oldest, *others = car_ages_descending
print(oldest, second_oldest, others)
```

这种全面解包的方式，比索引和切片更简洁，更易读，且不会出现需要同步边界索引的易错性问题

此外，星号表达式可以出现在任何位置，无论是在序列的起始、中间还是末尾，都可以使用它来获取所需的子序列。然而，你不能单独使用星号表达式，也不能在单层解包模式中使用多个星号表达式。

星号表达式总是返回一个列表，如果序列中没有剩余的元素，它会返回一个空列表。当你知道序列至少有 N 个元素时，这一特性尤其有用。

在处理迭代器时，你也可以使用解包语法。例如，你有一个生成器，它生成了一个 CSV 文件的所有行。使用索引和切片处理这个生成器的结果是可以的，但是需要多行代码并且视觉上较为嘈杂。使用星号表达式进行解包，可以很容易地将第一行（即头部）与迭代器的其他内容分开处理，这样做更为清晰：

```python
def generate_csv():
    yield ('Date', 'Make' , 'Model', 'Year', 'Price')
    # ...

it = generate_csv()
header, *rows = it
print('CSV Header:', header)
print('Row count: ', len(rows))
```

### Item 14: Sort by Complex Criteria Using the key Parameter

- `sort()`方法可以用于根据自然排序来重新排列列表的内容。
- 如果对象没有定义自然排序，`sort()`方法无法工作。
- 可以使用`sort()`方法的`key`参数提供一个函数，该函数返回用于排序的值。
- 通过从`key`函数返回元组，可以组合多个排序条件。可以使用一元减运算符来反转排序顺序。
- 对于无法进行负运算的类型，可以通过多次调用`sort()`方法，并使用不同的`key`函数和`reverse`值来组合多个排序条件。

> - Python的`list`类型提供了一个`sort()`方法用于根据各种标准对列表实例进行排序。默认情况下，`sort()`会按照列表项的自然升序排列。
>
> ```python
> pythonCopy codenumbers = [93, 86, 11, 68, 70]
> numbers.sort()
> print(numbers)  # 输出：[11, 68, 70, 86, 93]
> ```
>
> - 如果列表项是自定义对象，`sort()`方法可能无法工作，除非对象定义了自然排序。一种常见的做法是通过`sort()`方法的`key`参数来提供一个函数，这个函数会对每个列表项进行一些操作，返回一个可以进行自然排序的值。
>
> ```python
> pythonCopy codeclass Tool:
>     def __init__(self, name, weight):
>         self.name = name
>         self.weight = weight
>     def __repr__(self):
>         return f'Tool({self.name!r}, {self.weight})'
> 
> tools = [
>     Tool('level', 3.5),
>     Tool('hammer', 1.25),
>     Tool('screwdriver', 0.5),
>     Tool('chisel', 0.25),
> ]
> 
> tools.sort(key=lambda x: x.name)
> print(tools)  # 输出：[Tool('chisel', 0.25), Tool('hammer', 1.25), Tool('level', 3.5), Tool('screwdriver', 0.5)]
> ```
>
> - `key`函数也可以用于对字符串进行转换操作，如将所有字符串转换为小写后再进行排序。
>
> ```python
> pythonCopy codeplaces = ['home', 'work', 'New York', 'Paris']
> places.sort(key=lambda x: x.lower())
> print(places)  # 输出：['home', 'New York', 'Paris', 'work']
> ```
>
> - 如果需要使用多个排序标准，可以返回一个包含所有排序标准的元组。这对于需要同时按照升序和降序排序的情况非常有用。
>
> ```python
> pythonCopy codepower_tools = [
>     Tool('drill', 4),
>     Tool('circular saw', 5),
>     Tool('jackhammer', 40),
>     Tool('sander', 4),
> ]
> power_tools.sort(key=lambda x: (-x.weight, x.name))
> print(power_tools)  # 输出：[Tool('jackhammer', 40), Tool('circular saw', 5), Tool('drill', 4), Tool('sander', 4)]
> ```
>
> - 对于无法进行负值操作的类型，可以通过多次调用`sort()`方法并使用不同的`key`函数和`reverse`值来实现相同的效果。要注意的是，需要按照从最低等级排序到最高等级排序的顺序调用`sort()`方法。
>
> ```python
> pythonCopy codepower_tools.sort(key=lambda x: x.name)  # 按名字升序排序
> power_tools.sort(key=lambda x: x.weight, reverse=True)  # 按重量降序排序
> print(power_tools)  # 输出：[Tool('jackhammer', 40), Tool('circular saw', 5), Tool('drill', 4), Tool('sander', 4)]
> ```

### Item 15: Be Cautious When Relying on dict Insertion Ordering

在Python 3.5及之前的版本中，字典项的迭代顺序与插入顺序不一致，这是因为之前的字典类型通过使用内建的哈希函数和随机种子来实现哈希表算法，导致字典顺序不匹配插入顺序并在程序执行中随机混排。在Python 3.7及之后的版本中，字典会保持插入顺序，这意味着字典的打印顺序将始终与程序员最初创建的顺序相同。

### Item 16: Prefer get Over in and KeyError toHandle Missing Dictionary Keys

在Python中，常见的处理字典缺失键的方法有四种：使用in表达式，抛出KeyError异常，使用get方法，以及使用setdefault方法。

① 使用in表达式：这种方式会检查键是否存在于字典中，如果存在则获取键的值，否则将键的值设置为默认值。

```python
pythonCopy codekey = 'wheat'
if key in counters:
 count = counters[key]
else:
 count = 0
counters[key] = count + 1
```

② 抛出KeyError异常：如果尝试获取字典中不存在的键的值，Python会抛出KeyError异常。这种方式只需要访问一次键并进行一次赋值，所以相比使用in表达式更有效率。

```python
pythonCopy codekey = 'wheat'
try:
 count = counters[key]
except KeyError:
 count = 0
counters[key] = count + 1
```

③ 使用get方法：get方法会尝试获取键的值，如果键不存在于字典中，则返回第二个参数作为默认值。使用get方法也只需要访问一次键并进行一次赋值，代码更简洁。

```python
pythonCopy codekey = 'wheat'
count = counters.get(key, 0)
counters[key] = count + 1
```

④ 使用setdefault方法：setdefault方法会尝试获取键的值，如果键不存在于字典中，会将键的值设为默认值，并返回键的值。这种方式可以使代码更短，但其方法名可能会导致理解上的困扰。

```python
pythonCopy codekey = 'brioche'
who = 'Elmer'
names = votes.setdefault(key, [])
names.append(who)
```

虽然使用setdefault方法可以让代码更简短，但是这个方法并不总是最好的选择。这是因为setdefault方法会直接将默认值赋值给字典中的键，而不是复制一份。这可能导致意外的结果，尤其是当默认值是可变类型时。(但默认值是经典数值类型时比较不错)

在字典值为简单类型（如计数器）时，使用get方法是最简洁和明了的选择。在字典值为复杂类型（如列表）时，可以使用Python 3.8引入的赋值表达式(海象表达式 Item 10)使代码更简洁。如果在使用setdefault方法时发现性能开销大或可能引发异常，应考虑使用collections模块的Counter类或defaultdict类。

### Item 17: Prefer defaultdict Over setdefault toHandle Missing Items in Internal State

你应该考虑在以下情况中使用 `defaultdict`：

1. **当你需要字典的值是某种类型的集合（如list，set）并且你要对这些集合进行操作（例如添加元素）时。**在这种情况下，如果你使用常规的字典并尝试为一个不存在的键添加元素，你需要先检查键是否存在，如果不存在则先创建一个空集合。而`defaultdict`可以自动为你做这个。
2. **当你创建一个需要管理任意潜在键的字典时。**默认字典会自动为任何你访问但不存在的键创建一个默认值。这可以减少检查键是否存在的代码，使得你的代码更简洁。
3. **当你的字典需要一个不可变的默认值（如int，float）时。**这对于计数或计算总和非常有用。*<u>例如，如果你使用一个普通字典来计数，你需要先检查键是否存在，如果不存在，你需要设置默认值为0，然后再增加计数。`defaultdict(int)` 会自动将不存在的键的值设为0，你可以直接增加计数。</u>*

总的来说, 当你发现自己在编写额外的代码来处理字典中不存在的键时，`defaultdict` 可能就是你需要的工具。

> 1. 当你处理一个你没有创建的字典时，处理缺失键的方法有很多，`get`方法通常是比使用`in`表达式和`KeyError`异常更好的方法。然而，在一些使用情况下，`setdefault`看起来是最简洁的选项。
> 2. 本文中的例子是在记录世界各国我访问过的城市。在这种情况下，可以用`setdefault`方法将新城市添加到集合中，无论该国家名是否已经在字典中。
> 3. 但是，`setdefault`方法的名称可能让新读者感到困惑，难以立即理解代码的行为。而且，它的实现效率不高，因为它在每次调用时都会构造一个新的集合实例，无论给定的国家是否已经存在于数据字典中。
> 4. Python的`collections`模块中的`defaultdict`类可以简化这个常见的用例，自动存储一个键不存在时的默认值。你只需要提供一个函数，每**次键丢失时，它都会返回默认值。**
> 5. 使用`defaultdict`比使用`setdefault`更好，特别是在这种类型的情况下。当然，`defaultdict`也可能无法解决你的问题，但Python提供了更多的工具来解决这些限制。
>
> 让我们看下如何将`setdefault`改为使用`defaultdict`：
>
> 使用`setdefault`的代码：
>
> ```python
> pythonCopy codeclass Visits:
>     def __init__(self):
>         self.data = {}
> 
>     def add(self, country, city):
>         city_set = self.data.setdefault(country, set())
>         city_set.add(city)
> 
> visits = Visits()
> visits.add('Russia', 'Yekaterinburg')
> visits.add('Tanzania', 'Zanzibar')
> print(visits.data)
> ```
>
> 改为使用`defaultdict`的代码：
>
> ```python
> pythonCopy codefrom collections import defaultdict
> 
> class Visits:
>     def __init__(self):
>         self.data = defaultdict(set)
> 
>     def add(self, country, city):
>         self.data[country].add(city)
> 
> visits = Visits()
> visits.add('England', 'Bath')
> visits.add('England', 'London')
> print(visits.data)
> ```
>
> 通过比较，你可以看到使用`defaultdict`的代码更简洁，运行效率更高。

### Item 18: Know How to Construct Key-Dependent Default Values with \__missing__

如何通过 `__missing__` 方法来处理字典中不存在的键。当你无法预先确定所有可能的键，或者**默认值依赖于键，或者计算默认值的代价很高**时，`__missing__` 方法会非常有用。

在某些情况下，使用内置的`dict`类型的`setdefault`方法可以在处理缺失的键时使代码更短。在许多这样的情况下，更好的工具是使用内置模块`collections`的`defaultdict`类型。但是，有些时候，`setdefault`和`defaultdict`都不适合。

例如，如果你正在编写一个管理文件系统上社交网络头像的程序，你需要一个字典来映射头像路径名到开放的文件句柄，以便你可以根据需要读取和写入这些图片。可以通过使用常规`dict`实例和使用`get`方法以及赋值表达式（在Python 3.8中引入）来检查键的存在。

尝试使用`setdefault`方法和`defaultdict`都会遇到问题。在`setdefault`的例子中，即使路径已经在字典中，创建文件句柄的`open`函数也总是会被调用，这会导致一个额外的文件句柄，可能会与程序中已经打开的句柄冲突。

```python
try:
 handle = pictures.setdefault(path, open(path, 'a+b'))
except OSError:
 print(f'Failed to open path {path}')
 raise
else:
 handle.seek(0)
 image_data = handle.read()
```

在`defaultdict`的例子中，问题在于`defaultdict`期望传递给它的函数不需要任何参数，这意味着`defaultdict`调用的辅助函数不知道哪个特定的键正在被访问，从而无法调用`open`。

```python
from collections import defaultdict
def open_picture(profile_path):
 try:
 return open(profile_path, 'a+b')
 except OSError:
 print(f'Failed to open path {profile_path}')
 raise
pictures = defaultdict(open_picture)
handle = pictures[path]
handle.seek(0)
image_data = handle.read()
>>>
Traceback ...
TypeError: open_picture() missing 1 required positional 
argument: 'profile_path'
```

幸运的是，Python提供了另一种内置的解决方案。你可以子类化`dict`类型，并实现`__missing__`特殊方法来添加处理**缺失键的自定义逻辑**。

在以下代码示例中，我们将创建一个新的类，该类利用了上文定义的`open_picture`辅助方法：

```python
pythonCopy codeclass Pictures(dict):
    def __missing__(self, key):
        value = open_picture(key)
        self[key] = value
        return value

pictures = Pictures()
handle = pictures[path]
handle.seek(0)
image_data = handle.read()
```

在这个示例中，当`pictures[path]`字典访问发现路径键在字典中不存在时，`__missing__`方法被调用。这个方法必须为键创建新的默认值，将它插入到字典中，并返回给调用者。

> - 当创建默认值的计算成本较高或可能引发异常时，`dict`的`setdefault`方法不适合使用。
> - 传递给`defaultdict`的函数不能需要任何参数，这使得默认值不能依赖于正在访问的键。
> - 你可以定义自己的`dict`子类，并用`__missing__`方法来构造必须知道正在访问哪个键的默认值。****

## Chapter Ⅲ: Functions

### Item 19: Never Unpack More Than Three Variables When Functions Return Multiple Values

当一个函数返回四个或更多的值时，这会变得复杂和容易出错。首先，所有返回的值都是数字，所以很容易在无意中改变它们的顺序。其次，解包的代码行会变得很长，可能需要以各种方式折行，这会降低代码的可读性。

不要在解包时使用超过三个的变量。如果需要返回更多的值，最好定义一个轻量级的类或者 `namedtuple`，并让函数返回该类的实例。

```python
from collections import namedtuple

Stats = namedtuple('Stats', ['minimum', 'maximum', 'average', 'median', 'count'])

def get_stats(numbers):
 minimum = min(numbers)
 maximum = max(numbers)
 count = len(numbers)
 average = sum(numbers) / count
 sorted_numbers = sorted(numbers)
 middle = count // 2
 if count % 2 == 0:
   lower = sorted_numbers[middle - 1]
   upper = sorted_numbers[middle]
   median = (lower + upper) / 2
 else:
   median = sorted_numbers[middle]
 return Stats(minimum, maximum, average, median, count)

stats = get_stats(lengths)
print(f'Min: {stats.minimum}, Max: {stats.maximum}')
print(f'Average: {stats.average}, Median: {stats.median}, Count {stats.count}')
```

### Item 20: Prefer Raising Exceptions to Returning None

在Python中，应该优先使用异常（Exceptions）而不是返回None来处理特殊情况。

在具体代码实现中，返回None可能会导致错误，因为在条件表达式中，None和其他值（如零，空字符串）都会被解析为False。这可能会导致误解，比如一个函数返回了0，但在接下来的条件判断中，0和None都被当做False处理，导致程序错误

函数的最终版本包括类型注解和docstring，明确了函数输入、输出，以及可能引发的异常，使得函数的使用更加清晰，减少了错误的可能性。

```python
def careful_divide(a: float, b: float) -> float:
    """Divides a by b.
    Raises:
    ValueError: When the inputs cannot be divided.
    """
    try:
        return a / b
    except ZeroDivisionError as e:
        raise ValueError('Invalid inputs')
# 调用
x, y = 5, 2
try:
    result = careful_divide(x, y)
except ValueError:
    print('Invalid inputs')
else:
    print('Result is %.1f' % result)
```

### Item 21: Know How `Closures` Interact with Variable Scope

> Python 支持闭包，也就是说，函数可以引用在它们定义的作用域中的变量。例如，辅助函数`helper`能够访问`sort_priority`的参数`group`。
>
> Python中的函数是一等公民，这意味着你可以直接引用它们，将它们赋值给变量，将它们作为参数传递给其他函数，将它们用在表达式和if语句中等等。这就是`sort`方法可以接受一个闭包函数作为`key`参数的原因。

> 然后，该段落进一步讨论了当闭包尝试修改其封闭作用域的变量时可能出现的问题。如果尝试在闭包内部改变在外部作用域定义的变量值，Python将会在闭包内部创建一个新的变量。这会导致外部作用域的变量值并未发生变化，就像`sort_priority2`函数中发生的情况。
>
> 为了解决这个问题，Python 提供了`nonlocal`语句，它使得在闭包内部可以修改封闭作用域的变量。但是，该段落也建议我们在使用`nonlocal`时要小心，因为`nonlocal`的副作用可能难以跟踪，尤其是在长函数中。
>
> 当你的`nonlocal`的使用开始变得复杂时，更好的选择是将你的状态封装在一个辅助类中。例如，在最后一个例子中，`Sorter`类完成了与使用`nonlocal`相同的结果，虽然代码略长，但易于阅读。
>
> ```python
> # 基本的闭包使用
> def sort_priority(values, group):
>     def helper(x):
>         if x in group:
>             return (0, x)
>         return (1, x)
>     values.sort(key=helper)
> 
> numbers = [8, 3, 1, 2, 5, 4, 7, 6]
> group = {2, 3, 5, 7}
> sort_priority(numbers, group)
> print(numbers)  # 输出：[2, 3, 5, 7, 1, 4, 6, 8]
> 
> 
> # 尝试在闭包内修改外部作用域变量的错误示例
> def sort_priority2(numbers, group):
>     found = False
>     def helper(x):
>         if x in group:
>             found = True
>             return (0, x)
>         return (1, x)
>     numbers.sort(key=helper)
>     return found
> 
> found = sort_priority2(numbers, group)
> print('Found:', found)  # 输出：Found: False
> print(numbers)  # 输出：[2, 3, 5, 7, 1, 4, 6, 8]
> 
> 
> # 使用 nonlocal 的正确示例
> def sort_priority3(numbers, group):
>     found = False
>     def helper(x):
>         nonlocal found
>         if x in group:
>             found = True
>             return (0, x)
>         return (1, x)
>     numbers.sort(key=helper)
>     return found
> 
> found = sort_priority3(numbers, group)
> print('Found:', found)  # 输出：Found: True
> print(numbers)  # 输出：[2, 3, 5, 7, 1, 4, 6, 8]
> 
> 
> # 使用类的更好示例
> class Sorter:
>     def __init__(self, group):
>         self.group = group
>         self.found = False
> 
>     def __call__(self, x):
>         if x in self.group:
>             self.found = True
>             return (0, x)
>         return (1, x)
> 
> sorter = Sorter(group)
> numbers.sort(key=sorter)
> assert sorter.found is True
> ```

### Item 22: Reduce Visual Noise with Variable Positional Arguments

```python
def log(message, *values):
    if not values:
    	print(message)
    else:
        values_str = ', '.join(str(x) for x in values)
        print(f'{message}: {values_str}')

log('My numbers are', 1, 2)
log('Hi there')
```

### Item 23: Provide Optional Behavior with Keyword Arguments

**可选的关键字参数应始终通过关键字而不是位置传递。**

关键字参数有三个主要优点：

- 提高代码可读性：对于新读者来说，关键字参数使函数调用更清晰。
- 可以设置默认值：允许函数在你需要时提供额外功能，但是大多数时候你可以接受默认行为，避免代码重复和减少噪音。
- 向后兼容性：关键字参数为扩展函数的参数提供了一种强大的方法，同时仍然保持与现有调用者的向后兼容性。这意味着你可以提供额外的功能而无需迁移大量现有的代码。

> 如果你希望函数能接收任何命名的关键字参数，你可以使用**kwargs来收集那些参数。
>
> ```python
> def print_parameters(**kwargs):
>     for key, value in kwargs.items():
>     	print(f'{key} = {value}')
> print_parameters(alpha=1.5, beta=9, gamma=4)
> ```

### Item 24: Use None and Docstrings to Specify Dynamic Default Arguments

在Python中，**函数的默认参数*(like {}, [], or datetime.now())*只会在模块加载时被计算一次**。这可能导致一些问题，尤其是当默认参数是动态的（例如，`datetime.now()`）时。默认参数的这种行为可能会与程序员的预期不符, 

<u>因此，对于默认值是动态的情况，我们应该把默认参数设为`None`，然后在函数内部处理默认值的生成。在函数的文档字符串中，应该明确地说明默认参数的行为。</u>

```python
from typing import Optional
def log_typed(message: str,
    		when: Optional[datetime]=None) -> None:
    """Log a message with a timestamp.
    	Args:
    	message: Message to print.
    	when: datetime of when the message occurred.
   		Defaults to the present time.
    """
    if when is None:
    	when = datetime.now()
    print(f'{when}: {message}')
```



> 例如，假设我们有一个记录日志的函数，我们希望它能在记录每个日志条目时都能记录当前时间。我们可能会尝试下面的方法：
>
> ```python
> pythonCopy codefrom datetime import datetime
> 
> def log(message, when=datetime.now()):
>     print(f'{when}: {message}')
> ```
>
> 然而这种方法会导致所有的日志记录都有同样的时间戳，因为`datetime.now()`只在函数定义时被调用一次，之后就不会再变化了。
>
> 正确的做法是把默认参数设为`None`，然后在函数内部检查这个参数。如果这个参数为`None`，就生成一个新的默认值。例如：
>
> ```python
> pythonCopy codefrom datetime import datetime
> 
> def log(message, when=None):
>     """Log a message with a timestamp.
> 
>     Args:
>     message: Message to print.
>     when: datetime of when the message occurred.
>     Defaults to the present time.
>     """
>     if when is None:
>         when = datetime.now()
>     print(f'{when}: {message}')
> ```
>
> 同样的，对于默认参数是可变的（例如，字典或列表）的情况，也应该用同样的方法处理。因为如果一个函数的默认参数是可变的，那么这个参数在所有的函数调用中都是共享的。例如：
>
> ```python
> pythonCopy codeimport json
> 
> def decode(data, default={}):
>     try:
>         return json.loads(data)
>     except ValueError:
>         return default
> ```
>
> 这个函数在遇到无法解析的数据时，会返回同一个默认字典，这可能导致非常奇怪的结果。正确的做法是把默认参数设为`None`，然后在函数内部生成一个新的默认值：
>
> ```python
> pythonCopy codeimport json
> 
> def decode(data, default=None):
>     """Load JSON data from a string.
> 
>     Args:
>     data: JSON data to decode.
>     default: Value to return if decoding fails.
>     Defaults to an empty dictionary.
>     """
>     try:
>         return json.loads(data)
>     except ValueError:
>         if default is None:
>             default = {}
>         return default
> ```

### Item 25: Enforce Clarity with Keyword-Only and Positional-Only Arguments

- **关键字参数 (Keyword Arguments)**: Python 允许你使用关键字参数在函数调用时指定某些参数的值。这增加了代码的可读性，因为调用者可以清楚地看到每个参数的名称和传递的值。此外，关键字参数也可以有默认值，这使得这些参数在调用函数时可以是可选的。
- **关键字只参数 (Keyword-Only Arguments)**: 在函数定义中，参数列表中的 `*` 后的参数是关键字只参数，意味着它们只能通过关键字来提供，而不能通过位置。这样可以确保调用者在使用函数时清晰地了解他们的意图。
- **位置只参数 (Positional-Only Arguments)**: 在Python 3.8中，引入了位置只参数的概念。函数定义中参数列表的 `/` 之前的参数是位置只参数，它们只能通过位置提供，而不能通过关键字。这可以减少调用者对参数名称的依赖，使得我们可以在不影响现有调用者的情况下更改参数名称。

```python
# 使用关键字参数和默认值
def safe_division_b(number, divisor, ignore_overflow=False, ignore_zero_division=False):
    try:
        return number / divisor
    except OverflowError:
        if ignore_overflow:
            return 0
        else:
            raise
    except ZeroDivisionError:
        if ignore_zero_division:
            return float('inf')
        else:
            raise

# 使用关键字只参数
def safe_division_c(number, divisor, *, ignore_overflow=False, ignore_zero_division=False):
    try:
        return number / divisor
    except OverflowError:
        if ignore_overflow:
            return 0
        else:
            raise
    except ZeroDivisionError:
        if ignore_zero_division:
            return float('inf')
        else:
            raise

# 使用位置只参数
def safe_division_d(numerator, denominator, /, *, ignore_overflow=False, ignore_zero_division=False):
    try:
        return numerator / denominator
    except OverflowError:
        if ignore_overflow:
            return 0
        else:
            raise
    except ZeroDivisionError:
        if ignore_zero_division:
            return float('inf')
        else:
            raise

# 在参数列表中的 / 和 * 之间的参数可以通过位置或关键字提供
def safe_division_e(numerator, denominator, /, ndigits=10, *, ignore_overflow=False, ignore_zero_division=False):
    try:
        fraction = numerator / denominator
        return round(fraction, ndigits)
    except OverflowError:
        if ignore_overflow:
            return 0
        else:
            raise
    except ZeroDivisionError:
        if ignore_zero_division:
            return float('inf')
        else:
            raise

```

### Item 26: Define Function Decorators with functools.wraps 

装饰器是Python中一种特殊的语法，能在函数执行前后插入额外的代码。装饰器可以访问并修改输入参数、返回值和抛出的异常，对于强制语义、调试、注册函数等有很大的帮助。

举例来说，我们可以使用装饰器打印函数调用的参数和返回值。这在调试递归函数的函数调用堆栈时非常有用。可以通过使用 *args 和 **kwargs 将所有参数传递给被装饰的函数，下面给出一个示例：

```python
pythonCopy codedef trace(func):
 def wrapper(*args, **kwargs):
     result = func(*args, **kwargs)
     print(f'{func.__name__}({args!r}, {kwargs!r}) '
           f'-> {result!r}')
     return result
 return wrapper
```

使用 `@` 符号应用此装饰器到一个函数：

```python
pythonCopy code@trace
def fibonacci(n):
    """Return the n-th Fibonacci number"""
    if n in (0, 1):
        return n
    return (fibonacci(n - 2) + fibonacci(n - 1))
```

这种方法在打印每一级递归调用的参数和返回值时非常有用。但这也带来了一种副作用，即装饰器返回的函数名与原函数名不同。这可能会影响到执行自省（introspection）的工具，如调试器。

如 `help(fibonacci)` `pickle.dumps(fibonacci)` 等

为了解决这个问题，我们可以使用functools模块中的 `wraps` 助手函数。这是一个帮助你编写装饰器的装饰器。当你将其应用到包装器函数时，它会将内部函数的所有重要元信息复制到外部函数：

```python
pythonCopy codefrom functools import wraps

def trace(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        # do something before the function call
        result = func(*args, **kwargs)
        # do something after the function call
        print(f'{func.__name__}({args!r}, {kwargs!r}) '
              f'-> {result!r}')
        return result
    return wrapper

@trace
def fibonacci(n):
    """Return the n-th Fibonacci number"""
    if n in (0, 1):
        return n
    return (fibonacci(n - 2) + fibonacci(n - 1))
```

使用 wraps 装饰器可以确保函数元数据的正确传递，从而避免在使用一些执行自省的工具时出现异常。

总结一下，要点如下：

- Python的装饰器是一种运行时修改函数的语法。
- 使用装饰器可能会导致一些执行自省的工具出现异常行为。
- 当你定义自己的装饰器时，使用functools模块的wraps装饰器可以避免这些问题。

## Chapter Ⅳ: Comprehensions and Generators 

许多程序都是围绕处理列表、字典键值对和集合构建的。Python提供了一种特殊的语法，称为**推导式**，用于简洁地迭代这些类型并创建衍生数据结构。推导式可以显著提高执行这些常见任务的代码的可读性，并提供许多其他优势。

这种处理方式还可以扩展到具有生成器的函数上，生成器可以逐步返回函数的值流。调用生成器函数的结果可以在适当的任何地方使用迭代器（例如，用于循环、星号表达式）。生成器可以提高性能，减少内存使用量，并提高可读性。

### Item 27: Use Comprehensions nstead of map and filter

- 列表推导式比map和filter函数更清晰，因为它们不需要lambda表达式。
- 列表推导式允许你轻松地从输入列表中跳过项目，这是map函数在没有filter函数帮助的情况下无法实现的。
- 字典和集合也可以使用推导式来创建。

列表推导式是一种从其他序列派生新列表的简洁语法。比如，你想计算一个列表中每个数字的平方，你可以通过一个简单的for循环实现：

```python
pythonCopy code
a = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
squares = []
for x in a:
    squares.append(x**2)
print(squares)
# 输出：[1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
```

但使用列表推导式，你可以在一行代码内完成同样的事情：

```python
pythonCopy codesquares = [x**2 for x in a]
print(squares)
# 输出：[1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
# Or
alt = map(lambda x: x ** 2, a)
```

列表推导式比map函数更清晰，特别是在进行的不只是一个单参数函数操作时。map函数需要创建一个lambda函数进行计算，这在视觉上比较杂乱。

此外，列表推导式还可以很容易地过滤输入列表的项目，移除结果中的相应输出。如下所示，可以在列表推导式的循环后添加条件表达式来实现：

```python
pythonCopy codeeven_squares = [x**2 for x in a if x % 2 == 0]
print(even_squares)
# 输出：[4, 16, 36, 64, 100]
# V.S.
alt = map(lambda x: x**2, filter(lambda x: x % 2 == 0, a))
```

相比之下，使用map和filter函数来实现相同的效果会更难阅读。

字典和集合也有自己的推导式（字典推导式和集合推导式），这使得在编写算法时很容易创建其他类型的派生数据结构：

```python
pythonCopy codeeven_squares_dict = {x: x**2 for x in a if x % 2 == 0}
threes_cubed_set = {x**3 for x in a if x % 3 == 0}
print(even_squares_dict)
print(threes_cubed_set)
# 输出：
# {2: 4, 4: 16, 6: 36, 8: 64, 10: 100}
# {216, 729, 27}
```

### Item 28: Avoid More Than Two Control Subexpressions inComprehensions

在 Python 中，理解表达式（Comprehensions）是一种语法糖，可以让我们用一行代码就能完成原本需要多行才能完成的循环操作。例如，如果我们想要将一个二维列表（也可以看作是矩阵）中的所有元素放入一个新的一维列表中，可以使用包含两个 for 子表达式的列表理解：
```python
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
flat = [x for row in matrix for x in row]
```

另一个例子是，如果我们想要计算矩阵中每个元素的平方，我们可以使用嵌套的列表理解：
```python
squared = [[x**2 for x in row] for row in matrix]
```

然而，如果理解表达式中包含的控制子表达式超过两个，那么它们将变得很难理解，可能会降低代码的可读性。例如：
```python
my_lists = [
 [[1, 2, 3], [4, 5, 6]],
 ...
]
flat = [x for sublist1 in my_lists for sublist2 in sublist1 for x in sublist2]
```
在这种情况下，使用常规的循环和 if 语句，可能会更清晰：
```python
flat = []
for sublist1 in my_lists:
 for sublist2 in sublist1:
 flat.extend(sublist2)
```

此外，理解表达式也支持多个 if 条件，如果多个条件在同一层循环，它们之间有一个隐式的 and 表达式：
```python
a = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
b = [x for x in a if x > 4 if x % 2 == 0]
c = [x for x in a if x > 4 and x % 2 == 0] # b 和 c 是等价的
```

最后，如果理解表达式过于复杂，应该避免使用。更好的选择是使用常规的循环和条件语句，或者写一个帮助函数。

### Item 29: Avoid Repeated Work in Comprehensions by Using Assignment Expressions

关于在Python的comprehensions中避免重复计算的方法，使用Python 3.8引入的“海象运算符”（:=）来创建赋值表达式。

比如，在处理一个订单管理程序的情况下，我们需要重复引用同一个计算，我们需要找出库存中哪些订单可以被批量完成。在此情况下，我们可以使用字典推导式来避免重复的计算，但是这会导致一些可读性和一致性的问题。

下面是原始代码：
```python
stock = {
 'nails': 125,
 'screws': 35,
 'wingnuts': 8,
 'washers': 24,
}
order = ['screws', 'wingnuts', 'clips']

def get_batches(count, size):
 return count // size

result = {}
for name in order:
 count = stock.get(name, 0)
 batches = get_batches(count, 8)
 if batches:
 result[name] = batches

print(result)
```

而使用字典推导式的代码如下：
```python
found = {name: get_batches(stock.get(name, 0), 8)
 for name in order
 if get_batches(stock.get(name, 0), 8)}

print(found)
```

这段代码虽然更简洁，但是`get_batches(stock.get(name, 0), 8)`的表达式被重复了，这增加了阅读困难，并可能引入错误。为了避免这种问题，我们可以使用赋值表达式(`:=`运算符)，这将允许我们在单次循环中进行一次计算并存储其结果，然后在其他地方再次引用该变量，而不是重新计算。下面是修改后的代码：

```python
found = {name: batches for name in order
 if (batches := get_batches(stock.get(name, 0), 8))}

print(found)
```

这样，对于`get_batches`和`stock.get`的调用只进行了一次，不仅使代码更简洁，而且也提高了性能。

但是，作者也警告说，在赋值表达式在理解推导式的值部分被定义并在其他部分被引用时，可能会出现运行时错误。此外，如果在推导式中使用了赋值表达式，并且没有条件语句，那么循环变量就会泄漏到包含的作用域中，这通常是我们要避免的。

最后，这篇文章也提到，赋值表达式在生成器表达式中也能同样工作。

总的来说，主要的观点是：

- 赋值表达式可以使得推导式和生成器表达式在同一推导式中重用一个条件的值，这可以提高可读性和性能。
- 尽管可以在推导式或生成器表达式的条件之外使用赋值表达式，但应避免这样做。

### Item 30: Consider Generators Instead of Returning Lists

这个段落主要在讨论 Python 中列表与生成器的使用差异以及各自的优点。特别强调了在处理大量数据或需要产生序列数据时，选择使用生成器可能会更好。

首先，作者通过一个示例函数 `index_words`，展示了使用列表来保存和返回函数结果的一般方法。然后指出了这种方法的两个问题：

1. 代码相对密集和嘈杂，列表的 `append` 方法在每次找到新的结果时都需要被调用，这可能导致代码阅读困难。
2. 该函数要求在返回结果前，所有的结果必须存储在列表中。对于大数据输入，这可能会导致程序运行内存不足并崩溃。

然后，作者介绍了生成器的概念和用法，通过使用 `yield` 表达式的生成器函数，可以逐个产生结果，而不需要一次性生成所有的结果。

最后，作者使用一个文件读取的例子进一步展示了生成器如何处理大规模数据。在这个例子中，生成器每次只处理一行输入数据，并产生一个输出，这极大地降低了内存使用。

然而，作者也提醒我们，使用生成器的时候需要注意，生成器返回的迭代器是有状态的，不能重复使用。

下面是段落中的代码示例：

1. 列表的用法：

    ```python
    def index_words(text):
        result = []
        if text:
            result.append(0)
        for index, letter in enumerate(text):
            if letter == ' ':
                result.append(index + 1)
        return result
    address = 'Four score and seven years ago...'
    result = index_words(address)
    print(result[:10])  # [0, 5, 11, 15, 21, 27, 31, 35, 43, 51]
    ```

2. 使用生成器的方式：

    ```python
    def index_words_iter(text):
        if text:
            yield 0
        for index, letter in enumerate(text):
            if letter == ' ':
                yield index + 1
    it = index_words_iter(address)
    print(next(it))  # 0
    print(next(it))  # 5
    result = list(index_words_iter(address))
    print(result[:10])  # [0, 5, 11, 15, 21, 27, 31, 35, 43, 51]
    ```

3. 使用生成器处理大规模数据：

    ```python
    def index_file(handle):
        offset = 0
        for line in handle:
            if line:
                yield offset
            for letter in line:
                offset += 1
                if letter == ' ':
                    yield offset
    import itertools
    with open('address.txt', 'r') as f:
        it = index_file(f)
        results = itertools.islice(it, 0, 10)
        print(list(results))  # [0, 5, 11, 15, 21, 27, 31, 35, 43, 51]
    ```

关键点总结：

- 使用生成器比函数返回累积结果的列表更清晰。
- 生成器返回的迭代器产生的是生成器函数体内 `yield` 表达式传递的值集。
- 生成器可以为任意大的输入产生输出序列，因为其工作内存不包含所有的输入和输出。

### Item 31: Be Defensive When Iterating Over Arguments

* Item 32: Consider Generator Expressions for Large List Comprehensions
* Item 33: Compose Multiple Generators with yield from
* Item 34: Avoid Injecting Data into Generators with send
* Item 35: Avoid Causing State Transitions inGenerators with throw
* Item 36: Consider itertools for Working with Iteratorsand Generators

## Chapter Ⅴ: Classes and Interfaces 

* Item 37: Compose Classes Instead of NestingMany Levels of Built-in Types
* Item 38: Accept Functions Instead of Classes forSimple Interfaces
* Item 39: Use @classmethod Polymorphism toConstruct Objects Generically
* Item 40: Initialize Parent Classes with super
* Item 41: Consider Composing Functionalitywith Mix-in Classes
* Item 42: Prefer Public Attributes Over Private Ones
* Item 43: Inherit from collections.abc forCustom Container Types

## Chapter Ⅵ: Metaclasses and Attributes

## Chapter Ⅶ: Concurrency and Parallelism

## Chapter Ⅷ: Robustness and Performance

## Chapter Ⅸ: Testing and Debugging 

## Chapter Ⅹ: Collaboration 
