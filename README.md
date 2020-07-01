# СЭД Tessa

Ниже происходит препарирование системы TESSA 3.4, основанное на личном опыте внедрения.

1. [Общее описание системы](#общее-описание-системы)
   * [Карточка](#карточка)
   * [Представление](#представление)
   * [Схема данных](#схема-данных)
   * [Дерево объектов](#дерево-объектов)
   * [Работа с БД](#работа-с-бд)
   * [Отчеты](#отчеты)
   * [Создание карточки](#создание-карточки)
   * [Публикация](#публикация)
2. [Критика](#критика)
   * [TypeSafety](#type-safety)
3. [Вместо резюме](#резюме)

## Общее описание системы

**TESSA** - система электронного документооборота (**СЭД**) с возможностью автоматизации бизнес процессов (**BPM**). Построена на **.NET**.
Программирование расширений осуществляется в **VisualStudio 2017**
Разработка **GUI** может осуществляться как на основе **WPF**, так и **React.js**

У страницы системы есть два состояния: **Карточка** и **Представление**. **Представление** - настраиваемый список, например, список карточек.
Настройка списков (**Представлений**) гибко настраивается так называемыми метаданными - текстовое описание, а также шаблонизируемым **SQL** запросом.


## **Карточка**:

![Image of Yaktocat](https://github.com/1001011000101101/Tessa/blob/master/images/Card.png)

## **Представление**:

Представление (view) имеет похожий смысл, который имеет вьюшка в мире SQL. Грубо говоря, это просто табличка.

![Image of Yaktocat](https://github.com/1001011000101101/Tessa/blob/master/images/View.png)

В конструкторе карточек можно мышкой добавить контролы:

![Image of Yaktocat](https://github.com/1001011000101101/Tessa/blob/master/images/CardControls.png)


## Схема данных:

В системе есть wrapper для СУБД, через который можно создавать свои объекты БД:

![Image of Yaktocat](https://github.com/1001011000101101/Tessa/blob/master/images/DatabaseGuiWrapper.png)

## Дерево объектов:

Дерево объектов может состоять из трех элементов: Папка, Представление и так называемый Сабсет - подмножество (выборка) из родительского Представления.
Папка служит для группировки элементов, при выборе Папки в области справа от дерева ничего не отображается. То есть нельзя, например, отобразить карточку при выборе Папки.
При выборе Представления в Дереве объектов в правой области отображается таблица, сформированная либо SQL реквестом, либо же эта табличка может быть сформирована из C# кода. Кроме этого, дефолтное поведение правой области при выборе Представления, может быть переопределено Расширениями. Например, можно при выборе Представления в дереве нарисовать в WPF (в случае десктоп клиента) произвольные контролы.
Сабсеты представляют из себя подмножества Представления. Сабсеты ограничены в смысле отображения произвольного содержимого при выборе в Дереве.

![Object tree](https://github.com/1001011000101101/Tessa/blob/master/images/Tree.JPG)


## Работа с бд:

Что касается ORM, то его тут нет). Разработчики вместо ORM используют вот это: 

[bltoolkit](https://github.com/igor-tkachev/bltoolkit)
[ProgrammersGuide](https://mytessa.ru/docs/ProgrammersGuide/ProgrammersGuide.html#_%D0%B8%D1%81%D0%BF%D0%BE%D0%BB%D1%8C%D0%B7%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5_idbscope_%D0%B4%D0%BB%D1%8F_%D0%B2%D1%8B%D0%BF%D0%BE%D0%BB%D0%BD%D0%B5%D0%BD%D0%B8%D1%8F_%D0%B7%D0%B0%D0%BF%D1%80%D0%BE%D1%81%D0%BE%D0%B2_%D0%BA_%D0%B1%D0%B4)

О высокой скорости разработки в такой конфигурации говорить не приходится. Запрос к БД выглядит вот так:

```C#

db
        .SetCommand(
            "insert into Table (Column1, Column2) values (@Column1, @Column2)",
            db.Parameter("@Column1", value1),
            db.Parameter("@Column2", value2))
        .LogCommand()
        .ExecuteNonQuery();

```

Если представить ситуацию, когда нужно рефакторить базу данных (нормализация или ренейминг), то, уверен, программисту прощу уволиться, чем рефакторить вот такие стринговые запросы. Обращаю внимание: bltoolkit находится в состоянии suspended.

Остановлюсь на этом ужасе немного подробнее. 

Вот пример запроса на получение данных, способом, который предлагается разработчиками TESSA:

```C#
await using (this.dbScope.Create())
            {
                var db = dbScope.Db;

                var comand =
                     db.SetCommand(dbScope.BuilderFactory
                        .Select().C(null, "UserID", "UserName")
                        .From("RoleUsers")
                        .InnerJoin("Roles").On().C("RoleUsers", "ID").Equals().C("Roles", "ID")
                        .Where().C("RoleUsers", "TypeID").Equals().P("TypeID")
                        .And().C("Roles", "Name").Equals().P("RoleName")
                        .Build(),
                        db.Parameter("TypeID", 0, LinqToDB.DataType.Int32), db.Parameter("RoleName", "Мастера"))
                    .LogCommand();

                using var reader = await comand.ExecuteReaderAsync(cancellationToken);

                if (reader.HasRows)
                {
                    while (await reader.ReadAsync(cancellationToken))
                    {
                        string masterName = reader.GetValue<string>(1);
                        Guid masterId = reader.GetValue<Guid>(0);

                        result.Add(new DmMaster(masterId, masterName));
                    }
                }
            }
```

А вот запрос на получение точно таких же данных, но уже с ORM:

```C#

var dmMasters = db.RoleUsers.Where(x => x.TypeId == DmConstants.StaticRoleTypeId && x.Id == DmConstants.TessaDmMastersRoleId)
                .Select(x => new DmMaster()
            { 
                Id = x.UserId, 
                Name = x.UserName 
            }).ToList();

```


Тут мне могут возразить, что первый запрос будет работать быстрее. Возможно, но это не значит, что нужно писать на raw SQL. 
Эти времена давно прошли. Где-то это может быть уместно, но не во всем же проекте. Разница колоссальная! Не думал, что увижу такое в 2020) Сейчас время программиста стоит намного дороже, чем процессорное время, это нужно учитывать.
Скорость (соответственно, стоимость) внедрения системы TESSA будет страдать от без-ORM-ного подхода.
Справедливости ради нужно заметить, что в коде замечен нэймспейс LinqToDB.

Мы же пойдем своим, гуманным и типобезопасным путем: подключим живую ORM, например EntityFramework или Fluent NHibernate. 
На мой взгляд предпочтительнее EntityFramework так как не нужно руками писать маппинги, создание всех POCO классов на основе схемы БД в EF занимает ровно одну строку (Scaffold-DbContext). А для того, чтобы избежать проблем с зависимостями (разные версии одной и той же библиотеки), как с текущей версией TESSA, так и с будущими, можно развернуть netcore webapi. Получаем микросервисную архитектуру, data access layer будет микросервисом, можно еще выделить DTO в отдельный проект. Абстрагируемся не только от базы данных, но и от ORM) Наивысшая степень абстракции).

необходимые пакеты:

Microsoft.EntityFrameworkCore
Microsoft.EntityFrameworkCore.SqlServer
Microsoft.EntityFrameworkCore.Tools

Далее In Visual Studio, select menu Tools -> NuGet Package Manger -> Package Manger Console and run the following command:

```
PM> Scaffold-DbContext "Server=(local);Database=tessa;Trusted_Connection=True;" Microsoft.EntityFrameworkCore.SqlServer -OutputDir Models
```

- и webapi готов!


## Отчеты:

Тулинг для работы с отчетами типа СКД или FastReport здесь отсутствует! Это вам не low-code system.
Тут также два варианта: делать отчеты на голом WPF или React.js либо интегрировать что-то внешнее. Или можно например смиксовать: рядом подсадить еще один веб-сервис, который будет хостить тот же FastReport, и его страницы отображать прямо в расширениях WPF через WebBrowser. Снова имеем слабосвязанные и легко заменяемые элементы системы:

```
webBrowser.Navigate("http://local-report-tooling");

```


## Создание карточки:

Новую карточку можно создать достаточо просто:

В модальном окне:
```C#
var createCardContext = await advancedCardDialogManager.CreateCardAsync(cardTypeID: cardTypeId);
```

или на вкладке:

```C#
var createCardContext = await uiHost.CreateCardAsync(cardTypeID: cardTypeId);
```

Подписаться на события карточки, например, в файловый контрол загрузили новый файл, можно реализовав интерфейс ICardUIExtension.
Делается примерно так: в методе инициализации ICardUIExtension.Initialized() получаем модель карточки ICardModel, из нее достаем модель контрола IControlViewModel по его системному имени (алиасу), который задан во время проектирования:

```C#
cardModel.Controls.TryGet(Helpers.ControlAlias, out fileControlViewModel)
```
 Далее, делаем кастинг:
 
 ```C#
 var fileControl =  ((FileListViewModel)fileControlViewModel).FileControl;
 ```

Теперь можно подписываться на события:

```C#
fileControl.ContainerFileAdding += FileControl_ContainerFileAdding;
```

Здесь нужно заметить, что простого и интуитивно понятного способа передать что-то в методы ICardUIExtension нет. Например, если мы захотим передать caption текущего узла дерева (ITreeItem, или более частный случай IViewTreeItem - представление) в рабочем месте, то придется вместо простого создания карточки:

```C#
var createCardContext = await advancedCardDialogManager.CreateCardAsync(cardTypeID: cardTypeId);
```

сначала создать карточку вручную:

```C#
CardNewRequest cardNewRequest = new CardNewRequest() { CardTypeID = cardTypeId };
CardNewResponse cardNewResponse = cardRepository?.NewAsync(cardNewRequest).Result;
```

заполнить поля карточки:

```C#
cardNewResponse.Card.ID = Guid.NewGuid();
cardNewResponse.Card.Sections[DmDynamicNames.SectionDmProjects].Fields[DmDynamicNames.ViewCompositionId] = viewCompositionId;
```

сохранить карточку:

```C#
CardStoreRequest cardStoreRequest = new CardStoreRequest() { Card = cardNewResponse.Card };
CardStoreResponse cardStoreResponse = cardRepository?.StoreAsync(cardStoreRequest).Result;
```

и открыть ее:
```C#
var context = await this.uiHost.OpenCardAsync(cardID: cardNewResponse.Card.ID);
```

и теперь мы имеем возможность работать с данными внутри обработчиков:

```C#
string name = card.Sections[DmDynamicNames.SectionDmProjects].Fields.Get<string>(DmDynamicNames.Name);
```


## Публикация:
Доставка клиентсого приложения осуествляется на удивление легко. Пользователю не придется думать об обновлении приложения, для пользователя все происходит бесшовно. Это плюс!
[AdministratorGuide.html#apps-publish](https://mytessa.ru/docs/AdministratorGuide/AdministratorGuide.html#apps-publish)

Что же касается deploy-я на development сервер, то программисту предется думать об этом самостоятельно. Если мы разрабатываем клиент-серверные расширения, то все файлики копировать нужно вручную: остановить IIS (если Windows), скопировать новые extensions и снова запустить IIS. С клиентскими расширениями примерно тоже самое: закрыть клиента, скопировать, снова открыть. В зрелых продуктах данного класса (ECM-BPM) вопрос деплоинга решен, взять тот же DirectumRX. Я для этой задачи написал небольшой хелпер:
[ProjectPublicationHelpers](https://github.com/1001011000101101/ProjectPublicationHelpers).



## Критика


### Неудобное создание карточки

Из коробки процесс создания карточки на текущий момент (02 мая 2020) выглядит так:

![Image of Yaktocat](https://github.com/1001011000101101/Tessa/blob/master/images/CreateCard.png)

То есть пользователю предлагается выполнить избыточное кол-во операций, чтобы создать карточку. Для исправления ситуации служит механизм [расширений](https://mytessa.ru/docs/ProgrammersGuide/ProgrammersGuide.html#_%D1%80%D0%B0%D1%81%D1%88%D0%B8%D1%80%D0%B5%D0%BD%D0%B8%D1%8F).


### Неудобное удаление карточки

Из коробки процесс удаления карточки на текущий момент (02 мая 2020) выглядит так:

![Image of Yaktocat](https://github.com/1001011000101101/Tessa/blob/master/images/DeleteCard.png)

То есть пользователю предлагается выполнить избыточное кол-во операций, чтобы удалить карточку. Нужно учитывать, что при интенсивной работе с карточками, процесс удаления потребует доработки. На мой взгляд, тут нехватает панели стандартных операций, которые доступны непосредственно на экране карточки. Казалось бы, это не смертельно. Соглашусь, но ведь система призвана ускорять процессы в компании, и тут удобный GUI очень важен.

## TypeSafety

В мануалах присутствует ([тут](https://mytessa.ru/docs/ProgrammersGuide/ProgrammersGuide.html#_%D0%BF%D1%80%D0%B8%D0%BC%D0%B5%D1%80_%D1%80%D0%B0%D1%81%D1%88%D0%B8%D1%80%D0%B5%D0%BD%D0%B8%D0%B5_ui_%D0%BD%D0%B0_%D1%81%D0%BA%D1%80%D1%8B%D1%82%D0%B8%D0%B5_%D1%8D%D0%BB%D0%B5%D0%BC%D0%B5%D0%BD%D1%82%D0%BE%D0%B2_%D1%83%D0%BF%D1%80%D0%B0%D0%B2%D0%BB%D0%B5%D0%BD%D0%B8%D1%8F_%D0%BF%D0%BE_%D1%83%D1%81%D0%BB%D0%BE%D0%B2%D0%B8%D1%8E)) демонстрация не самых лучших практик программирования, а именно, проблема с [типобезопасностью](https://en.wikipedia.org/wiki/Type_safety)

Проблема тут в том, что в примерах кода используются имена контролов, то есть обычные string. Что будет с этим кодом, когда через N месяцев кто-то совершенно справедливо решит, что такой нэйминг, как **Control1** - это вообще говоря, не самая лучшая идея? Программист переименует **Control1** в что-нибудь более описательное, например **txtProjectName**. Решение (Solution) вполне себе прекрасно сбилдится, и эта проблема дойдет до пользователя! А если таких участков будет очень много?
Захардкоженных строк в коде быть не должно! Это ликбез. Все строковые литералы необходимо выносить в отдельный класс с константами. Или можно хранить константы в БД, и вытаскивать их в compile time в код автоматически (кодогенератор T4 и DAPPER).
Строковые литералы в коде - это глюкогенератор.

![TypeSafety issue](https://github.com/1001011000101101/Tessa/blob/master/images/TypeSafety.png)

## Резюме

Сравнительно невысокая цена системы на первоначальном этапе внедрения скомпенсируется на длинном временном интервале комплексностью разработки и поддержания.
То есть дешевле в итоге не получится вот почему:
1) На текущий момент система представляет из себя почти пустой каркас: все придется делать самостоятельно.
2) Высокий порог входа в разработку: сложная программная архитектура (слишком много WTF per Hour) влечет за собой необходимость в наличии высококвалифицированных специалистов, которых на рынке сейчас ноль целых, ноль десятых...

