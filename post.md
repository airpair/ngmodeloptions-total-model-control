Angular 1.3 introduced a new directive called `ng-model-options`. This gives you
more fine-tuned control over how Angular updates and manipulates your model.

*Edit: Play with ng-model-options with [this jsbin](http://jsbin.com/wihuje/edit?html,css,js,output)*

## Usage

It's very simple to use. You simply add the `ng-model-options` directive on the
same element that uses the `ng-model` directive. This is assigned to an object
which holds the configuration for your `ng-model-options`. Like so:

```markup
<input ng-model="vm.user.firstName" ng-model-options="vm.modelOptions" />
```

```javascript
vm.user = {
  firstName: 'Obi Wan'
};
vm.modelOptions = {}; // <-- this is what we'll be looking most closely at
```

*Note: You can also place the directive on the `<form>` element and all
`ng-model`s will receive these options as defaults.*

## Options

There are several options that are available to you. Some of these make more 
sense when used together. [`updateOn` and `debounce`](#updateon-and-debounce),
[`getterSetter` and `allowInvalid`](#gettersetter-and-allow-invalid), and
[`timezone`](#timezone).

### `updateOn` and `debounce`

So, let's say you have a search input field and you want it to query the server
without the user having to click a button ("Search" for example). If you do a
simple `$watch` function, or `ng-change` directive, you will end up sending a
server request for every single time the user types a character (even mistypes
and backspaces). This is not what we want. What we're looking for is a way to
make sure that the user has time to type what they mean to search, and then do
the actual search.

To do this before `ng-model-options` was a bit of a pain, and was not optimal
from a performance standpoint. With `ng-model-options`, this becomes much
simpler. So, taking the example we had above, here's what that would look like:

```javascript
vm.modelOptions = {
  debounce: 300
};
```

This means that Angular will not update the model until the user has stopped
changing the value of the `input` for 300 milliseconds. This is great! However,
what if the user moves out of the input? At that point, we know immediately that
they will not be changing anything else, right? Right. Enter: `updateOn`.

`updateOn` takes a string that represents the DOM events that you want you model
to update on. For example:

```javascript
vm.modelOptions = {
  updateOn: 'blur'
};
```

Now, the model will only be updated when the `input` element fires a `blur`
event. You can even specify multiple events like so:

```javascript
vm.modelOptions = {
  updateOn: 'blur mouseover'
};
```

Now the model value will be update when the `input` element fires a `blur` or a
`mouseover` event. This is obviously odd behavior, but it's instructive.

Also, you can specify that you want the model updated on `default` behavior:

```javascript
vm.modelOptions = {
  updateOn: 'default blur'
};
```

Obviously, `default` is not a DOM event, this is simply part of the
`ng-model-options` api. This configuration is not really useful until we combine
it with `debounce`. Like so:

```javascript
vm.modelOptions = {
  updateOn: 'default blur',
  debounce: { // <-- can be an object as well
    default: 300,
    blur: 0
  }
};
```

And now we've got the model updating after the user has finished typing for
300ms OR once they blur the `input`.

### `getterSetter` and `allowInvalid`

There are sometimes where the model is complex and cannot be easily bound to the
model we're trying to represent. `getterSetter` allows us to specify a function
as our `ng-model` rather than an actual value. Like so:

```javascript
var privateValue = 'Obi Wan';
vm.user = {
  firstName: getSetVal // <-- remember, this is what our input's ng-model is bound to
};

vm.modelOptions = {
  getterSetter: true
};

// Here's the getter/setter function
function getSetVal(newValue) {
  // if there is no newValue specified
  // then it's being invoked as a getter
  // But this  will be a problem later...
  if (angular.isDefined(newValue)) {
    privateValue = newValue;
  }
  return privateValue;
}
```

Now, we are storing the actual value of the model in our own `privateState`
variable. There is one problem with this though. If you're familiar with how
Angular treats the value of `ng-model` attributes with regards to validation,
then you will know that when user input is invalid, the model is set to
`undefined`. This is a problem because for our getter/setter, a value of
`undefined` will result in our `privateValue` not getting updated to the true
value of the model.

There is a way around this, but it is not default behavior so you need to be
careful when using this:

```javascript
vm.modelOptions = {
  getterSetter: true,
  allowInvalid: true
};
```

Specifying `allowInvalid` will cause the `ng-model` model value to be set
regardless of the validity of the field.

# timezone

Finally, `ng-model-options` allows you to specify a `timezone` property. Let's
change our input a little:

```markup
<input type="time" ng-model="vm.user.firstName" ng-model-options="vm.modelOptions" />
```

The only difference here is that we specified the `type` to be `time`. This is
one of the only situations where specifying the `timezone` property really makes
any sense. Now, let's use the timezone property:

```javascript
vm.modelOptions = {
  timezone: 'UTC'
};
```

In 1.3, the only allowed value for `timezone` is `'UTC'`. However, in 1.4, you
can specify any valid timezone. Here's how it works. By specifying UTC as the
timezone for these `modelOptions`, we're telling Angular that we want the
`$viewValue` (what the user sees and interacts with) to be in the `UTC`
timezone. However, the `$modelValue` (what really matters) is still in the
timezone of the browser's locale. This gives you some control over these values
and how the user interacts with your `ng-model` elements.

## In conclusion

`ng-model-options` is a very handy directive that makes what was once very
difficult to do, much easier to do.

## Resources

I have several lessons on [`egghead.io`](http://bit.ly/egghead-ng-model-options)
about these concepts. Also, I gave this as a talk at ng-conf 2015. Feel free to
watch [the video](http://youtu.be/k3t3ov6xHDw) or see [the jsbin](http://jsbin.com/qocekak/edit). And if you're serious about forms with
Angular, then I recommend you use
[angular-formly](http://formly-js.github.io/angular-formly/).

Feel free to each out to me on Twitter:
[@kentcdodds](https://twitter.com/kentcdodds).
