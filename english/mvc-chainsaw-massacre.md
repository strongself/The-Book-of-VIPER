It is not difficult to start a new project with the VIPER architecture as a foundation. But constantly developers have to support and enhance applications, that were initially developed with chaotic code base, without considering strict rules of design and architecture. It often happens that the first version of the project doesn't have a lot of requirements and they can fit on a single sheet of paper. Therefore, there is not enough attention paid to the application architecture at this stage. But terms of reference for the second version can have as many pages as "War and Peace"(by Leo Tolstoy) has. That's why, more complex task, but an interesting one at the same time, is to migrate an existing iOS application from a weak foundation to the solid one, that is based on a flexible and robust VIPER architecture.   

### Why MVC becomes Massive-View-Controller

**Model-View-Controller (MVC)** is a basic architectural pattern for iOS-applications offered by Apple. Usually this pattern explicitly or implicitly is being followed by all novice developers. Model-View-Controller is obviously a three-layered architectural pattern. When using it, all application objects, depending on the purpose, belong to one of three layers: Model (Model), presentation (View) or control (Controller). Each architectural layer is separated from the other by abstract boundaries, that are used for communication between objects from the different layers. The primary purpose of this architectural concept is separation of business logic (model) from its visualization (representation).

**The model layer** describes the data of an application and determines processing logic and storage. Model objects should not have a clear link with objects of the presentation layer; they don't contain any information how data could be visualized. As an example of objects of this layer can be data storage objects, parsers, network clients, etc.

**The Presentation** layer contains objects that a user can see. Objects of this layer are usually can be reused. These include classes like UILabel and UIButton.

**The Controller** layer mediates an interaction between presentation layer objects and model objects: controls user input and uses a model and a view to implement necessary reaction. Also objects of this layer are used for definition and coordination of application problems, life cycle management of other objects.

![Model-View-Controller](../Resources/MVCMassacre/MVC.png)

> **Notice**. Despite the fact that Apple in its documentation refers to the described architectural pattern as classic Model-View-Controller, however, it's correct name is **Model-View-Adapter (MVA)** or **mediating-controller MVC**.
> 
> MVA and MVC solve the same problem, but using different approaches. Classic MVC is represented in the form of a triangle, where the vertices are layers: the model, the view and the controller - and it enables communication between the model and the view bypassing the controller. MVA also has three layers on the same line in order to prevent direct communication between the model and the view (as shown above).
> 
> The term Model-View-Controller will be used further to avoid inconsistencies between Apple official documentation and this article.

According to the Apple recommendations, **presentation controllers (view controllers)** are the core of the control layer in an iOS-app . Each such object is responsible for

* view hierarchy management;
* scaling of presentation views to fit a specific device display size;
* updating presentation content in response to a change of data;
* processing of user input and transmitting data to the model layer;
* releasing of related resources in case of a insufficient memory;

The recommended way to describe a view controller and related view is a Storyboard editor. Using this tool, it is possible not only to specify which presentation objects belong to a specific view controller, but also to define hierarchical relationships and transitions between different controllers.
![Storyboard editor](../Resources/MVCMassacre/Storyboard editor.png)
Apple gives a few tips for creating a view controller.

* **Use view controller classes provided with the standard SDK.**

    The iOS SDK has a set of view controllers that solve specific tasks, starting from accessing a user's contact list up to displaying media data. A good practice is to use such controllers, provided by system libraries, in your applications.

* **Make a View Controller as independent as possible.**   

    A View Controller shouldn't be aware of the internal logic of another controller or his view hierarchy. The communication between two controllers should be done through an explicitly defined public interface. 

* **Do not store data in a view controller.** 
    
    A View controller is a mediator between the model layer and the presentation layer for data exchange. It can cache some data for quick access, validate it, but its main responsibility - to ensure that a view displays correct information.    

* **Use a view controller for reacting to external events.**
    
    The external events include: user input, system notifications (for example, when the keyboard appears), delegate methods for various handlers, for example [CLLocationManager](https://developer.apple.com/library/ios/documentation/CoreLocation/Reference/CLLocationManager_Class/index.html#//apple_ref/doc/uid/TP40007125-CH3)).      

These recommendations and advice, despite their usefulness, are too general, described without specifying concrete definition, therefore have a number of disadvantages. The major of them is uncontrolled growth of complexity of view controllers. Their tight coupling with the view life-cycle, which is the specific characteristic of the iOS SDK, requires generation of a response to events of this loop directly in the view controller. Implementation of handling of external events, user input is also happens in a view controller. Furthermore, a model often becomes too passive and is used exclusively for data access, but all the business logic still resides in a view controller. It worth mentioning that Apple almost does not describe important organizational problems of bi-directional data transfer between controllers, configuration of created controllers and so on, as a result a majority of developers think that these are responsibilities of the view controller.  

As a result, view controllers become a core of almost everything that happens in an application and, as a consequence, grow to giant size, turning into something called as **Massive-** or **Mega-View-Controller**. Similar type of object is an excellent example of the consequences of a poorly-designed development process that ignores the basic principles of design and architecture. 

![Massive-View-Controller](../Resources/MVCMassacre/MassiveViewControllerScheme.png)

The main disadvantages of using Massive-View-Controller are :

* **High complexity of support and development.** 
    
    The Massive-View-Controller code is difficult to modify and extend due to its tight coupling and unpredictable data streams. It is always risky to make changes, because this can result in a broken of existing functionality of an application, but more important this might be not recognized until a certain point in time.

* **A high entry threshold.**
    
    It can be a daunting task to find a required method in a such large amount of code: code structure is not obvious and requires time for exploring. 

* **Poorly testable code.**
    
    Due to the tight coupling Massive-View-Controller code is not coverable by unit tests. Thus, an attempt to check presentation logic leads to an explicit life-cycle method call that can indirectly lead to the instantiation of all related views, that violates the principles of unit-testing.

* **It is almost impossible to reuse code.**
    
    Implementation of new application features is carried out directly in a view controller. That's why it is impossible to reuse this part of the code without reusing the entire controller. As well as refactoring and decomposition of such large amount of code can be a non-trivial task. 

To avoid these drawbacks and prevent an application development process to result in a stalemate, it is necessary to strictly follow the requirements of a well-designed architectural pattern. Model-View-Controller, despite the weaknesses and non-obvious solutions to some design problems, due to the ease of use, it is still suitable for smaller applications or applications that require high speed of development. However, for large and complex projects, with long-term support and development cycle is better to use a more flexible architecture - VIPER.

### From poor implementation to a good one. Massive-View-Controller refactoring.

Faced with the task of developing an application whose codebase is burdened with the Massive-View-Controller, first of all you have to improve the structure of the project. As a result, this requires migration from a large and hardly maintainable Massive-View-Controller into something more convenient and flexible, for example, a VIPER-module. 
In the VIPER, as opposed to MVC, the view controller is the core of the view-layer, that allows not to burden it with additional logic and utilize it only for  implementation of the tasks directly related to data visualization. In this case, a view controller, receiving various external events (user input, system notifications of insufficient memory, etc.) passes them for processing to the lower layer - Presenter. All other non-visual module logic, implemented in MVC directly in a view controller (eg, creation of other modules or data transfer between them), is incapsulated in other layers as well.

There are three examples of transition from a Massive-View-Controller to a VIPER-module described further. A ViewController implementation from a real project (a guide to one of the cities in the world) is used as a starting point. This view controller contains the main navigation menu implementation.

![Storyboard editor](../Resources/MVCMassacre/MassiveViewControllerExample.png)

#### Refactoring examples

* **Transition between modules**

    This code snippet implements a transition to an inner section of the app. The direction of the transition is determined by the index of the selected menu table cell.
    
    ```objective-c
    - (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath {
        long index = indexPath.row;
        switch (index) {
            case MainMenuCellsCity: {
                UIStoryboard *sb = [TyphoonStoryboard storyboardWithName:@"Quarters"
                                                                 factory:AGAppDelegateMacros.storyboardFactory
                                                                  bundle:nil];
                UIViewController *vc = [sb instantiateViewControllerWithIdentifier:@"agquarterslistviewcontroller"];
                [self.navigationController pushViewController:vc
                                                     animated:YES];
            }
            break;
            case MainMenuCellsPlaces: {
                UIStoryboard *sb = [TyphoonStoryboard storyboardWithName:@"Places"
                                                                 factory:AGAppDelegateMacros.storyboardFactory
                                                                  bundle:nil];
                UIViewController *vc = [sb instantiateViewControllerWithIdentifier:@"agplacescategorieslist"];
                [self.navigationController pushViewController:vc
                                                     animated:YES];
            }
            break;
            case MainMenuCellsReferenceBook: {
                UIStoryboard *sb = [TyphoonStoryboard storyboardWithName:@"ReferenceBook"
                                                                 factory:AGAppDelegateMacros.storyboardFactory
                                                                  bundle:nil];
                UIViewController *vc = [sb instantiateViewControllerWithIdentifier:@"agreferencebookcategoriesviewcontroller"];
                [self.navigationController pushViewController:vc
                                                     animated:YES];
            }
            break;
        }
    }
    ```

    This implementation has a number of issues.
    
    * Presentation layer is burdened with additional logic for creating new modules and implementing transitions between them. 
    * If you continue to add new menu items into this method it will grow indefinitely, and may affect the current implementation.
    * It not easy to cover this code by unit tests.

    Therefore, while refactoring this code should be moved to other layers. There are several steps to achieve this
    
    1. We need to extend an output data protocol of the view-layer. 
       
       It is necessary to add a method that can be described as **"Show a menu item of the specified type, with transitioning from the source view-controller"**
       
       ```objective-c
       @protocol AGMainMenuControllerViewOutput <NSObject>

        @required
        - (void)showMenuSectionWithType:(MainMenuSectionType)sectionType
                     fromViewController:(UIViewController *)viewController;
        
        @end
       ```
       
    2. Refactor the initial method of the view controller.

       ```objective-c
       - (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath {
            NSInteger index = indexPath.row;
            [self.output showMenuSectionWithType:index
                              fromViewController:self];
        }
       ```
       Now the view controller delegates the processing of user input from the view-layer to the presenter, without doing anything extra. 
       
    3. Extend the input data protocol for the router of the VIPER-module that is being created. 

       Each of the three initial transitions should have its own method defined. 
      
      ```objective-c
      @protocol AGMainMenuRouterInput <NSObject>

      @required
      - (void)showCityFromViewController:(UIViewController *)viewController;
      - (void)showPlacesFromViewController:(UIViewController *)viewController;
      - (void)showReferenceBookFromViewController:(UIViewController *)viewController;
        
      @end
      ```
      
    4. Implement methods, which have been just added to the view-layer output data protocol, inside the presenter. 

       After getting information about user input (ie, the selected table cell number) from the view layer, the presenter chooses a transition and transfers the control to the router. 
       
       ```objective-c
       - (void)showMenuSectionWithType:(MainMenuSectionType)sectionType
                     fromViewController:(UIViewController *)viewController {
            switch (sectionType) {
                case MainMenuCellsCity:
                    [self.router showCityFromViewController:viewController];
                    break;
                case MainMenuCellsPlaces:
                    [self.router showPlacesFromViewController:viewController];
                    break;
                case MainMenuCellsReferenceBook:
                    [self.router showReferenceBookFromViewController:viewController];
                    break;
            }
        }
       ```
       
    5. Implement methods added to the protocol inside the router of the VIPER-module which is being created.

       In each method being implemented, it necessary to load the required storyboard from the factory, get a next view controller from it and perform a Segue. 
       
       ```objective-c
        - (void)showCityFromViewController:(UIViewController *)viewController {
            UIStoryboard *sb = [TyphoonStoryboard storyboardWithName:@"Quarters"
                                                             factory:AGAppDelegateMacros.storyboardFactory
                                                              bundle:nil];
            UIViewController *vc = [sb instantiateViewControllerWithIdentifier:@"agquarterslistviewcontroller"];
            [viewController.navigationController pushViewController:vc animated:YES];
        }
        
        - (void)showPlacesFromViewController:(UIViewController *)viewController {
            UIStoryboard *sb = [TyphoonStoryboard storyboardWithName:@"Places"
                                                             factory:AGAppDelegateMacros.storyboardFactory
                                                              bundle:nil];
            UIViewController *vc = [sb instantiateViewControllerWithIdentifier:@"agplacescategorieslist"];
            [viewController.navigationController pushViewController:vc animated:YES];
        }
        
        - (void)showReferenceBookFromViewController:(UIViewController *)viewController {
            UIStoryboard *sb = [TyphoonStoryboard storyboardWithName:@"ReferenceBook"
                                                             factory:AGAppDelegateMacros.storyboardFactory
                                                              bundle:nil];
            UIViewController *vc = [sb instantiateViewControllerWithIdentifier:@"agreferencebookcategoriesviewcontroller"];
            [viewController.navigationController pushViewController:vc animated:YES];
        }
       ```

    As a result, such implementation of the original problem will make code loosely coupled (for example, you can replace the router implementation and handle transitions differently), testable and more readable. Further code refactoring of this example, despite its necessity, will not affect logic segregation across the VIPER-module layers. 
    
    The results of logic segregation across the VIPER-module layers are shown in the figure below.
    
    ![First example result](../Resources/MVCMassacre/FirstExampleResult.png)

* **Reading data**

    In this example loading of a list of city quarters from the local database ([MagicalRecord](https://github.com/magicalpanda/MagicalRecord) library is used for loading) is implemented in the **viewDidLoad** method of the view controller being reviewed. In this case after loading, managed objects are passed for processing, that can lead to unpredictable behavior of an app in future.
    
    ```objective-c
    - (void)viewDidLoad {
        [super viewDidLoad];
    
        NSManagedObjectContext *context = [NSManagedObjectContext MR_defaultContext];
        NSString *sortTerm = NSStringFromSelector(@selector(priority));
        NSArray *quarters = [AGQuarter MR_findAllSortedBy:sortTerm ascending:YES inContext:context];
        [self handleLoadedQuarters:quarters];
    }
    ```
    
    While refactoring all data reading logic should be moved out of the view layer.For this
    
    1. A new method should be added to the output protocol of the view-layer.

       In this case it can be read as **"Obtain a list of city quarters."**

       ```objective-c
       @protocol AGMainMenuControllerViewOutput <NSObject>
        
       @required
       - (void)obtainQuarters;
        
       @end
       ``` 
       
    2. Carry out refactoring of the original method from the view controller. 

       ```objective-c
       - (void)viewDidLoad {
            [super viewDidLoad];
            [self.output obtainQuarters];
       }
       ```
       Now the view controller transfers control to the presenter, getting rid of logic that is not typical for it.
       
    3. Implement the method added to the protocol inside the presenter of VIPER-module being formed. 

       ```objective-c
       - (void)obtainQuarters {
            [self.view showSpinners];
            [self.interactor loadQuarters];
       }
       ```
       The module presenter transmits a data request to an interactor for processing, after changing the state of view-layer. To do this, the presenter uses the view-layer input protocol, and requests to display a loading progress bar. 
       
    4. Implement data loading inside the interactor.
 
       ```objective-c
       - (void)loadQuarters {
           NSManagedObjectContext *context = [NSManagedObjectContext MR_defaultContext];
           NSString *sortTerm = NSStringFromSelector(@selector(priority));
           NSArray *quarters = [AGQuarter MR_findAllSortedBy:sortTerm
                                                   ascending:YES
                                                   inContext:context];
           NSArray *ponsoQuarters = [self createPlainObjectsFrom:quarters];
           [self.output loadedQuarters:ponsoQuarters];
       }
       ```
       It is necessary to load data from the database, and then on the basis of received managed-objects create "flat" models (**PONSO, Plain Old NSObject**), which are not overloaded with extra features and predictably behave in a concurrent environment. 
        
    5. Pass the data received by the presenter for displaying.

       ```objective-c
       - (void)loadedQuarters:(NSArray *)quarters {
            [self.view hideSpinners];
            [self.view handleObtainedQuarters:quarters];
       }
       ``` 
       Presenter asks the view-layer to the hide progress bar and process the data.
       
    6. Displaying received data. 

       ```objective-c
       - (void)handleObtainedQuarters:(NSArray *)quarters {
            // Displaying quarters
       }
       ``` 
       Concrete implementation is not covered in this example. 
       
    As a result, new implementation will prevent the potential error associated with the use of managed-object in the view-layer, reduce code coupling and improve its testability.
    
    The results of the logic segregation across the VIPER-module layers are shown in the figure below. 
    
    ![Second example result](../Resources/MVCMassacre/SecondExampleResult.png) 
    
* **Objects configuration**

   In this example, the local image cache manager is lazily instantiated in the getter method, defined directly in the view controller.
   
   ```objective-c
   - (id<AGImageLocalCacheManager>)imageLocalCacheManager {
        if (_imageLocalCacheManager == nil) {
            NSFileManager *fileManager = [NSFileManager defaultManager];
            _imageLocalCacheManager = [AGImageLocalCacheManagerImplementation cacheManagerWithFileManager:fileManager];
        }
        return _imageLocalCacheManager;
   }
   ```
   
   But an object should not create needed helper objects by itself - it is fraught with duplication of the code for setting up dependencies. Therefore, special configurators are used to encapsulate this logic, you can use exisiting DI-frameworks for their implementation, for example, [Typhoon](https://github.com/appsquickly/Typhoon). In this instance for refactoring of this example code, you have to do the following
   
   1. Move the cache manager to the interactor layer as far as interaction with the local storage — this is it's responsibility, not the view-layer's. 

      ```objective-c
      @interface AGMainMenuInteractor : NSObject <AGMainMenuInteractorInput>

      @property (nonatomic, strong) id<AGImageLocalCacheManager> imageLocalCacheManager;
    
      @end
      ``` 
      
      The module interactor will keep a strong reference to the cache manager.
      
    2. Move configuration of module dependencies into a separate object — assembly. 

      ```objective-c
      @implementation AGMainMenuAssembly
      
      - (AGMainMenuInteractor *)interactorMainMenu {
            return [TyphoonDefinition withClass:[AGMainMenuInteractor class]
                                  configuration:^(TyphoonDefinition *definition) {
                                      [definition injectProperty:@selector(output)
                                                            with:[self presenterMainMenu]];
                                      [definition injectProperty:@selector(imageLocalCacheManager)
                                                            with:[self.coreHelpersAssembly imageLocalCacheManager]];
                                  }];
      }
      
      @end
      ```
      
      As far as the cache manager is a basic component and is reused between different modules, the good practice is to use separate configurator for it and similar entities. In this example it is **coreHelpersAssembly**.
      
    3. Add creation of the cache manager into the assembly for base objects. 

       ```objective-c
       @implementation AGCoreHelpersAssembly
       
       - (id<AGImageLocalCacheManager>)imageLocalCacheManager {
            return [TyphoonDefinition withClass:[AGImageLocalCacheManagerImplementation class]
                                  configuration:^(TyphoonDefinition *definition) {
                                      [definition useInitializer:@selector(cacheManagerWithFileManager:)
                                                      parameters:^(TyphoonMethod *initializer) {
                                                          [initializer injectParameterWith:[self fileManager]];
                                                      }];
                                  }];
        }
       
       @end
       ```
       
    As a result,the interactor is able to use the cache manager and transmit the data received from it to upper layers.
    
    ```objective-c
    - (void)loadBackgroundImage {
        AGPaperGuide *guide = [AGPaperGuide MR_findFirstInContext:[NSManagedObjectContext MR_defaultContext]];
        UIImage *guideBackgroundImage = [self.imageLocalCacheManager loadImageWithImageId:guide.backgroundImage.photoId];
        [self.output loadedBackgroundImage:guideBackgroundImage];
    }
    ```
    
    The resulting implementation eliminates potential problem of duplication while creating helper objects and increases code testability.
    
    The results of the logic segregation across the VIPER-module layers are shown in the figure below. 
    
    ![Third example result](../Resources/MVCMassacre/ThirdExampleResult.png)  

#### General useful tips to consider when moving from a Massive-View-Controller to a VIPER-module

Here are some tips that are useful when moving from  Massive-View-Controller to a VIPER-module. 

* Think of a view in a VIPER-module not as an object but as a layer which contains a lot of objects. For example, a good manner is to create separate objects in this layer that implement table delegates and most of other protocols. Also it is necessary to use the auxiliary entities for animations. 
* Classes dependent on **UIKit**, **Core Animation** or **Core Graphics** should not go beyond the view-layer.
* Use simple immutable objects in the view layer to interact with data, the implementation of which has no connection with the model layer. This ensures that data passed to the view layer for displaying has been already pre-loaded and can be displayed. 
* Complex views, that are part of a view controller's hierarchy, should implement display logic themselves. As an example, a custom date picker with additional properties for data visualization can be defined in a separate helper class. 
* If a view controller has a lot of responsibilities even after refactoring, a resulting VIPER-module have to be divided into several modules. 

#### Refactoring results    

If you answer "No" to the questions in this simple questionnaire, it probably indicates that the refactoring has been done correctly. 

* Whether a view controller interacts directly with a model?
* Does a view controller contain any business logic?
* Whether a view controller contains logic that is not related to the UI?

On average, refactoring of a single Massive-View-Controller, including writing test, takes 2-3 working days depending on experience and qualifications of a developer. In this case, after transformation of initial implementation into a VIPER-module, problems of Massive-View-Controller mentioned above are solved and understanding of data flow and control is greatly improved by code structuring.

It should be noted that it is not necessarily to transform all view controllers of a project into VIPER-modules. This should be considered, for example, at the moment when it becomes difficult to cover an MVC class by tests or it is difficult to reuse logic between classes. This is possible since VIPER is fully compatible with Model-View-Controller. Those, within the same project there can be controllers written in the MVC style, which are potentially waiting for refactoring, as well as VIPER-modules.

### Additional reading

* [Wikipedia: Model-View-Controller](https://ru.wikipedia.org/wiki/Model-View-Controller). 
* [Wikipedia: Model-View-Adapter](https://en.wikipedia.org/wiki/Model-view-adapter). 
* [Apple's DevPedia: Model-View-Controller](https://developer.apple.com/library/ios/documentation/General/Conceptual/DevPedia-CocoaCore/MVC.html#//apple_ref/doc/uid/TP40008195-CH32). 
* [Apple's CocoaEncyclopedia: Model-View-Controller](https://developer.apple.com/library/ios/documentation/General/Conceptual/CocoaEncyclopedia/Model-View-Controller/Model-View-Controller.html).
* [Apple's documentation: UIViewController class reference](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIViewController_Class/index.html#//apple_ref/doc/uid/TP40006926). 
* [Apple's documentation: view controller for iOS](https://developer.apple.com/library/ios/featuredarticles/ViewControllerPGforiPhoneOS/index.html#//apple_ref/doc/uid/TP40007457).
* [Apple's documentation: view controller design tips](https://developer.apple.com/library/ios/featuredarticles/ViewControllerPGforiPhoneOS/DesignTips.html#//apple_ref/doc/uid/TP40007457-CH5-SW1). 
* [WWDC 2012/236. The evolution of View Controllers on iOS](https://developer.apple.com/videos/play/wwdc2012/236/).
* [WWDC 2014/229. Advanced iOS Application Architecture and Patterns](https://developer.apple.com/videos/play/wwdc2014/229/).
* [iOS architecture patterns by Bohdan Orlov](https://medium.com/ios-os-x-development/ios-architecture-patterns-ecba4c38de52). 
* [objc.io. Issue №1. Lighter view controllers](https://www.objc.io/issues/1-view-controllers/). 
* [Model-View-Controller (MVC) in iOS: A Modern Approach by Rui Peres](https://www.raywenderlich.com/132662/mvc-in-ios-a-modern-approach).
