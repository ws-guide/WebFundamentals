project_path: /web/_project.yaml
book_path: /web/fundamentals/_book.yaml
description: Обработчики ввода потенциально являются источниками проблем с производительностью в приложениях, поскольку они могут заблокировать завершение кадров, а также вызывать дополнительную (и совершенно ненужную) работу по пересчету макета

{# wf_updated_on: 2015-03-19 #}
{# wf_published_on: 2000-01-01 #}

# Оптимизация обработчиков ввода {: .page-title }

{% include "web/_shared/contributors/paullewis.html" %}


Обработчики ввода потенциально являются источниками проблем с производительностью в приложениях, поскольку они могут заблокировать завершение кадров, а также вызывать дополнительную (и совершенно ненужную) работу по пересчету макета

### TL;DR {: .hide-from-toc }
- "Не используйте обработчики ввода с длительным рабочим циклом\_– они могут заблокировать прокрутку."
- Не изменяйте стили в обработчиках ввода.
- Оптимизируйте обработчики; запоминайте значения событий и реализуйте изменения стилей при следующем обратном вызове функции requestAnimationFrame.


## Не используйте обработчики ввода с длительным рабочим циклом

В самом быстром варианте, когда пользователь вводит данные на страницу, поток компоновки страницы может принять касание пользователя и просто переместить контент. Основной поток, в котором выполняется код JavaScript, перерисовка, рассчитывается макет и применяются стили, при этом задействован не будет вообще.

<img src="images/debounce-your-input-handlers/compositor-scroll.jpg" class="center" alt="Облегченная прокрутка; только компоновщик.">

Однако, если прикрепить обработчик ввода, например `touchstart`, `touchmove` или `touchend`, поток компоновщика должен будет подождать завершения выполнения рабочего цикла этого обработчика, поскольку может быть вызвана функция `preventDefault()` и прокрутка будет остановлена. Даже если функция `preventDefault()` не вызывается, компоновщик должен ждать, а выполняемая пользователем прокрутка будет заблокирована, что может привести к подвисанию или пропуску кадров.

<img src="images/debounce-your-input-handlers/ontouchmove.jpg" class="center" alt="Тяжелая прокрутка; компоновщик заблокирован на JavaScript.">

Короче говоря, рабочий цикл любых используемых обработчиков ввода должен выполняться быстро, чтобы компоновщик мог делать свою работу.

## Не изменяйте стили в обработчиках ввода

Рабочий цикл обработчиков ввода, например прокрутки и касаний, выполняется перед тем, как будут выполняться обратные вызовы функции `requestAnimationFrame`.

Если внутри одного из таких обработчиков вносится визуальное изменение, то в начале выполнения обратного вызова `requestAnimationFrame` необходимо будет применить изменение стиля. Если _после этого_ выполнить чтение визуальных свойств в начале обратного вызова requestAnimationFrame, как рекомендуется в статье "[Не используйте большие, сложные макеты и избегайте подтормаживания макетов](avoid-large-complex-layouts-and-layout-thrashing)", вы вызовите принудительный синхронный перерасчет макета!

<img src="images/debounce-your-input-handlers/frame-with-input.jpg" class="center" alt="Тяжелая прокрутка; компоновщик заблокирован на JavaScript.">

## Оптимизируйте обработчики прокрутки

Обе описанные выше неполадки имеют одно решение: следует всегда откладывать внесение визуальных изменений до следующего обратного вызова `requestAnimationFrame`:


    function onScroll (evt) {
    
      // Store the scroll value for laterz.
      lastScrollY = window.scrollY;
    
      // Prevent multiple rAF callbacks.
      if (scheduledAnimationFrame)
        return;
    
      scheduledAnimationFrame = true;
      requestAnimationFrame(readAndUpdatePage);
    }
    
    window.addEventListener('scroll', onScroll);
    

Кроме того, обработчики ввода при этом останутся необременительными, а это отлично, поскольку теперь прокрутка или касания при выполнении кода с большим объемом вычислений не будут блокироваться!


