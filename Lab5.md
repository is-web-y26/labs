# ЛР 5. Разработка схемы GraphQL

В результате выполнения данной лабораторной работы должна быть получена готовая схема GraphQL с возможностью выполнения запросов и мутаций во встроенной песочнице GraphQL.

## Подготовка приложения

NestJS предлагает на выбор два варианта интеграции GraphQL, Apollo Server (который используется по умолчанию) и Mercurius (работает только с Fastify). Для нас между ними нет особой разницы, но первый вариант поставляется со встроенной песочницей, поэтому предлагается использовать именно его.

Также на выбор предлагается два способа работы с GraphQL: отталкиваясь от кода (code-first) и отталкиваясь от схемы (schema-first). Подробнее про них можно (и нужно) почитать [в соответствующем разделе документации](https://docs.nestjs.com/graphql/quick-start), но в рамках лабораторной работы предлагается использовать подход code-first.

Опираясь на документацию, установите нужные пакеты и подключите модуль `GraphQLModule`.

## Разработка запросов, мутаций и обработчиков

Самый простой способ посмотреть на структуру файлов, которая нужна для корректной работы запросов - создать новый модуль с готовым ресурсом, выбрать в качестве транспорта GraphQL и согласиться сгенерировать CRUD-операции (см. ЛР 3). Выполнить команду поверх существующего модуля не получится, поскольку NestJS не сможет сгенерировать сервис, поэтому можно сгенерировать тестовый модуль, посмотреть его структуру и впоследствии удалить его.

Если обойтись без этой команды, то для базовой работы с GraphQL нужно реализовать три вещи:, **запрос**, который можно будет выполнить в песочнице, его **обработчик** (resolver) и **схему**, которая включает в себя описание как различных данных, так и доступных операций.

Сначала нужно описать схему в части оперируемых данных. При подходе code-first нужно описать аналоги DTO, object types (возвращаемые данные) и input object types (принимаемые данные). Для этого нужно создать соответствующие классы с полями и расставить декораторы у полей и классов. Все поля должны быть с описанием. Пример из документации:

```ts
import { Field, Int, ObjectType } from '@nestjs/graphql';
import { Post } from './post';

@ObjectType()
export class Author {
  @Field(type => Int)
  id: number;

  @Field({ nullable: true })
  firstName?: string;

  @Field({ nullable: true })
  lastName?: string;

  @Field(type => [Post])
  posts: Post[];
}
```

Эти классы автоматически превратятся в части общей схемы при сборке приложения.

Далее нужно реализовать непосредственно обработчики запросов и мутаций. Обработчик можно представить как контроллер из мира GraphQL: его основной задачей также является приём запроса/мутации, его первичная валидация и передача в сервис. Поскольку используется подход code-first, объявления запросов и мутаций для схемы сгенерируются автоматически по коду обработчика, только не забудьте поставить нужные декораторы.

Ещё один пример из документации:

```ts
@Resolver(() => Author)
export class AuthorsResolver {
  constructor(
    private authorsService: AuthorsService,
    private postsService: PostsService,
  ) {}

  @Query(() => Author)
  async author(@Args('id', { type: () => Int }) id: number) {
    return this.authorsService.findOneById(id);
  }

  @ResolveField()
  async posts(@Parent() author: Author) {
    const { id } = author;
    return this.postsService.findAll({ authorId: id });
  }
}
```

С большой вероятностью сервисы уже будут содержать необходимую бизнес-логику, нужно будет её только подключить.

Класс с обработчиком можно сгенерировать с помощью CLI:

```bash
nest generate resolver [name] --no-spec
```

Эта команда создаст пустой обработчик и подключит его в заданном модуле.

> [!IMPORTANT]
>
> **Запросы и мутации должны соответствовать операциям предметной области.** Например, если есть сущность со статьёй `Post`, у которой есть статусы "Опубликованная" и "Скрытая", должна быть не одна мутация `changeStatus(postId: ID, status: PostStatus)`, а две, `publish(postId: ID)` и `hide(postId: ID)`.

Реализуйте получение вложенных сущностей с помощью обработчика отдельного поля (field resolver) для более удобного получения связанной информации. При этом для запросов, которые возвращают большой объём данных, предусмотрите возможность пагинации. Чтобы избежать потенциальных проблем с производительностью, добавьте подсчёт сложности запроса и ограничение на максимальную сложность.

В итоге у вас по адресу http://localhost:3000/graphql должна быть доступна песочница с похожим интерфейсом:

![](https://i.imgur.com/VJ84fNC.png)

Перед сдачей лабораторной работы убедитесь, что вы знаете, как открывать песочницу, как в ней открыть схему GraphQL и как писать в ней запросы.
