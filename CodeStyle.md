Использование всеми разработчиками команды одного code style не менее важно, чем единая точка зрения на архитектуру приложения и ответственности разных объектов. Наличие соглашений по работе с VIPER-стеком не исключение.

### Общие правила для модуля

- Название модуля должно полностью отражать его назначение. Суффикс *Module* в название включать не следует. 
  
  **Пример:** `MessageFolder`, `PostList`, `CacheSettings`.
  
- Все элементы модуля разбиты по подпапкам в рамках одной папки модуля. 

  **Пример:**
  
  ```
  /NewPostUserStory
      /NewPostModule
          /Assembly
          /Interactor
          /Presenter
          /Router
          /View
      /ChooseAvatarModule
          /Assembly
          /Interactor
          /Presenter
          /Router
          /View
  ```
  
- Если по итогу написания модуля какие-то из его элементов остались неиспользованными, будь то классы, протоколы или методы, они удаляются.
- Все хелперы располагаются в подпапке своего слоя. 

  **Пример:**

  ```
/Interactor
      /UserInputValidator
        UserInputValidator.h
        UserInputValidator.m
      /PlainObjectMapper
        PlainObjectMapper.h
        PlainObjectMapperImplementation.h
        PlainObjectMapperImplementation.m
  ```
- Все методы, с помощью которых слои общаются друг с другом, должны быть синхронными.

  **Пример:**
  
  ```objc
  @interface InteractorInput
  - (void)obtainDataFromNetwork;
  ...
  
  @interface InteractorOutput
  - (void)didObtainDataFromNetwork:(NSArray *)data;
  ...
  ```
  
- Все методы протоколов, которыми закрыты элементы модуля, должны начинаться с глаголов - это помогает явно указать на то, что каждый из компонентов обладает поведением, а не состоянием.
- Методы, обозначающие начало действия, должны начинаться с глаголов, выражающих приказ или просьбу (глаголов повелительного наклонения).
- Методы, обозначающие завершение действия или процесса, констатацию факта должны начинаться с глагола прошедшего времени.


  **Пример:**
  
  ```objc
  - (void)obtainImageForPostId:(NSString *)postId;
  - (void)processUserInput:(NSString *)userInput;
  - (void)invalidateCurrentCache;
  ```
  
- В публичных интерфейсах всех классов и протоколов стараемся использовать forward-declaration, используя `#import` лишь при необходимости.

  **Пример:**
  
  ```objc
  #import <Foundation/Foundation.h>

  #import "PostListViewOutput.h"
  #import "PostListModuleInput.h"
  #import "PostListInteractorOutput.h"

  @protocol PostListViewInput;
  @protocol PostListRouterInput;
  @protocol PostListInteractorInput;
  @class PostListViewModelMapper;

  @interface PostListPresenter : NSObject <PostListModuleInput, PostListViewOutput, PostListInteractorOutput>

  @property (nonatomic, weak) id<PostListViewInput> view;
  @property (nonatomic, strong) id<PostListRouterInput> router;
  @property (nonatomic, strong) id<PostListInteractorInput> interactor;
  @property (nonatomic, strong) PostListViewModelMapper *postListViewModelMapper;

  @end
  ```

-----

### Слой Interactor
#### Класс `Interactor`

##### Наименование
`<ModuleName>Interactor.h / <ModuleName>Interactor.m`

##### Дополнительные правила

- Интерактор не держит состояния, только зависимости, расположенные в его открытом интерфейсе.

  **Пример:**
  
  ```objc
  @interface PostListInteractor : NSObject <PostListInteractorInput>

  @property (nonatomic, weak) id<PostListInteractorOutput> output;
  @property (nonatomic, strong) id<AccountService> accountService;

  @end
  ```
  
- Интерактор держит weak-ссылку на презентер. Переменная называется `output`.

  **Пример:** 
  
  `@property (nonatomic, weak) id<PostListInteractorOutput> output;`

#### Фасады над сервисами
##### Наименование
`<Feature>Facade.h / <Feature>Facade.m` 

##### Описание
В случае, если в интеракторах нескольких модулей есть повторяющаяся логика по использованию сервисов определенным образом, она инкапсулируется в отдельный фасад над сервисами. В качестве примера можно привести объект, реализующий логику пагинации разных лент постов - он умеет запрашивать список новых элементов, сранивать их с ранее закешированными, вычислять смещения и дырки - и многое другое. Благодаря выделению этих связей в отдельную сущность, пагинация может быть достаточно легко подключена для любого модуля списка элементов.

**Пример:** `PagingFacade`.

#### Протокол `<InteractorInput>`
##### Наименование
`<ModuleName>InteractorInput.h`

##### Описание
Содержит методы для общения с интерактором. Этим протоколом закрыт интерактор с точки зрения презентера.

##### Примеры методов

```objc
- (NSArray *)obtainNewsFromCache;
- (void)obtainMessageWithId:(NSString *)messageId;
- (void)performLoginWithUsername:(NSString *)username password:(NSString *)password;
```

##### Общие паттерны методов

- Если в данном модуле нам нужно самостоятельно решать, в какой момент просить данные из кэша, а в какой - из сети, допустимо явно указывать это интерактору (`obtainFromNetwork/-fromCache`).

#### Протокол `<InteractorOutput>`
##### Наименование
`<ModuleName>InteractorOutput.h`

##### Описание
Содержит методы, при помощи которых интерактор общается с вышестоящим слоем модуля. Этим протоколом обычно закрыт презентер.

##### Примеры методов

```objc
- (void)didObtainMessage:(Message *)message;
- (void)didPerformLoginWithSuccess;
```

##### Общие паттерны методов:

- В большинстве случаев в качестве префикса каждого метода используем `did` - это указывает на пассивную роль интерактора, который умеет выполнять ряд действий по запросу и уведомлять об их окончании.
- В случае отсутствия единой системы обработки ошибок заводятся пары методов (`didObtainWithSuccess/-withFailure`).

### Слой Presenter
#### Класс `Presenter`
##### Наименование
`<ModuleName>Presenter.h / <ModuleName>Presenter.m`

##### Дополнительные правила

- В отличие от всех остальных элементов, презентер обладает состоянием. Оно находится в приватном extension.
- Презентер держит weak-ссылку на view. Переменная называется `view`. 

  **Пример:**
  
  `@property (nonatomic, weak) id<PostListViewInput> view;`
  
- Презентер держит strong-ссылку на роутер. Переменная называется `router`.

  **Пример:**
  
  `@property (nonatomic, strong) id<PostListRouterInput> router;`
  
- Презентер держит strong-ссылку на интерактор. Переменная называется `interactor`.

  **Пример:**
  
  `@property (nonatomic, strong) id<PostListInteractorInput> interactor;`
  
- Если презентеру нужно держать объект, реализующий протокол `ModuleInput` дочернего модуля, переменная называется `<OtherModuleName>ModuleInput`.

#### Класс `State`
##### Наименование
`<ModuleName>State.h / <ModuleName>State.m`

##### Описание
В том случае, если текущий модуль обладает каким-либо состоянием, его можно выделить в отдельный объект, не обладающий никаким поведением и выступающий простым хранилищем данных.

**Пример:**

```objc
@interface PostListState

@property (nonatomic, assign) FeedType feedType;
@property (nonatomic, assign) BOOL hasHeader;
@property (nonatomic, strong) NSString *feedId;

@end
```

#### Протокол `<ModuleInput>`
##### Наименование
`<ModuleName>ModuleInput.h`

##### Описание
Содержит методы, при помощи которых с модулем могут общаться другие модули или его контейнер.

##### Дополнительные правила
- При использовании библиотеки ViperMcFlurry наследуется от протокола `<RamblerViperModuleInput>`.

##### Примеры методов

```objc
- (void)configureWithPostId:(NSString *)postId;
- (void)updateContentInset:(CGFloat)contentInset;
```

#### Протокол `<ModuleOutput>`
##### Наименование
`<ModuleName>ModuleOutput.h`

##### Описание
Содержит методы, при помощи которых модуль общается со своим контейнером или другими модулями.

##### Дополнительные правила
- При использовании библиотеки ViperMcFlurry наследуется от протокола `<RamblerViperModuleOutput>`.

##### Примеры методов

```objc
- (void)didSelectMenuItem:(NSString *)menuItem;
- (void)didPerformLoginWithSuccess;
```

### Слой Router
#### Класс `Router`
##### Наименование
`<ModuleName>Router.h / <ModuleName>Router.m`

##### Дополнительные правила
- При использовании библиотеки ViperMcFlurry держит weak-ссылку на `ViewController`, отвечающий за переходы этого модуля. Ссылка представляет собой свойство, закрытое протоколом `<RamblerViperModuleTransitionHandlerProtocol>`. Обычно эта переменная называется `transitionHandler`

#### Класс `Route`
##### Наименование
`<ModuleName>Route.h / <ModuleName>Route.m`

##### Описание
Если несколько роутеров в рамках одного приложения реализуют повторяющуюся логику по переходам на один экран - ее можно инкапсулировать в отдельном объекте-маршруте, который будет подключаться к нужным модулям. К примеру, это может пригодиться в случае экрана авторизации, на который можно перейти из разных модулей - настроек, профиля, бокового меню. Благодаря инкапсуляции этой логики в отдельном объекте, мы избавляемся от необходимости в роутере каждого из этих модулей писать один и тот же код.

##### Примеры методов

```objc
- (void)openAuthorizationModuleWithTransitionHandler:(id<RamblerViperModuleTransitionHandlerProtocol>)transitionHandler;
```

#### Протокол `<RouterInput>`
##### Наименование
`<ModuleName>RouterInput.h`

##### Описание
Содержит методы переходов на другие модули, которые могут быть вызваны презентером.

##### Примеры методов

```objc
- (void)openDetailNewsModuleWithNewsId:(NSString *)newsId
- (void)closeCurrentModule;
```

##### Общие паттерны методов

- Для консистентности все методы этого протокола начинаются либо на `open-` (открытие какого-либо модуля), `close-` (закрытие модуля) или `embed-` (встраивание дочернего модуля в контейнер).

### Слой View
#### Классы отображения (ViewController, View, Cell)
##### Наименование
`<ModuleName>View.h / <ModuleName>View.m`, `<ModuleName>ViewController.h / <ModuleName>ViewController.m`, `<ModuleName>Cell.h / <ModuleName>Cell.m`.

##### Дополнительные правила

- Все `IBOutlet`'ы и `IBAction`'ы (то есть все зависимости и интерфейс) View выносятся в его публичный интерфейс.
- View держит strong-ссылку на презентер. Переменная называется `output`.

  **Пример:**
  
  `@property (nonatomic, strong) id<PostListViewOutput> output;`

#### Класс DataDisplayManager
##### Наименование
`<ModuleName>DataDisplayManager.h / <ModuleName>DataDisplayManager.m`

##### Описание
Объект, закрывающий логику реализации `UITableViewDataSource` и `UITableViewDelegate`. Работает только с данными, ничего не знает о конкретных `UIView` экрана. Обычно протоколом не закрывается, потому что конкретному экрану чаще всего соответствует одна конкретная реализация DataDisplayManager.

##### Примеры методов

```objc
- (id<UITableViewDataSource>)dataSourceForTableView:(UITableView *)tableView;
- (id<UITableViewDelegate>)delegateForTableView:(UITableView *)tableView
                               withBaseDelegate:(id <UITableViewDelegate>)baseTableViewDelegate;
```

#### Класс CellObjectFactory
##### Наименование
`<ModuleName>CellObjectFactory.h / <ModuleName>CellObjectFactory.m`

##### Описание
Зачастую удобно бывает выносить логику по созданию моделей ячеек из DataDisplayManager'а в отдельный объект, который, по сути, преобразует обычные модели в CellObject'ы.

#### Протокол <ViewInput>
##### Наименование
`<ModuleName>ViewInput.h`

##### Описание
Содержит методы, при помощи которых презентер может управлять отображением или получать введенные пользователем данные.

##### Примеры методов

```objc
- (void)updateWithTitle:(NSString *)title;
- (NSString *)obtainCurrentUserInput;
```

#### Протокол <ViewOutput>
##### Наименование
`<ModuleName>ViewOutput.h`

##### Описание
Содержит методы, при помощи которых View уведомляет презентер об изменениях своего состояния.

##### Дополнительные правила
- В этом же протоколе находятся и методы, при помощи которых View уведомляет презентер о событиях своего жизненного цикла. 

##### Примеры методов

```objc
- (void)didTapLoginButton;
- (void)didModifyCurrentInput;
```

##### Общие паттерны методов
- В большинстве случаев в качестве префикса каждого метода используем `did` - это указывает на пассивную роль View, который умеет выполнять ряд действий по запросу и уведомлять об их окончании.

**Примеры:** 

```objc
- (void)didTriggerViewWillAppearEvent;
- (void)didTriggerMemoryWarningEvent;
```

### Слой Assembly
#### Класс Assembly
##### Наименование
`<ModuleName>Assembly.h / <ModuleName>Assembly.m`

##### Дополнительные правила

- В интерфейс Assembly выносится только метод, конфигурирующий View. Установка всего VIPER-стека - это детали реализации assembly, которые не должны быть известны окружающему миру.

##### Примеры методов

```objc

- (PostListViewController *)viewPostListModule;
- (PostListPresenter *)presenterPostListModule;

```

##### Общие паттерны методов

- Для более удобной автоподстановки у всех методов, создающих стандартные компоненты, указывается префикс `<ComponentName><ModuleName>Module`.

-----

### Комментарии

- Все методы протоколов и конкретных классов обязательно покрываются подробными javadoc-комментариями.

  
  **Пример:**
  
  ```objc
  @protocol PostListInteractorInput <NSObject>

  /**
   Метод возвращает модель поста по определенному индексу
 
   @param index Индекс поста в рамках просматриваемой категории
 
   @return Пост
   */
  - (PostModelObject *)obtainPostAtIndex:(NSUInteger)index;

  /**
   Метод возвращает общее количество постов для текущей ленты
 
   @return Количество постов
   */
  - (NSUInteger)obtainOverallPostCount;

  @end
  ```
  
- Комментарии пишутся к интерфейсам всех уникальных компонентов модуля (различные хелперы, ячейки).

  **Пример:**
  
  ```objc
  /**
   Ячейка, используемая для краткого отображения поста в списке
   */
  @interface PostListCell : UITableViewCell

  ...

  @end
  ```

- К интерфейсам всех неуникальных компонентов (интерактор, презентер, view), а также к их протоколам, пишется одинаковый комментарий, описывающий предназначение всего модуля.

  **Пример:**
  
  ```objc
  /**
   Модуль отвечает за отображение списка постов в любой из лент приложения.
   Используется как embedded-модуль.
   */
  @interface PostListAssembly : TyphoonAssembly
  ...
  @end
  ```

- В имплементации классов группы методов различных протоколов разбиваются при помощи `#pragma mark -`. 

  **Пример:**
  
  ```objc
  @implementation PostListPresenter

  #pragma mark - PostListModuleInput

  - (void)configureModuleWithPostListConfig:(PostListConfig *)config  {
       ...
  }

  #pragma mark - PostListViewOutput

  - (void)didTriggerPullToRefreshEvent {
       ...
  }

  #pragma mark - PostListInteractorOutput

  - (void)didProcessCacheTransaction:(CacheTransactionBatch *)transaction {
       ...
  }

  ```

-----

### Тесты

- Для каждого компонента модуля создается отдельный тест-кейс с названием вида `<ModuleName>ViewControllerTests.m`.
- Стараемся придерживаться правила один тест - одна проверка.
- Для разделения реализации теста на логические блоки используем нотацию *given/when/then*.
- Тесты методов различных протоколов разбиваются при помощи `#pragma mark -`.
- Так как в интерфейсе Assembly объявлены не все методы, для их проверки создается отдельный extension `_Testable.h`. В этом случае мы тестируем приватные методы, но такой подход обусловлен деталями реализации библиотеки *Typhoon*. В случае написания фабрики вручную, возможно проверить результаты работы компонента через его публичный интерфейс.
- Файловая структура модуля в тестах максимально повторяет файловую структуру проекта.
