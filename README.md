# TodoMVC in ~100 lines of vanilla JavaScript

Or: How I Learned to Stop Worrying and Love the DOM

## The problem: keeping the state and the UI in sync

How to keep the state and the UI in sync is a central problem when building web apps.

To begin, let's say you have some app state data stored in a JavaScript object. Let's say it looks like this:

```js
const state = {
    items: [
        { text: 'Foo' },
        { text: 'Bar },
    ],
}
```

Now, we want to display this in our HTML page. We'd like to render it somehow like this:

```html
<ul>
    <li>Foo</li>
    <li>Bar</li>
</ul>
```

It's easy enough to transform the data into HTML or DOM objects. But it gets tricky when the data changes:

```js
state.items.push({ text: 'Baz' })
// What happens now?
```

An even bigger problem arises when you want to change the data from the HTML UI:

```html
<ul>
    <li>Foo <button onclick="remove()">Remove</button></li>
    <li>Bar <button onclick="remove()">Remove</button></li>
</ul>
<script>
function remove() {
    // Need to remove element from the `items` array.
    // What to do here?
}
</script>
```

Things can get complicated quickly when rendering data manually this way.

How do you solve this? Answer: you don't. Rather than trying to solve the hard problem of keeping the UI in sync with the state, you ensure that the UI *never* changes (given any state). This is known as *declarative UI*.

Popular JavaScript libraries such as [React](https://react.dev) allow you to build UI declaratively, using various run-time or compile-time optimizations, such as virtual DOMs, to make re-rendering efficient. While this approach has a lot of advantages, there is one big problem: you're always fighting the DOM, and that adds a lot of complexity, and sometimes hefty performance penalties. Are there simpler solutions?

## What if you just put the state in the DOM?

Let's take a step back and consider our assumptions: (1) we have the data in JavaScript, and (2) this data needs to be rendered as HTML.

But wait. An HTML document *is* data, and the browser takes care of the rendering for you. So why don't we just put the data directly in the DOM?

Let's consider our earlier example again. Our state is stored in a JavaScript object:

```js
const state = {
    items: [
        { text: 'Foo' },
        { text: 'Bar },
    ],
}
```

To add an item, we would need to do something like this:

```js
state.items.push({ text: 'Baz' })
render() // somehow update the DOM
```

You could use a library or framework that handles this automatically for you. But *someone* has to do the chore of updating the DOM no matter what.

Now what if the state *is* the DOM? With [Web Components](https://developer.mozilla.org/en-US/docs/Web/API/Web_Components), we can store the same data as HTML elements, like so:

```html
<my-list>
    <my-item text="Foo"></my-item>
    <my-item text="Bar"></my-item>
</my-list>
```

To add an item, you just append a new element:

```js
const item = document.createElement('my-item')
item.setAttribute('text', 'Baz')
document.querySelector('my-list').append(item)
```

That's it! There's nothing more to do. What about removing an item? Thanks to the [`Element.remove()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/remove) method, it's as simple as

```js
item.remove()
```

Want to add a "Remove" button? No need to loop through the whole list just to remove an item. Just call `this.remove()` in your item component and you're done!

Reacting to changes can be achieved with the [Mutation Observer](https://developer.mozilla.org/en-US/docs/Web/API/MutationObserver) API:

```js
new MutationObserver(mutations => {
    console.log('some items were added or removed')
}).observe(document.querySelector('my-list'), { childList: true })
```

This is actually better than plain JavaScript objects, which can't really be observed (proxies are not observers). Now instead of fighting the DOM, you're using the DOM to your advantage.

What are some other advantages of storing data in the DOM? For one, remember that it's a *tree*. That means unlike JavaScript objects, you can keep and traverse lists and nested data very easily and efficiently. Have you ever used a tree or linked list library in JavaScript? Or even implemented one yourself? You don't have to! Just use the DOM!

## The result

To experiment with this approach, I made a [TodoMVC](https://todomvc.com) implementation. It contains only 101 lines of code in vanilla JavaScript, which is 1kb minified and gzipped.

But I'm cheating a little with that number. Because some bits aren't implemented with JavaScript at all. For example, filtering items by completion state is done using just CSS:

```css
#todos[data-filter="active"] [completed] {
    display: none;
}
#todos[data-filter="completed"] :not([completed]) {
    display: none;
}
```

Unlike most TodoMVC examples, there isn't a "store" where you have a bunch of `addTodo()`, `removeTodo()` methods and whatnot. Everything lives directly in the DOM. Even the data stored in `localStorage` is in HTML, not JSON!

## Conclusion: you're using the DOM wrong

For years, people have been keeping states in JavaScript and rendering them in the DOM. But that means you're unnecessarily keeping the data in two separate places—and then you complain about how keeping them in sync is difficult!

The DOM is designed to hold data. The "M" in "DOM" literally stands for "model". Using it only as a view is counterintuitive and counterproductive.

You might not need data binding, or virtual DOM, or state management, or the whole jungle... if the state is stored in the DOM, which can be implemented cleanly today with Web Components and Mutation Observers.
