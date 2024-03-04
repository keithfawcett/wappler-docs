# Create Component

### Introduction <a href="#introduction" id="introduction"></a>

Components in App Connect are elements that are placed in the html dom and can be used for different tasks. The following tasks are possible with components:

* Acts as a data source
* Render dynamic contents based on its attributes
* Update its content on data changes
* Trigger events that can call actions
* Has methods that can be called to perform an action

The component can be placed in the html using a custom tag or by defining it in the `is` attribute. All App Connect components are prefixed with `dmx-`.

**Custom tag name**

```
<dmx-value id="name" value="Patrick"></dmx-value>
```

**Using the `is` attribute**

```
<table is="dmx-datatable" dmx-bind:data="myDataset"></table>
```

Passing data is done using attributes. Values passed with attributes are static, to bind it to dynamic data use the `dmx-bind:` prefix on the attribute name and pass an expression as value.

Events handlers can also be attached using attributes, they start with `on` like `onclick` or `onchange`. In the attribute value you write a JavaScript expression. To call App Connect actions you use the prefix `dmx-on:` like `dmx-on:click`. You can call actions from other components and use dynamic data that is available in the current scope. The dynamic events also has some extra modifiers, for example `dmx-on:submit.prevent` will prevent the default action. Some other modifiers are `stop`, `self`, `ctrl`, `alt`, `shift` etc.

Example of keydown event which only triggers the action when `enter` key is pressed

```
<input name="search" dmx-on:keydown.enter="search(value)">
```

### Creating a component <a href="#creating-a-component" id="creating-a-component"></a>

A component definition has multiple properties and methods, most of them are optional depending on your needs. Registering a component is done like `dmx.Component(name, definition)`. Where `name` is the name of the component without the `dmx-` prefix and the defination is an object with properties and methods on it which we will explain later.

**The Value Component**

```javascript
dmx.Component('value', {
  initialData: {
    value: null
  },

  attributes: {
    value: {
      default: null
    }
  },

  methods: {
    setValue: function(value) {
      if (this.data.value !== value) {
        this.set('value', value);
        dmx.nextTick(function() {
          this.dispatchEvent('updated');
        }, this);
      }
    }
  },

  events: {
    updated: Event
  },

  render: function() {
    this.set('value', this.props.value);
  },

  update: function(props, fields) {
    if (fields.has('value')) {
      this.set('value', this.props.value);
      dmx.nextTick(function() {
        this.dispatchEvent('updated');
      }, this);
    }
  }
});
```

The `initialData` contains default values of the data from the component, you can also use a function that returns an object. The data set here can be accessed using expressions. Within the component we can access the data using `this.data`, setting/updating the data should be done using `this.set()` method. When the data value was changed it will request an update of all components on the page.

The `attributes` object describes all the attributes that the component has. The key is the attribute name, in our sample that attribute name is `value`. The default value for the attribute is `null`. You can also define a type like String, Number or Boolean. A Boolean type will be true when the attribute is on the element and false when it is not. The values are accessable using `this.props`. So `this.props.value` returns the value that is being set on the component like `value="Patrick"` or with `dmx-bind:value="name"`.

The `methods` are the actions added to the component, they can be called from dynamic events or in flow actions.

In the `events` it lists the available events.

The `render` function is called once when the DOM is being parsed on DOM ready. This is often the place where you initialize the component and render your html.

The `update` function is called when somewhere data/state was updated, the `props` argument returns the old property values while with `this.props` you get the new values, you can then compare them to see what was changed and do any necessary updates. The `fields` argument is a `Map` that contain property names that where changed.

In the value component we also see `this.dispatchEvent('updated');` which triggers the `updated` event after a value was updated. The `dmx.nextTick` was added to have a small delay so other components on the page can update first before the event handler is called.

The components are updated in order how they appear in the DOM. This means that if a component was depending on the value of this component and it is located before this component in the DOM it will not have the updated value and an extra update is required.

### The component API <a href="#the-component-api" id="the-component-api"></a>

Let’s go over some methods that are available on the component and when they are called.

```javascript
{
  // render is one of the first methods being called in your component and should only be called once
  render: function(node) {},
 
  // when data on the page is updated it will call first the beforeUpdate
  // returning false here will prevent the update and also the update of all the children
  beforeUpdate: function(fields) {},
  // the update method is where you do your update logic like updating its own data or html
  update: function(props, fields) {},
  // the updated is called after the component update is ready and also all children are updated
  // this is useful when children have to update first
  updated: function() {},

  // when a component is destroyed the following function will be called
  // the beforeDestroy is called first before anything is removed
  beforeDestroy: function() {},
  // the destroyed is called after the node is removed from DOM and all the
  // children are also destroyed
  destroyed: function() {},
```

Most of the methods do not have some default functionality, the exception is for the `render` method which default will call `this.$parse()` to parse the children. When you overwrite and have children that also needs to be parsed and rendered you’l need to call the `$parse` method.

```javascript
// default render method
render: function(node) {
  if (this.$node) {
    this.$parse();
  }
}
```

> **Internal Properties**\
> `this.$node`: a reference to the DOM Node\
> `this.parent`: the parent component\
> `this.children`: array with child components\
> `this.props`: the properties (values from attributes)\
> `this.data`: the data (exposed for expressions)\
> `this.name`: name of the component

> **Internal Methods**\
> `this.$parse(node)`: parse the DOM, you should call this in the render when you have html inside the component\
> `this.$addChild(name, node)`: add a child component\
> `this.$addBinding(expression, cb)`: add an expression binding, cb is called when value changes. the expression gets evaluated each time update is called\
> `this.$destroy()`: will remove the component from dom and calls destoy on all its children\
> `this.addEventListener(type, handler)`: add an event listerer to the component\
> `this.removeEventListener(type, handler)`: remove an event listener\
> `this.dispatchEvent(event, props, data, nsp)`: trigger an event on the component, will also trigger a native DOM event\
> `this.get(name, ignoreParents)`: get a value from the data, if it isn’t found on the current object it will look in the parent up to the global scope\
> `this.add(name, value)`: add a value to a data, will create an array or add new item to an existing array\
> `this.set(name, value)`: set a value in the data, will overwrite/update existing value\
> `this.del(name)`: remove value from data\
>
