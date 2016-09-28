### Foreword

There are lots of articles, presentations and videos related to VIPER in the Internet. Some of them are good and helpful, some are harmful. But it's better to study most of them.

**Important notice:** Materials in this list doesn't mean that we agree with authors and endorse it's ideas. It's possible that we just like fonts and found some jokes funny.

### Articles
- [objc.io #13 - Architecting iOS Apps with VIPER](http://www.objc.io/issues/13-architecture/viper/)

  **Authors:** Jeff Gilbert, Conrad Stoll.

  **Review:** *Unfading classics you should already read. Two things makes it famous: Firstly it was written it times of true objc.io, secondly it was written by the author of VIPER Jeff Gilbert.*

  *VIPER realization in this article should not be taken seriously. It's not optimal and buggy in some cases. But it has example projects on ObjC and SWIFT :)*

- [Introduction to VIPER](http://mutualmobile.github.io/blog/2013/12/04/viper-introduction/)

  **Author:** Jeff Gilbert.

  **Review:**   *Another must-read article, MutualMobile tells about their way to VIPER. You should pay attention to first paragraphs. There Jeff tells that use of such architecture was forced by need of UI tests. Also there's nice point about naming - first VIP letters and think out of E and R.*

  *But we're not agree with some points. Especially with Wireframe concepts, total prohibition of ManagedObject use out of Interactor and direct use of DataStore.*

- [Brigade’s Experience Using an MVC Alternative](https://medium.com/brigade-engineering/brigades-experience-using-an-mvc-alternative-36ef1601a41f)

  **Author:** Ryan Quan.

  **Review:** *On our opinion it's highest bidder to role of "The best introduction to VIPER". Good test, simple schemes, clear description of main ideas and principles. The one from popular tutorials, that advises to move business logic into service layer. This article is recomended at motivation material for your team, family and friends.*

  *Of course, here we have Wireframe. Moreover, move of DataManager(we call it ServiceFacade) from Interactor is so rare case, to advice it usage in most modules.*

- [The Clean Architecture](http://blog.8thlight.com/uncle-bob/2012/08/13/the-clean-architecture.html)

  **Authors:** Robert Martin.

  **Review:** *It's not about VIPER itself, but definitely must read. Uncle Bob talks about clean arcitecture and DI, draws circles and do lots of interesting things.*

  *It's hardly to use such architecture in our harsh reality, but it was exactly the thing that lead MutualMobile to create VIPER.*

### Podcasts
- [iPhreaks Show - VIPER](https://itunes.apple.com/ru/podcast/the-iphreaks-show/id634022060?mt=2&i=316803444)

  **Participants:** Conrad Stoll, Jeff Gilbert.

  **Review:** *Many things could be different if ideas from this podcast were told in the Objc.io article. Authors of VIPER tells about their motivation, refactoring approaches, creation of composite screens, testing and many other things. This podcast is one of the best things you can find about VIPER.*

### Videos
- [250 Days Shipping With Swift and VIPER](https://realm.io/news/altconf-brice-pollock-250-days-shipping-with-swift-and-viper/)

  **Speaker:** Brice Pollock.

  **Review:** *Сheerfully, Funny, SWIFTy. Coursera developer tells about VIPER experience. The team was not satisfied with canonical model and appended ViewModel, EventHandler and FlowController to it. Looks interesting, but data flow inside module looks frighteningly.*

- [Clean Architecture - VIPER](https://www.youtube.com/watch?v=OX4rLAJC7lw)

  **Speaker:** Sergi Gracia.

  **Review:** *Some information about elements responsibility, little more about testing, few words about SOLID and lots about Redbooth office and team. And great fonts in the keynote. Nothing special, just another one introduction into VIPER concept.*

  *The main claim is quality of the Video. It definitely should be seen with [slides in parallel](https://speakerdeck.com/sergigracia/clean-architecture-viper).*
