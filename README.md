# Derpibooru Unified Userscript UI Utility
## Introduction
*Derpibooru Unified Userscript UI Utility*, or *Derpi4U* for short, is a simple userscript library for script authors to implement user-changeable settings on [Derpibooru](https://derpibooru.org/).

It abstracts away the hassle of dealing with HTML elements, event listeners, and localStorage when all you want to do is add one or two options to your script for your users to choose from.

Settings for the various userscripts that implements this library will be placed in the [Content Settings](https://derpibooru.org/settings?active_tab=userscript) page of Derpibooru. This provides a single, logical, out-of-the-way location for users to find the settings for their installed scripts.

[![Screenshot](../master/Screenshots/settings-ui-thumbnail.jpeg)](https://raw.githubusercontent.com/marktaiwan/Derpibooru-Unified-Userscript-Ui/master/Screenshots/settings-ui.png)

## Quick Start Guide
First, add Derpi4U to your userscript with
````
// @require  https://raw.githubusercontent.com/marktaiwan/Derpibooru-Unified-Userscript-Ui/master/derpi-four-u.js
````
Now register your userscript with `ConfigManager`.
```` javascript
var config = ConfigManager(
    'Name of Your Script',
    'script_id',
    '[Optional] Description text'
);
````
This returns a `ConfigObject` with which you can manipulate setting entries for your script.

Next, register settings you want to present to your users with calls to `config.registerSetting`:
```` javascript
// register a setting entry
config.registerSetting({
    title: 'Setting title',
    key: 'key_a',
    description: '[Optional] A longer description of what this setting does.',
    type: 'checkbox',
    defaultValue: true
});
````
Each method call registers one setting entry. Each entry are presented on the Content Settings page in the order they are registered in. For a list of available entry types, consult [this table](#entry_type).

To retrieve the entry, simply call `config.getEntry` with the corresponding key:
```` javascript
// retrieve a stored setting entry
var a = config.getEntry('key_a');
````
### Notes
#### @match
If your script only runs on specific pages of Derpibooru, remember to add the following rules to your metadata block:
````
// @match https://derpibooru.org/settings
// @match https://trixiebooru.org/settings
````
#### For real-life usage examples, consult the following scripts:
[Derpibooru WebM Volume Toggle](https://derpibooru.org/forums/meta/topics/userscript-derpibooru-webm-volume-toggle-106)
## `ConfigObject`
A `ConfigObject` is returned by calls to `ConfigManager` or `ConfigObject.addFieldset`. It is used to add/read/change/remove user setting entries managed by Derpi4U.
### `ConfigObject.pageElement`
On the Settings page, this property corresponds to either the container element used to encapsulate all the input fields and display elements of your script when returned by `ConfigManager`, or the `<fieldset>` element when returned by `ConfigObject.addFieldset`.

When not on the Content Settings page, the property is set to `null`.
### `ConfigObject.registerSetting(entryConfig)`
Registers a setting entry to be used in your script. If on the Content Settings page, displays the input fields in your script container.

Returns the HTML element of the entry container, regardless if it's inserted or not.
#### Parameters
*entryConfig* is an object that contains:
 - `title`:  A `String` that is displayed next to the input element.
 - `key`:  A `String` used to identify the setting entry, and must be unique within your userscript.
   
   The key should follow the JavaScript naming restrictions. It must begin with an ASCII letter (upper or lower case) or an underscore (`_`) character.
    Subsequent characters must be letters, numbers, hyphens (`-`) or underscores (`_`).
 - `description`:  [Optional] A `String` that is displayed below the entry title and input element pair
 - `defaultValue`: The default value associated with the setting entry.
 - <a name="entry_type" href="#entry_type">`type`</a>: A `String` of either `checkbox`, `text`, `number`, `radio`, or `dropdown`.
   
   The value of `type` determines which input element to use on the Content Settings panel, and the data type used to store it:
   
   | Type       | Stored Data Type | HTML Element                     |
   | ---------- | ---------------- | -------------------------------- |
   | `checkbox` | Boolean          | `<input>` with `type="checkbox"` |
   | `text`     | String           | `<input>` with `type="text"`     |
   | `number`   | Number           | `<input>` with `type="number"`   |
   | `radio`    | String           | `<input>` with `type="radio"`    |
   | `dropdown` | String           | `<select>`                       |
   
 - `selections`: Required when the `type` property is of `radio` or `dropdown`.
   
   It should contain an `Array` of `{text:String, value:String}` objects used to define user selectable options in the dropdown list or radio button group.
   ```` javascript
   config.registerSetting({
       title: 'Pick a color',
       key: 'color',
       description: 'Pick a color from the dropdown list.',
       type: 'dropdown',  	// radio/dropdown
       defaultValue: 'r',
       selections: [
           {text: 'Red',   value: 'r'},
           {text: 'Green', value: 'g'},
           {text: 'Blue',  value: 'b'}
       ]
   });
   ````
   The `value` property of the selected item will be stored under the entry key.

#### Return value
`registerSetting()` returns a `<div>` element that is to be inserted when on the settings page.
````HTML
<div>
    <label>Entry Title</label>
    <input type="number" data-entry-key="key1">
    <div>
        <i>Descriptions here.</i>
    </div>
</div>
````
This allows you to apply additional styling and restriction to the display element:
````javascript
var input = config.registerSetting(/*....*/).querySelector('input');
input.setAttribute('min', '0');
input.setAttribute('max', '100');
input.setAttribute('step', '5');
````
### `ConfigObject.setEntry(key, value)`
`setEntry` allows you to manually store data into settings storage.
#### Parameter
*key* is a `String` identifying the setting entry.
#### Return value
`undefined`
### `ConfigObject.getEntry(key)`
Retrieves 
#### Parameter
*key* is a `String` identifying the setting entry.
#### Return value
The data stored in the setting entry.
### `ConfigObject.deleteEntry(key)`
Removes the key/value pair from storage.
#### Return value
`undefined`
### `ConfigObject.addFieldset(title, id, description)`
Appends a `<fieldset>` element to the the script container and returns a `ConfigObject`. Calling the new `ConfigObject.registerSetting` will add the new input elements inside `<fieldset>`.
```` javascript
var field = config.addFieldset(
    'Fieldset',
    'fieldset1',
    'Description'
);
field.registerSetting({
    title: 'Setting Title',
    key: 'key1',
    description: 'This setting entry is displayed inside the <fieldset> element.',
    type: 'checkbox',
    defaultValue: true
});
````
The `<fieldset>` element allows you to visually group related user settings together, but otherwise does not affect how the entries are store or accessed in your script. Thus calls to `field.getEntry` could be used to retrieve entries from outside the `<fieldset>`, and vice versa.

You may nest multiple `<fieldset>` elements by calling `addFieldset` on the new `ConfigObject`.
```` javascript
var subfield = field.addFieldset(
    'Subfield',
    'subfieldset',
    'This <fieldset> element is nested inside another <fieldset> element.'
);
````
