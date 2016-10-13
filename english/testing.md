##TDD and VIPER

###Short introduction to TDD

TDD means development through testing. Writing tests firstly, than you can start with realization. If classes written with help of protocols, they goes first. After all components are completed, they can be integrated.

With help of this approach developer reach full description of class behavior through tests. Tests can be used as example of what class can do. Although TDD allows us see how class structured even before we start using it.

More details you can find in [article](http://habrahabr.ru/company/rambler-co/blog/263087/) written by Andrew Rezanov

##VIPER

Usually separation for primary layers is enough for testing backend and Core layer, but if we work with 'huge' ViewController we come up with difficulties in Presentation layer testing. That's the reason why this layer usually have less tests. Let's find out how VIPER modules can help us with View test coverage.

Basic way for it is writing protocols mocks for class dependencies. Than calling interface methods, manipulate properties, check mocks methods calls.

###View testing

You can find several libraries which can help you with UI layer testing, we even have ability to write UI tests in Xcode, but is it all enough? From our point of view - no.

UI tests can help us with writing acceptance tests. Application becomes our black box and we try reach some results through application interface.

Main problem of the UI tests is if we have Massive View Controller whole View is black box with many different states and for testing it we need really big amount of tests.

How VIPER can help us with that? Logic for preparing and presenting data moved to Presenter. After that View responsible only for handling and throwing events into Presenter and displaying UI. You should notice that View isn't only UI, it's class which responsible for different interactions which reaching module. What it means for us? IBOutlet and IBAction **MUST** be public. If you do it you will get ability to test all taps, filed inputs and other things without tap simulation, search needed buttons through theres text and other unreliable things.

So, we can highlight two main test types:

- Interacting with IBOutlet / calling IBAction -> check that Presenter mock methods works correctly
- Calling protocol methods which helps Presenter connect to other classes -> check that IBOutlet / View changed correctly

Don't forget about ViewController / View lifecycle methods, which we should not forget to test either cause Presenter can't start setup View before `-viewDidLoad` or `-viewWillAppear` methods.

###Router testing

Router methods are using for presenting something from current ViewControler. According to that methods for presenting/dismissing current controller should be covered by tests. Although don't forget to test all animation transitions.

###Interactor testing

Interactor is connected to almost all services. It is used for working with Storage, creating PONSO-objects.

Most tests checks services calls dependent on responses from other services.

###Presenter testing

Presenter might be called as 'module linker' cause it handles requests from one part of module to another part of it. Most tests checks this interaction.

Notice that Presenter is module enter point. Presenter gets data from previous controller. And yes, we have to write tests for it.

Presenter is place where often we can find big amount of logic interactions. You should pay attention to it and while writing tests try pass to input different combinations of values.

###Assembly testing

Assembly setting up module components dependencies. Assembly tests responsible for checking that module consists of correct parts and all dependencies fulfilled.

Fortunately, if you have strict module structure, this tests can be created automatically. 