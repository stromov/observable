---

layout: yandex2

style: |
    /* собственные стили можно писать здесь!! */

    #observer-iterator img {
        width: 100%;
        margin-top: 50px;
    }


---

# ![](themes/yandex2/images/logo-{{ site.presentation.lang }}.svg){:.logo}

## {{ site.presentation.title }}
{:.title}

### ![](themes/yandex2/images/title-logo-{{ site.presentation.lang }}.svg){{ site.presentation.service }}

{% if site.presentation.nda %}
![](themes/yandex2/images/title-nda.svg)
{:.nda}
{% endif %}

<div class="authors">
{% if site.author %}
<p>{{ site.author.name }}{% if site.author.position %}, {{ site.author.position }}{% endif %}</p>
{% endif %}

{% if site.author2 %}
<p>{{ site.author2.name }}{% if site.author2.position %}, {{ site.author2.position }}{% endif %}</p>
{% endif %}

</div>

## Вспомним о Promise

{:.next#question}
- ...Соответсвует одной асинхронной операции
- ...Позволяет обработать результат асинхронной операции

## Observer + Iterator = Observable

{:.images #observer-iterator}
![](pictures/IteratorAndObserver.png)

## Promise и Observable

```js
const promise = new Promise((resolve, reject) => {/* some code */})
```

```js
const observable = new Observable(({next, complete, error}) => {/* some code */})
```

|                                            |  Promise   |  Observable |
+--------------------------------------------|------------|-------------+
|  Обрабатывает значение в случае успеха     |  resolve   |  next       |
|  Обрабатывает значение в случае ошибки     |  reject    |  error      |
|  Сигнализирует о завершении                |  resolve   |  complete   |


## Пример Observable

```js
const observable = new Observable(subscriber => {
  subscriber.next(1);
  subscriber.next(2);
  subscriber.next(3);
  setTimeout(() => {
    subscriber.next(4);
    subscriber.complete();
  }, 1000);
});
```

## Subscribe

```js
console.log('just before subscribe');
observable.subscribe({
  next(x) { console.log(x); },
  error(err) { console.error(err); },
  complete() { console.log('done'); }
});
console.log('just after subscribe');
```
{:style="float:left;"}
```js
    // console
    just before subscribe
    1
    2
    3
    just after subscribe
    4
    done
```
{:.image-right}
## Unsubscribe

Не забываем отписываться от потоков, особенно от <b>бесконечных</b>!

```js
const observable = interval(1000);
const subscription = observable.subscribe(console.log);

subscription.unsubscribe();
```

## Хелперы для создания Observable
{:.section}

### from, of, timer, interval, fromEvent

## from

```js
const observableFromPromise = from(new Promise(resolve => resolve('result')));
observableFromPromise.subscribe(console.log);

// result
```

```js
const observableFromArray = from([1,2,3]);
observableFromArray.subscribe(console.log);

// 1
// 2
// 3
```

## of

```js
of(10, 20, 30)
.subscribe(
  next => console.log('next:', next),
  err => console.log('error:', err),
  () => console.log('the end'),
);

// next: 10
// next: 20
// next: 30
// the end
```

## timer и interval

```js
timer(500, 1000).subscribe(console.log)

// 0
// 1
// 2
// ...
```

```js
interval(1000).subscribe(console.log)

// 0
// 1
// 2
// ...
```

## fromEvent

```js
const clicks = fromEvent(document, 'click');
clicks.subscribe(console.log);

// При клике на document выведется объект MouseEvent
```

## Методы комбинирования потоков
{:.section}

### merge, concat

## merge

```js
merge(observable, anotherObservable);
```

{:.images}
![](pictures/merge.png)

## concat
```js
concat(observable, anotherObservable)
``` 

{:.images}
![](pictures/concat.png)

## Обработка потока
{:.section}

### pipe, map, mapTo, filter, take, reduce, tap

## pipe

Метод pipe есть у каждого потока. Он принимает в качестве аргументов операторы для обработки потока.

```js
observable.pipe(...someOperators);
```

## map

```js
const observable = from([1,2,3]);
const result = observable.pipe(map(value => value * 10));

result.subscribe(console.log);

// 10
// 20
// 30
```

## mapTo

```js
const observable = from([1,2,3]);
const result = observable.pipe(mapTo(42));

result.subscribe(console.log);

// 42
// 42
// 42
```

## filter

```js
const observable = from([1,2,3]);
const result = observable.pipe(filter(value => value % 2));

result.subscribe(console.log);

// 1
// 3
```

## take

```js
const myInterval = interval(1000)
const result = myInterval.pipe(take(3))

result.subscribe(console.log);

// 0
// 1
// 2
```

## reduce

```js
const observable = from([1,2,3]);
const result = observable.pipe(reduce((acc, value) => acc + value, 0));

result.subscribe(console.log);

// 6
```

## Операторы можно объединять

```js
const observable = from([1,2,3]);
const result = observable.pipe(
        map(value => value * 10),
        filter(value => value % 20),
        reduce((acc, value) => acc + value, 0)
    );

result.subscribe(console.log);

// 40
```

## distinctUntilChanged

```js
const observable = from([1,2,2,3,3,3]);
const result = observable.pipe(distinctUntilChanged());

result.subscribe(console.log);

// 1
// 2
// 3
```

## tap

```js
const observable = from([1,2,3]);
const result = observable.pipe(x => someSideEffectFunction(x));

result.subscribe(console.log);
 
// 1
// 2
// 3
```

## Операторы высшего порядка
{:.section}

### mergeMap, switchMap

## mergeMap

```js
const letterObservable = of('a', 'b');
const result = letterObservable.pipe(
  mergeMap(letter => 
    interval(1000).pipe(take(2), map(i => `${letter} ${i}`))),
);

result.subscribe(console.log);

// a 0
// b 0
// a 1
// b 1
```
## mergeMap

{:.images}
![](pictures/mergeMap.png)

## switchMap

```js
const outerInterval = 
    interval(1500).pipe(take(2));
const result = outerInterval.pipe(
  switchMap(letter =>
    interval(1000).pipe(
      take(3),
      map(outerValue => 
        `${outerValue} ${i}`
        )
    )
  )
).subscribe(console.log);

```
{:style="float:left;"}
```js
    // console
    0 0
    0 1
    1 0
    1 1
    1 2
```
{:.image-right}

## switchMap

{:.images}
![](pictures/switchMap.png)

## Контакты 
{:.contacts}

{% if site.author %}

<figure markdown="1">

### {{ site.author.name }}

{% if site.author.position %}
{{ site.author.position }}
{% endif %}

</figure>

{% endif %}

{% if site.author2 %}

<figure markdown="1">

### {{ site.author2.name }}

{% if site.author2.position %}
{{ site.author2.position }}
{% endif %}

</figure>

{% endif %}

<!-- разделитель контактов -->
-------

<!-- left -->
<!-- - {:.skype}author -->
<!-- - {:.mail}author@yandex-team.ru -->
<!-- - {:.github}author -->

<!-- right -->
<!-- - {:.twitter}@author -->
<!-- - {:.facebook}author -->
- {:.telegram}@stromov

<!-- 

- {:.mail}author@yandex-team.ru
- {:.phone}+7-999-888-7766
- {:.github}author
- {:.bitbucket}author
- {:.twitter}@author
- {:.telegram}author
- {:.skype}author
- {:.instagram}author
- {:.facebook}author
- {:.vk}@author
- {:.ok}@author

-->
