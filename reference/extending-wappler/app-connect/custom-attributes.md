# Custom Attributes

### Introduction

Custom attributes are used to use expressions to interact with the DOM element. The following attributes are included in the core:

`dmx-bind`: bind a dynamic value\
`dmx-class`: toggle class based on expression\
`dmx-hide`: hide element based on expression\
`dmx-html`: set value as inner html\
`dmx-on`: dynamic events handler\
`dmx-repeat`: repeat the element\
`dmx-show`: show element based on expression\
`dmx-style`: dynamic style properties\
`dmx-text`: set value as inner text

The syntax for registering custom attributes is `dmx.Attribute(name, hook, function)`. The function you register gets as arguments the node (DOM element) and an attr object which we will explain in more detail later. The custom attribute will be prefixed with `dmx-` and can be applied to any DOM element within your app.

The custom attribute can be applied to all DOM elements on the page including components. They normally do just a single task like setting the content, toggle class or visibility of a node. The custom attribute function is only called once and within your function you then add your logic for updating, like watching for changes in an expression or listening for some DOM events.

### The class toggle attribute

This is the original code used in App Connect for the `dmx-class` attribute. The attribute allows you to toggle a class based on an expression, when the expression value is [truthy 1](https://developer.mozilla.org/en-US/docs/Glossary/Truthy) the class is added otherwise it is removed.

```javascript
dmx.Attribute('class', 'mounted', function(node, attr) {
  var className = attr.argument;
  this.$addBinding(attr.value, function(value, oldValue) {
    if (value != oldValue) {
      node.classList[value ? 'add' : 'remove'](className);
    }
  });
});
```

The code can be simplified for modern browsers, the above code was used to also support IE10.

```javascript
dmx.Attribute('class', 'mounted', function(node, attr) {
  this.$addBinding(attr.value, value => node.classList.toggle(attr.argument, !!value);
});
```

**usage**

```
<nav class="nav nav-pills">
  <a class="nav-link" href="/" dmx-class:active="browser1.location.pathname == '/'">Home</a>
  <a class="nav-link" href="/about" dmx-class:active="browser1.location.pathname == '/about'">About</a>
</nav>
```

Lets take a look at what the code does. We register an attribute with the name `class`, in the html we have to use it with the prefix `dmx-class`.

We execute the attribute function on the `mounted` hook. There are only 2 hooks: `before` and `mounted`. The `before` hook is when the parser comes to the DOM node and didn’t check the node yet, the only place it is used is in the the `repeat` attribute since it modifies the DOM node by duplicating it. The `mounted` hook is after the node has been parsed, when the node is also a component it will already have called the render function from it.

The attribute function has 2 arguments, the first is the DOM node and the second is an object that has properties about the parsed attribute. The custom attribute is not just a name/value as a normal attribute, it can have extra arguments and modifiers applied. The attr object has the following properties: `name`, `fullName`, `value`, `argument` and `modifiers`.

Best is to explain using an example, we have the followin attribute: `dmx-on:click.prevent.button:1.stop="log('Button 1 clicked')"`. This will result in the following attr object:

```javascript
{
  name: "on",
  fullName: "dmx-on:click.prevent.button:1.stop",
  value: "log('Button 1 clicked')",
  argument: "click",
  modifiers: {
    button: "1",
    prevent: true,
    stop: true
  }
}
```

The `this` keyword inside the attribute references the closest component. The method `this.$addBinding` registers an expression and the callback will be called when the value changes. In this callback we then update the node depending on the result.

### Examples

Here some examples of what is possible with custom attributes.

#### Intl attribute

The advantage of this attribute over a custom formatter is that it doesn’t execute on every update. The attribute will only update when the actual value was changed. This example will also make use of the attribute argument and modifiers.

```javascript
dmx.Attribute('intl', 'mounted', function(node, attr) {
  // convert kebab case to camel case
  const cc = (val) => {
    return String(val).replace(/-./g, m => m[1].toUpperCase());
  };

  // convert modifiers to options
  const options = (modifiers) => {
    const opts = {};

    for (let modifier in modifiers) {
      opts[cc(modifier)] = cc(modifiers[modifier])
    }

    return opts;
  };

  // the attribute value is an expression
  // with $addBinding we make sure the expression is evaluated with
  // each update request and the second argument is a callback
  // function that is called when the value was changed
  this.$addBinding(attr.value, value => {
    let locales = attr.argument || document.documentElement.lang || 'en';
    let type = typeof value;
    let text = value;

    switch (type) {
      case 'string':
        let date = dmx.parseDate(value);
        if (date.toString() != 'Invalid Date') {
          text = new Intl.DateTimeFormat(locales, options(attr.modifiers)).format(date);
        }
        break;

      case 'number':
        text = new Intl.NumberFormat(locales, options(attr.modifiers)).format(value);
        break;

      case 'object':
        if (Array.isArray(value)) {
          text = new Intl.ListFormat(locales, options(attr.modifiers)).format(value.map(String));
        }
        break;
    }

    node.innerText = text;
  });
});
```

**Usage**

```
<p>Price: <span dmx-intl:nl.style:currency.currency:eur="1.99"></span></p>
<p>Size: <span dmx-intl:nl.style:unit.unit:byte="1523662349"></span></p>
<p>Date: <span dmx-intl:nl.date-style:full="'2003-02-01'"></span></p>
<p>Time: <span dmx-intl:nl.time-style:full="'15:00:00'"></span></p>
<p>DateTime: <span dmx-intl:nl.date-style:full.time-style:full="'2003-02-01T15:00:00Z'"></span></p>
<p>List: <span dmx-intl:nl="[1,2,3,4]"></span></p>
```

results in:

```
Price: € 1,99
Size: 1.523.662.349 byte
Date: zaterdag 1 februari 2003
Time: 15:00:00 Midden-Europese standaardtijd
DateTime: zaterdag 1 februari 2003 om 16:00:00 Midden-Europese standaardtijd
List: 1, 2, 3 en 4
```

#### Var attribute

The var attribute is similar to the value component, but instead of a component which sets its data it sets the data on the closest component in the DOM or alternative as a global variable.

```javascript
dmx.Attribute('var', 'mounted', function(node, attr) {
  this.$addBinding(attr.value, value => {
    if (attr.modifiers.global) {
      dmx.global.set(attr.argument, value);
    } else {
      // this references the closest component
      this.set(attr.argument, value);
    }
  });
});
```

**usage**

```
<p>
  <!-- var is set on input component -->
  <input id="input1" type="number" value="2" dmx-var:var1="value * value">
  {{input1.var1}}
</p>

<p>
  <!-- var is set in global scope using the global modifier -->
  <input id="input2" type="number" value="2" dmx-var:var2.global="value * value">
  {{var2}}
</p>

<p>
  <!-- var is set on closes component in DOM since span is not a component -->
  <span dmx-var:var3="input1.value * input2.value"></span>
  {{var3}}
</p>
```

#### Inview attribute

This example doesn’t use an expression as value, the attribute lets you set a class on the element when it is in the viewport.

```javascript
(function() {
  const observer = new IntersectionObserver((entries) => {
    entries.forEach(entry => {
      const inview = entry.intersectionRatio > 0;

      if (inview !== entry.target.inview) {
        // store inview property for comparing
        entry.target.inview = inview;
        // toggle our inview class
        entry.target.classList.toggle(entry.target.inviewClass, inview);
      }
    })
  });

  dmx.Attribute('inview', 'mounted', function(node, attr) {
    // we add the value of the attribute as a class when node is inview
    node.inviewClass = attr.value;
    // observer the node
    observer.observe(node);
  });
})();
```

**usage**

```
<div dmx-inview="visible">
  ...
</div>
```

### App Connect 2 Beta

#### New way to watch expressions

The `$addBinding` has been renamed to `$watch`, the $addBinding will still keep working as an alias for `$watch`. With App Connect 2 the expressions are not evaluated on each change, with the new signals they subscribe directly to the data they are using and they will update directly when the data changes.

An important change is that the callback doesn’t have the oldValue anymore, if needed you should keep track of it yourself.

```
this.$watch(expression, value => {
  // your code here
});
```

\
