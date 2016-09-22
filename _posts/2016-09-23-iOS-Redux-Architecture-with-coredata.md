---
layout: post
title: "iOS Redux Architecture with CoreData"
date: 2016-09-23
categories: development
---

# iOS meets Redux

I started to lean Redux architecture recently.
**Redux** is super simple and easy to understand.

I want to user Redux architecture in iOS development, and I found [ReSwift](https://github.com/ReSwift/ReSwift).

**ReSwift** is the powerful framework to create Redux architecture iOS app.
I created a sample app using ReSwift with CoreData.

[GitHub - ReSwiftCoreData](https://github.com/micchyboy1023/ReSwiftCoreData)

---

# Inside the sample app

There is only one entity named `User`.
The sample is a simple application to add and update users.

![model]({{ site.baseurl }}/assets/posts/2016-09-23/model.png)

---

# State, Action, Reducer, and Store

There are four key component in Redux. State, Action, Reducer, and Store.

To understand Redux, read [ReSwift document](http://reswift.github.io/ReSwift/master/about-reswift.html) or [Redux document](http://redux.js.org).


## State

In this sample, I created one State named `AppState`.
`AppState` has all users.

{% highlight swift %}

struct AppState: StateType {

    var users: [User] = []

}

{% endhighlight %}

## Action

There are three actions, `AddUser`, `UpdateUserName` and `UpdateUserAge`.

{% highlight swift %}

struct AddUser: Action {
    let user: User
}

struct UpdateUserName: Action {
    let objectID: NSManagedObjectID
    let name: String

}

struct UpdateUserAge: Action {
    let objectID: NSManagedObjectID
    let age: Int16
}


{% endhighlight %}


## Reducer

`AppReducer` handles actions and create new state.

{% highlight swift %}

struct AppReducer: Reducer {

    func handleAction(action: Action, state: AppState?) -> AppState {

        var state = state ?? AppState()

        switch action {
        case let action as AddUser:
            state.users.append(action.user)

        case let action as UpdateUserName:
            let users = state.users.map({ user -> User in

                if user.objectID.isEqual(action.objectID) {
                    user.name = action.name
                }

                return user
            })

            state.users = users

        case let action as UpdateUserAge:
            let users = state.users.map({ user -> User in

                if user.objectID.isEqual(action.objectID) {
                    user.age = action.age
                }

                return user
            })

            state.users = users

        default:
            break
        }

        return state
    }
}


{% endhighlight %}


## Store

In `AppDelegate`, the store is initialized.

{% highlight swift %}

class AppDelegate: UIResponder, UIApplicationDelegate {

    var window: UIWindow?

    var mainStore: Store<AppState>!


    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
        // Override point for customization after application launch.

        let users = User.fetch(moc: persistentContainer.viewContext)
        mainStore = Store<AppState>(reducer: AppReducer(), state: AppState(users: users))

        return true
    }

    ...

{% endhighlight %}

To fetch saved users, I initialized the store in `func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool`.


# ViewControllers

There are two view controllers in this app, `TableViewController` and `DetailViewController`.

`TableViewController` shows all users.

`DetailViewController` shows selected user info and we can edit user info.

Both view controllers subscribe to mainStore in `viewWillAppear(_ animated: Bool)`, unsubscribe from mainStore in `viewWillDisappear(_ animated: Bool)`, and conform to `StoreSubscriber` protocol.

After reducers handle actions, `newState(state: AppState)` function is called. So we can update UI here depend on the state.

## TableViewController

`TableViewController` has all users in users property.
Inside `newState(state: AppState)`, users update to the latest state, and tableView is reloaded.

`TableViewController` has a right bar button item to add a new user.
When the button tapped, a new user is created and `AddUser` action is dispatched.

{% highlight swift %}

@IBAction func addButtonDidTap(_ sender: AnyObject) {
  let user = User(context: appDelegate.persistentContainer.viewContext)
  user.name = "Albert Einstein"
  user.age = 78
  appDelegate.saveContext()

  //Dispatch Action
  appDelegate.mainStore.dispatch(AddUser(user: user))       
}

{% endhighlight %}


When the row is selected, `TableViewController` shows `DetailViewController`.
In `prepare(for segue: UIStoryboardSegue, sender: Any?)`, user is injected to `DetailViewController`.

{% highlight swift %}

override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
  if segue.identifier == "Show" {
    let vc = segue.destination as! DetailViewController
    vc.user = self.users[self.tableView.indexPathForSelectedRow!.row]
  }
}

{% endhighlight %}

## DetailViewController

`DetailViewController` has two textfield, `nameTextField` and `ageTextField`.

When `nameTextField` is edited, `UpdateUserName` action is dispatched, and when `ageTextField` is edited, `UpdateUserAge` action is dispatched.

{% highlight swift %}

@IBAction func nameTextFieldEditingChanged(_ sender: UITextField) {
  appDelegate.mainStore.dispatch(UpdateUserName(objectID: self.user.objectID, name: sender.text ?? ""))
}


@IBAction func ageTextFieldEditingChanged(_ sender: UITextField) {
  let age = Int16(sender.text!) ?? -1
  appDelegate.mainStore.dispatch(UpdateUserAge(objectID: self.user.objectID, age: age))
}

{% endhighlight %}


If `nameTextField` is empty or `ageTextField` is incorrect, save button is disabled.

{% highlight swift %}

func newState(state: AppState) {

  guard let currentUser = state.users.filter({
      $0.objectID.isEqual(self.user.objectID)
  }).first else {
      return
  }

  self.user = currentUser

  if self.user.name == "" || self.user.age < 0 {
      self.saveButton.isEnabled = false
  } else {
      self.saveButton.isEnabled = true
  }

  self.nameTextField.text = self.user.name
  self.ageTextField.text = self.user.age < 0 ? "" : "\(self.user.age)"
}

{% endhighlight %}


## The Final Project

Please check my GitHub repository.

[GitHub - ReSwiftCoreData](https://github.com/micchyboy1023/ReSwiftCoreData)


---

# NEXT...

I want to integrate Redux with UndoManager.
