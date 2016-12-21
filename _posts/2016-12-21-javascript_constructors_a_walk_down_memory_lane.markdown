---
layout: post
title:  "Javascript Constructors: A Walk Down Memory Lane"
date:   2016-12-21 18:33:25 -0500
---


# A General Explanation

When I first read the requirements for the Rails + jQuery portfolio assessment, I had to pause for a while to remember what constructors were. I couldn’t recall exactly what they were or why and how they were used, other than they reminded me of classes in Ruby.

A constructor is a function that can be used to create an object prototype, kind of like a factory. It saves you the work of creating each object individually, as that wouldn’t adhere to the DRY principle. These new objects will have the same properties and methods, similar to a Ruby class. For example:

```
function Cat(name, breed, color) {
	this.name = name;
	this.breed = breed;
	this.color = color;
}
```

The first important detail is that the constructor function’s name starts with a capital letter, which identifies it as a constructor function. The parameters are the properties of the object, in this case, the name, breed, and color of the cat. A new Cat object will be initialized with name, breed, and color properties:

```
var cat1 = new Cat(“Koa”, “tabby”, “orange”);
var cat2 = new Cat(“T”, “Persian”, “white”);
```

Now when we can call `cat1. name`, we get the value “Koa”, `cat1.breed` is “tabby”, and `cat1.color` is “orange”.

The second important detail is to use the word `new` when creating a new instance of the object (creating new objects from the Cat prototype). If we don’t use the `new` keyword, then we are calling a function that doesn’t return anything. 

We can also add a method to the prototype:

```
Cat.prototype.meow = function() {
return “meow!”
}
```

Calling `cat1.meow();` will return “meow!”

# Using a constructor in my portfolio project

In my Rails + jQuery portfolio project, I have a users index page, and each user listed has a link to a ‘quick view’ to see individual user information without reloading the page.

This is what the User Constructor looked like:

```
function User(id, first_name, last_name, username, email, subscription) {
  this.id = id;
  this.first_name = first_name;
  this.last_name =  last_name;
  this.username = username;
  this.email = email;
  this.subscription = subscription;
}

User.prototype.naming = function() {
  return this.first_name + " " + this.last_name;
}
```

The method on the prototype is simply concatenating the user’s first and last names.

On the user index page, I have a div for each user id:

`<div id="user-<%= user.id %>"></div>`

User 1’s div will be `<div id=”user-1”>`, User 2’s will be `<div id=”user-2”>`, and so on.

Whenever the link to the ‘quick view’ is clicked, we are GET-ting the users.json response. The new user object, which grabs each attribute value from the json, is stored in a variable called `newUserInfo`.

```
var newUserInfo = new User(
	response["id"],
	response["first_name"],
	response["last_name"],
	response["username"],
	response["email"],
	response["subscription"]
);
```

Now I can call `newUserInfo.username` and get that particular user’s username, `newUserInfo.email` and get the user’s email, etc. I can insert this data into the DOM within each user’s div:

```
$(userDiv).html("<h3>" + newUserInfo.naming() + "</h3>" +
	"<h4>" + "Username: " + newUserInfo.username + "</h4>" +
	"<h4>" + "Email: " + newUserInfo.email + "</h4>" +
	"<h4>" + "Subscription Level: " + newUserInfo.subscription.level + "</h4>" +
	"<p>" + "Subscription Description: " + newUserInfo.subscription.description + "</p>" +
	"<span>" + "See more about this user: " + "</span>" +
	"<a href=" + "/users/" + newUserInfo.id + ">" + newUserInfo.username + "</a>" + "<p></p>"
	);

	$(userDiv).toggle();

});
```

I saved some typing by calling the method `naming()` on `newUserInfo` to concatenate the first and last names of each user. Since I had multiple users, this constructor worked just fine to display all of their information.

