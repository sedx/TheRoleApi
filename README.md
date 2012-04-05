# gem 'the_role'

## Bye bye CanCan, I got The Role!

Semantic, lightweight role system with an administrative interface

![TheSortableTree](https://github.com/the-teacher/the_role/raw/master/pic.png)

## What does it mean semantic?

Semantic - the science of meaning. Human should fast to understand what is happening in a role system.

Look at hash. If you can understand access rules - this role system is semantically.

``` ruby
role = {
  :pages => {
    :index => true,
    :show => true,
    :new => false,
    :edit => false,
    :update => false,
    :destroy => false
  },
  :articles => {
    :index => true,
    :show => true
  }
  :twitter => {
    :button => true,
    :follow => false
  }
}
```

### How it  works 

Role - is a two-level hash, consisting of the sections and rules.

**Section** may be associated with the name of a **controller**.

**Rule** may be associated with the name of **action** in the controller.

Section may contain a set of rules.

**Rule in Section** can be set to **true** and **false**, this provide **ACL** (**Access Control List**)

Role hash **stored in the database as YAML** string.
Using of hashes, makes it extremely easy to configure access rules in the role.

### Virtual sections and rules

Usually, we use real names of controllers as names of sections, and we use names of real actions in controllers as names of section's rules.

Like this:

``` ruby
current_user.has_role?(:pages, :show)
```

But you can also create virtual sections and rules:


``` ruby
current_user.has_role?(:twitter, :button)
current_user.has_role?(:facebook, :like)
```

These sections and the rules are not associated with real controllers and actions.
And you can use them as well as other access rules.









### Как это работает?

Роль - это двухуровневый хеш-массив состоящий из секций и правил.

Секция может ассоциироваться с именем конкретного контроллера.
Правило может ассоциироваться с именем действия в контроллере.

Секция может содержать в себе множество правил.
Правило в секции может принимать значение **true** или **false**, обеспечивая тем самым работу списка контроля доступа (**Access Control List** или **ACL**)

Ролевой хеш-массив храниться в базе данных в виде YAML строки.
Использование хеш-массивов позволяет чрезвычайно легко настраивать правила доступа конкретной роли.

### Виртуальные секции и правила

Мы можете ассоциировать секции и правила роли с именами контроллеров и действий.
Например, что бы проверить возможность доступа пользователя к контроллеру **pages** и действию **show** вам потребуется выполнить проверку следующего вида:

``` ruby
current_user.has_role?(:pages, :show)
```

Здесь имя секции и имя правила полностью соответствуют реальному контроллеру и реальному действию.

Однако, иногда требуется настраивать права доступа к действиям, которые не соответствуют реальным контроллерам/действиям вашего приложения.
Никто не запрещает вам создавать и настраивать виртуальные секции и правила.
Например, вы можете настроить отображение социальных кнопок только для пользователей с заданными правилами:

``` ruby
current_user.has_role?(:twitter, :button)
current_user.has_role?(:facebook, :like)
```

Здесь мы создали и настроили для конктретной роли секции **twitter** и **facebook** и создали правила **button** и **like**.
Фрагмент ролевого хеша здесь может выглядеть так:

``` ruby
just_fragment = {
  :twitter => {
    :button => true
  },
  :facebook => {
    :like => false
  }
}
```

### Кто такой Администратор?

Администратор - это пользователь, который может получить доступ к любым секциям и правилам вашего приложения.
Администратор является собственником любых объектов вашего приложения.
Администратор - это пользователь в ролевом хеше которого есть секция **system** с правилом **administrator**.

``` ruby
admin_role_fragment = {
  :system => {
    :administrator => true
  }
}
```

### Кто такой Модератор?

Модератор - это пользователь, который может получить доступ к любым действиям заданной секеции.
Модератор является собственником любых объектов соответствующего класса.
Модератор - это пользователь в ролевом хеше которого есть секция **moderator** с правилом, соответствующим имени реального или виртуального секции (контроллера).

``` ruby
moderator_role_fragment = {
  :moderator => {
    :pages   => true,
    :blogs   => false,
    :twitter => true
  }
}
```

Пользователь с указанным выше ролевым хешем является модератором рального контроллера **pages** и виртуальной секции правил **twitter**.
Данная роль позволяет выполнять любые действия в контроллере **pages**, а пользователь считается собственником всех объектов класса **pages**.
Данная роль позволяет выполнять любые действия в виртуальном контроллере **twitter**, т.е. метод **current_user.has_role?(:twitter, :any_crazy_name)** будет возвращать значение true

### Методы модели User

Обладает ли пользователь правом досутпа к данному действию контроллера?

``` ruby
current_user.has_role?(:pages,    :show)  => true | false
current_user.has_role?(:blogs,    :new)   => true | false
current_user.has_role?(:articles, :edit)  => true | false
```

Является ли пользователь модератором указанного контроллера?

``` ruby
current_user.moderator?(:pages)           => true | false
current_user.moderator?(:blogs)           => true | false
current_user.moderator?(:articles)        => true | false
```

Является ли пользователь администратором?

``` ruby
current_user.admin?                       => true | false
```

Является ли пользователь владельцем объекта?

``` ruby
current_user.owner?(@page)                => true | false
current_user.owner?(@blog)                => true | false
current_user.owner?(@article)             => true | false
```

Методы модели Role

``` ruby
# Найти роль с заданным именем
@role.find_by_name(:user)
```

``` ruby
# User like methods

@role.has?(:pages, :show)       => true | false
@role.moderator?(:pages)        => true | false
@role.admin?                    => true | false
```

## CRUD API

### CREATE


``` ruby
# Создать секцию прав
@role.create_section(:pages)
```

``` ruby
# Создать правило в указанной секции
@role.create_rule(:pages, :index)
```

### READ

``` ruby
# Получить роль в виде хеша
@role.to_hash => Hash

# Получить роль в виде YAML string
@role.to_yaml => String

# Получить роль в виде YAML string
@role.to_s => String
```

### UPDATE

``` ruby
# Обновить роль.
# Входящий аргумент - массив маска со значениями true
# Все правила исходного массива будут сброшены к значениям false
# Значения правил будут установлены на true только там
# где это указано во входящем массиве
new_role_hash = {
  :pages => {
    :index => true,
    :show => true
  }
}

@role.update_role(new_role_hash)
```

``` ruby
# Установить указанное правило на true
@role.rule_on(:pages, :index)
```

``` ruby
# Установить указанное правило на false
@role.rule_off(:pages, :index)
```

### DELETE

``` ruby
# Удалить секцию правил
@role.delete_section(:pages)

# Удалить указанное правило
@role.delete_rule(:pages, :show)
```


### ================================================

## TheRole

Gem for providing simple, but powerful and flexible role system for ROR3 applications.
Based on Hashes.

* Based on MVC semantik (easy to understand what's happening)
* Realtime dynamically management with simple interface
* Customizable

##  Installation

Gemfile

``` ruby
  gem 'the_role'
```

``` ruby
  bundle install
```

``` ruby
rake the_role_engine:install:migrations
  >> Copied migration 20111028145956_create_roles.rb from the_role_engine

rails g model user role_id:integer
rails g model role --migration=false

rake db:create && rake db:migrate
```

Creating roles for test

``` ruby
rake db:roles:test
  >> Administrator, Moderator of pages, User, Demo
```

Define aliases method for correctly work TheRole's controllers

**login_required** or any other method from your auth system

**access_denied** or any other method for processing access denied situation

``` ruby
class ApplicationController < ActionController::Base
  # You Auth system
  include AuthenticatedSystem

  # define aliases for correctly work of TheRole gem
  alias_method :the_login_required, :login_required
  alias_method :the_role_access_denied, :access_denied
end
```

##  Using with controllers

``` ruby
class ArticlesController < ApplicationController
  # Auth system
  before_filter :login_required, :except=>[:index, :show, :tag]
  # Role system
  before_filter :the_role_require,  :except => [:index, :show, :tag]
  before_filter :the_role_object,   :only => [:edit, :update, :rebuild, :destroy]
  before_filter :the_owner_require, :only => [:edit, :update, :rebuild, :destroy]

  # actions
  
end
```

**the_role_object** try to define instance variable by current controller name and params[:id]
@recipe for RecipesController
@user for UserController

Instance variable required for correctly work of **the_owner_require** method.

If you want to define other finder method, you should define **@the_role_object** for correctly work of **the_owner_require** method.

For example

``` ruby
class UsersController < ApplicationController
  before_filter :require_login, :except => [:new, :create]
  before_filter :the_role_require, :except => [:new, :create]
  before_filter :find_user, :only => [:edit]
  before_filter :the_owner_require, :only => [:edit]

  def new; end
  def create; end
  def cabinet; end
  def edit; end

  private

  def find_user
    @user = User.where(:login => params[:id]).first
    @the_role_object = @user
  end
end
```

##  Manage roles

``` ruby
rails s
```

**admin_roles_path** => **http://localhost:3000/admin/roles**

##  How it works

``` ruby
rails c

user = User.first
user.role = Role.where(:name => :demo).first
user.save

user.admin?
  => false
user.moderator? :pages
  => false
user.has_role? :pages, :index
  => true 
 

user.role = Role.where(:name => :moderator).first
user.save

user.admin?
  => false
user.moderator? :pages
  => true
user.has_role? :pages, :any_crazy_name
  => true

user.role = Role.where(:name => :admin).first
user.save

user.admin?
  => true
user.moderator? :pages
  => true
user.moderator? :any_crazy_name
  => true
user.has_role? :any_crazy_name, :any_crazy_name
  => true

```

Copyright (c) 2011 [Ilya N. Zykin Github.com/the-teacher], released under the MIT license