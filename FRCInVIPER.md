## NSFetchedResultsController в VIPER

Одной из наиболее удобных возможностей, которые нам предоставляет использование **CoreData** для работы с графом объектов, является `NSFetchedResultsController`. В рамках архитектуры MVC его использование достаточно очевидно - контроллер экрана реализует протокол `NSFetchedResultsControllerDelegate`:

```objc
- (void)controller:(NSFetchedResultsController *)controller didChangeObject:(id)anObject atIndexPath:(nullable NSIndexPath *)indexPath forChangeType:(NSFetchedResultsChangeType)type newIndexPath:(nullable NSIndexPath *)newIndexPath;
- (void)controllerWillChangeContent:(NSFetchedResultsController *)controller;
- (void)controllerDidChangeContent:(NSFetchedResultsController *)controller;
```

Эти методы действительно чрезвычайно удобны для того, чтобы прямо в них обновить стейт и вставить/перезагрузить/удалить некоторые из ячеек. За такую простоту, конечно, приходится платить:

- Еще одна ответственность у контроллера,
- Знание о CoreData выходит за пределы модельного/сервисного слоя,
- Мы жестко привязываемся к выбранному механизму уведомлений об изменениях состояния базы,
- *+100 строк* в и без того крупном классе.

Часть из поставленных проблем в некоторой степени решаются путем декомпозиции контроллера на составные объекты, в том числе и на элементы VIPER-стека.

В VIPER, в отличие от MVC, роль и место для `NSFetchedResultsController` не так четко определены. Однозначно не *View* - во-первых, она не может получать бизнес-сущности от кого-то из своего же слоя, во-вторых - в большинстве случаев стоит стараться вообще не использовать `NSManagedObject`'ы в чистом виде на верхнем уровне архитектуры.

Не подходит и *Presenter* - его ответственность - связывать между собой элементы модуля и держать стейт. *FRC* же - это отдельный источник данных, не связанный с интерактором. 

Размещать на сервисном слое - тоже неправильно, поскольку сервисы - объекты пассивные, умеющие только реагировать на команды от верхнеуровневых компонентов, и не содержащие в себе никакого дополнительного стейта.


Оптимальный вариант для размещения ответственности *FRC* - а под ней мы понимаем слежение за состоянием графа объектов, это интерактор:

- Он уже работает с другими бизнес-сущностями,
- В большинстве случаев он знает о том, что мы используем CoreData,
- Он является источником данных текущего модуля - не важно, что послужило причиной их появления - прямой запрос из презентера или получение уведомления от базы.

Мы пришли к следующему варианту работы с FRC на уровне интерактора:

##### Протокол CacheTracker

`CacheTracker` - протокол объекта, задачей которого является получение уведомлений об изменении состояния базы и формировании на их основе набора транзакций.

```objc
@protocol CacheTracker <NSObject>

/**
 Метод настраивает трекер кеша
 
 @param cacheRequest Запрос, описывающий поведение трекера
 */
- (void)setupWithCacheRequest:(CacheRequest *)cacheRequest;

/**
 Метод формирует батч транзакций исходя из текущего состояния кеша
 
 @return CacheTransactionBatch
 */
- (CacheTransactionBatch *)obtainTransactionBatchFromCurrentCache;

@end
```

##### Пример реализации `CacheTracker`

Приведенный вариант реализации построен как раз на работе с `NSFetchedResultsController`. Альтернативная имплементация протокола могла бы, к примеру, быть построенной на работе с `NSNotification`'ами, получаемыми от CoreData.

```objc
#pragma mark - Публичные методы

- (void)setupWithCacheRequest:(CacheRequest *)cacheRequest {
    NSManagedObjectContext *defaultContext = [NSManagedObjectContext MR_defaultContext];
    NSFetchRequest *fetchRequest = [self fetchRequestWithCacheRequest:cacheRequest];
    self.controller = [[NSFetchedResultsController alloc] initWithFetchRequest:fetchRequest
                                                          managedObjectContext:defaultContext
                                                            sectionNameKeyPath:nil
                                                                     cacheName:nil];
    self.controller.delegate = self;
    [self.controller performFetch:nil];
}

- (NSFetchRequest *)fetchRequestWithCacheRequest:(CacheRequest *)cacheRequest {
    NSFetchRequest *fetchRequest = [NSFetchRequest fetchRequestWithEntityName:[cacheRequest.objectClass entityName]];
    [fetchRequest setPredicate:cacheRequest.predicate];
    [fetchRequest setSortDescriptors:cacheRequest.sortDescriptors];
    return fetchRequest;
}

- (CacheTransactionBatch *)obtainTransactionBatchFromCurrentCache {
    CacheTransactionBatch *batch = [CacheTransactionBatch new];
    for (NSUInteger i = 0; i < self.controller.fetchedObjects.count; i++) {
        id object = self.controller.fetchedObjects[i];
        NSIndexPath *indexPath = [self.controller indexPathForObject:object];
        id plainObject = [self.objectsFactory plainNSObjectForObject:object];
        CacheTransaction *transaction = [CacheTransaction transactionWithObject:plainObject
                                                                   oldIndexPath:nil
                                                               updatedIndexPath:indexPath
                                                                     objectType:NSStringFromClass(self.cacheRequest.objectClass)
                                                                     changeType:NSFetchedResultsChangeInsert];
        [batch addTransaction:transaction];
    }
    
    return batch;
}

#pragma mark - Методы NSFetchedResultsControllerDelegate

- (void)controllerWillChangeContent:(NSFetchedResultsController *)controller {
    self.transactionBatch = [CacheTransactionBatch new];
}

- (void)controller:(NSFetchedResultsController *)controller didChangeObject:(NSManagedObject *)anObject atIndexPath:(NSIndexPath *)indexPath forChangeType:(NSFetchedResultsChangeType)type newIndexPath:(NSIndexPath *)newIndexPath {
    id plainObject = [self.objectsFactory plainNSObjectForObject:anObject];
    CacheTransaction *transaction = [CacheTransaction transactionWithObject:plainObject
                                                               oldIndexPath:indexPath
                                                           updatedIndexPath:newIndexPath
                                                                 objectType:NSStringFromClass(self.cacheRequest.objectClass)
                                                                 changeType:changeType];
    [self.transactionBatch addTransaction:transaction];
}

- (void)controllerDidChangeContent:(NSFetchedResultsController *)controller {
    if ([self.transactionBatch isEmpty]) {
        return;
    }
    
    [self.delegate didProcessTransactionBatch:self.transactionBatch];
}
```

##### Классы транзакций

`CacheTransaction` - объект, описывающий изменение одного `NSManagedObject`.

```objc
@interface CacheTransaction : NSObject

/**
 Измененный объект
 */
@property (strong, nonatomic, readonly) id object;

/**
 IndexPath объекта до его изменения
 */
@property (strong, nonatomic, readonly) NSIndexPath *oldIndexPath;

/**
 IndexPath объекта после его изменения
 */
@property (strong, nonatomic, readonly) NSIndexPath *updatedIndexPath;

/**
 Тип измененного объекта
 */
@property (strong, nonatomic, readonly) NSString *objectType;

/**
 Тип изменения
 */
@property (assign, nonatomic, readonly) NSFetchedResultsChangeType changeType;

+ (instancetype)transactionWithObject:(id)object
                         oldIndexPath:(NSIndexPath *)oldIndexPath
                     updatedIndexPath:(NSIndexPath *)updatedIndexPath
                           objectType:(NSString *)objectType
                           changeType:(NSUInteger)changeType;

@end

```

`CacheTransactionBatch` объединяет набор транзакций в один объект, предназначенный для передачи между слоями прямо к таблице.

```objc
@interface CacheTransactionBatch : NSObject

@property (strong, nonatomic, readonly) NSOrderedSet *insertTransactions;
@property (strong, nonatomic, readonly) NSOrderedSet *updateTransactions;
@property (strong, nonatomic, readonly) NSOrderedSet *deleteTransactions;
@property (strong, nonatomic, readonly) NSOrderedSet *moveTransactions;

/**
 Метод добавляет в батч новую транзакцию
 
 @param transaction Транзакция
 */
- (void)addTransaction:(CacheTransaction *)transaction;

/**
 Метод сообщает, содержит ли батч хоть одну транзакцию
 
 @return YES/NO
 */
- (BOOL)isEmpty;

@end

```

##### Пример интерактора

Интерактор не делает практически никакой работы - ему нужно только правильно преднастроить `CacheTracker` и стать его делегатом.

```objc
@implementation PostListInteractor

- (void)setupCacheTrackingWithCacheRequest:(CacheRequest *)cacheRequest {
    [self.cacheTracker setupWithCacheRequest:cacheRequest];
    CacheTransactionBatch *initialBatch = [self.cacheTracker obtainTransactionBatchFromCurrentCache];
    [self.output didProcessCacheTransaction:initialBatch];
}

#pragma mark - Методы протокола CacheTrackerDelegate

- (void)didProcessTransactionBatch:(CacheTransactionBatch *)transactionBatch {
    [self.output didProcessCacheTransaction:transactionBatch];
}

@end

```

##### Работа с таблицей

И финальная часть схемы - обработка таблицей полученного батча транзакций. В текущем варианте мы работаем с `UITableView` не напрямую, а с помощью фреймворка `Nimbus` - но сути действий это не меняет.

```objc
- (void)updateDataSourceWithTransactionBatch:(CacheTransactionBatch *)transactionBatch {
    for (CacheTransaction *transaction in transactionBatch.insertTransactions) {
        PostListCellObject *cellObject = [self generateCellObjectForPost:transaction.object];
        
        NSUInteger numberOfObjects = [self.tableViewModel lj_numberOfObjectsInSection:PostListSectionIndex];
        NSUInteger updatedRow = transaction.updatedIndexPath.row;
        [self.tableViewModel insertObject:cellObject
                                    updatedRow
                                inSection:PostListSectionIndex];
    }
    
    for (CacheTransaction *transaction in transactionBatch.updateTransactions) {
        PostListCellObject *cellObject = [self generateCellObjectForPost:transaction.object];
        NSIndexPath *oldIndexPath = [NSIndexPath indexPathForRow:transaction.oldIndexPath.row
                                                       inSection:PostListSectionIndex];
        [self.tableViewModel removeObjectAtIndexPath:oldIndexPath];
        [self.tableViewModel insertObject:cellObject
                                    atRow:transaction.updatedIndexPath.row
                                inSection:PostListSectionIndex];
    }
    
    NSMutableArray *removeIndexPaths = [NSMutableArray array];
    for (CacheTransaction *transaction in transactionBatch.deleteTransactions) {
        NSIndexPath *removeIndexPath = [NSIndexPath indexPathForRow:transaction.oldIndexPath.row
                                                          inSection:PostListSectionIndex];
        [removeIndexPaths addObject:removeIndexPath];
    }
    [self.tableViewModel lj_removeObjectsAtIndexPaths:[removeIndexPaths copy]];
    
    for (CacheTransaction *transaction in transactionBatch.moveTransactions) {
        PostListCellObject *cellObject = [self generateCellObjectForPost:transaction.object];
        NSIndexPath *oldIndexPath = [NSIndexPath indexPathForRow:transaction.oldIndexPath.row
                                                       inSection:PostListSectionIndex];
        [self.tableViewModel removeObjectAtIndexPath:oldIndexPath];
        [self.tableViewModel insertObject:cellObject
                                    atRow:transaction.updatedIndexPath.row
                                inSection:PostListSectionIndex];
    }
}
```