[React Router Official Documentation Translation](https://juejin.cn/post/6854573220470013966)

## Two Modes

* <BrowserRouter>, which is the history mode and is the default
* <HashRouter>, the hash mode

## Route Matching

* When rendering a `<Switch>`, it iterates through all child components (i.e., `<Route>` components) and stops at the first one whose path matches the current URL, then renders that `<Route>` component and ignores all others. Therefore, **you should place more specific route components first**.
* Typically, `<Route path="/">` is placed at the end of the `<Switch>`.

## exact and strict in react-router

* exact: Does not allow matching child routes, but does not control `/`
* strict: Does not care about child routes, but when the path ends with `/`, the URL must also have it to match

https://juejin.cn/post/6844903841217904648

## Hooks

* useHistory()
* useLocation()
* useParams()
* useRouteMatch()