# JavaScript анімації

JavaScript анімації можуть справитися з тим, з чим не зможе чистий CSS.

Наприклад, рух за складною траекторією з часовою функцією, відмінною від кривої Безьє, чи canvas-анімації.

## Використання setInterval

Анімація може бути реалізована як послідовність кадрів -- зазвичай незначних змін HTML/CSS властивостей.

Скажімо, якщо поміняти `style.left` з `0px` на `100px`, то елемент буде рухатися. А якщо ми збільшимо цю властивість з невеликою затримкою у `setInterval` щораз на `2px`, як-от 50 разів за секунду, тоді він буде рухатися плавно. За таким же принципом працює кінотеатр: 24 кадри за секунду достатньо, щоб картинка виглядала плавною.

Псевдо-код виглядатиме наступним чином:

```js
let timer = setInterval(function() {
  if (animation complete) clearInterval(timer);
  else increase style.left by 2px
}, 20); // змінювати на 2px кожні 20мс, приблизно 50 кадрів за секунду
```

Детальніший приклад анімації:

```js
let start = Date.now(); // запам'ятати час початку

let timer = setInterval(function() {
  // скільки часу пройшло з моменту початку?
  let timePassed = Date.now() - start;

  if (timePassed >= 2000) {
    clearInterval(timer); // завершити анімацію після 2 секунд
    return;
  }

  // відмалювати анімацію у момент timePassed
  draw(timePassed);

}, 20);

// по мірі того, як timePassed міняється від 0 до 2000
// лівий край отримує значення від 0px до 400px
function draw(timePassed) {
  train.style.left = timePassed / 5 + 'px';
}
```

Клікніть, щоб побачити демо:

[codetabs height=200 src="move"]

## Використання requestAnimationFrame

Давайте уявимо, що у нас є декілька анімацій, які запущені одночасно.

Якщо ми їх запустимо окремо одна від одної, то навіть якщо кожна з них має `setInterval(..., 20)`, браузер муситиме їх відмальовувати набагато частіше, ніж кожні `20мс`.

Це стається тому, що у них різний початковий час, тому "кожні 20мс" буде відрізнятися для різниих анімацій. Інтервали не вирівнюються між собою. Отже ми отримаємо декілька запущених анімацій у межах `20мс`.

Іншими словами, даний код:

```js
setInterval(function() {
  animate1();
  animate2();
  animate3();
}, 20)
```

...Є легшою версією, ніж три незалежні виклики:

```js
setInterval(animate1, 20); // незалежні один від одного анімації
setInterval(animate2, 20); // у різних місцях скрипта
setInterval(animate3, 20);
```

Ці декілька незалежних анімацій мали б бути згруповані, щоб спростити відмалювання для браузера і, таким чином, менше навантажувати процесор та мати плавніший вигляд.

Тут слід пам'ятати про наступне. Іноді процесор перевантажений, або є інші причини, щоб відмалювання відбувалося рідше (наприклад тоді, коли вкладка браузера прихована), тому дійсно не слід робити оновлення кожні `20мс`.

Але як дізнатися про це у JavaScript? Існує специфікація [Час анімації](http://www.w3.org/TR/animation-timing/) яка забезпечую функцію `requestAnimationFrame`. Вона призначена для вирішення всіх цих питань та навіть більше.

Синтакс:
```js
let requestId = requestAnimationFrame(callback)
```

Цей код планує запуск `колбек` функції на найближчий часовий період, коли браузер запускатиме анімацію.

Якщо зробити зміни елементів у `колбек` то вони будуть згруповані з іншим `requestAnimationFrame` колбеками та CSS анімаціями. Таким чином, буде проведено тільки один розрахунок геометрії та відмалювання, замість багатьох.

Можна використати повернене значення `requestId` щоб відмінити виклик:
```js
// відмінити заплановане виконання колбека
cancelAnimationFrame(requestId);
```

`Колбек` отримує один аргумент -- час, який пройшов з моменту початку завантаження сторінки у мілісекундах. Цей час також можна отримати, якщо викликати [performance.now()](mdn:api/Performance/now).

Зазвичай `колбек` виконується досить швидко, якщо процесор не перевантажений чи батарея ноутбука не є майже розряджена, чи не існує якась інша причина, яка перешкоджатиме цьому.

Код, написаний нижче, показує час між першими 10-ма виконаннями `requestAnimationFrame`. Зазвичай це 10-20мс:

```html run height=40 refresh
<script>
  let prev = performance.now();
  let times = 0;

  requestAnimationFrame(function measure(time) {
    document.body.insertAdjacentHTML("beforeEnd", Math.floor(time - prev) + " ");
    prev = time;

    if (times++ < 10) requestAnimationFrame(measure);
  })
</script>
```

## Структурована анімація

Тепер ми можемо зробити універсальнішу функцію анімації, на основі `requestAnimationFrame`:

```js
function animate({timing, draw, duration}) {

  let start = performance.now();

  requestAnimationFrame(function animate(time) {
    // timeFraction міняється з 0 на 1
    let timeFraction = (time - start) / duration;
    if (timeFraction > 1) timeFraction = 1;

    // прорахувати поточний стан анімації
    let progress = timing(timeFraction)

    draw(progress); // відмалювати

    if (timeFraction < 1) {
      requestAnimationFrame(animate);
    }

  });
}
```

Функція `animate` приймає 3 параметри, які описують суть анімації:

`duration`
: Загальний час анімації. Наприклад `1000`.

`timing(timeFraction)`
: Часова функція, подібна на CSS-властивість `transition-timing-function` що отримує частку часу, який пройшов (`0` на початку, `1` вкінці) та повертає завершення анімації (як `y` у кривій Безьє).

    Наприклад, дана лінійна функція описує анімацію, яка прогресує рівномірно та з однаковою швидкістю:

    ```js
    function linear(timeFraction) {
      return timeFraction;
    }
    ```

    Її графічне відображення:
    ![](linear.svg)

    Це зовсім як `transition-timing-function: linear`. Внизу описано ще більше цікавих варіантів.

`draw(progress)`
: Функція, яка отримує стан завершення анімації та відмальовує її. Значення `progress=0` позначає початковий стан анімації, а `progress=1` -- кінцевий стан.

    Це саме та функція, яка, власне, відмальовує анімацію.

    Вона може рухати елемент:
    ```js
    function draw(progress) {
      train.style.left = progress + 'px';
    }
    ```

    ...Чи будь-що інше, адже ми можемо анімувати все, що завгодно та як завгодно.


Let's animate the element `width` from `0` to `100%` using our function.

Click on the element for the demo:

[codetabs height=60 src="width"]

The code for it:

```js
animate({
  duration: 1000,
  timing(timeFraction) {
    return timeFraction;
  },
  draw(progress) {
    elem.style.width = progress * 100 + '%';
  }
});
```

Unlike CSS animation, we can make any timing function and any drawing function here. The timing function is not limited by Bezier curves. And `draw` can go beyond properties, create new elements for like fireworks animation or something.

## Timing functions

We saw the simplest, linear timing function above.

Let's see more of them. We'll try movement animations with different timing functions to see how they work.

### Power of n

If we want to speed up the animation, we can use `progress` in the power `n`.

For instance, a parabolic curve:

```js
function quad(timeFraction) {
  return Math.pow(timeFraction, 2)
}
```

The graph:

![](quad.svg)

See in action (click to activate):

[iframe height=40 src="quad" link]

...Or the cubic curve or even greater `n`. Increasing the power makes it speed up faster.

Here's the graph for `progress` in the power `5`:

![](quint.svg)

In action:

[iframe height=40 src="quint" link]

### The arc

Function:

```js
function circ(timeFraction) {
  return 1 - Math.sin(Math.acos(timeFraction));
}
```

The graph:

![](circ.svg)

[iframe height=40 src="circ" link]

### Back: bow shooting

This function does the "bow shooting". First we "pull the bowstring", and then "shoot".

Unlike previous functions, it depends on an additional parameter `x`, the "elasticity coefficient". The distance of "bowstring pulling" is defined by it.

The code:

```js
function back(x, timeFraction) {
  return Math.pow(timeFraction, 2) * ((x + 1) * timeFraction - x)
}
```

**The graph for `x = 1.5`:**

![](back.svg)

For animation we use it with a specific value of `x`. Example for `x = 1.5`:

[iframe height=40 src="back" link]

### Bounce

Imagine we are dropping a ball. It falls down, then bounces back a few times and stops.

The `bounce` function does the same, but in the reverse order: "bouncing" starts immediately. It uses few special coefficients for that:

```js
function bounce(timeFraction) {
  for (let a = 0, b = 1; 1; a += b, b /= 2) {
    if (timeFraction >= (7 - 4 * a) / 11) {
      return -Math.pow((11 - 6 * a - 11 * timeFraction) / 4, 2) + Math.pow(b, 2)
    }
  }
}
```

In action:

[iframe height=40 src="bounce" link]

### Elastic animation

One more "elastic" function that accepts an additional parameter `x` for the "initial range".

```js
function elastic(x, timeFraction) {
  return Math.pow(2, 10 * (timeFraction - 1)) * Math.cos(20 * Math.PI * x / 3 * timeFraction)
}
```

**The graph for `x=1.5`:**
![](elastic.svg)

In action for `x=1.5`:

[iframe height=40 src="elastic" link]

## Reversal: ease*

So we have a collection of timing functions. Their direct application is called "easeIn".

Sometimes we need to show the animation in the reverse order. That's done with the "easeOut" transform.

### easeOut

In the "easeOut" mode the `timing` function is put into a wrapper `timingEaseOut`:

```js
timingEaseOut(timeFraction) = 1 - timing(1 - timeFraction)
```

In other words, we have a "transform" function `makeEaseOut` that takes a "regular" timing function and returns the wrapper around it:

```js
// accepts a timing function, returns the transformed variant
function makeEaseOut(timing) {
  return function(timeFraction) {
    return 1 - timing(1 - timeFraction);
  }
}
```

For instance, we can take the `bounce` function described above and apply it:

```js
let bounceEaseOut = makeEaseOut(bounce);
```

Then the bounce will be not in the beginning, but at the end of the animation. Looks even better:

[codetabs src="bounce-easeout"]

Here we can see how the transform changes the behavior of the function:

![](bounce-inout.svg)

If there's an animation effect in the beginning, like bouncing -- it will be shown at the end.

In the graph above the <span style="color:#EE6B47">regular bounce</span> has the red color, and the <span style="color:#62C0DC">easeOut bounce</span> is blue.

- Regular bounce -- the object bounces at the bottom, then at the end sharply jumps to the top.
- After `easeOut` -- it first jumps to the top, then bounces there.

### easeInOut

We also can show the effect both in the beginning and the end of the animation. The transform is called "easeInOut".

Given the timing function, we calculate the animation state like this:

```js
if (timeFraction <= 0.5) { // first half of the animation
  return timing(2 * timeFraction) / 2;
} else { // second half of the animation
  return (2 - timing(2 * (1 - timeFraction))) / 2;
}
```

The wrapper code:

```js
function makeEaseInOut(timing) {
  return function(timeFraction) {
    if (timeFraction < .5)
      return timing(2 * timeFraction) / 2;
    else
      return (2 - timing(2 * (1 - timeFraction))) / 2;
  }
}

bounceEaseInOut = makeEaseInOut(bounce);
```

In action, `bounceEaseInOut`:

[codetabs src="bounce-easeinout"]

The "easeInOut" transform joins two graphs into one: `easeIn` (regular) for the first half of the animation and `easeOut` (reversed) -- for the second part.

The effect is clearly seen if we compare the graphs of `easeIn`, `easeOut` and `easeInOut` of the `circ` timing function:

![](circ-ease.svg)

- <span style="color:#EE6B47">Red</span> is the regular variant of `circ` (`easeIn`).
- <span style="color:#8DB173">Green</span> -- `easeOut`.
- <span style="color:#62C0DC">Blue</span> -- `easeInOut`.

As we can see, the graph of the first half of the animation is the scaled down `easeIn`, and the second half is the scaled down `easeOut`. As a result, the animation starts and finishes with the same effect.

## More interesting "draw"

Instead of moving the element we can do something else. All we need is to write the proper `draw`.

Here's the animated "bouncing" text typing:

[codetabs src="text"]

## Summary

For animations that CSS can't handle well, or those that need tight control, JavaScript can help. JavaScript animations should be implemented via `requestAnimationFrame`. That built-in method allows to setup a callback function to run when the browser will be preparing a repaint. Usually that's very soon, but the exact time depends on the browser.

When a page is in the background, there are no repaints at all, so the callback won't run: the animation will be suspended and won't consume resources. That's great.

Here's the helper `animate` function to setup most animations:

```js
function animate({timing, draw, duration}) {

  let start = performance.now();

  requestAnimationFrame(function animate(time) {
    // timeFraction goes from 0 to 1
    let timeFraction = (time - start) / duration;
    if (timeFraction > 1) timeFraction = 1;

    // calculate the current animation state
    let progress = timing(timeFraction);

    draw(progress); // draw it

    if (timeFraction < 1) {
      requestAnimationFrame(animate);
    }

  });
}
```

Options:

- `duration` -- the total animation time in ms.
- `timing` -- the function to calculate animation progress. Gets a time fraction from 0 to 1, returns the animation progress, usually from 0 to 1.
- `draw` -- the function to draw the animation.

Surely we could improve it, add more bells and whistles, but JavaScript animations are not applied on a daily basis. They are used to do something interesting and non-standard. So you'd want to add the features that you need when you need them.

JavaScript animations can use any timing function. We covered a lot of examples and transformations to make them even more versatile. Unlike CSS, we are not limited to Bezier curves here.

The same is about `draw`: we can animate anything, not just CSS properties.
