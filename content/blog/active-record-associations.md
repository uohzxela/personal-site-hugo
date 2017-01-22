+++
draft = false
date = "2014-05-27T10:35:51+08:00"
tags = []
comments = true
title = "Associations in Active Record"
showcomments = true
showpagemeta = true
categories = ["active record", "rails", "database schema"]
slug = ""

+++

Hi there. This week I’m going to blog about Active Record associations. They are used to declare relationships between model classes in Rails.

There are six types of AR associations in Rails:

1. `belongs_to`
2. `has_one`
3. `has_many`
4. `has_many :through`
5. `has_one :through`
6. `has_and_belongs_to_many`

In addition, there are three types of relationships that can expressed between model classes:

1. One-to-many
2. Many-to-many
3. One-to-one

This post will focus on each of the relationships, explaining how some associations work together to achieve a particular relationship between model classes.

<hr>

### One-to-many

Associations used:

1. `belongs_to`
2. `has_many`

Imagine you have two tables: Employee and Department. An employee can only belong to one department and one department can have many employees. If you have ever taken a database course before, this notation would look familiar:

	Employee 1 : n Department

This can be interpreted as follows: for each employee it can have only 1 department, for each department it can have multiple employees.

Clearly, from the notation above, you can easily intuit about the one-to-many relationship.

Here’s the code snippet on such a construction:

```ruby
class Employee < ActiveRecord::Base
  belongs_to :department
end

class Department < ActiveRecord::Base
  has_many :employees
end
```

Note the pluralization of the word “employee” when it is declared with `has_many` and the singular form of “department”.

It’s pretty self-explanatory. An employee belongs to a department as he/she cannot belong to more than one department. An department has many employees as a department is a collection of objects; in this case, it is a group of more than one person.

One caveat though: whichever table contains the statement `belongs_to :<table name>` must have the foreign id attribute of `<table name>`. That way, it can reference the row of the associated table it belongs to. In this case, the table Employee has an attribute called `department_id` as the Department table’s foreign id.

```ruby
class CreateEmployeess < ActiveRecord::Migration
  def change
    create_table :employees do |t|
      t.string   :name
      t.string    :email
      t.integer  :department_id
    end
  end
end
```

<hr>

### Many-to-many

Associations used:

1. `belongs_to`
2. `has_many`
3. `has_many :through`
4. `has_and_belongs_to_many`

Imagine you have two tables: Employee and Skill. Each employee has multiple skills (Java, Cobol etc) and each skill can be shared by multiple employees.

Notation:

	Employee n : n Skills

This is pretty self-explanatory. However, to implement this relationship in Rails is a bit tricky.

First thing first, you need to create a join table, preferrably named as `employees_skills` due to the Rails convention of naming join table in lexicographical order. Take note of the pluralization of both table names too.

Why create a join table for many-to-many relationship? Because we need to normalize the database to reduce redundancy. Yes we still can implement many-to-many relationship without the join table but I’m sure the employee table will contain a bunch of duplicated employee data along with distinct skill data:


Employee ID | Name | Email | Skill
------------|------|-------|------
1	|Alex	| alex@asdf.com	|Cobol
1	|Alex	| alex@asdf.com	|Java
1	|Alex	| alex@asdf.com	|Ruby
2	|Kevon	| kevon@asdf.com	|Lisp
2	|Kevon	| kevon@asdf.com	|Ruby
3	|Chen	| chen@asdf.com	|Haskell

Instead of the extremely redundant table above, why not we have a join table like this:

Employee ID | Skill
------------|------
1	|Cobol
1	|Java
1	|Ruby
2	|Lisp
2	|Ruby
3	|Haskell

Data from both tables is now stored in one place, and is referenced by their IDs instead of raw data. This leads to very efficient storage of data through database normalization.

To implement many-to-many relationships in Rails, there are two ways of doing this: `has_many :through` and `has_and_belongs_to_many`.

`has_and_belongs_to_many` (HMABT) is a more concise way of creating many-to-many relationship but as quoted from The Rails Way it is “practically obsolete in the minds of many Rails developers”. The reason is that the HMABT join model is not real; it cannot contain any attributes other than the foreign IDs of the two tables it is joining. If you want to add extra columns to the join table, you need to use `has_many :through`.

### has_many_and_belongs_to

Model classes:

```ruby
class Employee < ActiveRecord::Base
  has_and_belongs_to_many :skills
end

class Skill < ActiveRecord::Base
  has_and_belongs_to_many :employees
end
```

We need to create a new table as the join table, but since there’s no real join model using HMABT method (normally Rails will generate a table from model for us), we have to create the table through database migration.

Assuming the above statements are added directly to model classes, here is the corresponding migration to create join table:

```ruby
class CreateEmployeesAndSkills < ActiveRecord::Migration
    create_table :employees_skills, id: false do |t|
      t.belongs_to :employee
      t.belongs_to :skill
    end
  end
end
```

### has_many :through

`has_many :through` is a more verbose way of creating many-to-many relationship but you will get a real join model.

Model classes:

```ruby
class Employee < ActiveRecord::Base
  has_many :skillsets
  has_many :skills, through :skillsets
end

class Skill < ActiveRecord::Base
  has_many :skillsets
  has_many :employees, through :skillsets
end

class Skillset < ActiveRecord::Base
  belongs_to :employee
  belongs_to :skill
end
```

The difference here is that we have created an actual join model that acts just like any other models; this brings flexibility such as the freedom to add extra attributes to the join table and also to reference it directly.

<hr>

### One-to-one

Associations used:

1. `has_one`
2. `has_one :through`
3. `belongs_to`

`has_one` is like `has_many`, but when the database query is executed, a single object is retrieved instead. An example would be such that an employee only has one passport. `has_one :through` is used to reference an object through the intermediate singleton, e.g., an employee has only one passport number as referenced in his passport.

Notation:

	Employee 1 : 1 Passport 1 : 1 Passport Number

It can be interpreted as follows: an employee has a single passport which has a single passport number.

Model classes:

```ruby
class Employee < ActiveRecord::Base
  has_one :passport
  has_one :passport_number, through: passport
end

class Passport < ActiveRecord::Base
  belongs_to :employee
  has_one :passport_number
end

class PassportNumber < ActiveRecord::Base
  belongs_to :passport
end
```
