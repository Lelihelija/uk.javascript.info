У завданні <info:task/animate-ball> було анімовано тільки одну властивість. Тепер потрібно це зробити для ще однієї: `elem.style.left`.

Горизонтальні координати змінюються згідно з іншим законом: вони не "стрибають", а поступово збільшуються, посуваючи м’яч у праву сторону.

Ми можемо написати ще одну `animate` для цього.

Цього разу ми можемо використати часову функцію `linear`, але щось на кшталт `makeEaseOut(quad)` буде виглядати набагато краще.

The code:

```js
let height = field.clientHeight - ball.clientHeight;
let width = 100;

// анімація верху (стрибання)
animate({
  duration: 2000,
  timing: makeEaseOut(bounce),
  draw: function(progress) {
    ball.style.top = height * progress + 'px'
  }
});

// анімація лівого краю (рух у праву сторону)
animate({
  duration: 2000,
  timing: makeEaseOut(quad),
  draw: function(progress) {
    ball.style.left = width * progress + "px"
  }
});
```
