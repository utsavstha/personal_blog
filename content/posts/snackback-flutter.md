+++
title = "Snackback Flutter"
date = "2022-07-26T19:52:54+05:45"
author = "Utsav Shrestha"
authorTwitter = "" #do not include @
cover = ""
tags = ["", ""]
keywords = ["flutter", "js", "snackbar"]
description = ""
showFullContent = false
readingTime = false
hideComments = false
+++

# Snackbar for flutter

As an Android developer I love my Toasts (No pun intended). I’ve always found Toasts to be the simplest way to convey information to the user. But with the dawn of Snackbars many android developers have shifted from Toasts to Snackbars, not me. But when I started flutter the first thing I searched for to display messages was toast. But evidently flutter team has done a really good job with Snackbars. They’re highly customizable and very easy to implement.
Lets get started.
Snackbars are often useful to convey information about a specific action to the users. For example, If a user added an item to cart in an ecommerce application a snackbar saying, “Item added to cart” can be used.

Following steps can be followed to create a Snackbar:

- Create a Scaffold.
- Display Snackbar.
- Provide optional action.
  ##Create a Scaffold
  To create a Snackbar in flutter you will have to first create a Scaffold and then add a Snackbar to it.

```js
Scaffold(
 appBar: AppBar(
 title: Text(‘SnackBar Demo’),
 ),
 body: APageThatWillHaveSnackbar(),
);
```

## Display Snackbar

With Scaffold ready, a Snackbar can be displayed using a ScaffoldMessenger.

```js
final snackBar = SnackBar(content: Text(‘Yay! A SnackBar!’));
// Find the ScaffoldMessenger in the widget tree
// and use it to show a SnackBar.
ScaffoldMessenger.of(context).showSnackBar(snackBar);
```

## Provide an action (Optional)

Adding an action inside a snackbar is optional but often can come in handy.

```js
final snackBar = SnackBar(
  content: Text('Yay! A SnackBar!'),
  action: SnackBarAction(
    label: 'Undo',
    onPressed: () {
      // Some code to undo the change.
    },
  ),
);
```

## Complete Code

```js
import ‘package:flutter/material.js’;
void main() => runApp(SnackBarDemo());
class SnackBarDemo extends StatelessWidget {
 @override
 Widget build(BuildContext context) {
  return MaterialApp(
    title: ‘SnackBar Demo’,
    home: Scaffold(
    appBar: AppBar(
    title: Text(‘SnackBar Demo’),
   ),
   body: SnackBarPage(),
   ),
  );
 }
}
class SnackBarPage extends StatelessWidget {
 @override
 Widget build(BuildContext context) {
 return Center(
 child: ElevatedButton(
 onPressed: () {
 final snackBar = SnackBar(
 content: Text(‘Yay! A SnackBar!’),
 action: SnackBarAction(
 label: ‘Undo’,
 onPressed: () {
 // Some code to undo the change.
 },
 ),
 );
// Find the ScaffoldMessenger in the widget tree
 // and use it to show a SnackBar.
 ScaffoldMessenger.of(context).showSnackBar(snackBar);
 },
 child: Text(‘Show SnackBar’),
 ),
 );
 }
}
```

That’s it. You now have a Snackbar, display all those helpful messages to your users.
