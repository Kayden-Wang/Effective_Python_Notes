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

**段落摘要总结:**

这篇文章详细讲述了在Python中对函数参数进行多次迭代时需要注意的问题，特别是在处理迭代器或生成器时。由于这些对象只能遍历一次，所以在多次迭代时可能出现意想不到的结果或者丢失数据。文章提供了几种解决方案，包括复制输入迭代器、使用一个返回新迭代器的函数或者使用实现了迭代器协议的容器类型。这个协议定义了Python的for循环和相关表达式如何遍历容器类型。文章还讨论了如何通过检查一个对象是否可以迭代来保证函数和方法的参数不仅仅是迭代器。

**代码示例与解析:**

```python
# 计算访客百分比的函数
def normalize(numbers):
    total = sum(numbers)
    result = []
    for value in numbers:
        percent = 100 * value / total
        result.append(percent)
    return result

visits = [15, 35, 80]
percentages = normalize(visits)
print(percentages)
assert sum(percentages) == 100.0
# 输出：[11.538461538461538, 26.923076923076923, 61.53846153846154]
```

在上面的代码中，我们定义了一个函数`normalize`，它接收一个数字列表作为参数，并对每个数字计算其在总和中的百分比。这个函数可以很好地处理访客数量的列表，但是当我们尝试对一个生成器（例如从文件中读取访客数量）多次迭代时，就会出现问题，因为生成器只能被迭代一次。

解决这个问题的一种方法是复制输入迭代器：

```python
def normalize_copy(numbers):
    numbers_copy = list(numbers) # 复制迭代器
    total = sum(numbers_copy)
    result = []
    for value in numbers_copy:
        percent = 100 * value / total
        result.append(percent)
    return result
```

但是这可能会导致内存问题，因为复制可能非常大的迭代器可能会导致程序耗尽内存并崩溃。为了解决这个问题，可以接受一个函数作为参数，这个函数每次调用时都返回一个新的迭代器：

```python
def normalize_func(get_iter):
    total = sum(get_iter()) # 新迭代器
    result = []
    for value in get_iter(): # 新迭代器
        percent = 100 * value / total
        result.append(percent)
    return result
```

如果要检查参数是否是一个可以多次迭代的容器，可以使用Python的迭代器协议和`collections.abc.Iterator`类：

```python
from collections.abc import Iterator

def normalize_defensive(numbers):
    if isinstance(numbers, Iterator): # 另一种检查方式
        raise TypeError('Must supply a container')
    total = sum(numbers)
    result = []
    for value in numbers:
        percent = 100 * value / total
        result.append(percent)
    return result
```

**应用场景及应用原因总结:**

这些策略和代码适用于任何需要对函数参数进行多次迭代的情况。例如，我们可能需要分析大量数据，这些数据太大而无法全部装入内存，所以我们使用迭代器或生成器来一次处理一部分数据。但是，如果我们需要对这些数据进行多次迭代，就需要采取这里介绍的策略。

我们需要这样做的原因是Python的迭代器和生成器是"懒惰"的，也就是说它们只在需要时产生数据，而且只能迭代一次。这对于大数据处理非常有用，因为我们不需要一次性加载所有数据到内存，但也意味着我们不能简单地对这些数据进行多次迭代。

**实用代码段:**

```python
from collections.abc import Iterator

def normalize_defensive(numbers):
    if isinstance(numbers, Iterator): # 另一种检查方式
        raise TypeError('Must supply a container')
    total = sum(numbers)
    result = []
    for value in numbers:
        percent = 100 * value / total
        result.append(percent)
    return result

# 使用列表
visits = [15, 35, 80]
percentages = normalize_defensive(visits)
assert sum(percentages) == 100.0

# 使用容器
class ReadVisits:
    def __init__(self, data_path):
        self.data_path = data_path

    def __iter__(self):
        with open(self.data_path) as f:
            for line in f:
                yield int(line)

visits = ReadVisits('my_numbers.txt')
percentages = normalize_defensive(visits)
assert sum(percentages) == 100.0
```

### Item 32: Consider Generator Expressions for Large List Comprehensions

1. **段落摘要总结**： 这个段落讨论了Python中的生成器表达式以及它们在处理大型输入时如何优于列表推导式。列表推导式可能会因为创建了大量数据而消耗过多内存，特别是在处理大规模数据或无限数据流时，如读取一个巨大的文件或网络socket。生成器表达式是列表推导式和生成器的综合体，它不会在运行时产生整个输出序列，而是返回一个迭代器，一次只产生一个结果。生成器表达式还可以被组合在一起，通过传递一个生成器表达式的迭代器到另一个生成器表达式，以此方式可以在处理大型输入时，执行得非常快且内存效率高。但要注意，由生成器表达式返回的迭代器是有状态的，不能使用多次。

2. **根据段落行文逻辑，给出理解代码**：

   ```python
   # 代码①：使用列表推导式读取文件并返回每行字符的数量
   value = [len(x) for x in open('my_file.txt')]
   print(value)
   # 输出：[100, 57, 15, 1, 12, 75, 5, 86, 89, 11]
   
   # 代码②：使用生成器表达式的版本
   it = (len(x) for x in open('my_file.txt'))
   print(it)
   # 输出： <generator object <genexpr> at 0x108993dd0>
   
   # 逐步获取生成器表达式的输出
   print(next(it))  # 输出：100
   print(next(it))  # 输出：57
   
   # 生成器表达式可以进行组合
   roots = ((x, x**0.5) for x in it)
   print(next(roots))  
   # 输出：(15, 3.872983346207417)
   ```

3. **应用场景及应用原因总结**： 

   生成器表达式适用于处理大型输入，特别是那些可能导致内存溢出的情况，如处理大型文件或无限的网络socket。它们可以有效地避免内存问题，因为它们一次只产生一个输出。此外，生成器表达式可以很快地执行，当你需要操作大型输入流时，生成器表达式是一个很好的选择。然而，要注意生成器表达式返回的迭代器是有状态的，一次性的，不能使用多次。

4. **实用代码段:**

   ```python
   def read_large_file(file_path):
       """
       使用生成器表达式来读取大文件
       :param file_path: 文件的路径
       :return: None
       """
       # 使用生成器表达式来获取文件中每行的长度
       line_lengths = (len(line) for line in open(file_path))
   
       # 遍历迭代器
       for length in line_lengths:
           print(length)
   
   # 调用函数，传入大文件路径
   read_large_file('path_to_your_large_file')
   ```

### Item 33: Compose Multiple Generators with yield from

段落总结:

这个段落从《Effective Python》的“Item 33: 使用yield from组合多个生成器”一节中，详细介绍了在Python中如何使用"yield from"表达式来组合多个生成器。首先给出了一个图形程序动画的例子，显示了如何通过组合多个生成器的输出来创建动画。然后，提到了在实现这个动画的过程中存在的问题，即：如果有多个生成器需要组合，那么代码中就会有大量重复的for循环和yield表达式，这会降低代码的可读性。为了解决这个问题，文章介绍了"yield from"表达式，这个表达式可以使Python解释器自动处理嵌套的for循环和yield表达式，从而提高了代码的可读性和性能。最后，通过一个性能对比实验，证明了使用"yield from"的代码比手动迭代嵌套生成器并产生其输出的代码运行速度快。

代码理解与示例:

作者定义了两个生成器函数：`move`和`pause`。`move`生成器根据给定的`period`和`speed`，生成一系列代表速度的数据；`pause`生成器根据给定的`delay`生成一系列0，代表暂停。

```python
def move(period, speed):
    for _ in range(period):
        yield speed

def pause(delay):
    for _ in range(delay):
        yield 0
```
示例：

```python
for delta in move(3, 5.0):
    print(delta)

# Output
# 5.0
# 5.0
# 5.0
```

`animate`函数组合了`move`和`pause`生成器的输出来生成动画。但是，由于每个生成器都需要一个for循环和yield表达式，这使得代码有些重复和不清晰。

```python
def animate():
    for delta in move(4, 5.0):
        yield delta
    for delta in pause(3):
        yield delta
    for delta in move(2, 3.0):
        yield delta
```
示例：

```python
for delta in animate():
    print(delta)

# Output
# 5.0
# 5.0
# 5.0
# 5.0
# 0.0
# 0.0
# 0.0
# 3.0
# 3.0
```

使用"yield from"可以使代码更清晰，更直观。"yield from"让Python解释器自动处理嵌套的for循环和yield表达式。

```python
def animate_composed():
    yield from move(4, 5.0)
    yield from pause(3)
    yield from move(2, 3.0)
```
示例：

```python
for delta in animate_composed():
    print(delta)

# Output
# 5.0
# 5.0
# 5.0
# 5.0
# 0.0
# 0.0
# 0.0
# 3.0
# 3.0
```

4. "yield from"比手动迭代嵌套生成器并产生其输出更快。性能测试证明了这一点。

```python
import timeit
def child():
    for i in range(1_000_000):
        yield i

def slow():
    for i in child():
        yield i

def fast():
    yield from child()

baseline = timeit.timeit(
    stmt='for _ in slow(): pass',
    globals=globals(),
    number=50)

comparison = timeit.timeit(
    stmt='for _ in fast(): pass',
    globals=globals(),
    number=50)

reduction = -(comparison - baseline) / baseline
print(f'{reduction:.1%} less time')

# Output might be similar to:
# 13.5% less time
```

方法应用场景及应用原因:

在使用生成器的程序中，"yield from"可以用来合并多个生成器的输出。**这在需要组合多个生成器的输出，或者一个生成器的输出依赖于其他多个生成器的情况下非常有用**。与手动迭代每个生成器并yield其输出相比，使用"yield from"不仅可以使代码更清晰、可读性更高，还可以提高程序的性能。

实用代码段:

```python
def move(period, speed):
    for _ in range(period):
        yield speed

def pause(delay):
    for _ in range(delay):
        yield 0

def animate_composed():
    yield from move(4, 5.0)
    yield from pause(3)
    yield from move(2, 3.0)
```
以上代码定义了两个生成器`move`和`pause`，并通过"yield from"语句在`animate_composed`中组合了这两个生成器的输出。

### Item 34: Avoid Injecting Data into Generators with send

### Item 35: Avoid Causing State Transitions inGenerators with throw

1. **段落摘要总结**
   此段落主要介绍了Python生成器中throw方法的使用和其可能引发的问题，以及一个更好的替代方法。throw方法可在生成器中在最近执行的yield表达式处重新抛出异常。然而，这种使用方式降低了代码的可读性，因为需要额外的嵌套和代码以抛出和捕获异常。最后，作者建议避免使用throw方法，而是使用实现了__iter__方法和能引发特殊状态转换的方法的类。

2. **代码逻辑理解和示例**
   首先，作者使用了一个简单的例子来展示throw的使用：
   
   ```python
   class MyError(Exception):
       pass
   def my_generator():
       yield 1
       yield 2
       yield 3
   it = my_generator()
   print(next(it)) # 输出: 1
   print(next(it)) # 输出: 2
   print(it.throw(MyError('test error'))) # 抛出: MyError: test error
   ```
   在这个例子中，通过在生成器中使用throw方法，我们可以在生成器内部抛出我们自定义的异常。然后，我们可以捕获这个异常并处理它：

   ```python
   def my_generator():
       yield 1
       try:
           yield 2
       except MyError:
           print('Got MyError!')
       else:
           yield 3
           yield 4
   it = my_generator()
   print(next(it)) # 输出: 1
   print(next(it)) # 输出: 2
   print(it.throw(MyError('test error'))) # 输出: Got MyError!
   ```
   最后，作者建议使用实现了__iter__方法的类代替使用throw的方法：

   ```python
   class Timer:
       def __init__(self, period):
           self.current = period
           self.period = period
       def reset(self):
           self.current = self.period
       def __iter__(self):
           while self.current:
               self.current -= 1
               yield self.current
   def run():
       timer = Timer(4)
       for current in timer:
           if check_for_reset(): # 这里为示例所以没有实现这个函数
               timer.reset()
           announce(current) # 这里为示例所以没有实现这个函数
   run()
   ```
   在这个例子中，我们定义了一个Timer类，该类在每次迭代时减少当前的时间，直到时间用完为止。在每次迭代中，我们检查是否需要重置时间。如果需要，我们调用reset方法重置时间。

3. **应用场景及应用原因总结**
   使用生成器的throw方法可以在异步编程中处理异常。然而，这种做法会降低代码的可读性。当你需要在生成器中提供特殊行为（例如，重置计时器）时，使用实现了__iter__方法和能引发特殊状态转换的方法的类是一个更好的选择。它们使代码更易读，更易于理解。

   举个例子，使用这种方法可以用于编写计时器程序，支持偶尔的重置。例如，在网络请求的循环中，如果某个请求失败，我们可能希望重置计时器，重新开始请求。或者在游戏中，我们可能需要一个能够重置的计时器，以便在特定事件发生时重置游戏的时间。

4. **实用代码段**
   
   ```python
   class Timer:
       def __init__(self, period):
           self.current = period
           self.period = period
       def reset(self):
           self.current = self.period
       def __iter__(self):
           while self.current:
               self.current -= 1
               yield self.current
               
   def check_for_reset():
       # 在此处实现重置的逻辑
       pass
   
   def announce(remaining):
       print(f'{remaining} ticks remaining')
   
   def run():
       timer = Timer(4)
       for current in timer:
           if check_for_reset():
               timer.reset()
           announce(current)
   
   run()
   ```
   这是一个使用了Timer类的计时器程序。该程序在每次迭代时减少当前的时间，直到时间用完为止。在每次迭代中，我们检查是否需要重置时间。如果需要，我们调用reset方法重置时间，并通过announce函数宣布剩余时间。

### Item 36: Consider itertools for Working with Iteratorsand Generators

Python内建模块`itertools`的使用方法，这个模块包含许多有用的函数，可以帮助我们更有效地操作和管理迭代器。具体内容包括了三个主要部分：

1. 连接迭代器：使用`chain`，`repeat`，`cycle`，`tee`和`zip_longest`函数来连接或者复制迭代器。
2. 过滤迭代器中的项：使用`islice`，`takewhile`，`dropwhile`和`filterfalse`函数来过滤迭代器中的项。
3. 从迭代器中生成项的组合：使用`accumulate`，`product`，`permutations`，`combinations`和`combinations_with_replacement`函数从迭代器中生成项的组合。

**链接迭代器 Linking Iterators **

```python
import itertools

# chain函数可以连接多个迭代器成一个连续的迭代器
it = itertools.chain([1, 2, 3], [4, 5, 6])
print(list(it))  # 输出：[1, 2, 3, 4, 5, 6]

# repeat函数可以无限次输出一个值，或者指定输出次数
it = itertools.repeat('hello', 3)
print(list(it))  # 输出：['hello', 'hello', 'hello']

# cycle函数可以使一个迭代器的项无限次循环输出
it = itertools.cycle([1, 2])
result = [next(it) for _ in range (10)]
print(result)  # 输出：[1, 2, 1, 2, 1, 2, 1, 2, 1, 2]

# tee函数可以将一个迭代器分裂成多个平行的迭代器
it1, it2, it3 = itertools.tee(['first', 'second'], 3)
print(list(it1))  # 输出：['first', 'second']
print(list(it2))  # 输出：['first', 'second']
print(list(it3))  # 输出：['first', 'second']

# zip_longest函数是zip函数的变体，当一个迭代器耗尽后，它会输出占位符
keys = ['one', 'two', 'three']
values = [1, 2]
it = itertools.zip_longest(keys, values, fillvalue='nope')
longest = list(it)
print('zip_longest:', longest)  # 输出：zip_longest: [('one', 1), ('two', 2), ('three', 'nope')]

```

**过滤迭代器中的项**

```python
# islice函数可以在不复制的情况下对迭代器进行切片
values = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
first_five = itertools.islice(values, 5)
print('First five: ', list(first_five))  # 输出：First five: [1, 2, 3, 4, 5]

# takewhile函数返回迭代器中的项，直到断言函数首次返回False
values = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
less_than_seven = lambda x: x < 7
it = itertools.takewhile(less_than_seven, values)
print(list(it))  # 输出：[1, 2, 3, 4, 5, 6]

# dropwhile函数与takewhile函数相反，它跳过迭代器中的项，直到断言函数首次返回True
it = itertools.dropwhile(less_than_seven, values)
print(list(it))  # 输出：[7, 8, 9, 10]

# filterfalse函数是filter函数的相反，它返回断言函数返回False的所有项
evens = lambda x: x % 2 == 0
filter_false_result = itertools.filterfalse(evens, values)
print('Filter false:', list(filter_false_result))  # 输出：Filter false: [1, 3, 5, 7, 9]
```

**生成迭代器中项的组合**

```python
# accumulate函数通过应用一个函数将迭代器中的项折叠到一个运行值
values = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
sum_reduce = itertools.accumulate(values)
print('Sum: ', list(sum_reduce))  # 输出：Sum: [1, 3, 6, 10, 15, 21, 28, 36, 45, 55]

# product函数返回一个或多个迭代器的笛卡尔积
single = itertools.product([1, 2], repeat=2)
print('Single: ', list(single))  # 输出：Single: [(1, 1), (1, 2), (2, 1), (2, 2)]

# permutations函数返回长度为N的迭代器中项的有序排列
it = itertools.permutations([1, 2, 3, 4], 2)
print(list(it))  # 输出：[(1, 2), (1, 3), (1, 4), (2, 1), (2, 3), (2, 4), (3, 1), (3, 2), (3, 4), (4, 1), (4, 2), (4, 3)]

# combinations函数返回迭代器中项的无序组合，不重复
it = itertools.combinations([1, 2, 3, 4], 2)
print(list(it))  # 输出：[(1, 2), (1, 3), (1, 4), (2, 3), (2, 4), (3, 4)]

# combinations_with_replacement函数与combinations函数相同，但允许重复值
it = itertools.combinations_with_replacement([1, 2, 3, 4], 2)
print(list(it))  # 输出：[(1, 1), (1, 2), (1, 3), (1, 4), (2, 2), (2, 3), (2, 4), (3, 3), (3, 4), (4, 4)]

```

Python的内建模块`itertools`提供了一系列功能强大的函数，可以帮助我们更有效地处理迭代器。这些函数包括连接迭代器、过滤迭代器中的项以及从迭代器中生成项的组合等，这些功能在处理大型数据集、复杂数据流以及各种复杂的迭代情况时都会非常有用。

## Chapter Ⅴ: Classes and Interfaces 

### Item 37: Compose Classes Instead of Nesting Many Levels of Built-in Types

1. **段落摘要总结**：
   这段文字讲述了如何通过Python的类来替代复杂的内置类型嵌套。Python的内置字典类型非常适合用来维护动态的内部状态，但是当使用多层嵌套时，代码的复杂性会增加，使得维护成为一种挑战。此时，应考虑使用类来替代。作者以一个成绩记录的例子进行了说明，首先使用字典嵌套的方式实现，然后再使用类的方式重构，并比较了两种方式的优劣。作者最后提出，当你发现内部状态的字典变得复杂时，就该使用类进行重构。

2. **代码理解与示例**：

   以下是一个初始版本的例子，这个例子是一个简单的成绩记录系统，使用字典来存储学生的成绩：
   ```python
   class SimpleGradebook:
     def __init__(self):
       self._grades = {}
   
     def add_student(self, name):
       self._grades[name] = []
   
     def report_grade(self, name, score):
       self._grades[name].append(score)
   
     def average_grade(self, name):
       grades = self._grades[name]
       return sum(grades) / len(grades)
   
   book = SimpleGradebook()
   book.add_student('Isaac Newton')
   book.report_grade('Isaac Newton', 90)
   book.report_grade('Isaac Newton', 95)
   book.report_grade('Isaac Newton', 85)
   print(book.average_grade('Isaac Newton'))  # 输出: 90.0
   ```
   这个例子使用字典嵌套的方式处理更复杂的情况，即对每个学生的每门科目进行成绩记录：
   ```python
   from collections import defaultdict
   
   class BySubjectGradebook:
     def __init__(self):
       self._grades = {}
   
     def add_student(self, name):
       self._grades[name] = defaultdict(list)
   
     def report_grade(self, name, subject, grade):
       by_subject = self._grades[name]
       grade_list = by_subject[subject]
       grade_list.append(grade)
   
     def average_grade(self, name):
       by_subject = self._grades[name]
       total, count = 0, 0
       for grades in by_subject.values():
         total += sum(grades)
         count += len(grades)
       return total / count
   
   book = BySubjectGradebook()
   book.add_student('Albert Einstein')
   book.report_grade('Albert Einstein', 'Math', 75)
   book.report_grade('Albert Einstein', 'Math', 65)
   book.report_grade('Albert Einstein', 'Gym', 90)
   book.report_grade('Albert Einstein', 'Gym', 95)
   print(book.average_grade('Albert Einstein'))  # 输出: 81.25
   ```
   当要处理的情况变得更复杂，例如对成绩添加权重，这个例子中的代码就变得难以阅读和维护：
   ```python
   class WeightedGradebook:
     def __init__(self):
       self._grades = {}
   
     def add_student(self, name):
       self._grades[name] = defaultdict(list)
   
     def report_grade(self, name, subject, score, weight):
       by_subject = self._grades[name]
       grade_list = by_subject[subject]
       grade_list.append((score, weight))
   
     def average_grade(self, name):
       by_subject = self._grades[name]
       score_sum, score_count = 0, 0
       for subject, scores in by_subject.items():
         subject_avg, total_weight = 0, 0
         for score, weight in scores:
           subject_avg += score * weight
           total_weight += weight
         score_sum += subject_avg / total_weight
         score_count += 1
       return score_sum / score_count
   
   book = WeightedGradebook()
   book.add_student('Albert Einstein')
   book.report_grade('Albert Einstein', 'Math', 75, 0.05)
   book.report_grade('Albert Einstein', 'Math', 65, 0.15)
   book.report_grade('Albert Einstein', 'Math', 70, 0.80)
   book.report_grade('Albert Einstein', 'Gym', 100, 0.40)
   book.report_grade('Albert Einstein', 'Gym', 85, 0.60)
   print(book.average_grade('Albert Einstein'))  # 输出: 80.25
   ```
   当内部的字典嵌套变得复杂，代码也难以阅读和维护时，我们可以将其重构为类：
   ```python
   from collections import namedtuple, defaultdict
   
   Grade = namedtuple('Grade', ('score', 'weight'))
   
   class Subject:
     def __init__(self):
       self._grades = []
   
     def report_grade(self, score, weight):
       self._grades.append(Grade(score, weight))
   
     def average_grade(self):
       total, total_weight = 0, 0
       for grade in self._grades:
         total += grade.score * grade.weight
         total_weight += grade.weight
       return total / total_weight
   
   class Student:
     def __init__(self):
       self._subjects = defaultdict(Subject)
   
     def get_subject(self, name):
       return self._subjects[name]
   
     def average_grade(self):
       total, count = 0, 0
       for subject in self._subjects.values():
         total += subject.average_grade()
         count
   class School:
     def __init__(self):
       self._students = defaultdict(Student)
   
     def get_student(self, name):
       return self._students[name]
   
     def average_grade(self):
       total, count = 0, 0
       for student in self._students.values():
         total += student.average_grade()
         count += 1
       return total / count
   
   book = School()
   albert = book.get_student('Albert Einstein')
   math = albert.get_subject('Math')
   math.report_grade(75, 0.05)
   math.report_grade(65, 0.15)
   math.report_grade(70, 0.80)
   gym = albert.get_subject('Gym')
   gym.report_grade(100, 0.40)
   gym.report_grade(85, 0.60)
   print(book.average_grade())  # 输出: 80.25

​	在这个例子中，我们创建了`Grade`，`Subject`，`Student`和`School`四个类来替代字典的嵌套。这样做的优点是代码的阅读和维护性更强。例如，我们可以方便地添加新的方法到这些类中，如添加一个计算某科目最高分的方法。

### Item 38: Accept Functions Instead of Classes for Simple Interfaces

1. **段落摘要总结:**
   本段主要讨论在Python中如何使用函数和类来作为接口。在Python中，许多内建API允许通过传递函数来定制行为。同时，Python作为一种支持一等函数（first-class functions）的语言，允许函数和方法像其他值一样被传递和引用。段落还讲解了defaultdict类，它允许传入一个函数来处理访问到的缺失键值。进一步，作者还提到了如何通过闭包（closure）来跟踪状态，并通过类的`__call__`方法来达到相同的效果。最后，作者建议在需要函数来维护状态时，应考虑定义一个提供`__call__`方法的类，而不是定义一个有状态的闭包。

2. **代码理解与修正:**

    2.1 先来看一下如何通过传递函数作为参数来定制行为的例子：
    ```python
    # 按作者的思路, 我们首先看一个使用函数作为参数的例子
    names = ['Socrates', 'Archimedes', 'Plato', 'Aristotle']
    names.sort(key=len)
    print(names)
    # 输出: ['Plato', 'Socrates', 'Aristotle', 'Archimedes']
    ```

    2.2 再看看如何使用defaultdict类的例子：
    ```python
    from collections import defaultdict

    def log_missing():
        print('Key added')
        return 0

    current = {'green': 12, 'blue': 3}
    increments = [
        ('red', 5),
        ('blue', 17),
        ('orange', 9),
    ]

    result = defaultdict(log_missing, current)
    print('Before:', dict(result))
    for key, amount in increments:
        result[key] += amount
    print('After: ', dict(result))
    # 输出: 
    # Before: {'green': 12, 'blue': 3}
    # Key added
    # Key added
    # After: {'green': 12, 'blue': 20, 'red': 5, 'orange': 9}
    ```

    2.3 看一下如何通过闭包来维护状态：
    ```python
    def increment_with_report(current, increments):
        added_count = 0

        def missing():
            nonlocal added_count  # Stateful closure
            added_count += 1
            return 0

        result = defaultdict(missing, current)
        for key, amount in increments:
            result[key] += amount
        return result, added_count

    result, count = increment_with_report(current, increments)
    assert count == 2
    # 输出：无，但assert的断言成功。
    ```

    2.4 使用类来跟踪状态，并通过类的`__call__`方法使其能够像函数一样被调用：
    ```python
    class BetterCountMissing:
        def __init__(self):
            self.added = 0

        def __call__(self):
            self.added += 1
            return 0

    counter = BetterCountMissing()
    assert counter() == 0
    assert callable(counter)

    counter = BetterCountMissing()
    result = defaultdict(counter, current)  # Relies on __call__
    for key, amount in increments:
        result[key] += amount
    assert counter.added == 2
    # 输出：无，但assert的断言成功。
    ```

3. **应用场景及应用原因总结:**
   
    应用原因: Python的函数是一等公民，可以作为参数传递，可以用于自定义行为；而且Python支持闭包和类的__call__方法，使得在需要维护状态时，可以选择使用有状态的闭包或者定义一个类，这样使得代码更清晰，更易读，更易维护。
    
    常见应用场景例子:
    ① 当需要对列表进行特定规则排序时，可以定义一个函数作为key参数传给sort方法。
    ② 在处理字典时，如果需要对不存在的键进行特殊处理，可以使用defaultdict，传入一个函数来处理这些不存在的键。
    ③ 当需要一个有状态的函数，例如在函数调用过程中需要维护一个计数器，可以使用闭包或者定义一个类，并使用__call__方法。
    
4. **实用代码段：**
```python
class BetterCountMissing:
    def __init__(self):
        self.added = 0

    def __call__(self):
        self.added += 1
        return 0

from collections import defaultdict

current = {'green': 12, 'blue': 3}
increments = [
    ('red', 5),
    ('blue', 17),
    ('orange', 9),
]

counter = BetterCountMissing()
result = defaultdict(counter, current)  # Relies on __call__
for key, amount in increments:
    result[key] += amount

print(f"Missing keys added: {counter.added}")
# 输出: Missing keys added: 2
```

### Item 39: Use @classmethod Polymorphism to Construct Objects Generically

> 多态（Polymorphism）是面向对象编程中的一个重要概念，它指的是同一个方法或函数可以根据调用对象的不同而表现出不同的行为。多态使得我们可以使用统一的接口来处理不同类型的对象，而不需要关心对象的具体类型。

> 在Python中，`cls`通常用作类方法的第一个参数，表示当前的类对象。这与实例方法的第一个参数是`self`表示当前的实例对象类似。但`cls`并不是强制规定的，它只是一个约定俗成的命名方式，理论上你可以用其他任何有效的变量名来替换它。
>
> 当你使用`@classmethod`装饰器定义一个类方法时，该方法的第一个参数就是类对象本身。`cls`参数和`self`参数的主要区别在于，`self`是用于访问实例属性和方法的，而`cls`用于访问类属性和方法。也就是说，通过`cls`，你可以在类方法中访问类级别的属性和方法。
>
> 例如，以下是一个使用`@classmethod`和`cls`的代码示例：
>
> ```python
> class MyClass:
>     class_var = "I'm a class variable!"
> 
>     @classmethod
>     def class_method(cls):
>         print(cls.class_var)  # Access the class variable
> 
> MyClass.class_method()  # Output: I'm a class variable!
> ```
>
> 在上述代码中，`MyClass.class_method()`调用了类方法`class_method`，而该方法通过`cls`访问了类级别的变量`class_var`。

1. 段落摘要总结:

   这段文章主要讨论了如何使用`@classmethod`多态来构建对象。作者首先讲解了在Python中，类和对象都支持多态，然后通过MapReduce工作流程的实例，阐述了如何使用面向对象编程来构建这个系统。但是，这个实现方式存在一个问题，即创建对象的过程并不通用，如果我们要增加新的InputData或者Worker子类，就需要重写生成输入，创建工作者，以及mapreduce函数。解决这个问题的方式是利用类方法多态，使用`@classmethod`装饰器来创建类方法，使得我们可以在子类中重新定义这些方法，来满足各自的需求。这样，我们就可以通过调整参数，使得mapreduce函数具有通用性。

2. 按照段落行文逻辑，以代码帮助理解:

以下代码段主要描述了如何创建InputData和Worker的类和子类，并说明了直接创建对象的问题。

```python
# 定义一个输入数据的基类
class InputData:
    def read(self):
        raise NotImplementedError

# 定义一个输入数据的子类，从文件中读取数据
class PathInputData(InputData):
    def __init__(self, path):
        super().__init__()
        self.path = path

    def read(self):
        with open(self.path) as f:
            return f.read()

# 定义一个工作者的基类
class Worker:
    def __init__(self, input_data):
        self.input_data = input_data
        self.result = None

    def map(self):
        raise NotImplementedError

    def reduce(self, other):
        raise NotImplementedError

# 定义一个工作者的子类，计算文件中的行数
class LineCountWorker(Worker):
    def map(self):
        data = self.input_data.read()
        self.result = data.count('\n')

    def reduce(self, other):
        self.result += other.result

# 模拟生成输入数据和工作者的过程
import os
def generate_inputs(data_dir):
    for name in os.listdir(data_dir):
        yield PathInputData(os.path.join(data_dir, name))

def create_workers(input_list):
    workers = []
    for input_data in input_list:
        workers.append(LineCountWorker(input_data))
    return workers
```

在以上代码中，我们可以看到在创建输入数据和工作者对象的过程中，其实是依赖于具体的子类（PathInputData和LineCountWorker）。如果要增加新的子类，就需要修改`generate_inputs`和`create_workers`函数，这样的代码并不具有通用性。我们可以通过使用类方法多态来解决这个问题：

```python
# 使用类方法来定义输入数据的基类
class GenericInputData:
    def read(self):
        raise NotImplementedError

    @classmethod
    def generate_inputs(cls, config):
        raise NotImplementedError

# 使用类方法来定义输入数据的子类
class PathInputData(GenericInputData):
    def __init__(self, path):
        super().__init__()
        self.path = path

    def read(self):
        with open(self.path) as f:
            return f.read()

    @classmethod
    def generate_inputs(cls, config):
        data_dir = config['data_dir']
        for name in os.listdir(data_dir):
            yield cls(os.path.join(data_dir, name))

# 使用类方法来定义工作者的基类
class GenericWorker:
    def __init__(self, input_data):
        self.input_data = input_data
        self.result = None

    def map(self):
        raise NotImplementedError

    def reduce(self, other):
        raise NotImplementedError

    @classmethod
    def create_workers(cls, input_class, config):
        workers = []
        for input_data in input_class.generate_inputs(config):
            workers.append(cls(input_data))
        return workers

# 使用类方法来定义工作者的子类
class LineCountWorker(GenericWorker):
    def map(self):
        data = self.input_data.read()
        self.result = data.count('\n')

    def reduce(self, other):
        self.result += other.result

# 新的mapreduce函数，只需要调整参数就可以处理不同的输入数据和工作者类
def mapreduce(worker_class, input_class, config):
    workers = worker_class.create_workers(input_class, config)
    return execute(workers)
```

在上面的代码中，我们使用了类方法多态来构造对象，每个子类可以通过重写类方法来满足自己的需求。例如，我们可以定义一个从网络中获取输入数据的子类，只需要重写`generate_inputs`类方法就可以。

3. 方法应用场景及应用原因总结:

   这个方法的主要优点是使得代码更加通用，我们只需要调整参数，就可以处理不同的输入数据和工作者类。例如，我们可以通过定义新的输入数据类和工作者类，来处理不同来源的数据或执行不同的任务。例如，我们可以定义一个从网络中获取输入数据的类，或者定义一个计算文件大小的工作者类，都不需要修改`mapreduce`函数。

   应用场景包括：

   - 数据处理: 例如，我们可以定义不同的输入数据类来处理来自不同来源（例如文件、数据库、网络等）的数据，定义不同的工作者类来执行不同的数据处理任务（例如计算文件的行数、统计单词频率等）。
   - 网页爬虫: 例如，我们可以定义不同的输入数据类来处理不同的网页，定义不同的工作者类来执行不同的爬取任务（例如爬取链接、下载图片等）。

4. 实用代码段：

```python
import os
from threading import Thread

# 输入数据的基类
class GenericInputData:
    def read(self):
        raise NotImplementedError

    @classmethod
    def generate_inputs(cls, config):
        raise NotImplementedError

# 输入数据的子类，从文件中读取数据
class PathInputData(GenericInputData):
    def __init__(self, path):
        super().__init__()
        self.path = path

    def read(self):
        with open(self.path) as f:
            return f.read()

    @classmethod
    def generate_inputs(cls, config):
        data_dir = config['data_dir']
        for name in os.listdir(data_dir):
            yield cls(os.path.join(data_dir, name))

# 工作者的基类
class GenericWorker:
    def __init__(self, input_data):
        self.input
```

### Item 40: Initialize Parent Classes with super

1. 段落摘要总结:

   这段内容主要讲述了在Python中使用`super`函数进行父类初始化的重要性。在Python中，直接调用父类的`__init__`方法进行初始化是一种简单的方式，但是在处理多继承或者类继承层次复杂的情况下，这种方式可能会出现问题。比如，继承顺序和初始化顺序的冲突，或者是钻石继承（一个子类有两个或者以上的父类，并且这些父类又有一个共同的父类）导致的`__init__`方法被多次调用等等。为了解决这些问题，Python提供了`super`函数和标准方法解析顺序(MRO)。`super`函数确保在钻石继承的情况下，公共的超类只会被初始化一次；MRO则定义了超类被初始化的顺序。最后，这段内容还提供了一些`super`函数的其他用法，如直接调用`super()`，或者是提供两个参数`super(ExplicitTrisect, self)`，在需要访问子类中特定父类的功能时，这是必须的。

2. 根据段落行文逻辑, 提供代码来帮助理解:

   ① 这是一个直接调用父类`__init__`方法的例子，但是在复杂的类层次结构或者多重继承的情况下可能出问题:

   ```python
   class MyBaseClass:
       def __init__(self, value):
           self.value = value

   class MyChildClass(MyBaseClass):
       def __init__(self):
           MyBaseClass.__init__(self, 5)
   ```

   ② 钻石继承的一个例子，这会导致`MyBaseClass.__init__`被多次调用，从而出现预料之外的结果：

   ```python
   class MyBaseClass:
       def __init__(self, value):
           self.value = value

   class TimesSeven(MyBaseClass):
       def __init__(self, value):
           MyBaseClass.__init__(self, value)
           self.value *= 7

   class PlusNine(MyBaseClass):
       def __init__(self, value):
           MyBaseClass.__init__(self, value)
           self.value += 9

   class ThisWay(TimesSeven, PlusNine):
       def __init__(self, value):
           TimesSeven.__init__(self, value)
           PlusNine.__init__(self, value)

   foo = ThisWay(5)
   print('Should be (5 * 7) + 9 = 44 but is', foo.value)
   # Output: Should be (5 * 7) + 9 = 44 but is 14
   ```

   ③ 使用`super`函数进行父类初始化的例子，`super`确保了公共的超类只被运行一次，并且可以自动处理继承顺序：

   ```python
   class MyBaseClass:
       def __init__(self, value):
           self.value = value

   class TimesSevenCorrect(MyBaseClass):
       def __init__(self, value):
           super().__init__(value)
           self.value *= 7

   class PlusNineCorrect(MyBaseClass):
       def __init__(self, value):
           super().__init__(value)
           self.value += 9

   class GoodWay(TimesSevenCorrect, PlusNineCorrect):
       def __init__(self, value):
           super().__init__(value)

   foo = GoodWay(5)
   print('Should be 7 * (5 + 9) = 98 and is', foo.value)
   # Output: Should be 7 * (5 + 9) = 98 and is 98
   ```

3. 方法应用场景及应用原因总结:

   ① 应用原因: Python的`super`函数和MRO可以帮助我们更好的处理父类的初始化问题，尤其是在面临多继承或者复杂的类继承层次结构的时候，可以避免很多错误或者预料之外的情况发生。

   ② 常见应用场景: `super`和MRO的使用场景主要是在面临多继承或者复杂的类继承层次结构的时候。例如，你可能在设计一个复杂的GUI框架，其中有很多类和子类，每个类都有自己的初始化过程，`super`和MRO可以帮助你正确的管理这些类的初始化。另一个例子可能是你在设计一个游戏，有很多角色类，每个角色类又有很多子类，比如士兵、魔法师等，这些类和子类都有自己的初始化过程，`super`和MRO也可以在这里帮助你。

4. 实用代码段：

   ```python
   class MyBaseClass:
       def __init__(self, value):
           self.value = value
   
   class TimesSevenCorrect(MyBaseClass):
       def __init__(self, value):
           super().__init__(value)
           self.value *= 7
   
   class PlusNineCorrect(MyBaseClass):
       def __init__(self, value):
           super().__init__(value)
           self.value += 9
   
   class GoodWay(TimesSevenCorrect, PlusNineCorrect):
       def __init__(self, value):
           super().__init__(value)
   
   foo = GoodWay(5)
   print('Should be 7 * (5 + 9) = 98 and is', foo.value)
   # Output: Should be 7 * (5 + 9) = 98 and is 98
   ```

### Item 41: Consider Composing Functionality with Mix-in Classes

1. 段落摘要总结：
   这段文字主要介绍了Python中混入（Mix-in）类的概念和应用。混入是一个定义了一组附加方法但不定义自身实例属性的类，该类不需要它的构造器被调用。

   混入主要用于为子类提供一些附加功能，而不是用于表示对象的实体。它的主要优势是可以通过组合和层叠来减少重复代码并最大化代码复用。文章通过实例解释了如何使用混入类来将Python对象转换为字典，以便于序列化。同时，作者也说明了如何通过重写方法来解决混入可能会引发的循环引用问题。此外，还解释了如何使用混入来实现JSON序列化。

2. 代码理解和PEP8格式化：
   以下是文章中的一些代码段，按照作者的思路递进，以及PEP8格式进行修改。

① 定义混入类

> 在Python中，`self.__dict__`是一个特殊的字典，它用来存储一个对象的属性和对应的值。`self`关键字在类的方法中使用，它引用的是当前实例对象。`__dict__`就是该实例对象的属性字典，也可以说是实例对象的命名空间。

```python
class ToDictMixin:
    def to_dict(self):
        return self._traverse_dict(self.__dict__)

    def _traverse_dict(self, instance_dict):
        output = {}
        for key, value in instance_dict.items():
            output[key] = self._traverse(key, value)
        return output

    def _traverse(self, key, value):
        if isinstance(value, ToDictMixin):
            return value.to_dict()
        elif isinstance(value, dict):
            return self._traverse_dict(value)
        elif isinstance(value, list):
            return [self._traverse(key, i) for i in value]
        elif hasattr(value, '__dict__'):
            return self._traverse_dict(value.__dict__)
        else:
            return value
```
此段代码定义了一个混入类`ToDictMixin`，它提供了一个`to_dict`方法，可以将对象的属性转换为字典。在内部，`_traverse`方法会递归地遍历对象的属性，将它们转换为字典。

应用示例：
```python
class BinaryTree(ToDictMixin):
    def __init__(self, value, left=None, right=None):
        self.value = value
        self.left = left
        self.right = right

tree = BinaryTree(10,
    left=BinaryTree(7, right=BinaryTree(9)),
    right=BinaryTree(13, left=BinaryTree(11)))
print(tree.to_dict())

# 输出：
{
    'value': 10,
    'left': {'value': 7,
    'left': None,
    'right': {'value': 9, 'left': None, 'right': None}},
    'right': {'value': 13,
    'left': {'value': 11, 'left': None, 'right': None},
    'right': None}
}
```
在这个例子中，我们定义了一个二叉树类`BinaryTree`，它继承了`ToDictMixin`，因此可以使用`to_dict`方法。

② 处理混入类的循环引用问题
```python
class BinaryTreeWithParent(BinaryTree):
    def __init__(self, value, left=None, right=None, parent=None):
        super().__init__(value, left=left, right=right)
        self.parent = parent

    def _traverse(self, key, value):
        if (isinstance(value, BinaryTreeWithParent) and key == 'parent'):
            return value.value  # Prevent cycles
        else:
            return super()._traverse(key, value)
```
此段代码解决了混入类可能遇到的循环引用问题。在`BinaryTreeWithParent`中，我们重写了`_traverse`方法，当遇到'parent'属性时，我们直接返回其值，而不再递归遍历，从而避免了循环引用。

应用示例：
```python
root = BinaryTreeWithParent(10)
root.left = BinaryTreeWithParent(7, parent=root)
root.left.right = BinaryTreeWithParent(9, parent=root.left)
print(root.to_dict())

# 输出：
{
    'value': 10,
    'left': {'value': 7,
    'left': None,
    'right': {'value': 9,
    'left': None,
    'right': None,
    'parent': 7},
    'parent': 10},
    'right': None,
    'parent': None
}
```
这个例子展示了如何使用`BinaryTreeWithParent`类，并且不会出现循环引用问题。

③ 实现JSON序列化
```python
import json

class JsonMixin:
    @classmethod
    def from_json(cls, data):
        kwargs = json.loads(data)
        return cls(**kwargs)

    def to_json(self):
        return json.dumps(self.to_dict())
```
这个混入类`JsonMixin`提供了JSON序列化的功能。`from_json`方法将JSON字符串转化为类实例，而`to_json`方法则将类实例转化为JSON字符串。

应用示例：
```python
class DatacenterRack(ToDictMixin, JsonMixin):
    def __init__(self, switch=None, machines=None):
        self.switch = switch
        self.machines = machines

rack = DatacenterRack(switch={"ports": 5, "speed": 1e9}, machines=[{"cores": 8, "ram": 32e9, "disk": 5e12}])
serialized = rack.to_json()
print(serialized)  # 输出：序列化后的JSON字符串
deserialized = DatacenterRack.from_json(serialized)
print(deserialized)  # 输出：反序列化后的DatacenterRack对象
```
这个例子展示了如何使用`JsonMixin`来进行JSON序列化和反序列化。

3. 应用场景及应用原因总结：
   使用混入类的原因主要是因为其能够有效地提高代码复用性和减少代码冗余。混入类可以包含实例方法或类方法，依据你的需要而定。通过组合多个混入类，可以从简单的行为构建出复杂的功能。

   混入类常见的应用场景如下：
   - 序列化和反序列化：如上文的`ToDictMixin`和`JsonMixin`，可以将对象转换为字典或者JSON字符串，方便数据的存储和传输。

      - 添加日志功能：可以定义一个混入类，其中包含了写日志的方法，然后让需要日志功能的类继承这个混入类。

      - 授权和认证：在Web开发中，可以定义一个混入类，其中包含了用户认证的方法，然后让需要用户认证的View继承这个混入类。

4. 关于性能、效率和最佳实践的讨论：
   混入类的设计和使用需遵守一些最佳实践，这也能够确保你在使用时能够得到最好的性能和效率。

   - 使用组合而非继承：虽然混入类是通过继承来使用的，但你应该把它看作是一种组合机制，不要试图在混入类中定义实例属性和构造方法。混入类中只应包含方法，并且这些方法是独立于具体类的状态的。

   - 单一责任原则：每个混入类都应只关注于一件事情，比如序列化、写日志、用户认证等。这样可以保证混入类的通用性和可复用性。

   - 明确的接口：混入类应该有一个明确的接口。也就是说，混入类应该清楚地定义它需要什么样的方法或属性来完成它的工作。

   - 避免命名冲突：由于Python支持多重继承，所以可能会出现命名冲突的问题。你应该尽量让你的混入类的方法名具有描述性，以降低命名冲突的可能性。

     在性能和效率方面，混入类并不会引入额外的开销。实际上，由于混入类可以通过复用代码来减少代码量，所以它可能会带来性能上的微小提升。但请注意，由于混入类通常涉及到额外的方法调用，所以在一些极端的情况下，它可能会对性能产生一定的影响。但在大多数情况下，这个影响是可以忽略的。

5. 关于安全性的讨论：
   由于混入类只包含方法，不包含状态，所以它们通常不会引入安全问题。但是，你应该注意避免在混入类中引入可能会被恶意利用的功能。

   例如，如果你的混入类包含一个方法，该方法可以用来更改类的内部状态，那么恶意的代码可能会利用这个方法来修改类的行为。为了避免这种情况，你应该确保你的混入类的方法只能被你期望的类使用，而不能被其他类使用。

总的来说，混入类是一种强大的工具，可以帮助你编写出更为清晰、简洁和可复用的代码。通过正确地使用混入类，你可以提高代码的质量，同时也能提高你的开发效率。

### Item 42: Prefer Public Attributes Over Private Ones

1. 段落摘要总结

这个段落主要是讲解Python的公有属性和私有属性的概念和使用。

Python中，通过单下划线、双下划线前缀来表示属性的可见性。双下划线前缀的属性被称为私有属性，这是因为Python会对它们进行名称修饰，以防止在子类中被意外覆盖。然而，这并不能真正阻止外部访问，它们只是比没有下划线的公有属性更难访问。

段落的作者认为，除非你担心与子类的属性名称冲突，否则应尽可能地使用公有属性或者受保护的属性(单下划线前缀)，因为这样更利于代码的扩展。

2. 代码理解

让我们按照段落作者的思路逐步理解：

首先，理解公有属性和私有属性：

```python
class MyObject:
    def __init__(self):
        self.public_field = 5
        self.__private_field = 10

    def get_private_field(self):
        return self.__private_field


foo = MyObject()
print(foo.public_field)  # 输出: 5
print(foo.get_private_field())  # 输出: 10
```

尝试直接访问私有属性会抛出异常：

```python
try:
    print(foo.__private_field)
except AttributeError:
    print("Cannot access private field")  # 输出: Cannot access private field
```

然后，理解子类不能访问父类的私有属性：

```python
class MyParentObject:
    def __init__(self):
        self.__private_field = 71

class MyChildObject(MyParentObject):
    def get_private_field(self):
        return self.__private_field

baz = MyChildObject()
try:
    print(baz.get_private_field())
except AttributeError:
    print("Child object cannot access parent object's private field")  # 输出: Child object cannot access parent object's private field
```

但是，可以通过特殊的方法访问私有属性：

```python
print(baz._MyParentObject__private_field)  # 输出: 71
```

作者建议我们尽量使用公有属性，以便进行扩展。下面是一个例子：

```python
class MyBaseClass:
    def __init__(self, value):
        self._value = value  # 使用一个下划线前缀的受保护属性，而不是私有属性
    def get_value(self):
        return self._value

class MyStringClass(MyBaseClass):
    def get_value(self):
        return str(super().get_value()) 

class MyIntegerSubclass(MyStringClass):
    def get_value(self):
        return int(self._MyStringClass__value)

foo = MyIntegerSubclass(5)
print(foo.get_value())  # 输出: 5
```

3. 应用场景及应用原因总结

应用原因：私有属性并不能真正阻止外部访问，它们只是比公有属性更难访问。相反，公有属性更有利于代码的扩展和重用。只有在担心与子类属性名冲突的情况下，才应考虑使用私有属性。

```python
class ApiClass:
    def __init__(self):
        self.__value = 5 # Double underscore
    def get(self):
    	return self.__value # Double underscore
class Child(ApiClass):
    def __init__(self):
        super().__init__()
        self._value = 'hello' # OK!
a = Child()
print(f'{a.get()} and {a._value} are different')
>>>
5 and hello are different
```



应用场景：

- 如果你的类是其他程序员使用的API的一部分，那么使用私有属性可以防止与子类属性名称冲突。
- 如果你的类只是项目内部使用，或者你可以确定不会有其他类继承你的类，那么公有属性是更好的选择。

4. 实用代码段

这是一个实用的例子，展示如何在Python中使用公有属性，而不是私有属性：

```python
class MyPublicClass:
    def __init__(self, value):
        self.value = value

    def get_value(self):
        return self.value


class MyChildClass(MyPublicClass):
    def __init__(self, value):
        super().__init__(value)

    def print_value(self):
        print(self.get_value())


foo = MyChildClass(5)
foo.print_value()  # 输出: 5
```

### Item 43: Inherit from collections.abc for Custom Container Types

1. 段落摘要总结：
这段文字主要讲述了在Python编程中如何自定义类以实现容器行为。一般来说，我们可以直接继承Python的内置容器类型（如list或dict）来实现简单的容器功能，如作者第一个示例所示的FrequencyList类。但如果我们想要实现更复杂的行为，比如像列表一样支持索引但又不继承自列表的类型，直接继承内置容器类型会变得困难。在这种情况下，我们需要实现一些特殊的方法，如`__getitem__`和`__len__`。然而，要完全实现一个容器的行为，还需要实现更多的方法，这会非常繁琐。为了简化这个过程，Python提供了collections.abc模块，定义了一组抽象基类，这些基类提供了每种容器类型通常需要的所有方法。当我们从这些抽象基类继承并实现所需的方法时，该模块将为我们提供所有额外的方法，如index和count。如果我们忘记实现某些方法，该模块还会提示我们。

2. 代码理解与修改：
```python
# 示例一：定义一个FrequencyList类，继承自list，实现了统计元素频率的功能
class FrequencyList(list):
    def __init__(self, members):
        super().__init__(members)

    def frequency(self):
        counts = {}
        for item in self:
            counts[item] = counts.get(item, 0) + 1
        return counts

foo = FrequencyList(['a', 'b', 'a', 'c', 'b', 'a', 'd'])
print('Length is', len(foo))
foo.pop()
print('After pop:', repr(foo))
print('Frequency:', foo.frequency())
# 输出：
# Length is 7
# After pop: ['a', 'b', 'a', 'c', 'b', 'a']
# Frequency: {'a': 3, 'b': 2, 'c': 1, 'd': 1}

# 示例二：定义一个BinaryNode类，它不是list的子类，但我们可以实现__getitem__方法使其支持索引
class BinaryNode:
    def __init__(self, value, left=None, right=None):
        self.value = value
        self.left = left
        self.right = right

class IndexableNode(BinaryNode):
    def _traverse(self):
        if self.left is not None:
            yield from self.left._traverse()
        yield self
        if self.right is not None:
            yield from self.right._traverse()

    def __getitem__(self, index):
        for i, item in enumerate(self._traverse()):
            if i == index:
                return item.value
        raise IndexError(f'Index {index} is out of range')

# 但如果我们尝试调用len()函数，会发现出错，因为我们还需要实现__len__方法
# 这就是在自定义容器类型时所需要注意的问题，我们需要实现的方法可能比我们想象的要多

# 为了避免这种困扰，我们可以使用collections.abc模块
# 这个模块提供了一系列的抽象基类，当我们从这些类继承并实现了必须的方法后，它会为我们提供所有其他的方法
from collections.abc import Sequence

class SequenceNode(IndexableNode, Sequence):
    def __len__(self):
        for count, _ in enumerate(self._traverse(), 1):
            pass
        return count

# 现在我们就可以正确地使用len()函数了，而且还获得了index()和count()等方法
```
3. 应用场景与原因总结：
① 应用原因：当我们需要定义自己的容器类型时，直接继承Python的内置类型可能会遇到很多问题，因为实现所有的特殊方法非常繁琐。使用collections.abc模块可以帮助我们更容易地实现自定义容器类型，只需要实现必要的方法，其余的方法会自动获得。
② 应用场景：例如我们可能需要定义一个具有某种特定行为的序列类型（例如可以通过索引访问元素的二叉树），或者需要定义一个具有自定义方法（例如统计元素频率）的列表类型。在这些情况下，我们可以使用collections.abc模块来简化我们的工作。

4. 实用代码段：
```python
from collections.abc import Sequence

class BinaryNode:
    def __init__(self, value, left=None, right=None):
        self.value = value
        self.left = left
        self.right = right

class IndexableNode(BinaryNode):
    def _traverse(self):
        if self.left is not None:
            yield from self.left._traverse()
        yield self
        if self.right is not None:
            yield from self.right._traverse()

    def __getitem__(self, index):
        for i, item in enumerate(self._traverse()):
            if i == index:
                return item.value
        raise IndexError(f'Index {index} is out of range')

class SequenceNode(IndexableNode, Sequence):
    def __len__(self):
        for count, _ in enumerate(self._traverse(), 1):
            pass
        return count

tree = SequenceNode(
    10,
    left=SequenceNode(
        5,
        left=SequenceNode(2),
        right=SequenceNode(
            6,
            right=SequenceNode(7))),
    right=SequenceNode(
        15,
        left=SequenceNode(11))
)
print('Tree length is', len(tree))  # 输出：Tree length is 7
```

## Chapter Ⅵ: Metaclasses and Attributes  

Python的元类（Metaclasses）和动态属性（Dynamic Attributes）。

元类是Python的一个重要特性，它允许你在定义类时拦截Python的class声明，并提供特殊的行为。Python的动态属性让你能够自定义属性访问。这些特性能够帮助你从简单类平滑过渡到复杂类。但是，这些强大的特性也带来了很多陷阱，如动态属性可能会导致对象被覆盖和产生意想不到的副作用，元类可能会导致极其奇怪的行为，对新手来说难以理解。因此，你应遵循最少惊奇原则，只在实现众所周知的惯用法时使用这些机制。

① 动态属性示例：

```python
class DynamicAttributeClass:
    def __init__(self):
        self._attribute = "default"

    @property
    def attribute(self):
        return self._attribute

    @attribute.setter
    def attribute(self, value):
        self._attribute = value


dynamic_instance = DynamicAttributeClass()
print(dynamic_instance.attribute)  # 输出: default

dynamic_instance.attribute = "Changed"
print(dynamic_instance.attribute)  # 输出: Changed
```

这段代码展示了如何使用Python的动态属性特性，可以在运行时动态修改类的属性。

② 元类示例：

```python
class Meta(type):
    def __init__(cls, name, bases, attrs):
        print('Creating class:', name)
        super().__init__(name, bases, attrs)

class MyClass(metaclass=Meta):
    pass

# 输出: Creating class: MyClass
```

这段代码展示了如何使用Python的元类特性，通过拦截类的创建过程，我们可以在类创建时插入自定义的行为。

应用场景及应用原因总结:

① 动态属性可以用于创建灵活的API，这些API可以在运行时改变其行为。例如，Django模型和Tensorflow张量等使用了这个特性。

② 元类可以用于多种场景，包括在类创建时自动注册类，验证类定义，以及自动添加类属性和方法等。例如，Django ORM就使用了元类来将类与数据库表关联。

应用这些特性的主要原因是为了提高代码的灵活性和可复用性，同时减少代码的冗余。但是，由于它们增加了代码的复杂性，因此应谨慎使用。

用代码段:

```python
pythonCopy codeclass Meta(type):
    def __init__(cls, name, bases, attrs):
        print('Creating class:', name)
        super().__init__(name, bases, attrs)

class MyClass(metaclass=Meta):
    pass

class DynamicAttributeClass:
    def __init__(self):
        self._attribute = "default"

    @property
    def attribute(self):
        return self._attribute

    @attribute.setter
    def attribute(self, value):
        self._attribute = value

dynamic_instance = DynamicAttributeClass()
dynamic_instance.attribute = "Changed"
```

### Item 44: Use Plain Attributes Instead of Setter andGetter Methods

1. 段落摘要总结：

   这段文字主要讲述了在Python中不需要实现显式的setter和getter方法。取而代之，应始终从简单的公共属性开始实现。并且，如果后续需要在设置属性时添加特殊行为，可以迁移到@property装饰器及其相应的setter属性。另外，还可以使用@property让父类的属性变为不可变，以及进行一些类型检查和验证。最后，应避免在getter和setter方法中执行其他不可预期的副作用操作。

2. 代码演示及逐步理解：

   ①首先是旧式的方式，使用显式的getter和setter方法：
   
   ```python
   class OldResistor:
       def __init__(self, ohms):
           self._ohms = ohms

       def get_ohms(self):
           return self._ohms

       def set_ohms(self, ohms):
           self._ohms = ohms

   r0 = OldResistor(50e3)
   print('Before:', r0.get_ohms())
   r0.set_ohms(10e3)
   print('After: ', r0.get_ohms())
   ```
   
   输出：

   ```
   Before: 50000.0
   After: 10000.0
   ```

   ②然后是使用Python的方式，直接使用属性：

   ```python
   class Resistor:
       def __init__(self, ohms):
           self.ohms = ohms
           self.voltage = 0
           self.current = 0

   r1 = Resistor(50e3)
   r1.ohms = 10e3
   r1.ohms += 5e3
   print('After: ', r1.ohms)
   ```
   
   输出：

   ```
   After: 15000.0
   ```

   ③使用@property装饰器和setter方法来添加特殊行为：

   ```python
   class VoltageResistance(Resistor):
       def __init__(self, ohms):
           super().__init__(ohms)
           self._voltage = 0

       @property
       def voltage(self):
           return self._voltage

       @voltage.setter
       def voltage(self, voltage):
           self._voltage = voltage
           self.current = self._voltage / self.ohms

   r2 = VoltageResistance(1e3)
   print(f'Before: {r2.current:.2f} amps')
   r2.voltage = 10
   print(f'After: {r2.current:.2f} amps')
   ```
   
   输出：

   ```
   Before: 0.00 amps
   After: 0.01 amps
   ```
   
   ④使用setter进行类型检查和验证：

   ```python
   class BoundedResistance(Resistor):
       def __init__(self, ohms):
           super().__init__(ohms)
           self._ohms = ohms

       @property
       def ohms(self):
           return self._ohms

       @ohms.setter
       def ohms(self, ohms):
           if ohms <= 0:
               raise ValueError(f'ohms must be > 0; got {ohms}')
           self._ohms = ohms

   r3 = BoundedResistance(1e3)
   try:
       r3.ohms = 0
   except ValueError as e:
       print(e)
   ```
   
   输出：

   ```
   ohms must be > 0; got 0
   ```

   ⑤使用@property使得属性不可变：

   ```python
   class FixedResistance(Resistor):
       def __init__(self, ohms):
           super().__init__(ohms)
           self._ohms = ohms

       @property
       def ohms(self):
           return self._ohms

       @ohms.setter
       def ohms(self, ohms):
           if hasattr(self, '_ohms'):
               raise AttributeError("Ohms is immutable")
           self._ohms = ohms

   r4 = FixedResistance(1e3)
   try:
       r4.ohms = 2e3
   except AttributeError as e:
       print(e)
   ```
   
   输出：

   ```
   Ohms is immutable
   ```

3. 方法应用场景及应用原因总结：

   - 应用原因：使用@property能够实现对属性值的控制，包括类型检查、范围限制等。此外，还能实现当属性值改变时触发特定的行为，如更新其他属性等。

   - 常见应用场景：常用于实现只读属性，或者在设置属性值时要触发某些操作的情况。例如，在设计模型时，当属性值改变需要更新其他依赖此属性的属性时，可以用@property来实现。

4. 实用代码段：
   
   ```python
   class Resistor:
       def __init__(self, ohms):
           self._ohms = ohms
           self.voltage = 0
           self.current = 0
   
       @property
       def ohms(self):
           return self._ohms
   
       @ohms.setter
       def ohms(self, ohms):
           if ohms <= 0:
               raise ValueError(f'ohms must be > 0; got {ohms}')
           self._ohms = ohms
   ```
   
   这段代码定义了一个电阻类，使用@property装饰器限制了电阻值必须大于0，否则会抛出错误。

### Item 45: Consider @property Instead of Refactoring Attributes

> `__repr__` 是 Python 中的一个特殊方法（special method），用于定义对象的字符串表示形式。它返回一个字符串，用于表示对象的详细信息，通常用于调试和开发过程中。
>
> 当你在交互式环境中输出一个对象时，实际上会调用该对象的 `__repr__` 方法来获取其字符串表示形式。也可以使用 `repr()` 函数显式地调用 `__repr__` 方法来获取对象的字符串表示。
>
> 例如，如果你定义了一个名为 `Person` 的类，并为其定义了 `__repr__` 方法，你可以通过在对象上调用 `repr()` 或直接输出对象来获取其字符串表示形式。
>
> ```python
> class Person:
>     def __init__(self, name, age):
>         self.name = name
>         self.age = age
>     
>     def __repr__(self):
>         return f"Person(name='{self.name}', age={self.age})"
> 
> person = Person("Alice", 25)
> print(person)  # 输出: Person(name='Alice', age=25)
> print(repr(person))  # 输出: Person(name='Alice', age=25)
> ```
>
> 在上面的示例中，`__repr__` 方法返回了一个以类名为前缀的字符串，包含了对象的属性信息。这样做可以方便地查看对象的详细信息，有助于调试和理解程序的运行。

1. **段落摘要总结**

   本文是关于Python中的@property装饰器的讨论，它允许你在获取和设置属性时添加额外的行为。作者首先介绍了“漏桶算法”，该算法表示某种配额，一旦“桶”满了，这个配额就不能从一个时期延续到下一个时期。然而，原始的漏桶实现有一个问题：当配额为零时，无法知道“桶”开始时的配额是多少。作者使用@property来解决这个问题，通过新增的两个属性来跟踪最大配额和消耗的配额，并通过@property计算当前配额。最后，作者强调，在大规模使用@property时，应该考虑重构代码，以优化设计。

2. **段落行文逻辑与代码**

   - **原始Bucket类和其漏桶算法**
     
     首先，作者提出了Bucket类的问题：你不知道“桶”开始时的配额是多少。以下是Bucket类的代码：

     ```python
     from datetime import datetime, timedelta
     
     class Bucket:
         def __init__(self, period):
             self.period_delta = timedelta(seconds=period)
             self.reset_time = datetime.now()
             self.quota = 0
     
         def __repr__(self):
             return f'Bucket(quota={self.quota})'
     ```
     
     桶被填充后，每次消费者想要使用配额，都需要检查是否能扣除所需的配额量。以下是`fill`和`deduct`函数的代码：

     ```python
     def fill(bucket, amount):
         now = datetime.now()
         if (now - bucket.reset_time) > bucket.period_delta:
             bucket.quota = 0
             bucket.reset_time = now
         bucket.quota += amount
     
     def deduct(bucket, amount):
         now = datetime.now()
         if (now - bucket.reset_time) > bucket.period_delta:
             return False  # Bucket hasn't been filled this period
         if bucket.quota - amount < 0:
             return False  # Bucket was filled, but not enough
         bucket.quota -= amount
         return True  # Bucket had enough, quota consumed
     ```
     
     应用示例:

     ```python
     bucket = Bucket(60)
     fill(bucket, 100)
     print(bucket)  # Output: Bucket(quota=100)
     
     if deduct(bucket, 99):
         print('Had 99 quota')
     else:
         print('Not enough for 99 quota')
     print(bucket)  # Output: Bucket(quota=1)
     
     if deduct(bucket, 3):
         print('Had 3 quota')
     else:
         print('Not enough for 3 quota')
     print(bucket)  # Output: Bucket(quota=1)
     ```

   - **通过@property解决Bucket类的问题**

     为了解决上述问题，作者提出了一个新的Bucket类，该类引入了两个新的属性：`max_quota`和`quota_consumed`。

     ```python
     class NewBucket:
         def __init__(self, period):
             self.period_delta = timedelta(seconds=period)
             self.reset_time = datetime.now()
             self.max_quota = 0
             self.quota_consumed = 0
     
         def __repr__(self):
             return (f'NewBucket(max_quota={self.max_quota}, '
                     f'quota_consumed={self.quota_consumed})')
     
         @property
         def quota(self):
             return self.max_quota - self.quota_consumed
     
         @quota.setter
         def quota(self, amount):
             delta = self.max_quota - amount
             if amount == 0:
                 self.quota_consumed = 0
                 self.max_quota = 0
             elif delta < 0:
                 assert self.quota_consumed == 0
                 self.max_quota = amount
             else:
                 assert self.max_quota >= self.quota_consumed
                 self.quota_consumed += delta
     ```
     
     应用示例：

     ```python
     bucket = NewBucket(60)
     print('Initial', bucket)  # Output: Initial NewBucket(max_quota=0, quota_consumed=0)
     fill(bucket, 100)
     print('Filled', bucket)  # Output: Filled NewBucket(max_quota=100, quota_consumed=0)
     
     if deduct(bucket, 99):
         print('Had 99 quota')
     else:
         print('Not enough for 99 quota')
     print('Now', bucket)  # Output: Now NewBucket(max_quota=100, quota_consumed=99)
     
     if deduct(bucket, 3):
         print('Had 3 quota')
     else:
         print('Not enough for 3 quota')
     print('Still', bucket)  # Output: Still NewBucket(max_quota=100, quota_consumed=99)
     ```

3. **方法应用场景及应用原因总结**

   - **应用原因**

     使用@property的原因是，你可以在不修改调用方代码的情况下，对类的内部实现进行改变。这对于API的向后兼容性和代码的可维护性来说非常有用。特别是当你想给一个原本是简单数据的属性添加更复杂的行为时，@property是一个很有用的工具。

   - **应用场景**

     @property广泛应用于各种场景中，包括：

     1. 当你需要对属性值进行验证时。例如，如果你有一个`Person`类，其`age`属性必须在0到120之间，你可以使用@property来实现这个验证。
     2. 当你需要属性的值依赖于其他属性时。例如，你有一个`Rectangle`类，它有`width`和`height`属性，你可以使用@property来计算它的`area`。
     3. 当你需要在获取或设置属性时执行一些额外的操作时，如日志记录、事件触发等。

### Item 46: Use Descriptors for Reusable aproperty Methods

1. 段落摘要总结:
   
   本段内容是讲解Python的描述符(descriptor)如何用于重用@property方法，以实现属性的验证和行为的重复使用。初始的实现是在类中使用@property装饰器来设置和获取属性，并在设置属性时进行验证。然后作者通过示例指出这样做的问题，即无法重用验证代码，无法应用到不同的属性和不同的类。于是，作者引入了描述符，提供了__get__和__set__方法来实现属性的设置和获取，并在设置时进行验证。通过创建描述符类Grade，可以重用属性验证行为。但是，这个实现会有内存泄露的问题，因为描述符对象持有所有已设置过的对象的引用，导致它们无法被垃圾回收。最后，作者使用weakref模块中的WeakKeyDictionary解决了这个问题。WeakKeyDictionary会在Python运行时知道它持有对象的最后一个引用时，将对象从其项目集中移除。

2. 代码示例及解释:
   以下是一些用于理解这个概念的代码示例，以及它们的输出。

① 用@property实现属性验证:
```python
class Homework:
 def __init__(self):
     self._grade = 0

 @property
 def grade(self):
     return self._grade

 @grade.setter
 def grade(self, value):
     if not (0 <= value <= 100):
         raise ValueError('Grade must be between 0 and 100')
     self._grade = value

galileo = Homework()
galileo.grade = 95
print(galileo.grade)  # 输出：95
```
代码中的`@property`装饰器使得我们可以在设置`grade`属性的时候进行一些操作，如此例中的数值检查。

② 使用描述符重用属性验证:
```python
class Grade:
    def __init__(self):
        self._value = 0

    def __get__(self, instance, instance_type):
        return self._value

    def __set__(self, instance, value):
        if not (0 <= value <= 100):
            raise ValueError('Grade must be between 0 and 100')
        self._value = value

class Exam:
    math_grade = Grade()
    writing_grade = Grade()
    science_grade = Grade()

first_exam = Exam()
first_exam.writing_grade = 82
print(first_exam.writing_grade)  # 输出：82
```
这个例子使用描述符`Grade`重用属性验证行为，描述符提供了`__get__`和`__set__`方法来实现属性的设置和获取，并在设置时进行验证。

③ 使用WeakKeyDictionary解决内存泄露问题:
```python
from weakref import WeakKeyDictionary

class Grade:
    def __init__(self):
        self._values = WeakKeyDictionary()

    def __get__(self, instance, instance_type):
        return self._values.get(instance, 0)

    def __set__(self, instance, value):
        if not (0 <= value <= 100):
            raise ValueError('Grade must be between 0 and 100')
        self._values[instance] = value

class Exam:
    math_grade = Grade()
    writing_grade = Grade()
    science_grade = Grade()

first_exam = Exam()
first_exam.writing_grade = 82

second_exam = Exam()
second_exam.writing_grade = 75

print(first_exam.writing_grade)  # 输出：82
print(second_exam.writing_grade)  # 输出：75
```
这个例子使用`WeakKeyDictionary`解决了内存泄漏问题，`WeakKeyDictionary`会在Python运行时知道它持有对象的最后一个引用时，将对象从其项目集中移除。

3. 应用场景及应用原因总结:

   ① 应用原因:
   使用描述符可以更有效地重用属性的验证和行为，避免重复编写相同的代码，提高代码的可维护性。并且，使用WeakKeyDictionary可以避免因为描述符对象持有所有已设置过的对象的引用，导致的内存泄露问题。

   ② 应用场景:
   - 在需要对设置的属性进行验证的场景下，如设置学生的成绩，需要验证成绩在0到100之间；
   - 在需要将相同的属性行为应用到多个属性或多个类的场景下，如多个科目的成绩都需要进行相同的验证；
   - 在需要避免内存泄露的场景下，如在长时间运行的程序中，频繁地创建和删除对象，需要避免内存泄露。

4. 实用代码段:

```python
from weakref import WeakKeyDictionary

class Grade:
    def __init__(self):
        self._values = WeakKeyDictionary()

    def __get__(self, instance, instance_type):
        return self._values.get(instance, 0)

    def __set__(self, instance, value):
        if not (0 <= value <= 100):
            raise ValueError('Grade must be between 0 and 100')
        self._values[instance] = value

class Exam:
    math_grade = Grade()
    writing_grade = Grade()
    science_grade = Grade()

first_exam = Exam()
first_exam.writing_grade = 82

second_exam = Exam()
second_exam.writing_grade = 75

print(first_exam.writing_grade)  # 输出：82
print(second_exam.writing_grade)  # 输出：75
```
这个代码段演示了如何使用描述符进行属性的验证和行为的重复使用，并通过WeakKeyDictionary避免内存泄露。

### Item 47: Use `__getattr__, __getattribute__, and __setattr__` for Lazy Attributes

1. 段落摘要总结:
    这段文字详细介绍了Python的几个特殊方法, 如`__getattr__`, `__getattribute__`, 和 `__setattr__`，它们是如何用于Python中的惰性属性处理。`__getattr__`只有在属性不存在的情况下才会被调用，用于实现属性的动态绑定。`__getattribute__`不同，它会在每次属性访问时都被调用，允许在每次属性访问时执行更多的操作，如校验数据有效性。__setattr__也总是在属性被赋值时调用，可以用于拦截属性的赋值。需要注意的是，当在这些方法中访问其他属性时，必须使用`super()`，以避免无限递归。

2. 根据段落行文逻辑, 并给我代码来帮我进行理解: 

    (i) 使用__getattr__进行惰性属性绑定：

    ```python
    class LazyRecord:
        def __init__(self):
            self.exists = 5

        def __getattr__(self, name):
            value = f'Value for {name}'
            setattr(self, name, value)
            return value

    data = LazyRecord()
    print('Before:', data.__dict__)
    print('foo: ', data.foo)
    print('After: ', data.__dict__)
    ```

    输出：

    ```
    Before: {'exists': 5}
    foo: Value for foo
    After: {'exists': 5, 'foo': 'Value for foo'}
    ```

    (ii) 使用__getattribute__在每次属性访问时进行更多的操作：

    ```python
    class ValidatingRecord:
        def __init__(self):
            self.exists = 5

        def __getattribute__(self, name):
            try:
                value = super().__getattribute__(name)
                print(f'* Found {name!r}, returning {value!r}')
                return value
            except AttributeError:
                value = f'Value for {name}'
                print(f'* Setting {name!r} to {value!r}')
                setattr(self, name, value)
                return value

    data = ValidatingRecord()
    print('exists: ', data.exists)
    print('First foo: ', data.foo)
    print('Second foo: ', data.foo)
    ```

    输出：

    ```
    * Found 'exists', returning 5
    exists: 5
    * Setting 'foo' to 'Value for foo'
    First foo: Value for foo
    * Found 'foo', returning 'Value for foo'
    Second foo: Value for foo
    ```

    (iii) 使用__setattr__拦截属性的赋值：

    ```python
    class SavingRecord:
        def __setattr__(self, name, value):
            # Save some data for the record
            super().__setattr__(name, value)

    data = SavingRecord()
    print('Before: ', data.__dict__)
    data.foo = 5
    print('After: ', data.__dict__)
    data.foo = 7
    print('Finally:', data.__dict__)
    ```

    输出：

    ```
    Before: {}
    After: {'foo': 5}
    Finally: {'foo': 7}
    ```

3. 给我这个方法应用场景及应用原因总结:

    ① 应用的原因：这些特殊方法允许开发者在访问和修改属性时执行一些自定义操作，如动态绑定、数据校验和数据存储等。

    ② 常见应用场景例子：
       - 对象关系映射(ORM)：可以通过__getattr__实现字段的动态绑定，通过__setattr__拦截字段的赋值并同步到数据库。
       - 数据校验：在每次访问属性时，通过__getattribute__对数据进行校验。
       - 懒加载：对一些计算量大或需要从远程获取的属性，可以使用__getattr__实现懒加载。

4. 给我实用代码段:

    ```python
    class LazyDBRecord:
        def __init__(self, db_conn, record_id):
            self._db_conn = db_conn
            self._record_id = record_id
    
        def __getattr__(self, name):
            value = self._db_conn.fetch(self._record_id, name)
            setattr(self, name, value)
            return value
    
        def __setattr__(self, name, value):
            self._db_conn.save(self._record_id, name, value)
            super().__setattr__(name, value)
    
    # 示例代码, 模拟数据库连接
    class MockDBConn:
        def __init__(self):
            self.data = {
                1: {"name": "Alice", "age": 25},
                2: {"name": "Bob", "age": 30}
            }
    
        def fetch(self, record_id, field):
            return self.data[record_id][field]
    
        def save(self, record_id, field, value):
            self.data[record_id][field] = value
    
    db_conn = MockDBConn()
    record = LazyDBRecord(db_conn, 1)
    print('name:', record.name)  # 实际上会去数据库中查询
    print('age:', record.age)  # 实际上会去数据库中查询
    record.age = 26  # 实际上会去数据库中保存
    print('age:', record.age)  # 不需要查询，直接返回
    ```

### Item 48: Validate Subclasses with _ init_subclass_

1. 段落摘要总结:

   这个段落讨论了Python类定义过程中的验证和元类的使用，详细说明了使用元类来进行子类验证的方法，并展示了其存在的问题。在解决这些问题时，它引入了Python 3.6中的一个特性，`__init_subclass__`方法，用于在子类定义时进行验证。这个段落还讨论了`__init_subclass__`和元类相比的优势，包括更简单的代码、更好的错误提示、和更强大的功能（如处理多重继承和混合）。

2. 代码段：

   以下是这个段落中出现的主要代码段，并用PEP8格式修改了排版。同时，还包括了一些输出示例以帮助理解。

   ```python
   # 早期的使用元类进行子类验证的方法
   class Meta(type):
       def __new__(meta, name, bases, class_dict):
           print(f'* Running {meta}.__new__ for {name}')
           print('Bases:', bases)
           print(class_dict)
           return type.__new__(meta, name, bases, class_dict)

   class MyClass(metaclass=Meta):
       stuff = 123

       def foo(self):
           pass

   class MySubclass(MyClass):
       other = 567

       def bar(self):
           pass

   # 输出:
   # * Running <class '__main__.Meta'>.__new__ for MyClass
   # Bases: ()
   # {'__module__': '__main__', '__qualname__': 'MyClass', 'stuff': 123, 'foo': <function MyClass.foo at 0x105a05280>}
   # * Running <class '__main__.Meta'>.__new__ for MySubclass
   # Bases: (<class '__main__.MyClass'>,)
   # {'__module__': '__main__', '__qualname__': 'MySubclass', 'other': 567, 'bar': <function MySubclass.bar at 0x105a05310>}
   
   # 使用__init_subclass__进行子类验证的方法
   class BetterPolygon:
       sides = None  # Must be specified by subclasses

       def __init_subclass__(cls):
           super().__init_subclass__()
           if cls.sides < 3:
               raise ValueError('Polygons need 3+ sides')

       @classmethod
       def interior_angles(cls):
           return (cls.sides - 2) * 180

   class Hexagon(BetterPolygon):
       sides = 6

   # 输出:
   # assert Hexagon.interior_angles() == 720
   ```

3. 应用场景及应用原因总结:

   应用原因:

   `__init_subclass__`特性在子类定义时提供了一个运行点，允许我们在类定义时检查和修改类。这样我们可以确保每个子类都遵循某些约定或者满足某些条件。

   应用场景:

   - 在创建一个复杂的类层次结构时，我们可能想要对子类进行一些限制，如需要它们覆盖某些方法、或者需要它们具有某些特定的属性。这种情况下，我们可以使用`__init_subclass__`进行验证。
   - 当我们希望子类在继承时有某种特定的行为，如注册自己、或者修改自己的某些属性。这种情况下，`__init_subclass__`可以作为一个钩子，执行我们想要的代码。

4. 实用代码段:

   ```python
   class BetterPolygon:
       sides = None  # Must be specified by subclasses
   
       def __init_subclass__(cls):
           super().__init_subclass__()
           if cls.sides < 3:
               raise ValueError('Polygons need 3+ sides')
   
       @classmethod
       def interior_angles(cls):
           return (cls.sides - 2) * 180
   
   class Hexagon(BetterPolygon):
       sides = 6
   
   assert Hexagon.interior_angles() == 720
   ```

### Item 49: Register Class Existence with `_ init_subclass_`

### Item 50: Annotate Class Attributes with `__set_name_`
### Item 51: Prefer Class Decorators Over Metaclasses forComposable Class Extensions

## Chapter Ⅶ: Concurrency and Parallelism

## Chapter Ⅷ: Robustness and Performance

## Chapter Ⅸ: Testing and Debugging 

## Chapter Ⅹ: Collaboration 
