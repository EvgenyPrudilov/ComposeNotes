
ActivityResultApi

Есть старый способ получать результат от активити через startActivityForResult(), который помечен как deprecated. 
Новый способ - использовать ActivityresultApi. Суть в том, что мы создаём контракт, в котором описываем, какой результат мы хотим получить от запускаемой активити, а также, какие данные нам понадабяться на вход.

Есть класс ActivityResultcontract - это абстрактный класс и у него два параметра типа - 1) данные для запуска активити 2) данные для получения от этого экрана. Внутри него нам нужно будет переопределить два метода:

```kotlin
val contract = object : ActivityResultContract<Intent, String?>() {
  override fun createIntent(context: Context, input: Intent): Intent {
    return input
  }
  override fun parseResult(resultCode: Int, intent: Intent?): String? {
    if (resultCode == RESULT_OK) {
      return intent?.getStringExtra(UsernameActivity.EXTRA_USERNAME) ?: ""
    } 
    return null
  }
}
```

В методе createIntent нам нужно создать Intent для получения результата, а система запустит эту активити.
при завершении задачи, вызывается parseResult с кодом и результатом в интенте.

Теперь, когда контракт создан, нам нужно подписаться на результат выполнения этой задачи и создать таким образом объект лончер. После выполнения задачи будет сработан указанный колбэк(в который передаётся строка, которую мы ожидаем от активити):

```
val launcher = registerForActivityResult(constract) { it: String? ->
  if (!it.isNullOrBlank()) {
    userNameTextView.text = it
  }
}
```

Теперь, при клике по кнопке, нам нужно запустить этот лончер:

```
getUserNameButton.setOnClickListener { it: View! ->
  launch.launch(UsernameActivity.newIntent(this))
}
```

Почему мы создаём интент в launch.launch(), а не в createIntent? Нам никто не мешает так сделать:

```
...
override fun createIntent(context: Context, input: Intent): Intent {
  return UsernameActivity.newIntent(context)
}
...
getUserNameButton.setOnClickListener { it: View! ->
  launch.launch()
}
```

Но в таком случае есть проблема: класс контракт в качестве типа обязательно должен принимать два параметра, а не один. Туда можно засунуть даже Unit, но мы всё равно должны будем его передать:

```
...
override fun createIntent(context: Context, input: Unit): Intent {
  return UsernameActivity.newIntent(context)
}
...
getUserNameButton.setOnClickListener { it: View! ->
  launch.launch(Unit)
}
```

Если мы хотим создать активити для выбора и возвращения картинки, что можно сделать так:

```kotlin
val contract = object : ActivityResultContract<String, Uri?>() {
  override fun createIntent(context: Context, input: String): Intent {
    return Intent(Intent.ACTION_PICK).apply {
      type = input
    }
}
  override fun parseResult(resultCode: Int, intent: Intent?): Uri? {
    return intent?.data
  }
}
...
val launcherImage = registerForActivityResult(constract) { it: Uri? ->
  ImageFromGalleryImageView.setImageURI(it)
}
...
getImageButton.setOnClickListener { it: View! ->
  launcherImage.launch("image/*")
}
```

Создание контракта и лончера вполне можно было бы перенести в тело кликабельной функции. Но вызывать функцию registerForActivityResult нужно ДО состояния started, т.е. когда активити ещё не видна. Поэтому мы обязаны вызывать это всё в onCreate(). 

Мы сами создавали контракты, но на самом деле библиотека Андроид содержит большое количество готовых контрактов, которые можно использовать. То, что мы сделали выше, используется очень часто, поэтому такой контракт уже существует. Все контракты можно найти в классе ActivityResultContracts.

```
val contractUsername = ActivityResultContracts.StartActivityForResult()
val launcherUsername = registerForActivityResult(contractUsername) {
  if (it.resultCode == RESULT_OK) {
    usernameTextView.text = it.data?.getStringExtra(UsernameActivity.EXTRA_USERNAME) ?: ""
  }
}
```

Функция StartActivityForResult возвращает тип ActivityResult, т.к. на этапе определения функции parseResult мы не знаем, какой тип результата мы будем использовать. У него есть два поля: 1) resultCode: Int 2) data: Intent . Мы можем их использовать.

Контракт для картинки из галлереи:

```
val contractImage = ActivityResultContracts.GetContent()
val launcherImage = registerForActivityResult(contractImage) {
  imageFromGalleryImageView.setImageURI(it)
}
```


















```
