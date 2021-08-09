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
- [Chapter 7: Moving Features Between Objects](#chapter-7-moving-features-between-objects)
  - Move Method
  - Move Field
  - Extract Class
  - Hide Delegate (+ Remove Middle Man)
  - Introduce Local Extension
- [Chapter 8: Organizing Data](#chapter-8-organizing-data)
  - Replace Data Value with Object
  - Replace Array with Object
  - Replace Magic Number with Symbolic Constant
  - Replace Type Code with Class
  - Encapsulate Collection

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

---
<br>

# <b>Chapter 7: Moving Features Between Objects</b>

Generally, we aim to ensure that each field and method are placed in the correct classes and objects, ensuring that responsibilities belong to its rightful owner. The refactoring approaches in this chapter address these code smells and closely enforces SOLID's SRP (Single Responsibility Principle). Also, this often led to small and concise classes, improving readability.

<i>Note: For this chapter, I will be borrowing some concepts from ActiveRecord (Rails' ORM) to easier illustrate some of the pointers</i>

#### <b>>> Move Method</b>

<b>TL;DR: Move the method into the class that uses it most + Turn the old method into a simple delegation, or remove it altogether</b>

Sign(s) of code smell:
- A method is or will be, using or used by more features of another class than the class on which it
is defined.

```ruby
# Before
class Account
  attr :days_overdrawn
  has_one :account_type, class_name: AccountType.to_s

  def overdraft_charge
    return days_overdrawn * 1.5 if account_type.premium?
    days_overdrawn * 2
  end
end

class AccountType
  belongs_to :account, class_name: Account.to_s
  # ...
end
```

```ruby
# After
class Account
  attr :days_overdrawn
  has_one :account_type, class_name: AccountType.to_s

  def overdraft_charge
    account_type.overdraft_charge
  end
end

class AccountType
  belongs_to :account, class_name: Account.to_s

  def overdraft_charge
    return account.days_overdrawn * 1.5 if premium?
    account.days_overdrawn * 2
  end
end
```

Why it is better:
- This sets a better foundation for cleaner code down the road too, especially if there are multiple `AccountType` and each has its implementation of `overdraft_charge`. We can use [Replace Conditional with Polymorphism](https://refactoring.guru/replace-conditional-with-polymorphism), which follows the SOLID's Open-Closed Principle very closely.

---<br>

#### <b>>> Move Field</b>

<b>TL;DR: Move the field into the class that uses it the most</b>

Sign(s) of code smell:
- A field is, or will be, used by another class more than the class on which it is defined.

```ruby
# Before
class Account
  attr :interest_rate
  has_one :account_type, class_name: AccountType.to_s

  def interest(amount, days)
    interest_rate * amount * days
  end
end

class AccountType
  belongs_to :account, class_name: Account.to_s
  # ...
end
```

```ruby
# After
class Account
  has_one :account_type, class_name: AccountType.to_s

  # Solution 1
  def interest(amount, days)
    account_type.interest_rate * amount * days
  end

  # Solution 2 
  def interest(amount, days)
    interest_rate * amount * days
  end

  def interest_rate
    account_type.interest_rate
  end
end

class AccountType
  attr :interest_rate
  belongs_to :account, class_name: Account.to_s
  # ...
end
```

Why it is better:
- Similar to "Move Method"

Note:
- Solution 1 is the more preferred refactoring as `interest_rate` is removed from the class altogether, and no additional method has been introduced
- However, Solution 2 is "safer" and more recommended if there are insufficient test coverages, as there's a much lower chance of things breaking.

---<br>

#### <b>>> Extract Class</b>

<b>TL;DR: Extract logics into a new class</b>

Sign(s) of code smell:
- Classes overly bloated with methods that don't really fall under its concern and responsibilities

```ruby
# Before
class Person
  attr :area_code, :office_number

  def telephone_number
    "(#{@area_code} #{@office_number})"
  end
end
```

```ruby
# After
class Person
  attr :area_code, :office_number

  def telephone_number
    TelephoneNumber.new(@area_code, @office_number).call
  end
end

class TelephoneNumber
  def initialize(area_code, office_number)
    @area_code = area_code
    @office_number = office_number
  end

  def call
    "(#{@area_code} #{@office_number})"
  end
end
```

Why it is better:
- This is similar to using [Value Objects to avoid Primitive Obsession]({% post_url 2021-05-26-value-objects-to-avoid-primitive-obsession %})
  - Improve readability
  - Business logics are abstracted and hidden away

---<br>

#### <b>>> Hide Delegate (+ Remove Middle Man)</b>

<b>TL;DR: Create a new method to hide delegate class from client</b>

Sign(s) of code smell:
- A client is calling a delegate class of an object.

```ruby
# Before
class Person
  belongs_to :department, class_name: Department.to_s
end

class Department
  attr :manager
end

manager = john.department.manager
```

```ruby
# After
class Person
  belongs_to :department, class_name: Department.to_s

  def manager
    department.manager
  end
end

class Department
  attr :manager
end

manager = john.manager
```

Why it is better:
- Abstract logic on how the `Department` class works and hide it away from the client 

Note:
- IMO, I see a minimal benefit when the delegate class is immediate, especially at the cost of adding a method. However, the benefit multiplies with the number of delegation e.g. `john.department.manager.previous_job.salary`
- However, if you are using templating engines (like ERB for Rails) that allow the client to access the backend, this will be a great practice to separate the logic from the client.
- If the benefit is negligible, we can reverse it with <b>Remove Middle Man</b> (explanation skipped)

---<br>

#### <b>>> Introduce Local Extension</b>

<b>TL;DR: Create a new class (subclass OR wrapper) as an extension of the original class</b>

Sign(s) of code smell:
- A server class you are using needs several additional methods, but you can't modify the class.

```ruby
# Before
class JobListing
  # ...
end

def annual_salary_from_monthly(monthly_salary)
  monthly_salary * 12
end

j = JobListing.new
a = annual_salary_from_monthly(j.monthly_salary)
```

```ruby
# After (with subclass)
class JobListingInheritance < JobListing
  def annual_salary
    monthly_salary * 12
  end
end

j = JobListingInheritance.new
a = j.annual_salary
```

```ruby
# After (with wrapper class)
class JobListingWrapper
  def initialize(job_listing)
    @job_listing = job_listing
  end

  def annual_salary
    @job_listing.monthly_salary * 12
  end
end

j = JobListing.new
a = JobListingWrapper.new(j).monthly_salary
```

Why it is better:
- Replaces `annual_salary_from_monthly` by and
  - Give it a better name
  - Restraining it such that it won't be used in the wrong context

Note:
- The issue with using a wrapper class is that it is no longer the same instance and class as the original class. `inheritance.is_a?(JobListing)` will return true while `wrapper.is_a?(JobListing)` will return false.

---
<br>

# <b>Chapter 8: Organizing Data</b>

#### <b>>> Replace Data Value with Object</b>

<b>TL;DR: Turn the data item into an object</b>

Sign(s) of code smell:
- Simple objects aren't so simple and require quite a bit of manipulation

```ruby
# Before
class JobListing
  def initialize(salary_min, salary_max)
    @salary_min = salary_min
    @salary_max = salary_max
  end

  def salary_range_within_1000?
    @salary_max - @salary_min < 1000
  end
end

j = JobListing.new(4000, 5000)
return j.salary_range_within_1000?
```

```ruby
# After
class JobListing
  def initialize(salary_min, salary_max)
    @salary_range = SalaryRange.new(salary_min, salary_max)
  end
end

class SalaryRange
  def initialize(salary_min, salary_max)
    @salary_min = salary_min
    @salary_max = salary_max
  end

  def within_1000?
    @salary_max - @salary_min < 1000
  end
end

j = JobListing.new(4000, 5000)
return j.salary_range.within_1000?
```

Why it is better:
- This has been explained in greater detail in [Value Objects to avoid Primitive Obsession]({% post_url 2021-05-26-value-objects-to-avoid-primitive-obsession %})
  - Improved Readability
  - Business logics are abstracted and hidden away
  - The class can be extended easily

---<br>

#### <b>>> Replace Array with Object</b>

<b>TL;DR: Replace the array with an object that has a field for each element</b>

Sign(s) of code smell:
- The array is used to contain different objects e.g. <i>the 1st element is the name, 2nd is the age...</i>

```ruby
# Before
a = ['Adrian', 'Goh', 25]
puts "First name is #{a[0]}"
```

```ruby
# After

# Solution #1
Person = Struct.new(:first_name, :last_name, :age)
a = Person.new('Adrian', 'Goh', 25)
puts "First name is #{a.first_name}"

# Solution #2
Person = Struct.new(:first_name, :last_name, :age, keyword_init: true)
a = Person.new(first_name: 'Adrian', last_name: 'Goh', age: 25)
puts "First name is #{a.first_name}"
```

Why it is better:
- This is very similar to <b>Replace Data Value with Object</b>
- Also, give more context, e.g. `a[0]` doesn't tell you anything, but `a.first_name` does

Note:
- Solution #2 requires keywords to be included during initiation. This is more encouraged when there are many attributes in the object.

---<br>

#### <b>>> Replace Magic Number with Symbolic Constant</b>

<b>TL;DR: Replace the number with a constant instead</b>

Sign(s) of code smell:
- You have a literal number with a particular meaning.

```ruby
# Before
def price_in_usd(price_in_sgd)
  price_in_sgd * 0.74
end
```

```ruby
# After
SGD_TO_USD_CONVERSION_RATE = 0.74

def price_in_usd(price_in_sgd)
  price_in_sgd * SGD_TO_USD_CONVERSION_RATE
end
```

Why it is better:
- It might not seem much, but using constant makes updating so much simpler as we only have to update it in one place instead of finding all the instances (this multiplies with its usage count)
- 100% lead to improved contextualization

---<br>

#### <b>>> Replace Type Code with Class</b>

<b>TL;DR: Replace the type with a new class</b>

Sign(s) of code smell:
- A class has a numeric type code that does not affect its behavior.

```ruby
# Before
class Employee
  BACKEND = 'Backend Developer'
  FRONTEND = 'Frontend Developer'
  TITLE_OPTIONS = [BACKEND, FRONTEND]

  def initialize(title)
    @title = title
  end
end

Employee.new(Employee::BACKEND)
```

```ruby
# After
class Employee
  def initialize(title)
    @title = title
  end
end

class Title
  BACKEND = 'Backend Developer'
  FRONTEND = 'Frontend Developer'
  OPTIONS = [BACKEND, FRONTEND]
end

Employee.new(Title::BACKEND)
```

Why it is better:
- Keep `Employee` class small and readable

---<br>

#### <b>>> Encapsulate Collection</b>

<b>TL;DR: Make it return a read-only view + provide add/remove methods.</b>

Sign(s) of code smell:
- A method returns a collection

```ruby
# Before
class Company
  attr_accessor :people

  def initialize
    @people = []
  end
end

c = Company.new
c.people = [Person.new('Adrian'),
            Person.new('Goh')]
```

```ruby
# After
class Company
  attr_reader :people

  def initialize
    @people = []
  end
  
  def add_person(person)
    # perhaps some logic to validate if person can be added
    @people << person
  end
end

c = Company.new
c.add_person(Person.new('Adrian'))
c.add_person(Person.new('Goh'))
```

Why it is better:
- Preserve attribute's integrity e.g. Avoid adding an object that's not `Person` into `people`
- Avoid revealing too much to clients about the object's internal data structures.

---
<br>
