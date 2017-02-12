Creating a new VIPER module can be rather cumbersome and involves at least providing the following:

- **Five new classes** (Assembly, ViewController, Presenter, Interactor, Router),
- **Five new protocols** (ViewInput, ViewOutput, InteractorInput, InteractorOutput, RouterInput),
- **Five new tests** (AssemblyTests, ViewControllerTests, PresenterTests, InteractorTests, RouterTests).

Moreover, all the necessary relations need to be established, protocols need to be implemented, dependency injection set up - and plenty of other things in particular cases. This complexity brings two main problems:

- Too much time spent on routine work
- Typos become more frequent which adversely affects the code style and application logic

One way to tackle the issue is via creating Xcode templates (we were also following this approach some time ago in Rambler&Co). It solves the problems mentioned above, though having a number of disadvantages:

* Building a new template is a complex process due to cumbersome syntax
* Templates might just stop working after an Xcode update
* No convenient way of adding new templates to Xcode (Alcatraz does not count)
* Impossible to add template files to different targets (e.g. when generating auto tests)
* Xcode templates are highly developer dependent, not the project dependent

To be independent of Xcode IDE and its implementation details, we have decided to separate the process of code generation and came up with a small utility application - [Generamba](https://github.com/rambler-digital-solutions/Generamba).

![Generamba](http://s24.postimg.org/gej9cg1cl/generamba.jpg)

---

### Installation

```
gem install generamba
```

### Usage

#### Setting up the project

Before creating a module you need to configure the `Rambafile` by calling `generamba setup` (the file is located in the project’s root folder). It contains project name, prefix, company, module, paths to test folders and a list of used templates. The file is versioned in git and should be used by all developers maintaining the project.

#### Creating a new module

Generation of a new module is performed by calling `generamba gen ModuleName TemplateName`. As a result, it will create all files described in the given template - and they will be added to Xcode project, as well as to the file system. 

#### Working with templates

All templates which are used in the current project is described in `Rambafile`. The call to `generamba template install` will install provided templates one by one - either from a local folder, from a remote git repository or even from template catalog([including shared ones](https://github.com/rambler-digital-solutions/generamba-catalog)). All templates are stored in the project’s `Templates` folder.

The full list of commands and their options can be found in our [wiki](https://github.com/rambler-digital-solutions/Generamba).

The project is open sourced, so everyone who is willing [can help us in development, send us bug reports and share your ideas](https://github.com/rambler-digital-solutions/Generamba/issues). Besides, we will be happy to add new templates to [our catalog](https://github.com/rambler-digital-solutions/generamba-catalog) via Pull Requests.

### Other code generators
- [vipergen](https://github.com/teambox/viper-module-generator)
- [boa](https://github.com/team-supercharge/boa)
