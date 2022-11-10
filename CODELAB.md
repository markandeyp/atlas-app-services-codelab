summary: Synced apps with Flutter and Atlas App Services
id: synced-app-with-flutter-realm
categories: Flutter, Atlas App Services
environments: Web
status: Published
tags: flutter, mongodb, realm, atlas sync
authors: Markandey Pathak
feedback link: markandeypathak@live.com
# Synced apps with Flutter and Atlas App Services

## Overview
Duration: 0:02:00

Welcome to the workshop **Synced Apps with Flutter and Atlas App Services**. In this workshop, we will learn about what Atlas App Services are and how we can leverage them to build a synced _Shopping List_ application capable of working offline. 

### What we will build
![Login Screen](synced-app-with-flutter-realm/assets/images/login_screen.png)

![Login Screen](synced-app-with-flutter-realm/assets/images/home_screen.png)

![Login Screen](synced-app-with-flutter-realm/assets/images/add_item_screen.png)

### What will you need?
1. A **can-do** attitude
2. A **MongoDB** Account (we will create a _Free account_ if you don't already have one)
3. A machine with **Flutter** SDK installed
4. A **simulator** configured (Android/iPhone/MacOS)

## Atlas App Services
Duration: 0:05:00

_**Atlas Application Services**_ are fully-managed backend services and APIs that help you build apps, integrate services, and connect to your Atlas data faster.

### Atlas Device Sync
You're developing a mobile app. Your users want their data saved in the cloud and accessible from their other devices. Network access on a mobile device can be intermittent, so you write data locally on the device first. A background process then synchronizes the data to the cloud and resolves any conflicting writes.

Atlas Device Sync provides all of the above, so you can build better apps faster.

### Key Features
1. **Realm in the Front, MongoDB in the Back** - Atlas Device Sync is a bridge between client apps using the Realm SDKs and a MongoDB instance running in Atlas. Realm is a lightweight database optimized for mobile development.
2. **Robust & Secure** - Device Sync handles conflicts for you, so you don't have to write complex custom code to resolve conflicting writes from multiple clients. A user-based permissions system lets you control who can access which data.
3. **"Always-On"** **Experience** - Realm Database and Device Sync seamlessly handle intermittent connectivity so users can continue using your app regardless of their current network status.

## Creating an Atlas App
Duration: 0:05:00

### Create (or login to) your MongoDB account
Head [here](https://account.mongodb.com/account/login) to login or create a MongoDB account

### Create a cluster
![Create a cluster](synced-app-with-flutter-realm/assets/images/create_cluster.png)

### Create a database user (optional)
![Create a database user](synced-app-with-flutter-realm/assets/images/create_db_user.png)

### Allow network access (optional)
![Allow network access](synced-app-with-flutter-realm/assets/images/set_network.png)

### Create a new App
![Create a new App](synced-app-with-flutter-realm/assets/images/create_An_App.png)

![Create a blank App](synced-app-with-flutter-realm/assets/images/blank_app.png)
## Enabling Email Authentication
Duration: 0:02:00
![Enable email authentication](synced-app-with-flutter-realm/assets/images/enable_authentication.png)

## Creating a Flutter App
Duration: 0:03:00
### Create a new flutter project
```bash
flutter create shopping_buddy
```

Above command will generate the boilerplate flutter project. Open the project folder with your favorite IDE. The folder structure will look something like below

![A new flutter project structure](synced-app-with-flutter-realm/assets/images/project_folder_structure.png)

## Adding Realm package
Duration: 0:02:00

Now that we have our project created, to use realm SDK, we need to add the Realm as a dependency. Add below line in your `pubspec.yaml`

```yaml
  realm: ^0.6.0+beta
```

Run below command to download the dependencies. 

```bash
flutter pub get
```

Alternatively, you can run the below command to add and download the Realm SDK

```bash
flutter pub add realm
```

## Creating Atlas App
Duration: 0:02:00
Open `main.dart` and replace the contents with the code below

```dart
import 'package:flutter/material.dart';
import 'package:realm/realm.dart';
import 'package:realm_sync_demo/screens/login_screen.dart';

const appId = "flutter-realm-app-vshsp";

void main() {
  final App atlasApp = App(AppConfiguration(appId));
  runApp(MyApp(
    atlasApp: atlasApp,
  ));
}

class MyApp extends StatelessWidget {
  final App atlasApp;

  const MyApp({super.key, required this.atlasApp});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Shopping List',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: LoginScreen(atlasApp: atlasApp),
    );
  }
}
```

## User Login
Duration: 0:05:00

```dart
final Credentials credentials = Credentials.emailPassword(emailController.text, passwordController.text);
await atlasApp.logIn(credentials);
```

### The Login Screen

Let's create the login screen to allow the user to login using email and password. Create a directory named `screens` under `lib`. This directory will contain the screens for our application. Create a file named `login_screen.dart` and paste the below code.

```dart
import 'package:flutter/foundation.dart';
import 'package:flutter/material.dart';
import 'package:realm/realm.dart';
import 'package:realm_sync_demo/screens/home_screen.dart';
import 'package:realm_sync_demo/services/item_service.dart';

class LoginScreen extends StatelessWidget {
  final App atlasApp;
  final emailController = TextEditingController();
  final passwordController = TextEditingController();

  LoginScreen({Key? key, required this.atlasApp}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Padding(
          padding: const EdgeInsets.all(20),
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              const Icon(
                Icons.shopping_bag,
                size: 100,
                color: Colors.blueAccent,
              ),
              const SizedBox(
                height: 30,
              ),
              const Text("Login",
                  style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold)),
              const SizedBox(
                height: 30,
              ),
              TextField(
                  controller: emailController,
                  decoration: const InputDecoration(
                      hintText: "Email", border: OutlineInputBorder())),
              const SizedBox(
                height: 15,
              ),
              TextField(
                controller: passwordController,
                decoration: const InputDecoration(
                    hintText: "Password", border: OutlineInputBorder()),
                obscureText: true,
              ),
              const SizedBox(
                height: 15,
              ),
              ElevatedButton(
                  style: ElevatedButton.styleFrom(
                      padding: const EdgeInsets.all(20)),
                  onPressed: () async {
                    try {
                      final navigator = Navigator.of(context);
                      final Credentials credentials = Credentials.emailPassword(
                          emailController.text, passwordController.text);
                      await atlasApp.logIn(credentials);
                      if (atlasApp.currentUser != null) {
                        navigator.pushReplacement(
                            MaterialPageRoute(builder: (BuildContext context) {
                          return HomeScreen(
                              atlasApp: atlasApp,
                              itemService: ItemService(atlasApp.currentUser!));
                        }));
                      }
                    } on RealmException catch (error) {
                      if (kDebugMode) {
                        print("Error during login ${error.message}");
                      }
                    }
                  },
                  child: const Text("Login", style: TextStyle(fontSize: 20)))
            ],
          ),
        ),
      ),
    );
  }
}
```

## Creating Schema
Duration: 0:05:00

After login, we need to allow the users to manage their shopping list. To do this, we first will create schemas which will control the structure of our data.

Create a directory `schemas` under `lib`. Create a file named `item.dart` and paste the below code.

```dart
import 'package:realm/realm.dart';

part 'item.g.dart';

@RealmModel()
class _Item {
  @PrimaryKey()
  @MapTo("_id")
  late ObjectId id;
  late String text;
  late bool done;
  @MapTo("user_id")
  late String userId;
}
```

To generate the `RealmObject` based on the above model, run the below command. This will generate another file named `item.g.dart` which will be an extension of your `_Item` class declared above.

```bash
flutter pub run realm generate
```

## Enabling Flexible Sync
Duration: 0:05:00

After declaring the schema, we can start saving the data locally. However, we are building a synced app. In order to enable the sync between devices, we need to configure a flexible sync for our Realm application.

On MongoDB console UI, choose **_Device Sync_** option from left sidebar. We will use default configurations to enable a flexible sync.

![Enable Flexible Sync](synced-app-with-flutter-realm/assets/images/enable_sync.png)

## Opening a Synced Realm
Duration: 0:10:00

Now that we have our sync configured, we need to open a synced Realm in our application. 

```dart
Realm openRealm() {
    var realmConfig = Configuration.flexibleSync(user, [Item.schema]);
    var realm = Realm(realmConfig);
    realm.subscriptions.update((mutableSubscriptions) {
      mutableSubscriptions.add(realm.all<Item>());
    });
    return realm;
}
```

### Reading/Adding/Updating/Deleting Items

```dart
RealmResults<Item> getItems() {
    return realm.all<Item>();
}

add(String text) {
    realm
        .write(() => {realm.add<Item>(Item(ObjectId(), text, false, user.id))});
}

toggleStatus(Item item) {
    realm.write(() => {item.done = !item.done});
}

delete(Item item) {
    realm.write(() => {realm.delete(item)});
}
```

Create a directory `services` under `lib`. Create a new file named `item_service.dart` and paste the code below. This will contain a service class to easily work with our Realm.

```dart
import 'package:realm/realm.dart';
import '../schemas/item.dart';

class ItemService {
  final User user;
  late final Realm realm;

  ItemService(this.user) {
    realm = openRealm();
  }

  Realm openRealm() {
    var realmConfig = Configuration.flexibleSync(user, [Item.schema]);
    var realm = Realm(realmConfig);
    realm.subscriptions.update((mutableSubscriptions) {
      mutableSubscriptions.add(realm.all<Item>());
    });
    return realm;
  }

  RealmResults<Item> getItems() {
    return realm.all<Item>();
  }

  add(String text) {
    realm
        .write(() => {realm.add<Item>(Item(ObjectId(), text, false, user.id))});
  }

  toggleStatus(Item item) {
    realm.write(() => {item.done = !item.done});
  }

  delete(Item item) {
    realm.write(() => {realm.delete(item)});
  }
}
```

## The Home Screen
Duration: 0:10:00

We now will add another screen where the user can see the shopping list, add a new item and update/delete existing items.

Create a new file under `screens` directory named `home_screen.dart` and paste the code below:

```dart
import 'package:flutter/foundation.dart';
import 'package:flutter/material.dart';
import 'package:realm/realm.dart';
import 'package:realm_sync_demo/services/item_service.dart';

import '../schemas/item.dart';
import '../styles.dart';

class HomeScreen extends StatefulWidget {
  final App atlasApp;
  final ItemService itemService;

  const HomeScreen(
      {Key? key, required this.atlasApp, required this.itemService})
      : super(key: key);

  @override
  State<HomeScreen> createState() => _HomeScreenState();
}

class _HomeScreenState extends State<HomeScreen> {
  final _formKey = GlobalKey<FormState>();
  late RealmResults<Item> allItems;

  @override
  void initState() {
    super.initState();
    allItems = widget.itemService.getItems();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text("Shopping List")),
      body: StreamBuilder(
        stream: allItems.changes,
        builder: (BuildContext context,
            AsyncSnapshot<RealmResultsChanges<Item>> snapshot) {
          List<Item> items = [];
          if (snapshot.hasData) {
            items = snapshot.data!.results.toList();
          }

          return ListView.separated(
              itemBuilder: (BuildContext context, int index) {
                return ListTile(
                  tileColor: items[index].done
                      ? Colors.green.shade50
                      : Colors.transparent,
                  minLeadingWidth: 50,
                  leading: IconButton(
                    icon: Icon(
                      items[index].done
                          ? Icons.close_outlined
                          : Icons.check_outlined,
                      color: items[index].done
                          ? Colors.grey.shade500
                          : Colors.green.shade500,
                      semanticLabel: "Mark done",
                    ),
                    onPressed: () {
                      widget.itemService.toggleStatus(items[index]);
                    },
                  ),
                  trailing: IconButton(
                    icon: Icon(
                      Icons.delete_outline,
                      color: Colors.red.shade500,
                      semanticLabel: "Delete item",
                    ),
                    onPressed: () {
                      widget.itemService.delete(items[index]);
                    },
                  ),
                  title: Text(items[index].text),
                );
              },
              separatorBuilder: (BuildContext context, int index) {
                return Divider(
                    height: 0, thickness: 1, color: Colors.grey.shade400);
              },
              itemCount: items.length);
        },
      ),
      floatingActionButtonLocation: FloatingActionButtonLocation.centerFloat,
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          showDialog(context: context, builder: dialogBuilder);
        },
        tooltip: "Add item",
        child: const Icon(
          Icons.add,
          size: 50,
        ),
      ),
    );
  }

  Widget dialogBuilder(BuildContext context) {
    late String itemName;

    return Dialog(
        backgroundColor: Colors.transparent,
        child: Container(
          decoration: BoxDecoration(
              color: Colors.white, borderRadius: BorderRadius.circular(10)),
          padding: Styles.largePadding,
          child: Wrap(
            children: [
              Form(
                  key: _formKey,
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      ...createTextFormField("Item name",
                          placeholder: "Enter item name",
                          isRequired: true, onSaved: (value) {
                        itemName = value!;
                      }),
                      const SizedBox(
                        height: 10,
                      ),
                      Center(
                        child: ElevatedButton(
                            style: ElevatedButton.styleFrom(
                                padding: Styles.largePadding),
                            onPressed: () {
                              if (_formKey.currentState!.validate()) {
                                _formKey.currentState!.save();
                                if (kDebugMode) {
                                  print(itemName);
                                }
                                widget.itemService.add(itemName);

                                Navigator.pop(context);
                              }
                            },
                            child: const Text(
                              "Add",
                              style: Styles.label,
                            )),
                      )
                    ],
                  )),
            ],
          ),
        ));
  }

  List<Widget> createTextFormField(String label,
      {String? placeholder,
      bool isRequired = false,
      Function(String?)? onSaved}) {
    return [
      Text(label, style: Styles.label),
      const SizedBox(
        height: 5,
      ),
      TextFormField(
        onSaved: onSaved,
        validator: (value) {
          return isRequired && (value == null || value.isEmpty)
              ? "$label is required"
              : null;
        },
        decoration: InputDecoration(hintText: placeholder),
      )
    ];
  }
}
```

## Summary
Duration: 0:03:00

## Congratulation
Duration: 0:01:00

![Congratulations](synced-app-with-flutter-realm/assets/images/congrats.png)