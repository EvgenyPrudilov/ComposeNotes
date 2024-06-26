----------------
Знакомство с Material Components
----------------
 
Material компоненты - это элементы интерфейса сделанные в соответствии с материал дизайном - правила, разработаные гуглом, по которым стоит строить пользовательский интерфес: какие использовать шрифты, отступы, цвета, элементы, ...
Разработчики JetPack Compose создали много элементов, которые соответствуют этим правилам.
Один из таких элементов - Button - внутри неё могут быть другие элементы. Такой подход называется SlotAPI - вместо того, чтобы жёстко указать, что кнопка будет принимать текст, разработчики создали контейнер, внутри которого можно помещать другие элементы. Причём в конце composable функции элемента UI  мы можем принимать не одну composable функцию(которая будет содержать другие функции), а множество(например как в AlertDialog).

Когда мы пишем

```
@Composable
inline fun Column(
    modifier: Modifier = Modifier,
    verticalArrangement: Arrangement.Vertical = Arrangement.Top,
    horizontalAlignment: Alignment.Horizontal = Alignment.Start,
    content: @Composable ColumnScope.() -> Unit
) {
```

это значит что последний параметр будет экстеншеном на ColumnScope, т.е. она его расширяет. Поэтому в качестве объекта this в лямбде будет выступать интерфейс ColumnScope, а знчит мы можем вызывать методы, которые есть в этом интерфейсе, например, Modifier.weight(), котороый нет, например, в Text() (если текст не находится в колом, то у нас будет ошибка). 

Так можно делать, если нам нужно, например, вызывать функции только лишь внутри какого-то другого элемента. Например:

@Composable
private fun TestText(
    count: Int,
    text: String,
) {
    repeat(count) {
        Text(
            modifier = Modifier.weight(1f),
            text = text,
        )
    }
}

Это просто функция, у которой нет атрибута weight - это значит, что будет ошибка. Но  мы можем сказать, что её можно вызываться только внутри другого элемента, объявив её расширением:

@Composable
private fun ColumnScope.TestText(...) {...}

Т.е. вызывать её мы можем вызывть внутри ColumnScope, т.е. внутри Column.

У элемента Button будет скоуп this: RowScope, т.е. элементы внутри него будут располагаться подряд, как в строке. Потому, что функция Button принимает в конце следующее:

@Composable
fun Button(
...
content: @Composable RowScope.() -> Unit
) {
    ...
}

--------------------
Примеры простых элементов:

Лайтовая обведённая кнопка: 

OutlinedButton(
    onClick = {
        // действие при клике.
    }
) {  // то, что видим на кнопке
    Text(
        text = "Hello"
    )
}

Текстовое поле с лейбом:

TextField(
    value = "Value",
    onValueChange = {
        // действие при изменениии
    },
    label = {
        Text(text = "label name")
    }
)

Деалоговое окно:

AlertDialog(
    onDismissRequest = {},
    conffirmButton = {
        Text(modifier = Modifier.padding(8.dp), text = "Yes")
    }
    },
    title = {
        Text(text = "Are you sure?")
    },
    text = {
        Text(text = "Do you want to delete this file?")
    },
    dismissButton = {
        Text(modifier = Modifier.padding(8.dp), text = "No")
    }
)

--------------------

Scaffold - composable функция - элемент пользовательского интерфейса, который, по сути, представляет весь экран. Она принимает очень много параметров, в том числе:
* topBar - верхняя часть экрана
* bottomBar - нижняя панель навигации на экране  
* floatingActionButton - круглая кнопка, по которой обычно происходит какое-то действие, например, добавление нового элемента
* drawerContent - боковое меню -  функция, которая является расширением для ColumnScope
* content - сам контент, который находится на экране. Принимает PaddingValues - отступы
Функция, как и многие другие, построена по принципу SlotAPI - она предоставляет очень много слотов, в которые мы можем вставлять свои composable функции.

Scaffold(
        topBar = {
            TopAppBar(
                title = {
                    Text(
                        "Simple TopAppBar",
                        maxLines = 1,
                        overflow = TextOverflow.Ellipsis
                    )
                },
                navigationIcon = {  
                    IconButton(onClick = { /* doSomething() */ }) {
                        Icon(
                            imageVector = Icons.Filled.Menu,
                            contentDescription = "Localized description"
                        )
                    }
                },
            )
        },
    )
}

В TopAppBar мы с помощью:
* title - устанавливаем заголовок
* navigationIcon - устанавливаем иконку 

IconButton - обычная иконка, но на которую можно установить колбэк функцию для обработки нажания.
В Icons находятся иконке, которые уже встроены в Compose Material

Для создания нижнего бара:

fun BottomNavigationSample() {
    var selectedItem by remember { mutableStateOf(0) }
    val items = listOf("Songs", "Artists", "Playlists")

    NavigationBar() {
        items.forEachIndexed { index, item ->
            NavigationBarItem(
                icon = {
                    Icon(it.icon, contentDescription = null)
                },
                label = {
                    Text(text = stringResource(id = it.titleResId))
                },
                onClick = {},
                selected = false,
                colors = NavigationBarItemDefaults.colors(
                            unselectedIconColor = MaterialTheme.colorScheme.onSecondary,
                            unselectedTextColor = Color.Gray,
                            selectedIconColor = MaterialTheme.colorScheme.onPrimary,
                            selectedTextColor = Color.White
                        )
            )
        }
    }
}

Где
* items - отображаемые строки
* NavigationBar - установить для каждого элемента коллекции иконку, лэйбл, колбэк, значение - отмечен ли элемент. 
NavigationBar предоставляет RowScope



----------------------
Для версии material 3 

ModalNavigationDrawer(
    drawerContent = {
        ModalDrawerSheet {
            Text("Drawer title", modifier = Modifier.padding(16.dp))
            Divider()
            NavigationDrawerItem(
                label = { Text(text = "Drawer Item") },
                selected = false,
                onClick = { /*TODO*/ }
            )
            // ...other drawer items
        }
    }
) {
    Scaffold(
        topBar = {
            TopAppBar(
                title = {
                    Text(text = "title")
                },
                navigationIcon = {
                    IconButton(onClick = {}) {
                        Icon(
                            imageVector = Icons.Filled.Menu,
                            contentDescription = null
                        )
                    }
                }
            )
        },
        bottomBar = {
            BottomAppBar {
                IconButton(onClick = { }) {
                    Icon(Icons.Filled.Menu, contentDescription = "Меню")
                }
                Spacer(Modifier.weight(1f, true))

                IconButton(onClick = { }) {
                    Icon(Icons.Filled.Info, contentDescription = "Информация о приложении")
                }
                IconButton(
                    onClick = { },
                    enabled = false
                ) {
                    Icon(
                        Icons.Filled.Search,
                        contentDescription = "Избранное"
                    )
                }
            }

        }
    ) {
        Text(text = "Hello", modifier = Modifier.padding(it))
    }
}

----------------------

sealed класс - это обычный класс, наследники которого должны быть объявленны внутри этого же файла/класса или, с определённой версии котлин, внутри одного пакета. Очень удобно создать класс и внутри объявить всех его наследников.

Отрисока элементов происходит в момент вызова отрисовки функции. Т.о., осли мы меняем значения переменной из колбэка в UI composable элементе, перерисовка не проиходит, потому что эти функции больше никто не вызывает.

Рекомпозиция - все функции в compose могут быть вызваны множество раз, в зависимости от различных состояний: какое-то состояние экрана изменилось -> его нужно перерисовать -> функции compose вызываются снова. Для этого нужно, чтобы эти compose функции зависили от состояния экрана.

Чтобы создать какой-то стейт, можно вызвать функцию mutableStateOf(value)

@Composable
... {
    val selectedItem = mutableStateOf(0)

    ...
    selectedItem.value == 0
    on click = {
        selectedItem.value = index
    }
}

Если внутри composable функции есть стейт, то функция от него зависит - если стейт меняется, то функция вызвается снова, т.о. происходит рекомпозиция. Но у нас всегда вызвается функция

val selectedItem = mutableStateOf(0)

, поэтому стейт по сути не меняется, хотя перед этим в selectedItem.value при клике было положено значение. Т.о. новое значение не переживает рекомпозицию. 
А если мы кликаем на то значение, при котором в стейт записывается уже имеющееся там значение, рекомпозиция не проиходит.

Чтобы исправить это, мы записываем следующее:

val selectedItem = remember {
    mutableStateOf(0)
}

этот стейт будет установлен единожды, а при клике будет устанавливаться новое значение. При использовании функции remember стейт будет сохраняться между рекомпозициями.

Для получения строки из strings можно использовать следующую функцию:

Text(text = stringResource(id = it.titleResId))

-------------------------------------

Кнопка для добавления или ещё чего

floatingActionButton = {
    FloatingActionButton(onClick = {}) {
        Icon(
            painter...
        )
    }
}

Если мы хотим, чтобы при клике на эту кнопку появлялся SnackBar(что-то вроде алёрт бар), мы не сможем вызывать её в onClick(), т.к. мы не везде могжем вызывать composable функции.
Вместо этого мы сразу в элементе Scaffold должны объявить SnackBar - он будет реагировать на определённый стейт(состояние SnackBar - например, видимый или нет). При клике на кнопку, мы будем менять состояние и этот SnackBar будет отобраажаться.

Параметр Scaffold - snackbarHost - предоставляет место для снекбара.

val snackbarHostState = SnackbarHostState()
Scaffold(
    snackbarHost = {
        SnackbarHost(
            hostState = snackbarHostState 
        )
    },

    floatingActionButton = {
    FloatingActionButton(onClick = {
        snackbarHostState.showSnackbar(
            "This is snack bar!"
        )

    }) {
        Icon(
            painter...
        )
    }
}


)

Но так сделать не получиться, т.к. функция showSnackbar является suspend функцией. А знаачит она доложна быть вызвана из другой саспенд функции. Для Compose у нас есть специальный скоуп, который мы можем для этого использовать:

val scope = rememberCoroutineScope()

это специальный скоуп, который был сделан для работы в Compose:

...
FloatingActionButton(onClick = {
    scope.launch {
        snackbarHostState.showSnackbar(
            "This is snack bar!"
        )
    }
...

У showSnackbar есть перегрузка, которую можно использовать так:

...
FloatingActionButton(onClick = {
    scope.launch {
        snackbarHostState.showSnackbar(
            message = "This is snack bar!",
            actionLabel = "Hide FAB",
            duration = SnackbarDuration.Long,
        )
    }
...

благодаря этому мы можем на баре разместить кнопку с текстом, на который можно кликать, и увеличить длительность появления бара. При клике происходит действие по умолчанию - скрывается снекбар.

У снекбара возвращаемый тип - SnackbarResult, у которого два параметра: Dismissed - снекбар просто исчез(время отображения вышло), ActionPerformed - мы нажали не кнопку(после чего он исчез). Это значение можно сохранить в переменную, а уже после этого проверить её и выполнить какое-то действие. Например, можно убрать эту floating кнопку. Но как это сделать? С помощью рекомпозиции - можно добавить какой-нибудь стейт и связать его с этой кнопкой. Если мы не хотим, чтобы кнопка отрисовывалась на экране, мы можем её функцию просто не вызывать:

val fabIsVisible = remember {
    mutableStateOf(true);
}
Scaffold(
    floatingActionButton = {
        if (fabIsVisible.value) {
            FloatingActionButton(
                onClick = {
                    scope.launch {
                        val action = snackbarHostState.showSnackbar(
                            message = "This is snack bar!",
                            actionLabel = "Hide FAB",
                            duration = SnackbarDuration.Long,
                        )
                        if (action == SnackbarResult.ActionPerformed) {
                            fabIsVisible.value = false
                        }
                    }
                }
            }
            ...

Очень важно!: если меняется состояние snackbarHostState, то происходит перекомпозиция только лишь тех функций, которые зависят от этого состояния - в данном случае Scaffold. Для fabIsVisible - floatingActionButton.
