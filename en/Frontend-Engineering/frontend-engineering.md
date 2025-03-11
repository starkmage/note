## What Factors Need to be Considered in Frontend Engineering

### Modularization

Simply put, modularization means **breaking down a large file into smaller interdependent files, then performing unified assembly and loading**. This is the only way to enable multi-person collaboration.

#### JS Modularization

Refer to the summary in 'Frontend Modularization'

#### CSS Modularization

You can use @import to import CSS files, but for CSS modularization, one of the most important issues is **the global pollution problem of selectors**.

From a tooling perspective, the community has created three solutions: Shadow DOM, CSS in JS, and CSS Modules.

- Shadow DOM is the WebComponents standard. It can solve the global pollution problem, but currently many browsers are not compatible, so it's still far off for us;
- CSS in JS completely abandons CSS, using JS or JSON to write styles. This method is radical, cannot leverage existing CSS technologies, and handling pseudo-classes and other issues is difficult;
- CSS Modules still uses CSS, but lets JS manage dependencies. It maximizes the combination of CSS ecosystem and JS modularization capabilities, currently appearing to be the best solution. **Vue's scoped style is also considered one type**.

#### Resource Modularization

Webpack's power lies not only in its unification of various JS module systems, but more importantly in its universal module loading concept, meaning that all resources can and should be modularized.

After resource modularization, there are three benefits:

1. Simplified dependency relationships. All CSS and image resource dependencies uniformly follow the JS route, eliminating the need to handle CSS preprocessor dependencies separately or deal with image merging and font image path issues during code migration;
2. Integrated resource processing. Now loaders can do various things with different resources, like the complex vue-loader, etc.
3. Clearer project structure.

### Componentization

First, componentization ≠ modularization.

Modularization is just at the file level, breaking down code or resources; while **componentization is at the design level, breaking down the UI (user interface)**.

**Each structurally complete unit containing template (HTML) + style (CSS) + logic (JS) broken down from the UI is called a component**.

More importantly, componentization is a **divide-and-conquer mindset**.

Everything on the page is a component. A page is a large component that can be broken down into several medium components, then medium components can be further broken down into several small components, and small components can be broken down until they become DOM elements. DOM elements can be seen as the browser's own components, serving as the basic units of components.

Componentization is actually a further abstraction of object-oriented programming in the form of template (HTML) + style (CSS) + logic (JS) trinity. So besides encapsulating the components themselves, we also need to properly handle relationships between components, such as **(logical) inheritance**, **(style) extension**, **(template) nesting** and **inclusion**, etc., all of which can be categorized as **dependencies**.

Componentization framework—Vue

### Standardization

- Directory structure specification
- Coding standards
- Frontend-backend interface specifications
- Documentation standards
- Component management standards
- Git branch management
- Commit description standards

### Automation

Any simple mechanical repetitive labor should be completed by machines

* Automated building
* Automated deployment
* Automated testing

## Reference Articles

[Who can introduce web frontend engineering? - Zhao Yusen's answer - Zhihu](https://www.zhihu.com/question/24558375/answer/139920107)

https://segmentfault.com/a/1190000016226284

https://github.com/fouber/blog/issues/10