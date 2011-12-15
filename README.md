Simple User Messaging System UI using the Mailboxer plugin (https://github.com/ging/mailboxer).

Overview of features:
- To send messages internally between users only, in email style
- Folders: Inbox, Sent, Drafts, Trash. List showing vital info and each is sortable. Checkbox to do delete multiple emails.
- General email search
- Compose new message with: multiple receivers, multiple attachments. If this is a reply, quote the replied message.
- Message view (1 message only, no threading gmail style), options to download attachment and delete the message.

This Messaging System also exists as a Refinery CMS engine

Installing the gem
==================

Include gems and run the bundle command

````ruby
# Gemfile
gem 'messaging', git: 'git://github.com/frodefi/rails-messaging.git'
gem 'mailboxer', git: 'git://github.com/dickeytk/mailboxer.git'
````

Install messaging

````
$ bundle install
$ rails generate messaging:install
````

*Overwrite model file when asked*

Run migrations

````
$ rake db:migrate
````

Set the default host and set a root route (required for Devise)

````ruby
# config/environments/development.rb
config.action_mailer.default_url_options = { :host => 'localhost:3000' }
````

These steps will work for standard Rails applications.

Make the rails-messaging work for RefineryCMS 2.0
===============

Remove this line from /config/routes.rb

````ruby
devise_for :messaging_users
````

Replace content of /app/models/messaging_user.rb as below

````ruby
class MessagingUser < Refinery::User

end
````

Create /app/decorators/models/messaging/message_decorator.rb and insert the below content

````ruby
Messaging::Message.class_eval do

  def recipients=(string='')
    @recipient_list = []
    string.split(',').each do |s|
      unless s.blank?
        recipient = Refinery::User.find_by_email(s.strip)
        @recipient_list << recipient if recipient
      end
    end
  end

end
````

Create /app/decorators/models/refinery/user_decorator.rb and insert the below content

````ruby
Refinery::User.class_eval do
  include Mailboxer::Models::Messageable
  acts_as_messageable

  def name
    self.to_s
  end

  def mailboxer_email(message)
    email
  end

  def to_s
    email
  end
end
````

Create /app/decorators/controllers/messaging/application_controller_decorator.rb and insert the below content

````ruby
Messaging::ApplicationController.class_eval do
  skip_filter :authenticate_messaging_user!
  before_filter :login_required
  helper_method :current_messaging_user

  protected
  def login_required
    redirect_to main_app.new_refinery_user_session_path unless current_refinery_user
  end

  def current_messaging_user
    current_refinery_user
  end
end
````

Enabling Search
===============

To enable search, install the config file

````
$ rails g sunspot_rails:install
````

Install the development solr server

````ruby
# Gemfile
gem 'sunspot_solr'
````

Start the development solr server

````
$ rake sunspot:solr:run
````

Enable search in mailboxer

````ruby
# config/initializers/mailboxer.rb
config.search_enabled = true
````
