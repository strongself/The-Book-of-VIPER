## Application Modularization

Any project can be divided into several logical parts with clearly defined functionality. In Instagram app, the main part of the application - news feed, in which we can view the photos, navigate to the screen to create a comment or send a message. The next tab is a list of likes, from which we can move on to a particular photo. Each of these screens is a self-contained part of the application that performs the task well, and knowing how to initiate the transition to the other screens if necessary. We call these kinds of elements to modules.

In the case of Instagram app, especially Profile screen, they are the photos module (1), information about the user module (2), the tabs for switching the photos list to other modules (3).

![Profile Instagram](https://raw.githubusercontent.com/rambler-ios/The-Book-of-VIPER/master/Resources/instagram_example.png)

## VIPER-module structure

In order to perform its mission module, it is necessary to solve several problems. It is required to implement the business logic module, networking, database, render the user interface. VIPER describes the role of each component and how they interact with each other. So, VIPER-module consists of the following components:

- **View:** responsible for displaying data on the screen and notifies the Presenter of the user's actions. But **View** never asks for data. It just gets them from the presenter.

- **Interactor:** contains all the business logic required for this module.

- **Presenter:** receives information about the user's actions from **View** and transforms it into requests to **Router**, **Interactor**. Receives data from **Interactor**, prepares them and sends **View** to display.

- **Entity**: model objects that do not contain any business logic.

- **Router**: responsible for the navigation between the modules.

## What we have changed

VIPER appears in its original form on [Mutual Mobile](https://www.objc.io/issues/13-architecture/viper/). We have worked with this approach, and soon realized that it has a few problems:

1) In the original version of the VIPER, **Wireframe** is responsible for **Routing** component. But it is also responsible for assembly of the module, the transition to which it carries out, and putting down all the dependencies in the module. This is bad because it violates the principle of [Single Responsibility](https://en.wikipedia.org/wiki/Single_responsibility_principle).

So, we decide to split into two parts of **Wireframe**. First, **Router** is responsible only for the transitions between the modules. Second, **Assembly** is responsible for affixing the module assembly and dependencies between all its components. In our projects, we use [Typhoon](https://github.com/appsquickly/Typhoon) for this purpose, a great library for Dependency Injection, and manually **Assembly** of the code is not even called.

2) **Interactor** hides a business logic. It sounds pretty scary, because behind it lies a lot of work, which is split between the specialized classes. Moreover, the same code is often reused within Interactor. We needed a common understanding of how to do it so as not to reinvent their bikes in each project.

We decide to introduce an additional layer of services. **Service** - the object responsible for the work with your specific type of model objects. For example, a news service for the news corresponds to a specific category list, as well as detailed information about each news. Authorization service is responsible for, in fact, authorization, password recovery, update session and so on. In **Service**, in turn, it has dependencies on lower-level objects, responsible for working with the network or database.

**Services** are injected into the **Interactor**. As a result, **Interactor** mainly serves as a facade, interact with the service and sends the resulting data to presenters. We as a team have agreed that we have not go to levels above **Interactor** when using Core Data NSManagedObject. Therefore **Interactor** also occurs NSManagedObject conversion to Plain Old NSObject, then there is a simple NSObject.

3) In the VIPER unit as **View** often acts UIViewController. And sometimes in the controller contains code that is not directly related to the tasks of View. For example, working with tables and collections. These objects are created with protocols. So, it is intended that their implementation should be moved to separate object. But, in the example of the Mutual Mobile code, tables are in the UIViewController directly.

We don't like it, and we decide that the View is in general not a single object and layer. In addition to the controller, this layer may contain additional objects, which are injected into the controller and take on some of his work. An example of such an object in our projects is DataDisplayManager, implements and methods UITableViewDatasource UITableViewDelegate, or their codes for collections.

4) data transmission between the modules is not covered in VIPER original. Our solution to this problem is described in the chapter [the transitions between the modules](ModuleTransitions.md) details but do not dwell on it. It is enough to say that each module can be input and output interfaces - ModuleInput protocols and ModuleOutput. The former is responsible for transferring the input data, such as article identifier for displaying articles. The latter is responsible for transmitting the result of the module. For example, we can send out the item selected by the user when working with the menu settings module.

## The final module circuit

To illustrate all of the above, we suggest to familiarize with the final scheme VIPER unit. Don't be afraid. Not every module must contain a number of objects. The purpose of this scheme is the most fully displaying our approach to architecture as an example of a complex module.

![VIPER Driving Module](https://raw.githubusercontent.com/rambler-ios/The-Book-of-VIPER/master/Resources/module_structure.png)
