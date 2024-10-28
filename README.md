## On Waterfall, Agile, and Writing Programs (Somewhat) Declaratively

> I've been having some thoughts lately and wanted to share them to see whether anyone is working in
> this direction. I haven't written any code on this yet — it's all just speculation at this point.
> Just let me know if this rings any bells for you.

### Mixing Agile and Waterfall
First off, I am working with two teams in parallel. One team uses a waterfall-like approach with careful selection of features to support, capturing requirements with presentations, mockups, and endless discussions going back and forth. Typically, after several months, not a single line of code has been written yet. However, coding itself is simple, as all the technicalities have already been fleshed out beforehand. The other team employs an agile-like approach with bottom-up design, rapid iteration and testing, leading to endless code refactoring and rewrites as the team discovers new functionality based on feedback and realizes it does not fit into the existing architecture.

The requirements capturing phase is useful; we better understand our problem (that’s enough of a reason by itself, really). It also helps to automate some mundane tasks down the road(such as scaffolding with Yeoman-like tools, API generation) and write safer code(by automatically checking pre- and post-conditions, etc). Here’s the first idea: what if we write these requirements in a way that allows them to act as a minimal proof of concept demo right away, instead of being just “plain old” presentations or diagrams? Think of this as a mishmash of two approaches, taking the best of both worlds. It’s close to RAD tools and the idea behind Smalltalk—rapidly exploring main points and working out the details later—but not quite. Let me explain.

### Specialization Axis
High-level requirements have to be captured in a declarative, semantic form, and the industry as a whole is indeed slowly moving toward using it everywhere. See for yourself: we describe UI declaratively (see QML and XAML), infrastructure (Puppet’s JSON-like syntax, Kubernetes’ YAML), settings (YAML), diagrams (DrawIO’s XML), technical docs (DocBook, DITA), interprocess communication (Google Protobuf), project building (Gradle, Make), and dependency injection (XML in Spring), except for the code itself. We are close to having a unified declarative language to cover everything, but we’re not quite there yet. Anyway, syntactically, it boils down to describing the properties of an object (think of attributes in XML) and its children, if any, to express nested structures. Think of two axes in XML: a horizontal one to represent attributes and a vertical one to represent children. This leads to the next idea: to introduce a third axis, the specialization axis.

This way, we can reimagine diagrams, docs, and code itself as specializations of an abstract high-level description. I’ll take state machine formalism as an example; there are many others, ofc. Suppose we are developing an ATM interface. We start by capturing states like `Idle`, `CardInserted`, `ShowBalance`, and transitions like `insert-card` and `enter-pin`. This immediately pays off by allowing us to reason about UX correctness (hardly anyone talks about this aspect), among other things, enforcing that there is a path from any state to the initial Idle state. In other words, no matter how the transaction goes, the ATM will eventually be ready to serve another client.
Having this model in place, we can now specialize each state, such as `<state title="Idle">` (in XML syntax) as `<<rectangle fontColor="#666666" fontSize="17">>` to represent it as a rectangle in a diagram, as `<<frame>><GridLayout/></frame>>` for GUI specialization, and as `<<section>>`  in  tech-docs specialization. Transitions could be specialized as entry points for API specialization, as command-line arguments for the terminal interface, as buttons in the GUI, as database queries and as object methods that implement actual processing in raw code.

Here, I used `<< >>` marks to distinguish specialized tags. The particular syntax is not important; the idea is to have a main tree with top-level (upstream) definitions and also others to define semi-independent(but synchronized) specializations (downstream). You would never see all of them at the same time(I remember working with spaghetti of PHP/HTML/CSS/JS/SQL/regexp too vividly); instead, you either request the upstream tree or one of the specializations, and it gets built automatically by replacing each upstream node with its specialization, if any. When requesting a diagram, you’d get a valid and styled diagram without any details spilling over from other trees.  Note that if a specialization doesn't exist yet, in the simplest case, each node could be substituted with its default implementation. Available transitions in each state could be represented, let’s say, by a horizontal row of buttons. This will do for demo purposes just fine.

We are looking at a system to manage (declarative, semantic) trees, so some (but not all) changes in one downstream tree would propagate (merge) into upstream and consequently be reflected in other trees as well, making them all synchronized. You won’t merge any conflicting changes (like removing a node in a diagram that already has corresponding specializations elsewhere) until you decide how to resolve them. That’s different from tools that just skip tags(or metadata in general) they don’t understand. Seeing how complicated modern Git has become, this approach would likely have a lot of hidden edge cases also, but the idea of working on different aspects collaboratively with different representations while keeping them in sync is a compelling one, don’t you think? This would require having a unified declarative syntax for all aspects from UI to database queries, think of going full XML or JSON extended to support specializing. 

### Put Code into Context
A major stumbling block in the top-down approach is that the code is always considered the ultimate source of truth; the mere implementation becomes its own specification. We generate docs and diagrams from the code, not the other way around; otherwise, the docs become outdated at some point. Here comes the next idea: what if we reverse this and allow the code, documentation, diagrams, and UX specializations to compile (or interpret) together? In other words, let the code exist as part of a larger context. The dev process would be seen as incrementally adding more implementation details instead of rewriting existing ones. Existing languages are admittedly not very well suited for this, but depending on the language, we can do the following:
  * **Interfaces**: _Compiling_  docs or UI would actually mean generation of interfaces for the raw code to provide implementation for. 
  * **Decorators**: Many common tasks, such as user authorization or input validation, are DSL-friendly and can easily be moved upstream. Decorators can be autogenerated to check these requirements and be applied on top of raw functions automatically.
  * **Dependency Injection**: The idea is to discourage (or even forbid) instantiating classes directly, deriving instead which (and how) classes to instantiate from upstream and passing them to raw code as parameters.
  * **Permissions, contracts, and policies**: These can be enforced by static checkers, fuzzy testers, and similar tools by deriving and generating specifications in tool-specific formats from upstream.

### In Short
To summarize, the idea is to improve the development process by starting with capturing high-level requirements in an “active” unified declarative format, before any development, that can simultaneously serve either as a diagram, UI mockup, or a bare-bones demo to offset the time and efforts needed for the initial design. These representations will remain in a coherent state, with changes in one form being reflected in others, managed by tooling that supports the specialization axis technique. Raw implementation code will be augmented or statically checked to ensure it’s properly harmonized with the rest of the project.  This requires a unified format in the first place to represent all artifacts from UI to database queries and the development of compatible tooling. The idea absolutely not new; it’s rather another take on RAD tools, design by contract, type first programming, code generation from UML, DSLs, etc.
