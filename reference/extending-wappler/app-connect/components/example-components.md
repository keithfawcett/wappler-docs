# Example Components

### Weather Datasource Component

This sample component will use the weather api to get the current weather at a specific location. You probably don’t want to expose the api key in the html normally, but in this case it is for explaining the workings. If you want to use it in production you probably want to do the api calls in a server action and call the server action from the component. We will here call the api directly for simplicity.



```javascript
dmx.Component('weather-api', {
  initialData: {
    location: {},
    current: {}
  },

  attributes: {
    // api key from weatherapi.com
    key: {
      type: String,
      default: ''
    },

    q: {
      type: String,
      default: 'auto:ip'
    }
  },

  methods: {
    refresh: function() {
      this.refresh();
    }
  },

  events: {
    updated: Event
  },

  render: function(node) {
    // we call the refresh method on our initial render to get the weather data
    this.refresh();
    // we do not call $parse here, this prevents any children from being rendered
    // this component should not have any children since it is purely a datasource
  },

  update: function(props) {
    // if location was changed we need to refresh the data
    if (props.q !== this.props.q) {
      this.refresh();
    }
  },

  refresh: function() {
    // our refresh function will call the weather api and then update the component data
    fetch(`http://api.weatherapi.com/v1/current.json?key=${this.props.key}&q=${this.props.q}`)
      // get the json response
      .then(response => response.json())
      // set our own data with the response from the api
      .then(data => this.set(data))
      // dispatch an updated event
      .then(() => this.dispatchEvent('updated'));
  }
});
```

**usage**

```
<dmx-weather-api id="weather" key="apikey"></dmx-weather-api>
The weather in {{weather.location.name}} is {{weather.current.temp_c}}&#8451; and it is {{weather.current.condition.text}}.
```

### Weather Widget Component

For the weather widget we are going to extend the Weather Datasource Component and render some html that will visually show the current weather.

```javascript
dmx.Component('weather-widget', {
  extends: 'weather-api',

  render: function(node) {
    // we going to listen to our own updated event
    // when data was updated we will render the widget
    this.addEventListener('updated', () => this.renderWidget());
    // we call the render from the weather api component first
    dmx.Component('weather-api').prototype.render.call(this, node);
  },

  renderWidget: function() {
    const { current, location } = this.data;
    // we keep the render simple by just setting the innerHTML
    // using a template string
    this.$node.innerHTML = `
      <div style="background: #036; background-image: linear-gradient(180deg, black, transparent); color: #fff; border-radius: 1rem; font-family: sans-serif; padding: 1rem;">
      <div style="display: flex; flex-direction: column; gap: 1rem;">
          <div style="font-size: 1.5rem; text-align: center">${location.name}, ${location.country}</div>
          <div style="display: flex; gap: 1rem; justify-content: space-around">
            <div style="display: flex; flex-direction: column; align-items: center;">
              <img style="height: 100%" src="${current.condition.icon}" alt="${current.condition.text}">
              <div style="font-size: .825rem">${current.condition.text}</div>
            </div>
            <div style="text-align: center;">
              <div style="font-size: 5rem">${current.temp_c}&#8451;</div>
              <div style="font-size: .825rem">Feels like ${current.feelslike_c}&#8451;</div>
            </div>
            <div style="font-size: .825rem">
              Wind: ${current.wind_kph} kmph ${current.wind_dir}<br>
              Precip: ${current.precip_mm} mm<br>
              Presure: ${current.pressure_mb} mb<br>
              Humidity: ${current.humidity}%<br>
              UV Index: ${current.uv}
            </div>
          </div>
          <div style="font-size: .825rem; text-align: center">
            Data last updated from weatherapi.com at ${current.last_updated}
          </div>
        </div>
      </div>
    `;
  }
});
```

**usage**

```
<dmx-weather-widget id="weather" key="apikey"></dmx-weather-widget>
```

### Datatable Component

Simple component that renders a table based on a data source

```javascript
dmx.Component('datatable', {
  attributes: {
    data: {
      type: Array,
      default: []
    }
  },

  render: function(node) {
    // render the table
    this.renderTable();
  },

  update: function(props) {
    // dmx.equal is a helper function the does a deep compare
    // which is useful when comparing arrays and objects
    if (!dmx.equal(this.props.data, props.data)) {
      this.renderTable();
    }
  },

  renderTable: function() {
    // Data is null/undefined, probably loading the data
    if (!Array.isArray(this.props.data)) {
      this.$node.innerHTML = `
        <tr>
          <td>Loading...</td>
        </tr>
      `;
      return;
    }

    // Empty results or no data set
    if (!this.props.data.length) {
      this.$node.innerHTML = `
        <tr>
          <td>There are no results...</td>
        </tr>
      `;
      return;
    }

    this.$node.innerHTML = `
      <thead>
        <tr>
          ${Object.keys(this.props.data[0]).map(key => `<th>${key}</th>`).join('')}
        </tr>
      </thead>
      <tbody>
        ${this.props.data.map(row => `
          <tr>
          ${Object.values(row).map(value => `<td>${value}</td>`).join('')}
          </tr>
        `).join('')}
      </tbody>
    `;
  }
});
```

**usage**

```
<table class="table" is="dmx-datatable" dmx-bind:data="myDataset"></table>
```

### Resize Observer Component

The resize oberser component keeps track of changes to the dimensions of the element, the width and height are available as data properties and can be used in expressions. When the dimensions of the element changes it will also trigger the resize event which can be listened to.

```javascript
dmx.Component('resize-observer', {
  initialData: function() {
    const rect = this.$node.getBoundingClientRect();

    return {
      width: rect.width,
      height: rect.height
    };
  },

  events: {
    resize: Event
  },

  render: function(node) {
    this.observer = new ResizeObserver(entries => {
      this.set('width', entries[0].contentRect.width);
      this.set('height', entries[0].contentRect.height);
      this.dispatchEvent('resize');
    }).observe(node);
  },

  beforeDestroy: function() {
    this.observer.unobserver(this.$node);
    delete this.observer;
  }
});
```

#### Usage

```
<div id="resizable" is="dmx-resize-observer">
  {{resizable.width}}x{{resizable.height}}
</div>
```

### Shadow DOM component

For App Connect 1:

```
dmx.Component('shadow', {
  attributes: {
    content: {
      type: 'string',
      default: ''
    }
  },

  render (node) {
    this.shadowRoot = node.attachShadow({mode: 'open'});
    this.shadowRoot.innerHTML = this.props.content;
  },

  update (props) {
    if (props.content !== this.props.content) {
      this.shadowRoot.innerHTML = this.props.content;
    }
  }
});
```

For App Connect 2 Beta:

```
dmx.Component('shadow', {
  attributes: {
    content: {
      type: 'string',
      default: ''
    }
  },

  init () {
    this._shadowRoot = this.$node.attachShadow({mode: 'open'});
  },

  render () {
    this._shadowRoot.innerHTML = this.props.content;
  },

  performUpdate (updatedProps) {
    if (updatedProps.has('content')) {
      this.render();
    }
  }
});
```

#### Usage

```
<dmx-if dmx-bind:condition="payload.parts.where('mimeType', 'text/html', '==')">
  <dmx-shadow dmx-bind:content="payload.parts.where('mimeType', 'text/html' , '==' )[0].body.data.decodeBase64()"></dmx-shadow>
</dmx-if>
```
