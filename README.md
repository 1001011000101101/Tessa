# СЭД Tessa

Ниже собрана полезная информация по системе TESSA 3.4

1. [Общее описание системы](#общее-описание-системы)
   * [Карточка](#карточка)
   * [Представление](#представление)
   * [Схема данных](#схема-данных)
   * [Дерево объектов](#дерево-объектов)
   * [Работа с БД](#работа-с-бд)
2. [Критика](#критика)
   * [TypeSafety](#type-safety)
3. [Описание проекта автоматизации проектного управления в строительной отрасли](#описание-проекта-автоматизации-проектного-управления-в-строительной-отрасли)
   * [Задача №1: Структура проекта](#задача-1-структура-проекта)

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

Если представить ситуацию, когда нужно рефакторить базу данных (нормализация или ренейминг), то, уверен, программисту прощу уволиться, чем рефакторить вот такие стринговые запросы.

Выход здесть простой: подключить нормальную ORM, например EntityFramework database first. Что я в итоге и сделал.






![Object tree](https://github.com/1001011000101101/Tessa/blob/master/images/Tree.JPG)

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

![TypeSafety issue](https://github.com/1001011000101101/Tessa/blob/master/images/TypeSafety.png)



## Описание проекта автоматизации проектного управления в строительной отрасли

### Задача №1: Структура проекта

Необходимо дать пользователю возможность создавать проекты с представленной структурой папок (Разделы проекта). Причем при выборе каждой папки в области справа должен отображаться набор контролов, необходимый для имплементации бизнес-логики. 
 
![Project structure image](https://github.com/1001011000101101/Tessa/blob/master/images/ProjectTree.png)


Добавим в админке в дерево объектов Представление "Проекты" и привяжем к нему расширение, входящее в стандартную поставку системы - "CreateCardExtension". Данное расширение добавляет одну кнопку в верхнюю панель Представления "Создать карточку".

![CreateCardExtension image](https://github.com/1001011000101101/Tessa/blob/master/images/CreateCardExtension.png)

Расширение в Visual Studio выглядит так:
Данное расширение работает как в десктоп-клиенте, так и в веб-клиенте.

![CreateCardExtensionVisualStudio image](https://github.com/1001011000101101/Tessa/blob/master/images/CreateCardExtensionVisualStudio.png)

Далее необходимо перехватить событие сохранение (и удаления) карточки нового проекта и написать код добавления подчиненных Представлений в дерево объектов.
Сказано - сделано:

Регистрация перехватчиков на server-side:

```C#
public override void RegisterExtensions(IExtensionContainer extensionContainer)
        {
            // Store
            extensionContainer
                .RegisterExtension<ICardStoreExtension, DmProjectNewCardStoreExtension>(x => x
                    .WithOrder(ExtensionStage.AfterPlatform, 15)
                    .WithUnity(this.UnityContainer)
                    .WhenCardTypes(Helpers.DmProjectNewID)
                    .WhenAnyStoreMethod())
                ;


            // Delete
            extensionContainer
                .RegisterExtension<ICardDeleteExtension, DmProjectNewCardDeleteExtension>(x => x
                    .WithOrder(ExtensionStage.AfterPlatform, 15)
                    .WithUnity(this.UnityContainer)
                    .WhenCardTypes(Helpers.DmProjectNewID)
                    .WhenAnyStoreMethod())
                ;
        }
```

Далее, дерево объектов хранится в бд (Workplaces) Представление в дереве выглядит так:

```json
#view(Alias:DmProjectFolder1, Caption:1.Графики авансирования, CompositionId:$$DmProjectFolder1CompositionId$$, IsNode:True, ParentCompositionId:$$DmProjectFolder1ParentCompositionId$$, RowCounterVisible:Hidden, EnableAutoWidth:True) {
				#layout(Caption:1.Графики авансирования, CompositionId:$$DmProjectFolder1LayoutCompositionId$$) {
					#content {
						#data_view(Alias:DmProjectFolder1, CompositionId:$$DmProjectFolder1CompositionId$$) 
					}
				}
				#extension(TypeName:Tessa.Extensions.Client.Views.DmProjectFolder1ViewExtension, Order:0) {}
			}
```

В данном случае можно редактировать это дерево напрямую. Парсинг прост: разбиваем дерево на лексеммы ('#view', '(', ')', '{', '}') и считаем фигурные скобки.
Это самый короткий путь, но отнюдь не самый правильный! Правильно, конечно же, делать это через API ([TreeItemFactory](https://mytessa.ru/docs/api/html/M_Tessa_UI_Views_Workplaces_Tree_TreeItemFactory__ctor.htm)). Потому что если в следующих релизах работа с данным деревом подвергнется редизайну, придется переделывать.

В результате имеем вот такую заготовку:

![DmProjectTree image](https://github.com/1001011000101101/Tessa/blob/master/images/DmProjectTree.JPG)


