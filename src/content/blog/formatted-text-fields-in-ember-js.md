---
title: "Formatted Text Fields in Ember.js"
description: "A common need in most applications is custom text fields which display a formatted value. These can be date pickers, formatted numbers or one of many other uses. Often times…"
pubDate: "2013-11-29T00:00:00"
---

A common need in most applications is custom text fields which display a formatted value. These can be date pickers, formatted numbers or one of many other uses. Often times there are existing libraries we can leverage to do the work for us, but one problem we still have is getting the unformated back into our application. We can work around this by following a very reusable pattern in custom views.

### Binding directly to the value

With Em.TextField, we usually just bind a property our controller to the value property of the view, such as:

    {{view Em.TextField valueBinding="currency"}}

This works great because every time our value changes, our currency property gets updated. However, let's say we want to display money with a currency symbol, but we want our value to save as the raw number without any currency symbol. We'll extend the text field and do this exactly as we did before:

    {{view Em.CurrencyField valueBinding="currency"}}

Before going into the view code, we'll just assume that it formats the value. The problem is when we change the value on the view, our currency property will have that formatted value.

### Passing in a reference instead of binding

While we could add some logic into our controller to handle this, we could end up having to rewrite the same code in multiple spots if we plan on reusing this field throughout our application. A better way to do this is allow the view to handle the formatting and unformatting of the currency value. To do this, we will pass in a reference to our currency value to the view, but we won't bind it to the value property.

    {{view Em.CurrencyField currencyBinding="currency"}}

Here we are binding the currency property on our controller to the currency property on the view instead of value. To wire all this up, here is our custom view:

    App.CurrencyField= Em.TextField.extend({
      init: function() {
        this._super();
        var value = accounting.formatMoney(this.get("currency"));
        this.set("value", value);
      },

      updateCurrency: function() {
        var currency = accounting.unformat(this.get("value"));
        this.set("currency", currency);
      }.observes("value")
    });

Let's walk through what is going on here. First, we are extending Em.TextField to get all of the existing logic there.  
Next, we override the init method. In here we are formatting the currency passed in and setting the value, which will be displayed. The reason we are doing this in the init instead of didInsertElement is because we don't want to trigger a change on the value property. If you are wiring up a field that uses a jQuery plugin, you likely could avoid this and do that setup in the didInsertElement method.
In order to respond to changes of the input and get the unformatted value back to the currency property on our controller, we add the updateCurrency method which observes the value property of our view. In here we unformat the currency and set it to the currency property on our view, which is bound to the currency property on our controller. Now we are in full control of what format our currency is stored in within our application and not dependent on a 3rd party library's format.

### One more thing

Hopefully this helps you see how easy it is to create custom text fields in which we have full control over the format in which our bound value is output in. One more trick I have learned is to create a Handlebars helper for our view. We can do this with one line:

    Em.Handlebars.helper("currency-field", App.CurrencyField);

The benefit of this is that we are able to make our template more readable by replacing what we had above with the following:

    {{currency-field currency=currency}}

This helps reduce the noise in our template.

You can see a full example of this code [here](http://emberjs.jsbin.com/uYiRale/8/edit).
