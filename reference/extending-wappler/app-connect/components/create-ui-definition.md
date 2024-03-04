# Create UI Definition

To define a UI for your App Connect components, you need to create a special HJSON file that describes your UI. HJSON is a more readable JSON format that makes it easier to write. It describes the components, dynamic attributes, and events, and also which files to copy and link.

Note: If you created the extension folder as above and the App Connect was selected as option, the file was created for you.

You can place your Hjson file in `/app_connect/components.hjson`\


An example is:

```json
{
  components: [
    {
      type: 'dmx-example-component',
      selector : 'dmx-example-component, [is=dmx-example-component]',
      groupTitle : 'Components',
      groupIcon : 'fa fa-lg fa-cube',
      title : 'Example Component: @@id@@',
      icon : 'fa fa-lg fa-cubes',
      state : 'opened',
      anyParent: true,
      template: '<dmx-example-component id="@@id@@"></dmx-example-component>',
      baseName: 'comp',
      help: 'nice component',
      dataScheme: [
        {name: 'name', type: 'text'}
      ],
      outputType: 'object',
      dataPick: true,
      properties : [
        {
          group: 'Example Component Properties',
          variables: [
            { name: 'compId', attribute: 'id', title: 'ID', type: 'text', defaultValue: '', required: true },
            { name: 'compWidth', attribute: 'width', title: 'Width', type: 'text', defaultValue: '100%'},
            { name: 'compHeight', attribute: 'height', title: 'Height', type: 'text', defaultValue: '400'},
            { name: 'compValue', attribute: 'value', title: 'Initial Value', type: 'text', defaultValue: ''},
          ]
        },
      ],
      actionsScheme: [
        {
          addTitle: 'Set Value',
          title : 'Set Value',
          name : 'changeText',
          icon : 'fa fa-lg fa-play',
          state : 'opened',
          help: 'Set Value',
          properties : [
            {
              group: 'Set Value Properties',
              variables: [
                {
                  name: '1', optionName: '1', title: 'New Value', type: 'text',
                  dataBindings: true, defaultValue: '', required: true,
                  help: 'replace value with new one'
                }
              ]
            }
          ]
        },
      ],
      children: [],
      allowed_children: {},
      copyFiles: [
        {src: 'includes/dmx-example-component.js', dst: 'js/dmx-example-component.js'},
        {src: 'includes/dmx-example-component.css', dst: 'css/dmx-example-component.css'}
      ],
      linkFiles: [
        {src: 'js/dmx-example-component.js', type: 'js', defer: true},
        {src: 'css/dmx-example-component.css', type: 'css'}
      ],
      cssOrder: [],
      jsOrder: []
    }
  ],
  attributes: [
    { name: 'dmx-example-component-value', attributeStartsWith: 'dmx-bind', attribute: 'value', title: 'Value', type: 'boolean',
      display: 'fieldset', icon: 'fa fa-lg fa-chevron-right',
      groupTitle: 'Example Component', groupIcon: 'fa fa-lg fa-cubes',
      defaultValue: false, show: ['valueValue'], noChangeOnHide: true,
      groupEnabler: true, children: [
        { name: 'valueValue', attributeStartsWith: 'dmx-bind', attribute: 'value', isValue: true, dataBindings: true,
          title: 'Value:', type: 'text', help: 'Choose dynamic data binding.',
          defaultValue: '', initDisplay: 'none'
        }
      ], allowedOn: {
        'dmx-example-component' : true
      }
    }
  ],
  events: [
    { name: 'dmx-example-component-updated', attributeStartsWith: 'dmx-on', attribute: 'updated', title: 'updated', type: 'boolean',
      display: 'fieldset', icon: 'fa fa-lg fa-chevron-right',
      groupTitle: 'Example Component', groupIcon: 'fa fa-lg fa-cubes',
      defaultValue: false, show: ['updatedValue', 'updatedMods'], noChangeOnHide: true,
      groupEnabler: true, children: [
        { name: 'updatedValue', attributeStartsWith: 'dmx-on', attribute: 'updated', isValue: true, actionsPicker: true,
          title: 'Action:', type: 'text', help: 'Choose the action to execute.',
          defaultValue: '', initDisplay: 'none' //, required: true
        },
        { name: 'updatedMods', attributeStartsWith: 'dmx-on', attribute: 'updated', isModifiers: true, title: 'Modifiers:', type: 'enum',
          defaultValue: '', initDisplay: 'none', valuesType: 'event_modifiers' //values: EVENT_MODIFIERS
        }
      ], allowedOn: {
        'dmx-example-component' : true
      }
    }
  ],
  static_events: [
    { name: 'dmx-example-component-updated', attribute: 'onupdated', title: 'updated', type: 'boolean',
      display: 'fieldset', icon: 'fa fa-lg fa-play-circle-o',
      groupTitle: 'Example Component', groupIcon: 'fa fa-lg fa-cubes',
      defaultValue: false, show: ['updatedValue'], noChangeOnHide: true,
      groupEnabler: true, children: [
        { name: 'updatedValue', attribute: 'onupdated', isValue: true, behaviorsPicker: true,
          title: 'Action:', type: 'text', help: 'Choose the action to execute.',
          defaultValue: '', initDisplay: 'none' //, required: true
        }
      ], allowedOn: {
        'dmx-example-component' : true
      }
    }
  ]
}
```

The code is divided in four main structures:

* components - a list of all components definitions
* attributes - a list of all dynamic attributes
* events - a list of all dynamic events
* static\_events - a list of all static events

Lets go through then one by one.

### The components section <a href="#the-components-section" id="the-components-section"></a>

| Key                   | Type              | Description                                                                                                                                                                                                                                                                                                                                                                                                                 |
| --------------------- | ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **type**              | text, required    | a unique name for your component. Must start with _dmx-_ for App Connect components. Use your own prefix after that to make it unique                                                                                                                                                                                                                                                                                       |
| **selector**          | text, optional    | a CSS selector to identify your component. If not specified this component definition will be used as template only.                                                                                                                                                                                                                                                                                                        |
| **priority**          | text, optional    | if more component selectors match, sort by priority to display the match with lowest priority number                                                                                                                                                                                                                                                                                                                        |
| **framework**         | text, optional    | if this component is part of a framework, specify the name here. Possible names are `bootstrap4`, `bootstrap5`, `framework7_7`, `app_connect`. when component type starts with dmx- then `app_connect` is the default.                                                                                                                                                                                                      |
| **groupTitle**        | text, required    | a group title to show all components with the same group name together                                                                                                                                                                                                                                                                                                                                                      |
| **groupIcon**         | text, required    | a font awesome icon for the group, use also [color modifiers](https://community.wappler.io/t/wappler-extensibility-app-connect-ui-controls-reference/48503#color-modifiers)                                                                                                                                                                                                                                                 |
| **title**             | text, required    | a title that describes your component                                                                                                                                                                                                                                                                                                                                                                                       |
| **icon**              | text, required    | a font awesome icon for action, use also [color modifiers](https://community.wappler.io/t/wappler-extensibility-app-connect-ui-controls-reference/48503#color-modifiers)                                                                                                                                                                                                                                                    |
| **state**             | text, optional    | can be ‘open’ or ‘closed’ to indicate if rendered in the App Structure tree if the node should be initially closed or open to show its children                                                                                                                                                                                                                                                                             |
| **anyParent**         | boolean, optional | if true it indicates that this component can be placed anywhere. So it is a kind of global component. If not specified the component can only be placed in a parent that has it as child                                                                                                                                                                                                                                    |
| **template**          | text, required    | specify an html template that contains the initial code of the component when inserted. Use special variables as @@id@@ to add component ID                                                                                                                                                                                                                                                                                 |
| **baseName**          | text, optional    | specify initial component ID prefix                                                                                                                                                                                                                                                                                                                                                                                         |
| **dataScheme**        | array, optional   | an array of objects with name and type that define the output data for your action                                                                                                                                                                                                                                                                                                                                          |
| **dataPick**          | boolean, optional | is the main action name pickable as data                                                                                                                                                                                                                                                                                                                                                                                    |
| **properties**        | array, optional   | an array of groups with properties defining the input of the action. You can have multiple groups, see [UI Control Reference 10](https://community.wappler.io/t/wappler-extensibility-app-connect-ui-controls-reference/48503)                                                                                                                                                                                              |
| **actionsScheme**     | array, optional   | an array of actions for this component                                                                                                                                                                                                                                                                                                                                                                                      |
| **children**          | array, optional   | an array of children component names                                                                                                                                                                                                                                                                                                                                                                                        |
| **allowed\_children** | object, optional  | object with keys of allowed children and value how many are allowed or -1 for unlimitted                                                                                                                                                                                                                                                                                                                                    |
| **copyFiles**         | array, optional   | an array of files or folders to be copied from the extension root folder (src) to the web root folder (dst)                                                                                                                                                                                                                                                                                                                 |
| **linkFiles**         | array, optional   | an array of files to be linked when this component is used. Can be also full cdn url. Specify type of file `js`/`css`, and additional attributes like `defer`, `integrity`, `crossorigin`, `module`. Also add `detect='_regexp_'` to enter a full regexp (escaped for string) to detect the different variations of the include and match them as found. Otherwise exact match is needed or the include will be added again |
| **cssOrder**          | array, optional   | an array of css file names to make sure reorder if needed so that the order is enforced                                                                                                                                                                                                                                                                                                                                     |
| **jsOrder**           | array, optional   | an array of js file names to make sure reorder if needed so that the order is enforced                                                                                                                                                                                                                                                                                                                                      |

#### Component Actions Scheme <a href="#component-actions-scheme" id="component-actions-scheme"></a>

When specifying different component actions you can enter the properties passed on each action call.\
Those are passed in exact order given. Example:

```javascript
actionsScheme: [
    {
      addTitle: 'Go To',
      title : 'Go To',
      name : 'goto',
      icon : 'fa fa-lg fa-chevron-right',
      state : 'opened',
      help: 'Navigate to an URL',
      properties : [
        {
          group: 'Go To Properties',
          variables: [
            {
              name: '1', optionName: '1', title: 'URL', type: 'file',
              dataBindings: true, defaultValue: '', routePicker: true, required: true, expressionRequired: true,
              help: 'Enter the url'
            },
            {
              name: '2', optionName: '2', title: 'Internal', type: 'boolean',
              defaultValue: false, help: 'Use internal routing, for partial refresh', show: ['3'], hide: []
            },
            {
              name: '3', optionName: '3', title: 'Title', type: 'text',
              dataBindings: true, defaultValue: '', expressionRequired: true, initDisplay: 'none',
              help: 'Enter the title to be used when just partial refresh is done'
            }
          ]
        }
      ]
    },
....
```

You can however also pass is group parameters in single object parameter with `optionsInObject`, like this:

```javascript
actionsScheme: [
    {
      addTitle: 'General Find Place',
      title : 'General Find Place',
      name : 'findPlaceFromQuery',
      icon : 'fa fa-lg fa-search',
      state : 'opened',
      help: 'General Find Place. The most generic and cheap query<br><span style="color: #d28445; padding-left: 90px; display: block; line-height: 1.6em;">Warning heavy pricing may apply.<br>See <a href="javascript:void(0)" style="color: indianred;" onclick="nw.Shell.openExternal(\'https://developers.google.com/maps/billing/gmp-billing#nearby-search\')"">Google Places Nearby Pricing</a></span>',
      properties : [
        {
          group: 'General Find Place Properties',
          optionsInObject: true,
          variables: [
            {
              name: 'query', optionName: 'query', title: 'Query', type: 'text',
              dataBindings: true, defaultValue: '',
              help: 'Enter the query for the place',
            },
            { name: 'bindBounds', optionName: 'bindBounds', title: 'Bind Bounds', type: 'boolean',
              defaultValue: false, help: 'Search inside current map boundaries only',
              show: [], hide: ['latitude', 'longitude', 'radius']
            },
            { name: 'latitude', optionName: 'latitude', title: 'Latitude', type: 'text',
              defaultValue: '', dataBindings: true
            },
```

### The attributes section <a href="#the-attributes-section" id="the-attributes-section"></a>

The attributes section represents all dynamic attributes. Those are groups of one or more input fields that are toggled as single dynamic attribute. Usually those dynamic groups are identified with `attributeStartsWith` with value `dmx-bind`

You can also build your own generic custom attributes, see:

* [Writing Custom Attributes for App Connect 6](https://community.wappler.io/t/writing-custom-attributes-for-app-connect/48573)

| Key              | Type             | Description                                                                                                                                                                                                                                                                                                                                                                                                                 |
| ---------------- | ---------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **name**         | text, required   | a unique name for your dynamic attribute. Usually also prefixed with component type                                                                                                                                                                                                                                                                                                                                         |
| **groupTitle**   | text, required   | a group title to show all attributes with the same group name together                                                                                                                                                                                                                                                                                                                                                      |
| **groupIcon**    | text, required   | a font awesome icon for the group, use also [color modifiers](https://community.wappler.io/t/wappler-extensibility-app-connect-ui-controls-reference/48503#color-modifiers)                                                                                                                                                                                                                                                 |
| **title**        | text, required   | a title that describes your attribute. Make sure it is unique!                                                                                                                                                                                                                                                                                                                                                              |
| **icon**         | text, required   | a font awesome icon for attribute                                                                                                                                                                                                                                                                                                                                                                                           |
| **display**      | text, required   | must be value `fielset` for toggler group                                                                                                                                                                                                                                                                                                                                                                                   |
| **groupEnabler** | text, required   | must be value `true` for toggler group                                                                                                                                                                                                                                                                                                                                                                                      |
| **type**         | text, required   | must be value `boolean` for toggler group                                                                                                                                                                                                                                                                                                                                                                                   |
| **show**         | text, required   | add the values of all children inputs, so they are shown when the group is on                                                                                                                                                                                                                                                                                                                                               |
| **children**     | array, required  | an array with all the children inputs for this toggle group. Make sure they all have initDisplay: ‘none’ to be initially hidden. Further see the [UI Control Reference 10](https://community.wappler.io/t/wappler-extensibility-app-connect-ui-controls-reference/48503)                                                                                                                                                    |
| **allowedOn**    | object, optional | specifies on which components this attribute can be used. Set component type there.                                                                                                                                                                                                                                                                                                                                         |
| **notAllowedOn** | object, optional | specifies on general elements this attribute does not apply.                                                                                                                                                                                                                                                                                                                                                                |
| **copyFiles**    | array, optional  | an array of files or folders to be copied from the extension root folder (src) to the web root folder (dst)                                                                                                                                                                                                                                                                                                                 |
| **linkFiles**    | array, optional  | an array of files to be linked when this formatter is used. Can be also full cdn url. Specify type of file `js`/`css`, and additional attributes like `defer`, `integrity`, `crossorigin`, `module`. Also add `detect='_regexp_'` to enter a full regexp (escaped for string) to detect the different variations of the include and match them as found. Otherwise exact match is needed or the include will be added again |

### The events section <a href="#the-events-section" id="the-events-section"></a>

The events section represents all dynamic events. Those are groups of one or more input fields that are toggled as a single dynamic event. Usually those dynamic groups are identified with `attributeStartsWith` with value `dmx-on`

| Key              | Type             | Description                                                                                                                                                                                                                                                              |
| ---------------- | ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **name**         | text, required   | a unique name for your dynamic event. Usually also prefixed with component type                                                                                                                                                                                          |
| **groupTitle**   | text, required   | a group title to show all events with the same group name together                                                                                                                                                                                                       |
| **groupIcon**    | text, required   | a font awesome icon for the group, use also [color modifiers](https://community.wappler.io/t/wappler-extensibility-app-connect-ui-controls-reference/48503#color-modifiers)                                                                                              |
| **title**        | text, required   | a title that describes your event. Make sure it is unique!                                                                                                                                                                                                               |
| **icon**         | text, required   | a font awesome icon for event                                                                                                                                                                                                                                            |
| **display**      | text, required   | must be value `fielset` for toggler group                                                                                                                                                                                                                                |
| **groupEnabler** | text, required   | must be value `true` for toggler group                                                                                                                                                                                                                                   |
| **type**         | text, required   | must be value `boolean` for toggler group                                                                                                                                                                                                                                |
| **show**         | text, required   | add the values of all children inputs, so they are shown when the group is on                                                                                                                                                                                            |
| **children**     | array, required  | an array with all the children inputs for this toggle group. Make sure they all have initDisplay: ‘none’ to be initially hidden. Further see the [UI Control Reference 10](https://community.wappler.io/t/wappler-extensibility-app-connect-ui-controls-reference/48503) |
| **allowedOn**    | object, optional | specifies on which components this attribute can be used. Set component type there.                                                                                                                                                                                      |
| **notAllowedOn** | object, optional | specifies on general elements this eventdoes not apply.                                                                                                                                                                                                                  |

### The static events section <a href="#the-static-events-section" id="the-static-events-section"></a>

The static events section represents all static events. Those are groups of one or more input fields that are toggled as single static event. Usually those dynamic groups are identified with single `onxxxx` attribute

| Key              | Type             | Description                                                                                                                                                                                                                                                              |
| ---------------- | ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **name**         | text, required   | a unique name for your static event. Usually also prefixed with component type                                                                                                                                                                                           |
| **groupTitle**   | text, required   | a group title to show all events with the same group name together                                                                                                                                                                                                       |
| **groupIcon**    | text, required   | a font awesome icon for the group, use also [color modifiers](https://community.wappler.io/t/wappler-extensibility-app-connect-ui-controls-reference/48503#color-modifiers)                                                                                              |
| **title**        | text, required   | a title that describes your event. Make sure it is unique!                                                                                                                                                                                                               |
| **icon**         | text, required   | a font awesome icon for event                                                                                                                                                                                                                                            |
| **display**      | text, required   | must be value `fielset` for toggler group                                                                                                                                                                                                                                |
| **groupEnabler** | text, required   | must be value `true` for toggler group                                                                                                                                                                                                                                   |
| **type**         | text, required   | must be value `boolean` for toggler group                                                                                                                                                                                                                                |
| **show**         | text, required   | add the values of all children inputs, so they are shown when the group is on                                                                                                                                                                                            |
| **children**     | array, required  | an array with all the children inputs for this toggle group. Make sure they all have initDisplay: ‘none’ to be initially hidden. Further see the [UI Control Reference 10](https://community.wappler.io/t/wappler-extensibility-app-connect-ui-controls-reference/48503) |
| **allowedOn**    | object, optional | specifies on which components this attribute can be used. Set component type there.                                                                                                                                                                                      |
| **notAllowedOn** | object, optional | specifies on general elements this eventdoes not apply.                                                                                                                                                                                                                  |
