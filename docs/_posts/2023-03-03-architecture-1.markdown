---
layout: post
title:  Mobile Architecture, UIKit
---

Every app has one or more view controllers, which are at the center of almost everything you do. There are two types of view controllers:

- *Content view controllers* (*Contents* for short) display your app’s data.
- *Container view controllers* (*Containers* for short) display other view controllers.

However, some view controllers are of both types, in a typical navigation flow:

```swift
class ExampleViewController: UITableViewController {

    ...

    override func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        ...
        let next = NextViewController.instantiate(...)
        next.delegate = self
        navigationController?.pushViewController(next, animated: true)
    }
}

extension ExampleViewController: NextViewControllerDelegate {

    ...
}
```

As shown, `ExampleViewController` inherits from `UITableViewController`, a *content*. On the other hand, it instantiates and displays `NextViewController`. This is a definite violation of the [single responsibility principle](https://en.wikipedia.org/wiki/Single-responsibility_principle). Fortunately, everything related to other view controllers can be delegated to a real container, `ExampleViewController` is thus simplified to a *content*.

![Linear](/assets/230303/one.png)

But there is a new code smell. The navigation controller is the only real container in the navigation flow, so after simplifying, it is the delegate of all *contents*. In a long run, the navigation controller must be a [god object](https://en.wikipedia.org/wiki/God_object) as the number of *contents* increasing.

![All to one](/assets/230303/two.png)

The view controller hierarchy could be considered as a tree, where leafs are *contents* and branches are *containers*. And there is a negative correlation between the degree and height of a tree node, this is the key to lighten the load of the navigation controller:

1. Group *contents* by purpose, and encapsulate *contents* of each group into a new *container*.

2. Make each new *container* a [facade](https://en.wikipedia.org/wiki/Facade_pattern) masking view controllers it encapsulates and providing a purpose-specific interface.
3. Remove grouped view controllers from the navigation controller, but call them indirectly through interfaces of new *containers*.
4. Examine whether it is necessary to repeat grouping new *containers* by bigger purpose.

There is an [sample](https://github.com/zzyptr/Sample/tree/UIKit): `DivisionViewController` displays a list of divisions and is encapsulated into `AddressSelectionController` to provide hierarchical address selection. In the second round, `AddressSelectionController` and the other two view controllers are grouped together to compose `PaymentMethodController`. As a result, `RootController` no longer talks to underlying view controllers, but just launches `PaymentMethodController`.

![Hierarchical](/assets/230303/three.png)





This is the first in a series of articles on mobile architecture. In the next article, the architecture will be applied in SwiftUI.







### References:

1. [View Controller Programming Guide for iOS](https://developer.apple.com/library/archive/featuredarticles/ViewControllerPGforiPhoneOS)
2. [Creating a custom container view controller](https://developer.apple.com/documentation/uikit/view_controllers/creating_a_custom_container_view_controller)
