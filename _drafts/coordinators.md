
## Flow Coordinators

Modelled after Fowlers [Application Controller](https://martinfowler.com/eaaCatalog/applicationController.html) and heavily inspired by [Soroush Khanlou](http://khanlou.com/2015/10/coordinators-redux/).


> A centralized point for handling screen navigation and the flow of an application.
Martin Fowler


### Protocols

Basic foundation

```
protocol Coordinator: class {
    var childCoordinators: [Coordinator] { get set }

    func begin()
}
```


Navigatoin based "flows"

```
protocol NavigationCoordinator: Coordinator {
    var navigationController: UINavigationController { get }

    init(using navigationController: UINavigationController)
}
```


Modal based "flows"

```
protocol ModelCoordinator: Coordinator {
    var navigationController: UINavigationController { get }
	
	var onComplete: () -> Void { get set }

    init(using navigationController: UINavigationController)
}
```