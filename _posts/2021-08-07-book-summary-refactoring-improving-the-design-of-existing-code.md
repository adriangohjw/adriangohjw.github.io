---
layout: post
title:  "[Book Summary] Refactoring: Improving the Design of Existing Code"
date:   2021-08-07 18:08:00 +0800
categories: posts
---

Recently, I started reading <b>Refactoring: Improving the Design of Existing Code</b> by Martin Fowler. I thought it would be useful to translate my learnings into bite-size content in Ruby for future references.

Note:
- As the source is in Java, I have modified the code to make more sense in Ruby
- As I am trying to keep this concise such that my future self can easily refer to it (hi future self if you are reading this), I will be skipping the implementation steps as I find it to be rather straightforward once we grasped the concept.

# <b>Table of Content</b>
- [Chapter 6: Composing Methods](#chapter-6-composing-methods)
  - Extract Method
  - Introduce Explaining Variable
  - Inline Method
  - Replace Temp with Query
  - Split Temporary Variable
  - Remove Assignments to Parameters
  - Replace Method with Method Object

---
<br>

# <b>Chapter 6: Composing Methods</b>

#### <b>>> Extract Method</b>

<b>TL;DR: Turn the fragment into a method whose name explains the purpose of the method</b>

Sign(s) of code smell:
- The method is too long
- Code requires comment to understand its purpose

```ruby
# Before
def printOwing(amount)
  print_banner

  # print details
  puts("name: #{name}")
  puts("amount: #{amount}")
end
```

```ruby
# After
def printOwing(amount)
  print_banner
  print_details(amount)
end

def print_details(amount)
  puts("name: #{name}")
  puts("amount: #{amount}")
end
```

Why it is better:
- Code can be reused
- Improves readability
- Possible to test code in parts

Note:
- Descriptive method names are needed

---<br>

#### <b>>> Introduce Explaining Variable</b>

<b>TL;DR: Put the result of a complicated expression, or parts of it, in a temporary variable</b>

Sign(s) of code smell:
- Expression is long, complicated and hard-to-read (usually conditional statement)

```ruby
# Before
if ((platform.upcase.index_of('MAC') > -1) &&
    (browser.upcase.index_of('IE') > -1) &&
    resize > 0)
  # ...
end
```

```ruby
# After
is_platform_mac = platform.upcase.index_of('MAC') > -1
is_browser_ie = browser.upcase.index_of('IE') > -1
was_resized = resize > 0

if (is_platform_mac && is_browser_ie && was_resized)
  # ...
end
```

Why it is better:
- Improves readability

Note:
- Almost always, able to refactor using Extract Method instead. This is preferred only when there are many local variables, making it difficult to extract

---<br>

#### <b>>> Inline Method</b>

<b>TL;DR: Put the method's body into the body of its callers and remove the method.</b>

Sign(s) of code smell:
- The method name is as clear as its implementation

```ruby
# Before
def rating
  more_than_5_late_deliveries? ? 2 : 1
end

def more_than_5_late_deliveries?
  late_deliveries.count > 5
end
```

```ruby
# After
def rating
  late_deliveries.count > 5 ? 2 : 1
end
```

Why it is better:
- Needless indirection is harder to read

---<br>

#### <b>>> Replace Temp with Query</b>

<b>TL;DR: Extract expression into a method and replace all temp references with it</b>

Sign(s) of code smell:
- Temps are local --> likely to be useless without the context of the method they are in

```ruby
# Before
def some_method
  # ...
  base_price = quantity * item_price
  if base_price > 1000
    return base_price * 0.95
  else
    return base_price * 0.98
  end
end
```

```ruby
# After
def some_method
  # ...
  if base_price > 1000
    return base_price * 0.95
  else
    return base_price * 0.98
  end
end

def base_price
  quantity * item_price
end
```

Why it is better:
- Logic is abstracted and you don't have to worry about its implementation
- Code can be reused
- Cleaned up code likely to open up room for more and easier refactorings

Note:
- Most of the time, performance degradation is very much negligible

---<br>

#### <b>>> Split Temporary Variable</b>

<b>TL;DR: Make a separate temporary variable for each assignment</b>

Sign(s) of code smell:
- A temporary variable is assigned to more than once but is not a loop variable nor a collecting temporary variable

```ruby
# Before
temp = 2 * (height + width)
puts(temp)
temp = height * width
puts(temp)
```

```ruby
# After
perimeter = 2 * (height + width)
puts(perimeter)
area = height * width
puts(area)
```

Why it is better:
- Ensure each variable follows SRP (Single Responsibility Principle)
- Reusing a temp for multiple things are confusing for readers
- More likely lead to better naming of the variables

---<br>

#### <b>>> Remove Assignments to Parameters</b>

<b>TL;DR: Use a temporary variable instead</b>

Sign(s) of code smell:
- Parameters have been reassigned (especially if it refers to a completely different object)

```ruby
# Before
def discount(value)
  value = 2 if value > 50
  # ...
end
```

```ruby
# After
def discount(value)
  result = value
  result = 2 if value > 50
  # ...
end
```

Why it is better:
- Much clearer if you use only the parameter to represent what has been passed in because that is a consistent usage

---<br>

#### <b>>> Replace Method with Method Object</b>

<b>TL;DR: Turn the method into its object so that all the local variables become fields on that object.
You can then decompose the method into other methods on the same object.</b>

Sign(s) of code smell:
- The method is long with many local variables that cannot be extracted

```ruby
# Before
class Account
  # Note: Don't worry about what this logic means, it doesn't matter
  def discount(value, quantity)
    a = (value * quantity) + delta
    b = (value + quantity) + 100
    b -= 20 if a > 100
    c = b * 7
    # and so on
  end

  def delta
    # ...
  end
end
```

```ruby
# After
class Account
  def some_function(value, quantity)
    Discount.new(self, value, quantity).call
  end

  def delta
    # ...
  end
end

class Discount
  def initialize(account, value, quantity)
    @account = account
    @value = value
    @quantity = quantity
  end

  def call
    a = (@value * @quantity) + @account.delta
    b = (@value + @quantity) + 100
    b -= 20 if a > 100
    c = b * 7
    # and so on
  end
end
```

Why it is better:
- Can refactor `call` into smaller methods given that logic is encapsulated in the `Discount` object
