
---------------------
Навигация
---------------------

Нам нужно, чтобы при переходе на другую вкладку у нас отображалось другое содержимое, т.е. вызывалась другая Composable функция внутри контента - content параметр - Scaffold. А значит внутри контента нужно определять, какая вкладка сейчас выбрана, т.е. нам нужен стейт, который можно прямо определить в BottomNavigation.

Определим livedata для текущего элемента навигации:

private val _selectedNavItem = MutableLiveData<NavigationItem>(NavigationItem.Home)
val selectedNavItem: LiveData<NavigationItem> = _selectedNavItem

fun selectNavItem(item: navigationitem) {
    _selectednavItem.value = item
}

Теперь при клике на определённый item будем:

@Composable
fun MainScreen(viewModel: MainViewModel) {
    val selectedNavItem by viewModel.selectedNavItem.observeAsState(NavigationItem.Home)
        ...
        BottomNavigationItem(
            selected = selectedNavItem == item
            onClick = {
                viewModel.selectNavItem(item) 
            }
        )
        ...
    ) {
        when(selectedNavItem) {
            NavigationItem.Home -> {
                HomeScreen(...)
            }
            NavigationItem.Favourite -> Text(text = "Favourite")
            NavigationItem.Profile -> Text(text = "Profile")
        }

    }

Теперь при клике на табы, у нас будут вызываться соответствующие функции. но с таким подходом к навигации есть несколько проблем:
1) При переходе по табам Composable функции не складываются в бэкстек(при переходе с одного экрана на другой - если кликнуть на кнопку назад, то должен открыться предыдущий экран. В нашем случае этого не происходит и приложение закрывается)
2) При переходе по разным табам у нас происходи не рекомпозиция - все функции вызываются заново. А если у этих функций внутри есть состояние - оно не будет сохраняться

Чтобы всё это решить, нужно написать очень много кода.

----------------------------------

Есть готовое решение от гугл - JetPack Compose Navigation:

implementation("androidx.navigation:navigation-compose:2.5.2")

Мы объекту навигации будем передавать путь - куда нужно перейти - это просто строка с названием экрана.

Обычно, создают отдельный класс Screen, наследники которого представляют собой экраны со своими названиями, которые нужны для навигации:

sealed class Screen(
    val route: String
) {
    object NewsFeed : Screen(ROUTE_NEWS_FEED)
    object Favourite : Screen(ROUTE_FAVOURITE)
    object Profile : Screen(ROUTE_PROFILE)
    
    private companion object {
        const val ROUTE_NEWS_FEED = "news_feed"
        const val ROUTE_FAVOURITE = "favourite"
        const val ROUTE_PROFILE = "profile"
    }
}

Навигация - это граф, который описывает все возможные переходы. Для его  создания делаем так - вызываем NavHost(у которой есть несколько перегрузок):

@Composable
fun AppNavGraph(
    navHostController: NavHostController
) {
    NavHost(
        navController = navHostController,
        startDestination = Screen.NewsFeed.route,

    )
}

* startDestination - экран, который будет открываться первым
* builder - здесь будем строить весь граф приложения: направления и переходы, которые будут присутствовать в графе.

Для создания нового направления используется функция composable(), которая внутри вызывает addDestination(). Она принимает:
* route - String - название экрана, на который нужно перейти
* arguments - List<NamedNavArgumnet> - параметры, которые мы мжоем передать экрану при открытии
* deepLinks - List<NavDeepLinks> - диплинки??? что это???
* content - контент, который мы будем отображать при переходе на данный экран

NavHost(...) {
    composable(
        Screen.NewsFeed.route
    ) {


    }
}

Внутри можно было бы вызывать функцию экрана, но так делать не рекомендуется. Вместо этого лучше передать колбэк функции, которые уже и будут вызываться:

fun AppNavGraph(
    ...
    homeScreenContent: @Composable () -> Unit,
    favouriteScreenContent: @Composable () -> Unit,
    profileScreenContent: @Composable () -> Unit
) {
    ...
    composable(
        Screen.NewsFeed.route
    ) {
        homeScreenContent()
    }

    composable(
        Screen.Favourite.route
    ) {
        favouriteScreenContent()
    }
    ...

}

Наш граф готов. Теперь его можно использовать на главном экране. Объект navHostController будет хранить стейт навигации - мы у него будем вызывать navigate, передавать имя экрана, на который хотим перейти, и он будет делать всю работу:

val navHostController = rememberNavController()
...
) { paddingValues ->
    AppNavGraph(
        navHostController = navHostController,
        homeScreenContent = { HomeScreen(viewModel = ...) },
        favouriteScreenContent = { TextCounter(name = ...) },
        profileScreenContent = { TextCounter(name = ...) },
    )
}

Но сейчас BottomNavigation ничего не знает про наш AppNavGraph и элементы, которые мы в нём отображаем также никак не связаны с нашей навигацией:

BottomNavigationItem(
    ...
    onClick = {
        navHostController.navigate(item.screen)
    }
)

Нам нужен стейт, в котором будет храниться текущий открытый экран. И на изменения в этом стейте мы будем реагировать и подсвечивать кнопку открывшегося экрана. Такой стейт можно получить у navHostController'а:

BottomNavigation {
    val navBackStackEntry by navHostController.currentBackStackEntryAsState()
    currentRoute = navBackStackEntry?.destination?.route
    ...
    BottomNavigationitem(
        ...
        selected = currentRoute = item.screen.route
    )

}

Если экран меняется, то будет меняться стейт, а значит будет происходить рекомпозиция.

Теперь у нас работает Бэкстек. Но при переходе назад, мы не создавали компоузэбл функцию - она была взята из стека, но при этом её состояние сбросилось. Когда мы возвращаемся назад на экран, то функция дожна быть вызвана заново, и это та же компоузэбл функция, которая уже вызывалась ранее, а значит должна была произойти рекомпозиция и состояние должно было сохраниться(мы использовали remember). Дело в том, что когда функция помещается в бэкстек - она умирает(как будто мы переворачиваем устройство). Значит нам нужно восстанавливать её состояние. Это можно  сделать с помощью rememberSaveable:

fun TextCounter(name: STring) {
    var count by rememberSaveable {
        mutableStateOf(0) 
    }
}

Ещё проблема: если много раз кликнуть на таб, то множество функций будет лежать в бэкстеке и при клике назад, мы будем их все по очереди пересоздавать. Это можно решить добавив 
