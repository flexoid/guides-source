As you learn about Ember, you'll see code like `Ember.Component.extend()` and
`Model.extend()`. Here, you'll learn about this `extend()` method, as well
as other major features of the Ember object model.

### Defining Classes

To define a new Ember _class_, call the [`extend()`][1] method on
[`Ember.Object`][2]:

[1]: http://emberjs.com/api/classes/Ember.Object.html#method_extend
[2]: http://emberjs.com/api/classes/Ember.Object.html

```javascript
const Person = Ember.Object.extend({
  say(thing) {
    alert(thing);
  }
});
```

This defines a new `Person` class with a `say()` method.

You can also create a _subclass_ from any existing class by calling
its `extend()` method. For example, you might want to create a subclass
of Ember's built-in [`Ember.Component`][3] class:

[3]: http://emberjs.com/api/classes/Ember.Component.html

```javascript {data-filename=app/components/todo-item.js}
export default Ember.Component.extend({
  classNameBindings: ['isUrgent'],
  isUrgent: true
});
```

### Overriding Parent Class Methods

When defining a subclass, you can override methods but still access the
implementation of your parent class by calling the special `_super()`
method:

```javascript
const Person = Ember.Object.extend({
  say(thing) {
    alert(`${this.get('name')} says: ${thing}`);
  }
});

const Soldier = Person.extend({
  say(thing) {
    // this will call the method in the parent class (Person#say), appending
    // the string ', sir!' to the variable `thing` passed in
    this._super(`${thing}, sir!`);
  }
});

let yehuda = Soldier.create({
  name: 'Yehuda Katz'
});

yehuda.say('Yes'); // alerts "Yehuda Katz says: Yes, sir!"
```

In certain cases, you will want to pass arguments to `_super()` before or after overriding.

This allows the original method to continue operating as it normally would.

One common example is when overriding the [`normalizeResponse()`][4] hook in one of Ember-Data's serializers.

A handy shortcut for this is to use a "spread operator", like `...arguments`:

[4]: http://emberjs.com/api/data/classes/DS.JSONAPISerializer.html#method_normalizeResponse

```javascript
normalizeResponse(store, primaryModelClass, payload, id, requestType)  {
  // Customize my JSON payload for Ember-Data
  return this._super(...arguments);
}
```

The above example returns the original arguments (after your customizations) back to the parent class, so it can continue with its normal operations.

### Creating Instances

Once you have defined a class, you can create new _instances_ of that
class by calling its [`create()`][5] method. Any methods, properties and
computed properties you defined on the class will be available to
instances:

[5]: http://emberjs.com/api/classes/Ember.Object.html#method_create

```javascript
const Person = Ember.Object.extend({
  say(thing) {
    alert(`${this.get('name')} says: ${thing}`);
  }
});

let person = Person.create();

person.say('Hello'); // alerts " says: Hello"
```

When creating an instance, you can initialize the values of its properties
by passing an optional hash to the `create()` method:

```javascript
const Person = Ember.Object.extend({
  helloWorld() {
    alert(`Hi, my name is ${this.get('name')}`);
  }
});

let tom = Person.create({
  name: 'Tom Dale'
});

tom.helloWorld(); // alerts "Hi, my name is Tom Dale"
```

Note that for performance reasons, while calling `create()` you cannot redefine an instance's
computed properties and should not redefine existing or define new methods. You should only set simple properties when calling
`create()`. If you need to define or redefine methods or computed
properties, create a new subclass and instantiate that.

By convention, properties or variables that hold classes are
PascalCased, while instances are not. So, for example, the variable
`Person` would point to a class, while `person` would point to an instance
(usually of the `Person` class). You should stick to these naming
conventions in your Ember applications.

### Initializing Instances

When a new instance is created, its [`init()`][6] method is invoked
automatically. This is the ideal place to implement setup required on new
instances:

[6]: http://emberjs.com/api/classes/Ember.Object.html#method_init

```javascript
const Person = Ember.Object.extend({
  init() {
    alert(`${this.get('name')}, reporting for duty!`);
  }
});

Person.create({
  name: 'Stefan Penner'
});

// alerts "Stefan Penner, reporting for duty!"
```

If you are subclassing a framework class, like `Ember.Component`, and you
override the `init()` method, make sure you call `this._super(...arguments)`!
If you don't, a parent class may not have an opportunity to do important
setup work, and you'll see strange behavior in your application.

Arrays and objects defined directly on any `Ember.Object` are shared across all instances of that object.

```javascript
const Person = Ember.Object.extend({
  shoppingList: ['eggs', 'cheese']
});

Person.create({
  name: 'Stefan Penner',
  addItem() {
    this.get('shoppingList').pushObject('bacon');
  }
});

Person.create({
  name: 'Robert Jackson',
  addItem() {
    this.get('shoppingList').pushObject('sausage');
  }
});

// Stefan and Robert both trigger their addItem.
// They both end up with: ['eggs', 'cheese', 'bacon', 'sausage']
```

To avoid this behavior, it is encouraged to initialize those arrays and object properties during `init()`. Doing so ensures each instance will be unique.

```javascript
const Person = Ember.Object.extend({
  init() {
    this.set('shoppingList', ['eggs', 'cheese']);
  }
});

Person.create({
  name: 'Stefan Penner',
  addItem() {
    this.get('shoppingList').pushObject('bacon');
  }
});

Person.create({
  name: 'Robert Jackson',
  addItem() {
    this.get('shoppingList').pushObject('sausage');
  }
});

// Stefan ['eggs', 'cheese', 'bacon']
// Robert ['eggs', 'cheese', 'sausage']
```

### Accessing Object Properties

When accessing the properties of an object, use the [`get()`][7]
and [`set()`][8] accessor methods:

[7]: http://emberjs.com/api/classes/Ember.Object.html#method_get
[8]: http://emberjs.com/api/classes/Ember.Object.html#method_set

```javascript
const Person = Ember.Object.extend({
  name: 'Robert Jackson'
});

let person = Person.create();

person.get('name'); // 'Robert Jackson'
person.set('name', 'Tobias Fünke');
person.get('name'); // 'Tobias Fünke'
```

Make sure to use these accessor methods; otherwise, computed properties won't
recalculate, observers won't fire, and templates won't update.