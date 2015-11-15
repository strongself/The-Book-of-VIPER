**VIPER** - Architecture approach for application development(particularly iOS), based on ideas of [Robert Martin](http://blog.cleancoder.com/), for his article [The Clean Architecture](https://blog.8thlight.com/uncle-bob/2012/08/13/the-clean-architecture.html).

![Clean Architecture](http://i.imgur.com/rt9bUjo.png)

**Main goals of VIPER**:

- Increase of test coverage for Presentation level, usually built from *Massive View Controllers*.
- Partition of most heavy application classes into smaller parts with distinct responsibility.

it is important to note that VIPER is not list of rules and templates. It's list of recomendations how to build flexible, testable and reusable architecture. We, the iOS Team of **Rambler&Co**, have adopted some canonical principles and formed our best practices for some cases.

Firstly VIPER cat break the mind, especially for developers without team work experience. It's because lack of understanding that application parts should be independent and have maximal test coverage. But VIPER is helpful even for small applications.

**Pros and cons of VIPER:**

Pros:

- **Increase of testablity** for application presentation layer.
- **Modules are independent** form each other. It allows separate development of modules and reuse.
- **Main architecture approaches are defined**. So it's much easier to add new developer into team or move project to another team.

Cons:

- Highly **increases class count** in the project, as well as complexity of new module creation.
- Some principles **doesn't work with UIKit** out of the box.
- **Lack of recommendations**, best practices and complex application examples.

Other part of our manual tells more about each of this points. Especially it will be told how to beat those cons.

**VIPER history timeline:**

- **08.2012** - Article [The Clean Architecture](https://blog.8thlight.com/uncle-bob/2012/08/13/the-clean-architecture.html) by Robert Martin
- **12.2013** - Article [Introduction to VIPER](http://mutualmobile.github.io/blog/2013/12/04/viper-introduction/) by [MutualMobile](http://mutualmobile.github.io/)
- **06.2014** - Issue #13. objc.io [Architecting iOS Apps with VIPER](https://www.objc.io/issues/13-architecture/viper/) by MutualMobile
- **07.2014** - [iPhreaks Show podcast](https://itunes.apple.com/ru/podcast/the-iphreaks-show/id634022060?mt=2&i=316803444) by MutualMobile. History of VIPER, goals and usage.
- **04.2015** - Rambler&Co VIPER hackatone leads to first application.
- **12.2015** - Rambler&Co has dozen of VIPER application in development and [released in AppStore](https://itunes.apple.com/ru/developer/rambler-internet-holdings/id395455934).
