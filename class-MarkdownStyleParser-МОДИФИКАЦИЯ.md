Реализуем поддержку синтаксиса `<--mds ... -->` . Для этого:

### 1. Модифицируем парсер
```php
class MarkdownStyleParser {
    public function parse(string $html): string {
        return preg_replace_callback(
            '/<--mds\s*(.*?)\s*-->/s', // Ищем <--mds ... -->
            function ($matches) {
                $tagParser = new TagParser();
                return $tagParser->parse(trim($matches[1]));
            },
            $html
        );
    }
}
```

### 2. Пример использования
```html
<--mds
div.item
  h3 Hello!
-->
```

**Результат:**
```html
<div class="item">
  <h3>Hello!</h3>
</div>
```

### 3. Особенности подхода
1. **Минималистичный синтаксис**  
   Всего 2 дополнительных символа (`<--mds` и `-->`), но есть нюансы:
   - `<--mds` технически не является валидным HTML-комментарием (должно быть `<!--`).
   - Браузер может отобразить содержимое как текст, если парсер не обработает блок.

2. **Альтернатива с валидными комментариями**  
   Лучше использовать стандартные комментарии с маркером:
   ```html
   <!-- mds -->
   div.item
     h3 Hello!
   <!-- /mds -->
   ```
   Тогда содержимое гарантированно не будет отображаться в браузере.

### 4. Как сделать полностью валидным
Если важно сохранить HTML-валидацию, используем гибридный подход:
```php
// Регулярка для валидных HTML-комментариев
preg_replace_callback(
    '/<!--\s*mds\s*-->(.*?)<!--\s*\/mds\s*-->/s', 
    ...
);
```

**Шаблон в HTML:**
```html
<!-- mds -->
div.item
  h3 Hello!
<!-- /mds -->
```

### 5. Плюсы вашего варианта (`<--mds ... -->`)
- **Краткость** — меньше символов для ввода.
- **Уникальность** — легко искать в коде.

### 6. Минусы
- **Нарушение стандартов** — невалидный HTML.
- **Риск отображения** — браузер может показать содержимое.

### Итог
Лучше использовать стандартные комментарии с маркером (`<!-- mds -->`), чтобы:
- Сохранить валидность HTML.
- Гарантировать игнорирование содержимого браузерами.
- Легко интегрироваться с инструментами (IDE, линтеры). 

Но если краткость критична, можно реализовать ваш вариант с пометкой в документации о необходимости обработки парсером.
