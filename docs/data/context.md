# Context Data in Expressions

The nomral way of passing data to Handlebars templates via the template function. This data context is accessible from the root template context using the Handlebars expression.
```js
var rootContext = {foo: 'bar'};
var template = Handlebars.compile('{{foo}}');
template(rootContext); // => 'bar
```

When using the each helper we change the template context to that of what we are looping.

```js
var rootContext = {stooges: ['Larry', 'Curly', 'Moe']};
var template = Handlebars.compile('{{#each stooges}}{{this}},{{/each}}');
template(rootContext); // => Larry,Curly,Moe,
```

# Context Data in Parials

When using partials the data context remains the root by default.
```js
Handlebars.registerPartial('partial', '{{this}},');
var rootContext = {stooges: ['Larry', 'Curly', 'Moe']};
var template = Handlebars.compile('{{#each stooges}}{{> partial}}{{/each}}');
template(rootContext); // => Larry,Curly,Moe,
```

You can change the context of the partial by setting the second param.
```js
Handlebars.registerPartial('partial', '{{this}},');
var rootContext = {stooges: ['Larry', 'Curly', 'Moe'], author: 'Dan'};
var template = Handlebars.compile('{{#each stooges}}{{> partial ../author}}{{/each}}');
template(rootContext); // => Dan,Dan,Dan,
```

# Context Data in Helpers

Helper functions can be invoked from any number of contexts (eg parials and loops) within a template. Therefore, Handlebars sets the **this** context of the helper function to the current template data context. Therefore, helpers are aware of the **current** context even if you don't directly pass any data to a helper.

```js
Handlebars.registerHelper('helper', function() {
  return this.stooges.join(',');
  });
var rootContext = {stooges: ['Larry', 'Curly', 'Moe'], author: 'Dan'};
var template = Handlebars.compile('{{helper}}');
template(rootContext); // => Larry,Curly,Moe,
```

You can pass data to the helper as positional param.
```js
Handlebars.registerHelper('helper', function(author) {
  return author + ',';
  });
var rootContext = {stooges: ['Larry', 'Curly', 'Moe'], author: 'Dan'};
var template = Handlebars.compile('{{helper author}}');
template(rootContext); // => Dan,
```

We can change the context of the helper by invoking it in a each loop.
```js
Handlebars.registerHelper('helper', function(stooge) {
  return stooge + ',';
  });
var rootContext = {stooges: ['Larry', 'Curly', 'Moe'], author: 'Dan'};
var template = Handlebars.compile('{{#each stooges}}{{helper this}}{{/each}}');
template(rootContext); // => Larry,Curly,Moe,
```

But since Handlebars set the helper **this** context we could write:
```js
Handlebars.registerHelper('helper', function() {
  return this + ',';
  });
var rootContext = {stooges: ['Larry', 'Curly', 'Moe'], author: 'Dan'};
var template = Handlebars.compile('{{#each stooges}}{{helper}}{{/each}}');
template(rootContext); // => Larry,Curly,Moe,
```

Helpers are also passed as the last options argument. You can always get access to the root context from the `options.data.root`.

```js
Handlebars.registerHelper('helper', function(options) {
  return options.data.root.author + ',';
  });
var rootContext = {stooges: ['Larry', 'Curly', 'Moe'], author: 'Dan'};
var template = Handlebars.compile('{{#each stooges}}{{helper}}{{/each}}');
template(rootContext); // => Dan,Dan,Dan,
```

This is true even if we nest much deeper. Therefore, the root data is never far.

```js
Handlebars.registerHelper('helper', function(options) {
  return options.data.root.author + ',';
  });
var rootContext = {stooges: ['Larry', 'Curly', 'Moe'], dogs: ['Lilly', 'Moca'], author: 'Dan'};
var template = Handlebars.compile('{{#each stooges}}{{#each ../dogs}}{{helper}}{{/each}}{{/each}}');
template(rootContext); // => Dan,Dan,Dan,Dan,Dan,Dan,
```

