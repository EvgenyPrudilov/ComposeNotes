-----------------
Основы JetPack Compose
-----------------

При использования Compose мы не используем view элементы и макеты, поэтому в ресурсах нет директории layout. Абсолютно весь UI строиться через composable функции. Корень для вызова всех composable функций - setContent():

setContent {
	Text(text = "Hello, android!")
}

public fun ComponentActivity.setContent(
    parent: CompositionContext? = null,
    content: @Composable () -> Unit
) {
  ...
}

параметр parent - может отсутствовать, тогда будет null
параметр content - функция, помеченная аннотацией @Composable - значит мы можем передать лямбду в конце, которая автоматически станет composable функцией.

Если мы помечаем какую-либо функцию аннотацией @Composable, то компилятор понимает, что она работает в отрисовке. Для отрисовки каждого элеманта интерфейса мы всегда используем соответствующие функции, например, Text().

Объвленные composable функции - это обычные функции, помеченные аннотацией @Composable. Важно, что имена этих функций мы пишем с большой буквы: обычные функции мы начинаем с маленькой буквы, когда они выполняют какие-то действия, а функции в compose описывают какое-либо состояние части экрана(поэтому разработчики решили, что их стоит выделить заглавной буквой). В остальном - это обычные функции, в которых мы можем объявлять переменные, использовать циклы, ...

Мы не можем вызывать такие функции просто так - они могут вызываться только внутри других composable функций, иначе - будет ошибка.

Мы можем помечать функцию аннотацией @Preview, чтобы она отображалась итеративно в студии. Но в таком случае не должно быть никаких параметров. Если нам нужны параметры, то можно создать другую функцию без параметров и вызвать в ней нашу изначальную фукнцию с параметрами.

Элемент Column является аналогом LinearLayout с вертикальной ориентацией.

Column {
    Text(
        text = "Hello ", color = Color.Green
    )
    Text(
        text = "Good ", color = Color.Blue
    )
}

Всё, что вызывается внутри будет расположено друг под другом. Можно, например, использовать циклы:

Column {
    repeat (10) {
        Text(
            text = "Hello ", color = Color.Green
        )
    }

}

Элемент Row является аналогом LinearLayout с горизонтальной ориентацией.

Одним из параметром многих элементов является объект типа Modifier - используется, чтобы модифицировать элементы каким-то образом: добавить отступы, установить высоту/ширину, ... Параметр не обязательный. 
Modifier строиться через шаблон Builder, т.е. мы можем через точку все желаемые атрибуты:

Column(
    modifier = Modifier
      .background(color = Color.White)
      .padding(all = 16.dp)
) {
    ...
}

Все элементы по умолчанию все элементы занимают столько места, сколько нужно для отображения их содержимого, т.е. wrap_content.
Модификаторы fillMaxHeight и fillMaxWidth являются аналогами для match_parent.
Модификатор fillMaxSize - занять всё пространство и по высоте, и по ширине.

Элемент Box используется, когда нужен какой-то контейнет(который можно как-то отформатировать) для другого элемента.

Если атрибуты модификатора width и height одинаковы, то можно использовать один атрибук size с нужным параметром.
У Row есть параметр horizontalArrangement, который можно установить в Arrangement.SpaceEvenly, чтобы между всеми элементами было одинаковое растояние(и даже с краю), SpaceBetween, чтобы только между элементами было одинаковое расстояние, SpaceAround - между элементами одинаковое расстояние, а с боков растояние = половите того, что между элементами.
То же у Column, только verticalArrangement.
Есть атрибут padding - в Compose нет margin - указывает отступы.

Если часто встречается одна и таже структура элементов, то её можно вынести в отдельную функцию(ей можно присвоить модификатор доступа private, т.к. мы не будем использовать её из вне), а затем вызывать эту функцию столько, сколько нужно. 

Элемент Card, который является аналогом View Card из View системы андроид.
У него есть параметр Shape, у которого по умолчанию значение MaterialTheme.shapes.medium, которое равно RoundedCornerShape(4.dp) - он скругляет углы элемента. У него есть параметр shape, у которого не очень много параметров: RoundedCornerShape(8.dp) - закругляем по радиусу(все углам по 8 dp одинаково, но можно и всем по отдельности задарь радиус), RectangleShape - просто прямоугольная форма(это объект - скобки не нужны), CircleShape - круглая форма.
Чтобы сделать скруглённые углы только сверху, можно использовать следующее:

shape = RoundedCornerShape(4.dp).copy(
    bottomEnd = CornerSize(0.dp),
    bottomStart = CornerSize(0.dp)
) 

или

shape = RoundedCornerShape(
    topEnd = 4.dp,
    topStart = 4.dp
)

Есть у него параметр border, которая имеет тип BorderStroke, которая принимает ширину в dp и цвет.
Чтобы установить цвета нужно использовать 

colors = CardDefaults.cardColors(
    containerColor = Color.DarkGray, //Card background color
    contentColor = Color.White  //Card content color,e.g.text
)

При создании проекта автоматически создаётся файл темы с темой по умолчанию:

InstagramTheme - это функция, которая принимает три параметра:
 * darkTheme - используется isSystemInDarkTheme(), которая автоматически вычисляет, используется ли сейчас на устройстве тёмная тема.
 * content - это composable функция
 * dynamicColor - пока не знаю, что это

Если в setCompose вызовем InstagramTheme, а уже из него будем рисовать весь наш интерфес, то мы сможем использовать цвета из этой темы. Мы туда передаём content, colors, ... через MaterialTheme объект. Благодаря этому мы сможем использовать палитры для тёмной и светлой паллитры через этот самый объект: MaterialTheme.colorScheme.background.
Так есть, например, цвет backgroung - фон, а есть onBackgroung - цвет поверх фона

полезно делать так для сравнения влияния тем:

@Preview
@Composable
private fun PreviewCardLight() {
    InstagramTheme(
        darkTheme = false
    ) {
        InstagramProfileCard()
    }
}

@Preview
@Composable
private fun PreviewCardDard() {
    InstagramTheme(
        darkTheme = true
    ) {
        InstagramProfileCard()
    }
}

Если мы видим, что что-то в элементах интерфейса, например, Text, в параметрах стоит по умолчанию и = Unspecified(например, Color.Unspecified), то это значит, что значение наследуется от родительской composable функции.
параметр fontSize - размер шрифта( в единицах sp)
параметр fontWeight - сделать текст жирным
параметр fontFamily - семейство шрифтов
параметр textDecoration - подчёркивание, зачёркивание, комбинированное

Если нам нужно по разному оформлять отрывки в тексте, то можно использовать перегруженную функцию @composable Text, которая принимает annotatedString (через buildAnnotatedString), в которой можно использовать функцию append("...") для добавления текста. Внутри annotatedString мы долны использовать 

withStyle(SpanStyle(param = value, param2 - value2, ...)) {
	текст со стилем
}

Например

Text(
    buildAnnotatedString {
        withStyle(SpanStyle(fontWeight = FontWeight.Bold)) {
            append("Hello")
        }
        withStyle(SpanStyle(textDecoration = TextDecoration.Underline)) {
            append(" ")
        }
        withStyle(SpanStyle(fontSize = 30.sp)) {
            append("world")
            append("!")
        }

    }
)

Для работы с картинками, нужна Composable функция Image, у которой есть несколько перегрузок:
1) pointer: Pointer - отображение как растровых, так и векторных картинок
2) imageVector: ImageVector - только для векторных изображений
3) bitmap: ImageBitmap - только для битмап

contentDescription - параметр нужен для описания функции для слепых людей

Image(
    imageVector = Icons.Rounded.MoreVert, // стандартные иконки
    contentDescription = null
)

Image(
    modifier = Modifier
        .fillMaxSize(),
    painter = painterResource(id = R.drawable.ic_launcher_background),
    contentDescription = ""
)

contentScale - по умолчанию принимает  ContentScale.Fit - масштабирования нет, отображаем картинку так, как она есть на самом деле. FillBounds - заполнить по всему экрану, FillWidth - заполнить весь экран по ширине, а по высоте - сколько нужно, FillHeight - заполнить весь экран по высоте, а по ширине - сколько нужно. При этом картинка по необходимости растягиевается по всему экрану, а остальное обрезается.

У модификатора есть атрибут clip, которой мы можем передать параметр типа Shape, например CircleShape - картинка будет обрезана по кругу.

Image(
    modifier = Modifier
        .fillMaxSize()
        .clip(CircleShape),
    painter = painterResource(id = R.drawable.ic_launcher_background),
    contentDescription = "",
    //contentScale = ContentScale.FillHeight
)

Использование растровых изображений нежелательно в андроид разработке, т.к. устройств очень много, у них у всех разные дисплеи. Поэтому картинка будет нормальной на одном экране, а на другом растянуто, и из-за этого приходится в один проект добавлять много одинаковых картинок для разных экранов.
Поэтому удобнее использовать векторных изображения, которые поставляются в формате svg.

Последовательность модификаторов важна, т.к. они выполняются в том порядке, в котором мы их указали.

У кнопки Button есть два обязательных параметра - лямбда функция с действием, которое должно быть выполнено при клике, и лямбду в внутренними элементами интерфейса.

есть специальный элемент Spacer, который нужен для того, что заполнять какие-то пространства. Это как бы ещё один элемент, который вставляется между другими:

Spacer(
    modifier = Modifier
        .width(8.dp)
)

Есть такой элемент Surface, который используется для задания фона. И внутри которого мы располагаем все наши элементы интерфейса. Под капотом он использует Box

Surface(
    modifier = Modifier.fillMaxSize(),
    color = MaterialTheme.colorScheme.background
) {
    ...
}

В файле Color.kt мы указываем используемые цвета. 

При установке цветов в теме у нас может быть конфликт имёт. В таком случае можно установить аннотацию @SuppressLint("ConflictingOnColor"):

@SuppressLint("ConflictingOnColor")
private val DarkColorScheme = darkColorScheme(
    primary = Black900,
    secondary = Black900,
    onPrimary = Color.White,
    onSecondary = Black500
)

Когда мы работаем с векторными иконками удобно вместо функции Image использовать Icon - у нё есть атрибут tint, который позволяет указать цвет для данной иконки.

Все строки удобно вынасить в файл строк. Поместим курсор на строке, нажмём alt+enter -> extract string resource.

Можно растянуть картинку с помощью параметра
contentScale = ContentScale.FillWidth