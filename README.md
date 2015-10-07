## Froala-Reactive

Froala-Reactive provides a template-based, reactive wrapper around the [Froala WYSIWYG HTML Editor](https://froala.com/wysiwyg-editor/v2.0/pricing), designed to play nicely with [Meteor Framework](https://www.meteor.com/) client-side templates.

Note that Froala Editor requires a [license for commercial use](https://froala.com/wysiwyg-editor/v2.0/pricing).

#### Installation

You can install Froala-Reactive using Meteor's package management system  The Froala-Reactive package automatically includes the separate froala:editor package which provides the actual Froala Editor javascript library:

```bash
meteor add froala:editor-reactive
```

#### Basic Usage

Froala-Reactive provides a [Template inclusion tag](https://github.com/meteor/meteor/blob/devel/packages/spacebars/README.md#inclusion-tags) which wraps the underlying Froala-Editor jQuery plugin.  In it's simplest form, add it to your template like this:

```html
<template name="myTemplate">
  <div>
    {{> froalaReactive _onbeforeSave=doSave _value=myDoc.myHTMLField}}
  </div>
</template>
```

```javascript
Template.myTemplate.helpers({
  doSave: function () {
    var self = this;
    return function (e, editor) {
      // Get edited HTML from Froala-Editor
      var newHTML = editor.getHTML();
      // Do something to update the edited value provided by the Froala-Editor plugin, if it has changed:
      if (!_.isEqual(newHTML, self.myDoc.myHTMLField)) {
        console.log("onSave HTML is :"+newHTML);
        myCollection.update({_id: self.myDoc._id}, {
          $set: {myHTMLField: newHTML}
        });
      }
      return false; // Stop Froala Editor from POSTing to the Save URL
    }
  }
})
```

Where:

* The "myTemplate" template has a data context that contains a 'myDoc' property, which itself contains '_id' and 'myHTMLField' properties.
* The `onBeforeSave` argument provides a callback function (the `doSave` helper function) to handle the Froala-Editor save event.
* The `_value` argument provides the HTML string that you want to display and edit

Here, we are triggering the update of the underlying 'myDoc' document record in the 'myCollection' collection when the Froala Editor 'beforeSave' event triggers.  We could easily have used the 'blur' or 'contentChanged' events instead.

The final line in the callback stops Froala Editor from generating its own AJAX call to post the updated HTML contents, because we have used the awesomeness of Meteor to do that for us instead.

Note that Froala-Reactive *does not* automatically update the edited `_value`, you
have to provide your own Froala-Editor event handler(s) to do that.

However, Froala-Reactive *will* reactively update the displayed `_value` HTML immediately if you have assigned it to a data context property or template helper function which changes its value any time after the template has rendered (e.g. if the underlying collection document is updated from the server, or another action on the client).

#### Options and Events

You can provide callbacks for any of the Froala-Editor [events](https://froala.com/wysiwyg-editor/v2.0/docs/events) by specifying `_on<event name>` arguments in the `{{> froalaReactive}}` inclusion tag with name of template helper functions that must return a function with the expected Froala-Editor event function signature.

For example, to set up a callback for the [afterUploadPastedImage](https://froala.com/wysiwyg-editor/v2.0/docs/events#afterUploadPastedImage) event:

```html
{{> froalaReactive ...  _onafterUploadPastedImage=imagePasted ...}}
```

```javascript
Template.myTemplate.helpers({
  imagePasted: function () {
    var self = this;
    return function (e, editor, img) {
      // Do something
    };
  }
});
```

Note that the event name used in the `_on<event name>` argument name must be exactly the same as used in the Froala Editor `on('froalaEditor.<event name>', function ....)` callback declaration.  The Froala-Reactive code simply extracts the <event name> string from the inclusion tag argument, and appends it to the `froalaEditor.` string when setting up the underlying Froala-Editor plugin callback

Similarly, you can pass any of the Froala-Editor [options](https://froala.com/wysiwyg-editor/v2.0/docs/options) to the underlying Froala-Editor plugin object, by simply declaring them as arguments to the `froalaReactive` inclusion tag.  Also, if any of these option argument values are set to values on your template's data context, or from return vaues from template helpers, Froala-Reactive will call the Froala Editor `option` setter method to change them if any of them change values once your template has been rendered.

```html
{{> froalaReactive ... language=getLanguage ...}}
```

```javascript
Template.myTemplate.helpers({
  getlanguage: function () {
    return Session.get('language');
  }
})
```

Note that some option values cannot be changed after initialisation (e.g. [inlineMode](https://froala.com/wysiwyg-editor/v2.0/docs/options#inlineMode)) ... please refer to the Meteor-Editor documentation.

#### Methods

You can invoke any of the Froala Editor [methods](https://froala.com/wysiwyg-editor/v2.0/docs/methods) directly on the `editor` object in your Froala Editor event callback functions.  See above for an example of calling `editor.getHTML()`.

#### Gotchas

1. Remember that you must provide one or more `_on` callbacks to handle changing the froalaEditor contents, if you want use the Meteor Framework to do so.
2. If two or more users are actively editing the same underlying state (e.g. the same property of the same document in a collection), and you have set up a contentChanged event handler, or an autosaving Froala Editor, then the content will keep changing.  Their local caret cursor will keep resetting and jumping around.  To avoid this, you may want to implement some kind of locking mechanism, to only one user can initiate an edit session at a time.  To do this properly requires implementing something like Operational Transform!

#### Acknowledgements

This package is based on the implementation of the [x-editable-reactive-template](https://github.com/davidworkman9/x-editable-reactive-template) package.

#### License

This package is released under the MIT License (see LICENSE).

Froala Editor has [3 different licenses](https://froala.com/wysiwyg-editor/v2.0/pricing) for commercial use.
For details please see [License Agreement](https://froala.com/wysiwyg-editor/v2.0/terms).
