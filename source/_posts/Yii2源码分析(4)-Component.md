---
title: Yii2源码分析(4)-Component
date: 2018-12-04 10:57:48
tags:
    - Yii2
    - PHP
categories:
    - Yii2
comments: true
---

`Component`是实现`behavior`、`event`和`property`特性的基类，提供`行为`和`事件`的能力；`Component`可包含多个`behavior`，`behavior`可以包含多个`event`。
<!-- more -->
`Event`是一种将自定义代码`注入`到现有代码中的方法，当触发事件时，通过`call_user_func()`回调事件处理函数达到注入的效果；`Behavior`可用于增加现有组件功能而无需修改代码的方式。

## 组件(Component)的构成

### 成员变量

1. **$_events** 存储组件绑定的事件
2. **$_eventWildcards** 存储时间通配符
3. **$_behaviors** 组件绑定的行为

### 方法

1. **__get($name)** 返回组件的属性值, 不仅实现`BaseObject`类中的基本功能, 也会获取`Behavior`中的属性, 达到把行为中的属性注入到组件中的效果
2. **__set($name, $value)** 设置组件的属性. 会依次检查`组件中的setter`、`event`和`behavior中的setter`
3. **__isset($name)** 检验属性是否被设置
4. **__unset($name)** 设置组件的属性为`null`
5. **__call($name, $param)** 调用不存在的方法时，检查行为中是否存在
6. **__clone()** 克隆组件清除所有变量的值
7. **hasProperty()** 是否包含属性，会依次检查`component的getter和setter`、`component的变量`以及`绑定在组件上的行为的属性`
8. **canGetProperty()、canSetProperty()** 检查属性是否有读写能力，检查范围同`hasProperty()`
9. **hasMethod()** 检查组件、行为中是否存在方法
10. **behaviors()** 绑定在组件上的行为列表，数组结构表示如下：
    ```php
    'behaviorName' => [
        'class' => 'BehaviorName',
        'property1' => 'value1',
        'property2' => 'value2'
    ]
    ```
11. **hasEventHandlers()** 组件是否包含事件处理函数
12. **on()** 将事件绑定到组件
13. **off()** 将事件与组件解绑
14. **trigger()** 触发组件事件
15. **getBehavior()** 获取行为对象
16. **getBehaviors()** 获取组件所有行为
17. **attachBehavior()、attachBehaviors()** 绑定行为到组件上
18. **detachBehavior()、detachBehaviors()** 解绑组件上的行为
19. **ensureBehaviors()** 确保`behaviors()`上的所有行为已经绑定到组件上
20. **attachBehaviorInternal()** 绑定一个行为对象到组件的`$_behaviors`属性上

可以将`Component`的方法分为三类：

1. 属性类
2. 事件类
3. 行为类

## 如何实现property特性

`Component`继承与`BaseObject`，因此只需要继承、覆盖魔术方法`__get()、__set()`即可实现`属性`特性。下面是`__get()、__set()`实现代码：

### __get()

1. 检查组件中是否包含`getter`方法
2. 遍历所有行为是否存在属性

```php
public function __get($name)
{
    // step1 检查组件是否办好getName()方法
    $getter = 'get' . $name;
    if (method_exists($this, $getter)) {
        return $this->$getter();
    }

    // step1 行为中是否包含属性
    $this->ensureBehaviors();
    foreach ($this->_behaviors as $behavior) {
        if ($behavior->canGetProperty($name)) {
            return $behavior->$name;
        }
    }

    if (method_exists($this, 'set' . $name)) {
        throw new InvalidCallException('Getting write-only property: ' . get_class($this) . '::' . $name);
    }

    throw new UnknownPropertyException('Getting unknown property: ' . get_class($this) . '::' . $name);
}
```

### __set()

1. 检查组件是否存在set方法
2. 是否以`on `开头的绑定事件
3. `as `绑定行为
4. 检查行为中是否存在set方法

```php
public function __set($name, $value)
{
    $setter = 'set' . $name;
    if (method_exists($this, $setter)) {
        // set property
        $this->$setter($value);

        return;
    } elseif (strncmp($name, 'on ', 3) === 0) {
        // on event: attach event handler
        $this->on(trim(substr($name, 3)), $value);

        return;
    } elseif (strncmp($name, 'as ', 3) === 0) {
        // as behavior: attach behavior
        $name = trim(substr($name, 3));
        $this->attachBehavior($name, $value instanceof Behavior ? $value : Yii::createObject($value));

        return;
    }

    // behavior property
    $this->ensureBehaviors();
    foreach ($this->_behaviors as $behavior) {
        if ($behavior->canSetProperty($name)) {
            $behavior->$name = $value;
            return;
        }
    }

    if (method_exists($this, 'get' . $name)) {
        throw new InvalidCallException('Setting read-only property: ' . get_class($this) . '::' . $name);
    }

    throw new UnknownPropertyException('Setting unknown property: ' . get_class($this) . '::' . $name);
}
```

## 行为的绑定

重载`Component::behavior()`方法即可绑定行为到组件上。

**内部原理**：

首先我们会注意到`ensureBehaviors()`这个方法会在`__get()、__set()`以及`__call()`等魔术方法中调用，保证行为中的属性被注入到`Component`。

通过查看调用栈可以看出函数的调用顺下如下：

```php
1. Component->ensureBehaviors()
2. Component->attachBehaviorInternal()
3. Behavior->attach()
4. Component->on()
5. Component->ensureBehaviors()
```

### ensureBehaviors

```php
public function ensureBehaviors()
{
    if ($this->_behaviors === null) {   // 判断成员变量是否===null，注意是恒等于
        $this->_behaviors = [];         // 这一句很关键，如果没有这一句，会形成一个死循环
        foreach ($this->behaviors() as $name => $behavior) {
            $this->attachBehaviorInternal($name, $behavior);
        }
    }
}
```

上面的代码很简单，通过`foreach`把行为通过`attachBehaviorInternal()`绑定到`$this`组件上。

请一定注意`$this->_behaviors = [];`这一段代码，如果我们忽略这一句的重要性，将会影响我们理解。

当第一次执行`Component->ensureBehaviors()`时，会把`$this->_behaviors = [];`，当第二次执行`Component->ensureBehaviors()`函数时，由于`$this->_behaviors === null`从而直接退出。

### attachBehaviorInternal

```php
private function attachBehaviorInternal($name, $behavior)
{
    if (!($behavior instanceof Behavior)) {
        $behavior = Yii::createObject($behavior);
    }
    if (is_int($name)) {
        $behavior->attach($this);
        $this->_behaviors[] = $behavior;
    } else {
        if (isset($this->_behaviors[$name])) {
            $this->_behaviors[$name]->detach();
        }
        $behavior->attach($this);
        $this->_behaviors[$name] = $behavior;
    }

    return $behavior;
}
```

这是一个私有函数，可以看出绑定行为只能在`Component`中进行。

这个函数主要做了下面几件事：

1. 判断`$behavior`是否为继承`Behavior`类的实例，如果不是则通过`Yii::createObject()`创建；
2. 是否以匿名方式绑定，若是直接绑定；
3. 若是命名行为，判断行为是否存在`Component`中，如果则先解绑改行为，然后重新绑定到`Component`上。

从执行步骤可以看出，如果是命名行为，后声明的会先声明的行为。但是匿名行为不会。

### attach

```php
public function attach($owner)
{
    $this->owner = $owner;
    foreach ($this->events() as $event => $handler) {
        $owner->on($event, is_string($handler) ? [$this, $handler] : $handler);
    }
}
```

`attach`是`Behavior`中的方法。做两件事：

1. 设置行为的`$owner`，将行为依附在`Component`上；
2. 遍历行为中的事件，将事件绑定到`Component`上。

### on

```php
public function on($name, $handler, $data = null, $append = true)
{
    $this->ensureBehaviors();

    if (strpos($name, '*') !== false) {
        if ($append || empty($this->_eventWildcards[$name])) {
            $this->_eventWildcards[$name][] = [$handler, $data];
        } else {
            array_unshift($this->_eventWildcards[$name], [$handler, $data]);
        }
        return;
    }

    if ($append || empty($this->_events[$name])) {
        $this->_events[$name][] = [$handler, $data];
    } else {
        array_unshift($this->_events[$name], [$handler, $data]);
    }
}
```

`on()`函数的作用是把事件依附在`Component`上，如果包含通配符，在放到`$_eventWildcards`成员变量中；如果是正常的命名事件，则放到`$_events`;

## 行为响应事件

通过行为注入，可以在不修改现有类的情况下，更改、扩展类对于事件的响应和支持。如何将行为与`Component`的事件关联起来，就必须通过`Behavior`的`events()`方法。

```php
public function events()
{
    return [
        BaseActiveRecord::EVENT_BEFORE_INSERT => [$this, 'callback_function'],
    ];
}
```

通过覆盖`events()`方法，当`Behavior`与`Component`绑定时，会调用`Behavior::attach()`方法。

## 结语

到此，`Component`的重要特性基本上已经梳理了，重点理解组件是如何绑定行为以及事件是如何依附在组件之上和行为是如何响应事件等原理。
