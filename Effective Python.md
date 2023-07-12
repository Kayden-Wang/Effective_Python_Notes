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

* Item 11: Know How to Slice Sequences
* Item 12: Avoid Striding and Slicing in a Single Expression
* Item 13: Prefer Catch-All Unpacking Over Slicing
* Item 14: Sort by Complex Criteria Using the key Parameter

## Chapter Ⅲ: Functions 

## Chapter Ⅳ: Comprehensions and Generators 

## Chapter Ⅴ: Classes and Interfaces 

## Chapter Ⅵ: Metaclasses and Attributes

## Chapter Ⅶ: Concurrency and Parallelism

## Chapter Ⅷ: Robustness and Performance

## Chapter Ⅸ: Testing and Debugging 

## Chapter Ⅹ: Collaboration 
