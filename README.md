# Plotly

This library wraps the [Plotly](https://plot.ly/) [REST API] (https://plot.ly/rest/), allowing you to graph and style data obtained from Imp-connected sensors.

This class allows for simple creation of time-series data graphs while exposing access for styling graphs using all features of the Plotly API.  Note that this library requires creation of a Plotly account.

### Callbacks
Almost all methods in this class (including the constructor) take an optional *callback* argument.
This is a function that takes arguments *response* and *plot*, where *response* is a table representing a response from the Plotly servers and *plot* is a reference to the plot object.  The *response* object mirrors that provided in the callback to [httprequest.sendasync()](https://electricimp.com/docs/api/httprequest/sendasync/) with the addition of a *decoded* field that contains the JSON body of the response in a Squirrel table.

**Note that while the constructor will return immediately, it is only safe to operate on the resulting object once the callback has been called.**  The *plot* argument to the callback is provided for this purpose.  It is also the user's responsibility to ensure at this step that the construction has succeeded by checking the HTTP response code and/or Plotly response messages.

## Constructor: Plotly(*userName, userKey, FileName, worldReadable, traces [, callback]*)

To create a plot, you need to call the constructor with your Plotly authentication information and some basic data about the new graph.

To find your *userName* and *userKey*, go to the Plotly settings and copy the Username and API key as highlighted below.  Note that the *userKey* is **not** your password, but is an API key that Plotly provides for developers.  Whenever you cycle your API key (e.g. by clicking "Generate a new key"), you will have to update this value in your code as well.

![Plotly settings screenshot] (images/plotly_user_settings.png)

Let *fileName* be the file name you would like this graph to have in your Plotly account.

Let *worldReadable* be true if you would like this graph to be accessible to anyone with a link to the plot.  If this is false, the graph will only be accessible to your account by viewing the plots you own.

Let *traces* be a list of the data point names you would like to graph.  Each Plotly graph can display many concurrent values known as traces, but you must list them all here before plotting them.

```squirrel
local callback = function(response, plot){
    server.log(response.body);
    server.log("See plot at " + plot.getUrl());
}
myPlot <- Plot("<YOUR_USERNAME>", "<YOUR_API_KEY>", "weather_data", true, ["temperature", "inside_humidity", "outside_humidity"], callback);
```

## Class Methods

## plot.getUrl()

Returns a string with the URL of the graph that this object generates.  Note that if *worldReadable* was set to false in the constructor, this link will only be viewable when logged into Plotly.

```squirrel
local plotUrl = myPlot.getUrl();
```

## plot.setTitle(*title [, callback]*)

Sets the title that will be displayed on this graph.

```squirrel
myPlot.setTitle("Weather at Station 7");
```

## plot.setAxisTitles(*xAxisTitle, yAxisTitle*)

Sets the labels that will be applied to the standard x- and y-axes on this graph.  If either argument is null or empty, that axis title will not be changed.

```squirrel
myPlot.setAxisTitles("Time", "Temperature (°F)");
```

## plot.addSecondYAxis(*axisTitle, traces [, callback]*)

Adds a second y-axis on the right side of the graph and assigns the specified traces to it.  *traces* should be a list of the string names of traces as passed in the *traces* argument to the constructor.

```squirrel
myPlot.addSecondYAxis("Humidity (%)", ["inside_humidity", "outside_humidity"]);
```

## plot.setStyleDirectly(*styleTable [, callback]*)

Sets the style of the graph by passing a description directly to the Plotly API.  This allows for advanced styling options that this library does not have specific methods for.

*styleTable* should be a Squirrel list or table that will be parsed into JSON.  See the [Plotly API docs] (https://plot.ly/rest/) for details on how to format this argument.

Note that there are several caveats to using this method:

- This will entirely overwrite style parameters previously set using methods like `AddSecondAxis` or `setStyleDirectly`.
- If there is an error in formatting *styleTable*, an error may be printed to the console or the call may silently fail.

```squirrel
local style =
[
    {
        "name" : "temperature",
        "type": "scatter",
        "marker": {"symbol": "square", "color": "purple"}
    },
    {
        "name" : "inside_humidity",
        "type": "scatter",
        "marker": {"symbol": "circle", "color": "red"}
    }
];
myPlot.setStyleDirectly(style);
```

## Plotly.setLayoutDirectly(*layoutTable [,callback]*)

See documentation for `setStyleDirectly`.

## Plotly.plot(*dataObjs [, callback]*)

Appends data to the Plotly graph.  This method takes an array *dataObjs* of Squirrel tables in the following form:

```squirrel
{
    "name" : <TRACE_NAME>,
    "x" : [<X_VALUE_1, X_VALUE_2, ...>],
    "y" : [<Y_VALUE_1, Y_VALUE_2, ...>]
    "z" : [<OPTIONAL_Z_VALUE_1>, <OPTIONAL_Z_VALUE_2>, ...]
}
```

Note that the "x", "y", and "z" fields hold arrays of integers or strings and the "z" field is optional.

Each element in *dataObjs* must have a name field that corresponds to a trace name as passed into the constructor.  To add multiple data points to a trace, either add them to the traces data arrays or make multiple calls to this method.

```squirrel
myPlot.post([
    {
        "name" : "temperature",
        "x" : [timestamp],
        "y" : [latest_temperature]
    },
    {
        "name" : "inside_humidity",
        "x" : [timestamp],
        "y" : [latest_humidity]
    }
]);
```

## Static Methods

## Plotly.getPlotlyTimestamp()

Returns a timestamp string that Plotly will automatically recognize and style correctly.  Use this for your x-value on time-series data.

```squirrel
local timestamp = myPlot.getPlotlyTimestamp();
myPlot.post(
{
    "name" : "temperature",
    "x" : [timestamp],
    "y" : [latest_temperature]
});
```

## License

The Plotly library is licensed under the [MIT License](./LICENSE).

