Creation of a new module is one of the bottlenecks in VIPER, especially from the viewpoint of an outsider. The bare minimum needed in order to create a new module-screen is:

- **Five new classes** (Assembly, ViewController, Presenter, Interactor, Router),
- **Five new protocols** (ViewInput, ViewOutput, InteractorInput, InteractorOutput, RouterInput),
- **Five new tests** (AssemblyTests, ViewControllerTests, PresenterTests, InteractorTests, RouterTests).

Moreover, all the necessary relations need to be established, protocols need to be implemented, dependency injection set up - and plenty of other things in particular cases. This complexity brings two main problems:

- Too much time spent on simple routine work
- Increased probability of typos, which may harm not only code style but the application logic as well

One of the ways to tackle this issue, which we also have been using in Rambler&Co - is making our own templates for Xcode. Such approach solves all aforementioned issues, but is has a variety of its own disadvantages:

* Building a new template is a complex process due to cumbersome syntax
* Sometimes after an Xcode update templates might just stop working
* No convenient way of adding new templates to Xcode (Alcatraz does not count)
* No way to add template files to different targets (for example, when generating auto tests)
* Templates and code generation set up is oriented toward a specific user, not the project itself

In order to be free of implementation details and the way our IDE works, we have decided to separate the process of code generation and came up with a small utility application - [Generamba](https://github.com/rambler-ios/Generamba).

![Generamba](http://s24.postimg.org/gej9cg1cl/generamba.jpg)

---

### Install

```
gem install generamba
```

### Functionality

#### Project parameters set up

All the parameters needed for the code generation is contained in file `Rambafile` located in project root folder. It is tuned semiautomatically by calling `generamba setup`. The file contains these kinds of data: project name, prefix, company, module and test folders paths, a list of used templates. This file is versioned in git and used by all developers in the project.

#### New module generation

Generation of a new module is performed by calling `generamba gen ModuleName TemplateName`. As a result, it will create all files described in the given template - besides they will be added to Xcode project, as well as to the file system. 

#### Working with templates

All templates which are used in the current project is described in `Rambafile`. The call to `generamba template install` will install provided templates one by one - either from a local folder or from a remote git repository or even from template catalog([including shared ones](https://github.com/rambler-ios/generamba-catalog)). All templates are stored in the project’s `Templates` folder.

The full list of commands and their options can be found in out [wiki](https://github.com/rambler-ios/Generamba/wiki/Available-Commands).

The project is open sourced, so everyone who is willing [can help us in development, send us bug reports and share your ideas](https://github.com/rambler-ios/Generamba/issues). Besides, we will be happy to add new templates to [our catalog](https://github.com/rambler-ios/generamba-catalog) via Pull Requests.

### Other code generators
- [vipergen](https://github.com/teambox/viper-module-generator)
- [boa](https://github.com/team-supercharge/boa)
