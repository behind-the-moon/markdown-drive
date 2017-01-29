## Другой взгляд на разработку приложений для Smart TV.

Не смотря на то, что Smart TV появились на рынке СНГ и стали набирать популярность достаточно давно (~2010) - технологии/подходы
разработки приложений для них сильно отстают во времени, порой обвивая приятными воспоминаниями из 7х или ранее годов.

Мне бы хотелось поделиться результатами моего исследования области разработки приложений для SmartTV, подчеркнуть некоторые 
недостатки и конечно же рассказать о найденных решениях и просто прикольных штуках.

### Что такое приложения для Smart TV ?

В "приложение для Smart TV" я вкладываю определение как web приложения, специальным таким образом подогнанное под телевизор.
По большому счету, отличий не так и много, по большей части это несовместимость/отсутствие некоторого API на различных устройствах, к тому же не стоит забывать что телевизор не компьютер и его ресурсы более ограничены.

Ключевые (как по мне) отличия:

* Ограниченные ресурсы.
* Специальная навигация.
* Местами неожиданное поведение на различных устройствах.
* Немного запутанные способы тестирования.
* Неожиданные варианты деплоймента и обновления приложения.

Если относительно первых трех пунктов немного понятно и многие уже с ними сталкивались, то с оставшимися есть ряд проблем, которые каждый должен решать самостоятельно. Как например запуск тестов в окружении приложения на отдельно взятых устройствах, варианты обновления и запуска конечного приложения.

Но я хочу рассказать именно о первых двух.

### Почему ?

Существует очень много замечательных инструментов для разработки web приложений, но большинство из них не создавалось даже с 
мыслью о том что это будет использоваться на телевизорах. С другой стороны, предназначенные для телевизоров интсрументы не очень приспособлены для разработки веб приложений: некоторые отстали во времени, некоторые разработаны в лучших традициях и практиках лохматых годов или же перенесены из других языков без оглядки, на текущий этап развития фронт-энд разработки. Возможно, это больно осознавать, но, блин, на дворе 2017й год!

Мне очень близок здоровый (или нездоровый) минимализм, не люблю, знаете ли, разрабатывать с использованием библиотечки в 10кб раздуваемую до мегабайта при разработке `hello world`'a. Безусловно, можно смело сказать о том, что там очень много классного функционала, но я считаю более правильным держать только используемый.

Но это же очевидно ! - скажете вы. Не для всех, далеко не для всех, я видела, из чего состоят и как работают некторые, весьма популярные приложения. 

### Мысли

* Когда речь заходит о производительности веб приложения, первое, что приходит на ум, - это **virtual-dom**, оптимальный алгоритм мутации дерева елементов. Было бы неплохо применить подобную практику для разработки производительных приложений для телевозира.

* **Навигация**, камень преткновения практически всех разработчиков приложений, управляемых пультом.

Выглядит она примерно вот так:

[![Spatial-Demo](https://raw.githubusercontent.com/behind-the-moon/markdown-drive/habrahabr/contrib/spatial-navigation.gif)](http://codepen.io/linuxenko/full/MJONar/)

Очень хотелось бы избавиться/сократить кол-во упоминаний движения кнопок при разработке приложений, избежать селекторов и создание кучи листенеров для перехода от элемента к элементу.

* К вышесказанному хотелось бы иметь **лаконичный синтаксис** описания подобных элементов и в то же время не теряя контроль над ними, например так:

```js
<div focusable={true}></div>
<div></div>
<div focusable={true} onEnterx={() => alert('Ola!')}></div>
```

И чтоб все это было прозрачненько, и само по себе работало и обновлялось при мутациях, вот было бы круто, не правда ли ? Так вот, все эти идеи я уже реализовала, и далее просто приведу пример использования.

### Пример приложения

Буду использовать `babel` для приятного `jsx` синтаксиса управление уходит в `spatial-virtual-dom` обвернутый в `cakejs` и какой-нибудь `qunit` для простоты демонстрации.

Может показаться немного специфичным, надеюсь удалось раскрыть в комментариях все заклинания:

<spoiler title="Пример кода">
```js
/** @jsx h */   // прагма

/**
 * или import {s...} from 'cakejs2-spatial; если `npm i cakejs2-spatial`
 */
const {spatial, Cream, create} = cake; 

/**
 * Создаем гиперскрипт и назначаем клавиши
 */
const h = spatial({ keys: {
  LEFT: 37,
  RIGHT: 39,
  UP: 38,
  DOWN: 40,
  ENTER: 13
}});

/**
 * создание апы, сохраняем инстанс для тестов
 */
const app = create({ 
  element : document.getElementById('application')
})
.route('*', 'rectangles');

Cream.extend({   // компонент
  _namespace: 'rectangles',
  render : createRectangles()
});

function createRectangles () {
  let isFocusable = true;
  
  return function () {
    return (
      <div className="rectangles">
        <div focusable={isFocusable} className="rectangles--left"></div>
        <div focusable={isFocusable} className="rectangles--left"></div>
        <div focusable={isFocusable} className="rectangles--right"></div>
        <div focusable={isFocusable} className="rectangles--right"></div>
        <div focusable={isFocusable} className="rectangles--right"></div>
        <div focusable={isFocusable} className="rectangles--left"></div>
        <div focusable={isFocusable} className="rectangles--left"></div>
      </div>
    )
  }
}
```
</spoiler>
[Пример приложения (CODEPEN)](http://codepen.io/linuxenko/pen/MJONar)

Тестировать пока сложновато, нужны шорткуты (PR велкоме):

<spoiler title="Код теста">
```js
QUnit.test('Spatial test', function( assert ) {
  const rectangles = app.tree.children[0];
  assert.ok( 7 === app.tree.sn._collection.length, 'Should spatialize 7 elements');
  assert.ok( null === app.tree.sn._focus, 'Should not focus any of them');
  
  app.tree.sn.focus(rectangles.children[2].el);
  assert.ok(rectangles.children[2].el === app.tree.sn._focus, 'Should focus third children');
  app.tree.sn.move('left'); // element floats left
  assert.ok(rectangles.children[3].el === app.tree.sn._focus, 'Should move focus left');
});
```
</spoiler>
[Пример тестов (CODEPEN)](http://codepen.io/linuxenko/pen/MJONBP)

### Итоги
 
 * Размер библиотек не привышает нескольких десятков килобайт, в сжатом виде ~10 (от дефлата зависит)
 * Производительные мутации, вот реализация [js-repaint-perfs](https://15lyfromsaturn.github.io/js-repaint-perfs/cakejs/index.html) для тортика.
 * Прозрачная реализация навигации, изкоробки.
 * Для надстроек есть эвенты из навигации и самого DOM'a.

### Ссылки

* [Implementing TV remote control navigation (MDN)](https://developer.mozilla.org/en-US/docs/Mozilla/Firefox_OS_for_TV/TV_remote_control_navigation)
* [cakejs2-spatial (NPM)](https://www.npmjs.com/package/cakejs2-spatial)
* [spatial-virtual-dom (GitHub)](https://github.com/linuxenko/spatial-virtual-dom)
* [Демка другого приложения с использованием данного стека](https://public-isbkayrpog.now.sh/)
* [Контакт по вопросам (Twitter)](https://twitter.com/linuxenko)

Прошу прощения за ридмистайл. Спасибо!
