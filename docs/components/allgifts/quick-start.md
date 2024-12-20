# Быстрый старт

Из-за большого кол-ва создаваемых товаров, старт трудно назвать «быстрым», тем не менее я постарался максимально упростить процедуру настройки.

## Установка и настройка

Лучше всего выгружать товары на новый пустой сайт, потому что при выгрузке создаётся структура каталога сервиса, с которого происходит первичная выгрузка.

1. Установить miniShop2, msOptionsPrice2 и allGifts.

2. Создать шаблон для разделов и прописать в настройке ms2_template_category_default`

3. Создать шаблон для товаров и прописать в настройках `ms2_template_product_default` и `msoptionsprice_working_templates`

4. Очень желательно включить настройку `ms2_product_id_as_alias`

5. Указать системные настройки:

    - `msoptionsprice_allow_zero_count` = Да
    - `msoptionsprice_allow_zero_mass` = Да

6. Создать опции miniShop2, которые требуется заполнять при выгрузке и указать их ключи в системных настройках компонента allGifts, в разделе «Поля» - <https://prnt.sc/pfgu08>. Для быстрой настройки данного пункта можно [воспользоваться скриптом](#skript-dlya-bystrogo-sozdaniya), запустив его через компонент Console.

7. Если вы создали поле для резервов, то его ключ (без `options-`) необходимо прописать в системной настройке `msoptionsprice_exclude_modification_options`. **С версии 1.3.7 не обязательно.**

8. В системных настройках компонента прописать доступы к API сервисов выгрузки.

На этом настройка завершена, переходим к запуску!

### Скрипт для быстрого создания опций

Убедитесь, что ваш список уже созданных опций не пересекается со списком в скрипте!

```php
<?php
$modx->setLogLevel(xPDO::LOG_LEVEL_FATAL);
$ms2 = $modx->getService('miniShop2');
$categories = [];
$q = $modx->newQuery('msCategory', ['class_key' => 'msCategory'])->select('id');
if ($q->prepare()->execute()) {
  $categories = @$q->stmt->fetchAll(PDO::FETCH_COLUMN)?:[];
}
foreach ([
  'size' => null,
  'color' => null,
  'stocks' => [
    'caption' => 'Склады',
    'type' => 'combo-options',
  ],
  'gender' => [
    'caption' => 'Пол',
    'type' => 'combo-options',
  ],
  'brand' => [
    'caption' => 'Бренд',
    'type' => 'combo-options',
  ],
  'material' => [
    'caption' => 'Материал',
    'type' => 'combo-options',
  ],
  'product_size' => [
    'caption' => 'Размеры товара',
    'type' => 'combo-options',
  ],
  'density' => [
    'caption' => 'Плотность',
    'type' => 'combo-options',
  ],
  'branding' => [
    'caption' => 'Вид нанесения',
    'type' => 'combo-options',
  ],
  'memory' => [
    'caption' => 'Объем памяти',
    'type' => 'combo-options',
  ],
  'dating' => [
    'caption' => 'Датировка',
    'type' => 'combo-options',
  ],
  'calendars' => [
    'caption' => 'Календари',
    'type' => 'combo-options',
  ],
  'type_caps' => [
    'caption' => 'Тип кепки',
    'type' => 'combo-options',
  ],
  'migalki' => [
    'caption' => 'Моргалки',
    'type' => 'combo-options',
  ],
  'packing' => [
    'caption' => 'Упаковка',
    'type' => 'combo-options',
  ],
  'group' => [
    'caption' => 'Группа товаров',
    'type' => 'textfield',
  ],
  'main_product' => [
    'caption' => 'Главный товар группы',
    'type' => 'combo-boolean',
  ],
  'source' => [
    'key' => 'service',
    'caption' => 'Источник',
    'type' => 'textfield',
  ],
  'count' => [
    'key' => 'instock',
    'caption' => 'Кол-во на складе',
    'type' => 'numberfield',
  ],
  'reserves' => [
    'caption' => 'В резерве',
    'type' => 'numberfield',
  ],
] as $key => $field) {
  if (!empty($field)) {
    if (!isset($field['key'])) {
      $field = array_merge(['key' => $key], $field);
    }
    if ($option = $modx->getObject('msOption', ['key' => $field['key']])) {
      $field['id'] = $option->get('id');
      print_r('Опция ' . $field['key'] . ' уже существует.' . PHP_EOL);
    } else {
      $modx->error->reset();
      $response = $ms2->runProcessor('mgr/settings/option/create', array_merge([
        'category' => 0,
      ], $field));
      $response = $response->getObject();
      if (!empty($response['id'])) {
        $field['id'] = $response['id'];
        print_r('Опция ' . $field['key'] . ' успешно создана.' . PHP_EOL);
      } else {
        print_r('Процесс остановлен! Не удалось создать опцию ' . $field['key'] . '. Ошибка: ' . print_r(@$modx->error->errors[0]['msg'], 1));
        continue;
      }
    }
    if (!empty($field['id'])) {
      foreach ($categories as $category_id) {
        if (!$modx->getCount('msCategoryOption', [
          'option_id' => $field['id'],
          'category_id' => $category_id,
          'active' => true,
        ])) {
          $modx->error->reset();
          $ms2->runProcessor('mgr/settings/option/assign', [
            'option_id' => $field['id'],
            'category_id' => $category_id,
            'value' => '',
          ]);
        }
      }
    }
    if ($modx->getCount('msOption', ['key' => $field['key']])) {
      if ($setting = $modx->getObject('modSystemSetting', ['key' => 'ag_field_' . $key])) {
        $setting->set('value', 'options-' . $field['key']);
        $setting->save();
      }
    }
  } else {
    if ($setting = $modx->getObject('modSystemSetting', ['key' => 'ag_field_' . $key])) {
      $setting->set('value', $key);
      $setting->save();
    }
  }
}
```

## Запуск

::: warning
**Запускайте скрипт от имени юзера, под которым крутится сайт!**

Если запустить от рута или другого юзера, то в кеше будут созданы папки с чужими правами и MODX начнёт сыпать ошибки о невозможности удалить файл или директорию в кеше.

Для этого, если вы зашли в терминал от рута или судо юзера, во всех примерах запуска, перед `php …` пропишите `sudo -u{user}`, чтобы получилось примерно так:

```bash
sudo -u{user} php script.php
```

где `{user}` - это имя пользователя в системе, под которым крутится сайт, в ином случае конструкцию `sudo -u{user}` нужно удалить.
:::

Теперь нужно сделать предварительную выгрузку разделов каталога. Проще говоря, создадим структуру каталога по подобию структуры сервиса, который запускаем первым. В данном случае это Gifts.ru. Для этого в терминале запускаем скрипт выгрузки с параметром `--categories=1`:

```bash
php /path_to_site/core/components/allgifts/cron/import/run.php --service=giftsru --categories=1
```

На полную выгрузку товаров с сервиса может потребоваться часов 5-10. Наберитесь терпения и дождитесь окончания работы!

Когда первичная выгрузка товаров со структурой каталога завершена, то можно прописать задание в крон, примерно так:

```shell
0 2 * * * sudo -u{user} php /path_to_site/core/components/allgifts/cron/import/run.php --log=0 --service=giftsru
```

Где `{user}` - это имя пользователя в системе, под которым крутится сайт.
