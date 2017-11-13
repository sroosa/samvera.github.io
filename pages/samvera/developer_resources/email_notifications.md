---
title: "Email Notifications"
a-z: ["Email Notifications"]
keywords: Email, Notifications
categories: How to Do All the Things - Glossary
permalink: email_notifications.html
folder: samvera/how-to/
sidebar: home_sidebar
tags: [development_resources]
toc: false
---

### Sending Notifications by Email
Hyrax uses a gem called [mailboxer](https://github.com/mailboxer/mailboxer) to send notifications within the application. You can also make it send these notifications by email. Information in this guide is drawn from [Messaging with Rails and Mailboxer](https://www.sitepoint.com/messaging-rails-mailboxer/) by Ilya Bodrov-Krukowski.

#### 1. Ensure you can send email from your application.
There is plenty of documentation for how to configure ActionMailer for various mail services. The [Action Mailer Basics](http://guides.rubyonrails.org/action_mailer_basics.html) guide is a good place to start.

Given a file `config/environments/development.rb`:
```ruby
config.action_mailer.perform_deliveries = true
config.action_mailer.delivery_method = :smtp
config.action_mailer.smtp_settings = {
  address: 'email-smtp.us-east-1.amazonaws.com',
  port: '587',
  user_name: 'MY_USER_NAME',
  password: 'MY_PASSWORD',
  enable_starttls_auto: true
}
```

And a file `app/mailers/test_mailer.rb`:

```ruby
class TestMailer < ApplicationMailer
  def test_email
    mail(
      from: "you@yourdomain.com",
      to: "you@yourdomain.com",
      subject: "Test mail",
      body: "Test mail body"
    )
  end
end
```

On a rails console, you should be able to run `TestMailer.test_email.deliver` and see your email delivered.

#### 2. Tell mailboxer to use email
Edit the initializer for mailboxer at `config/initializers/mailboxer.rb`:

```ruby
Mailboxer.setup do |config|
  #Configures if you application uses or not email sending for Notifications and Messages
  config.uses_emails = true

  #Configures the default from for emails sent for Messages and Notifications
  config.default_from = "no-reply@mailboxer.com"

  #Configures the methods needed by mailboxer
  config.email_method = :mailboxer_email
  config.name_method = :display_name
  [...]
end
```

#### 3. Ensure your user model responds to the mailboxer methods
Now that you can send email from your application, make sure your User model has the methods it needs to mailboxer-style email. In the initializer, note that we set `name_method` to `:display_name`. Devise User objects in Hyrax have a method called display_name that works well here. However, the Devise `email` method won't work because the argument signature doesn't match what mailboxer is expected. Instead, edit `app/models/user.rb` and add a `mailboxer_email method`, like this:

```ruby
# Mailboxer (the notification system) needs the User object to respond to this method
# in order to send emails
def mailboxer_email(_object)
  email
end
```

That's it! Now when your Hyrax app sends notifications, it should send them by email as well as within the application.