Для реалізації стрибання ми можемо викристати CSS властивість `top` та `position:absolute` для м’яча, який знаходиться всередині поля з `position:relative`.

Нижні координати поля є `field.clientHeight`. CSS властивість `top` вказує на верхню границю м’яча. Отже її слід міняти від `0` до `field.clientHeight - ball.clientHeight`, це і буде кінцева найнижча позиція верхньої границі м’яча.

Щоб отримати ефект "стрибання", ми можемо використати часову функцію `bounce` у режимі `easeOut`.

Ось кінцевий варіант коду анімації:

```js
let to = field.clientHeight - ball.clientHeight;

animate({
  duration: 2000,
  timing: makeEaseOut(bounce),
  draw(progress) {
    ball.style.top = to * progress + 'px'
  }
});
```
