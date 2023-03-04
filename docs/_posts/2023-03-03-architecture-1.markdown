---
layout: post
title:  Mobile Architecture, UIKit
---

Every app has one or more view controllers which are at the center of almost everything you do. There are two types of view controllers:

- *Content view controllers* display your app’s data.
- *Container view controllers* display other view controllers.

In view of the above, a view controller is either a content view controller or a container view controller. However, some view controllers are of both types, in a typical navigation flow:

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

`ExampleViewController` inherits from a content view controller. On the other hand, it instantiates and displays `NextViewController`. This is a definite violation of the [single responsibility principle](https://en.wikipedia.org/wiki/Single-responsibility_principle). Fortunately, everything related to other view controllers can be delegated to a real container, `ExampleViewController` is thus simplified to a content view controller.

![one](/assets/230303/one.png)

After simplifying, the delegate of all content view controllers is the navigation controller, the only real container in the navigation flow. The navigation controller will become heavier as the number of content view controllers increases, and will become a [god object](https://en.wikipedia.org/wiki/God_object) someday.

![two](/assets/230303/two.png)

There is a solution to improve scalability by better arranging content view controllers:

- Group content view controllers by purpose.
- Encapsulate content view controllers of each group into a new container view controller.
- Make new container view controllers a mediator between the navigation controller and content view controllers.

For example, there are two content view controllers that display `Card` and `BillingAddress` which are the components of `PaymentMethod`. Based on the above solution, the two content view controllers will be grouped together and encapsulated into `PaymentMethodController`. Instead of talking to them, the navigation controller displays `PaymentMethodController` and receives `PaymentMethod` from it. The sample code is [here](https://github.com/zzyptr/sample-uikit).

![three](/assets/230303/three.png)

After refactoring, more container view controllers are introduced to share the work of the navigation controller. The view controller hierarchy is restructured into a tree, where leaf nodes are content view controllers and parent nodes are container view controllers. Other benefits of refactoring:

- Clearer role division between two types of view controllers, content view controllers thus are more reusable.
- No dependency between a view controller and its sibling, this means it is easy to partial compile, run and debug, the flow of developing is more agile.
- Evolvable, it is easy to separate a large container view controller into several smaller container view controllers (a [divide and conquer strategy](https://en.wikipedia.org/wiki/Divide_and_conquer_algorithm)).





### References:

1. [View Controller Programming Guide for iOS](https://developer.apple.com/library/archive/featuredarticles/ViewControllerPGforiPhoneOS)
2. [Creating a custom container view controller](https://developer.apple.com/documentation/uikit/view_controllers/creating_a_custom_container_view_controller)
