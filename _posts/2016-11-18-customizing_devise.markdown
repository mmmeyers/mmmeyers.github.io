---
layout: post
title:  "Customizing Devise"
date:   2016-11-18 10:37:41 -0500
---


When I was creating my Meowbox Rails application, one of the instructions was to create a sign up page that just asked for a user’s first name, last name, username, and email address. Doing so would register the user as a visitor. Then after that, they could sign up as a subscriber to a particular subscription level, setting up a password, which would then change their role to a subscriber. This wasn’t something I was used to, as all of the applications and labs I had worked on had users just sign up once with email addresses or usernames, and passwords.

To accomplish this, I needed to customize Devise, which was one of the hardest parts of this project since Devise does so much ‘magic’ behind the scenes, and I was hesitant to change anything because I wasn’t sure that I could fix it if it broke. I decided to go step by step. 

The first thing I did was to create a registrations controller that inherited from the Devise registrations controller:

`class RegistrationsController < Devise::RegistrationsController`

Next, I needed to tell the router to use that controller:

`devise_for :users, :controllers => { registrations: 'registrations'}`

Then, on the users/new page, I used an if-statement to test if the user was nil. If so, then the devise/registrations/new page would render:

```
<% if current_user.nil? %>
  <%= render "/devise/registrations/new" %>
<% end %>
```

Normally, the Devise `registrations/new` form will ask for an email and password. Since my application wasn’t asking for that information just yet, I had to tweak the form a little to not ask for a password or password confirmation. In the registrations controller, I had to whitelist the :first_name and :last_name params for sign up. The `sign_up_params` method actually comes from the Devise registrations controller:

```
  def sign_up_params
    params.require(:user).permit(:first_name, :last_name, :username, :email, :password, :password_confirmation)
  end
```

To redirect to a specific page after sign-up (registration), I had to tell the registrations controller where to send the user:

```
  def after_sign_up_path_for(resource)
   "/subscriptions"
  end
```
	
To redirect to a specific page for a user’s account that has *not yet been confirmed*, I had to tell the registrations controller where to send that user:

```
  def after_inactive_sign_up_path_for(resource)
   "/subscriptions"
  end
```

Note that both of the above are private methods.

Finally, to redirect to a specific page after a user has successfully logged in, I added the following to the application controller, also as a private method:

```
  def after_sign_in_path_for(resource)
     "/subscriptions"
  end
```


Once the new user registration form was submitted, the user would receive an email to confirm that they did indeed sign up. To simulate the sending and receiving of emails, I used an SMTP server called Mailcatcher. The setup was simple – all I needed to do was install it by running `gem install mailcatcher `in my terminal. Every time I wanted to use it, I just ran `mailcatcher` in my terminal and went to `localhost:1080` in the browser. It will work with any test email you want to send and receive so you can see if your mailers are working and what they look like. Also, be sure to add the `:confirmable` module to your `user.rb` file.

In the confirmation email is a link to ‘Confirm my account’. When the user clicks on it, they are taken back to the Meowbox application to a page to set their password. Here, they will enter their email address, and an email will be sent to create their new password by clicking on a ‘Change my password’ link. This link takes the user back to the Meowbox application once again, where they can enter a password and then confirm it. Once that’s done, an alert appears at the top of the subscriptions index page (root) indicating that they have signed in. This user is now considered a visitor.

To become a subscriber, they would choose the subscription level they want and click on the link to subscribe. All of the fields in the form – first name, last name, username, and email, are auto-filled since we are now updating the user, and Rails knows which user it is by finding it in the user table using the user id.

[https://github.com/plataformatec/devise/wiki/How-To:-Redirect-to-a-specific-page-on-successful-sign-up-(registration)](http://) to the wiki on Devise redirects.
