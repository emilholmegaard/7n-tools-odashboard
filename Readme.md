![Odashboard][logo] Odashboard [![Build Status](https://travis-ci.org/nrkno/odashboard.svg?branch=master)](https://travis-ci.org/nrkno/odashboard)
===

A configurable dashboard with a simple plugin architecture. Written in Node.js with Express, Socket.io and Angular.

## Getting started

### Installing Odashboard as a dependency

Odashboard can be installed from `npm` as a dependency.

**Example usage**
In your folder `my-dashboard/`

```
npm i @nrk/odashboard -S
```

Create configuration files:
```
my-dashboard/
├── config/
│   ├── widgets.js
│   └── dataSources.js
└── package.json
```

> Check out config examples of [`dataSources`](./config/widgets.template.js) and [`widgets`](./config/dataSources.template.js)

Add `npm` script to `package.json` start dashboard
```json
{
  "scripts": {
    "start": "odashboard"
  }
}
```

**Command line arguments**

You can pass arguments to `odashboard` to override default options

 * [`-w`, `--widgetconfig`] - path to config file
 * [`-d`, `--datasourceconfig`] - path to config file
 * [`-p`, `--port`] - port to run on

### Forking and using odashboard

```bash
> npm install

> npm run test

> npm run build

> npm start
```

The application answers on port 3000.

How it may look:
![Example][screenshot]

For some use cases, having access to all code and plugins might be better than installing as a dependency.

To configure your dashboard you need to edit two files:

* In `config/widgets.js` you define your widgets and their properties.
* In `config/dataSources.js` you define the datasources to your widgets.

Most widgets need a compatible datasource to run, but some (like the iframe-plugin) only need you to define a widget.

## Plugins
Below you find current list of available plugins. The generic plugin lets you combine different sources with different plugins. 

| Plugin | Description |
|--------|-------------|
|[Generic](./src/plugins/generic)|Generic plugin for combining data sources and widgets. This might be all you need! |
|[IFrame](./src/plugins/iframe)|Embed an external web resource in an iframe|

### Generic plugin
The generic plugin makes it possible to use a wide range of widgets with a set of different datasources. The availabe widgets are:

| Widget type | Description |
|-------------|-------------|
| [String](./src/plugins/widgets/string) | Show a string |
| [Number](./src/plugins/widgets/number) | Show a number |
| [Timestamp](./src/plugins/widgets/timestamp) | Show a timestamp | 
| [Checkmark](./src/plugins/widgets/checkmark) | Show a checkmark |
| [Build](./src/plugins/widgets/build) | Show build information from build tool | 
| [Queue](./src/plugins/widgets/queue) | Show the numbers of messages on a queue (with color coded warning levels) |
| [Pie chart](./src/plugins/widgets/piechart) | Draws a pie chart |
| [Line chart](./src/plugins/widgets/linechart) | Draws a line chart |

The widgets can display data from the following datasources:

| Datasource type | Description |
|-------------|-------------|
| [JSON Endpoint](./src/plugins/sources/json-endpoint) | Returns JSON formatted data from an url |
| [Application insights](./src/plugins/sources/appinsights)| Info from app insights | 
| [Azure service bus](./src/plugins/sources/azureservicebus) | Returns number of messages on an Azure service bus topic |
| [TeamCity](./src/plugins/sources/teamcity) | Returns TeamCity build information in a odashboard build message compatible with build widget |
| [Google analytics](./src/plugins/sources/google-analytics)| Returns real time data from Google Analytics| 
| [RabbitMQ](./src/plugins/sources/rabbitmq)| Returns number of messaages on a RabbitMq queue| 

Each plugin is documented in [the plugin folder](./src/plugins/)

## How to setup widgets
Widgets are defined in `config/widgets.js`. Each widget is defined as a json-object with the following fields:

| Fields        |Type| Description           | Optional  |
| ------------- |---|:-------------:| -----:|
| plugin      |String| The type of plugin, must correspond to one the available plugins | no |
| datasourceId     |String|The id of the datasource, must correspond to an available datasource|   no |
| url     |String|An url to where you should be taken if you click the widget| yes (default value: #) |
| width |String| The width of the widget (in px, % or em) | yes (default value: 20%) |
| height |String| The height of the widget (in px, % or em)| yes (default value: 100%) |

In addition each plugin may have it's own fields. Default values are defined in `widgetDefaults` in `config/appconfig.js`

This is an example config for a number widget using the Generic-plugin:
```
var myWidget = {
    plugin: "generic"
    datasourceId: "myDataSource1",
    url: "http://moreinfoaboutmywidget.com",
    // Plugin specific fields
    widgetType: "number",
    displayName: "Look ma, a number",
    fieldName: "field",
 }
```

## How to setup datasources
Datasources are defined in `config/dataSources.js`. Each datasource is defined as a json-object with the following fields:

| Fields        |Type| Description           | Optional  |
| ------------- |---|:-------------:| -----:|
| id      |String| The id of the datasource. Must be unique.  | no |
| plugin     |String|The type of plugin, must correspond to one the available plugins|   no |
| updateInterval |Number| How often update the dashboard in ms | no |
| timeout |Number|How long in ms before a request to the datasource should timeout| yes (default value: 1500)|
| auth |Object | What authentication the datasource should utilize in its web requests, see next section for details | yes (default value: undefined) |

A datasource has an id, a plugin, and an updateInterval. In addition each plugin will have it's own specific fields. In this example we use the Simple-plugin, which only has a url. Notice how the id corresponds to the datasourceId in our widget above.

```
{
    id: 'myDataSource1',
    plugin: 'generic',
    updateInterval: 6000,
    timeout: 2500,
    auth: undefined,

    // Plugin specific fields
    source: 'json-endpoint',
    config: {
      url: "http://myhost.com/myapi/test"
    } 
}

```

#### Datasources Authentication

Odashboard supports basic authentication, ntlm and header api keys. To use the authentication you must first define the auth object in the datasource.

For Basic autentication, set the type to `basic` and provide `username` and `password` in the options-object.

Basic auth example:
```
auth: {
  type: "basic",
  options: {
    username: 'user',
    password: 'password'
  }
}

```

For Api-keys, set the type to `apiKey`. In the options object you should provide your key in the `key` field. Because the field to provide API-keys vary from API to API, you must also provide what header field to put it under in the `headerName` field.

Api key example:
```
auth: {
  type: "apiKey",
  options: {
    key: 'mySecretKey',
    headerName: 'X-Api-Key'
  }
}

```

For NTML you must set the type to `ntlm`. In the options object you must provide `username`, `password`, `domain` and `workstation`. The NTLM auth is built using the [httpntlm package](https://www.npmjs.com/package/httpntlm).

NTML example:

```
auth: {
  type: "ntlm",
  options: {
    username: 'user',
    password: 'password',
    domain: 'WORKGROUP',
    workstation: 'MY-COMPUTER'
  }
}

```
## Tabs and rows

To make the widget available you need to add it to a row in the dashboard. A row has an array of widgets. A row is defined like this:

```
var myFirstRow = {
    title: "My widgets",
    widgets: [myWidget, myOtherWidget]
}
```

The rows in turn need to be included in the dashboardConfig-object. The rows can be added in two different ways, depending on if you need multiple tabs (pages) in your dashboard or not.

If you only want one page, you can add the rows directly in the `rows`-field.
```
var dashboardConfig = {
    title: "My Dashboard",
    rows: [myRow, mySecondRow]
};
```

If you have so many widgets that you need multiple pages, define an array of row-arrays in the `tabs`-field:
```
var dashboardConfig = {
    title: "My Dashboard",
    tabs: [
      [myFirstRow, mySecondRow],
      [pageTwoRow, pageTwoSecondRow]
    ]
};
```

If both `tabs` and `rows` are present, `tabs` will be used.

### Tab cycling

In `config/appconfig.js` you'll find 2 settings
 * `tabCycleAutoStart: boolean` - control if cycle is started automatically or not
 * `tabCycleInterval: number` - control the time to spend on each tab (milliseconds)


## Why another dashboard?

Odashboard origins from a hackathon at NRK. We wanted to display build information from TeamCity together with queue information from RabbitMQ on a big screen in our office. Instead of looking to existing open source dashboards like Dashing, we wanted to get some experience with Node.js, and thus Odashboard was born. At first it was a hard coded mess, but over time it evolved to the plugin oriented configurable application it is today. And then we thought other people might enjoy it as much as we do, so we open sourced it.

The strength of Odashboard is that you can get your dashboard up and running in minutes, and all you need to do is edit some config files. Try it out!


[logo]: https://github.com/nrkno/odashboard/blob/master/public/img/odashboard-logo-alt-128x92.png?raw=true "Odashboard"
[screenshot]: https://github.com/nrkno/odashboard/blob/master/public/img/screenshot01.PNG?raw=true "An example dashboard"
