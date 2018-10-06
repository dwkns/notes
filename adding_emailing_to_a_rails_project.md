###Adding emailing to a Rails Project.

Add an ActionMailer to your app.

	$ rails g mailer Gmailer create_email
	
Call it with :

	mail = Gmailer.create_email to, from, subject, message
	
In your `Gmailer` ActionMailer

	def create_email to, from, subject, message
	    @from = from
	    @to = to
	    @subject = subject
	    @message = message
	
	    mail(to: to, subject: subject, from: from)
  	end
  	
The instance variables allow access to these properties in `view/gmailer/create_email.html.erb` which is the body of your email.

Use 

	config.action_mailer.delivery_method = :file

in `config/enviroments/development` to write the email to a file in `temp/mails`

Use :

	config.action_mailer.delivery_method = :smtp
	config.action_mailer.perform_deliveries = true
	config.action_mailer.raise_delivery_errors = true
	config.action_mailer.smtp_settings = {
	     :authentication => :plain,
	     :address => "smtp.mailgun.org",
	     :port => 587,
	     :domain => "sandbox4672.mailgun.org",
	     :user_name => "postmaster@sandbox4672.mailgun.org",
	     :password => "8tts4f1y-pb1"
	}

To use the [Mailgun](http://www.mailgun.com) service.

Add `gem 'capybara-email'` to your `Gemfile` to enable email testing. You can then create websteps such as :

		Then(/^an email should be sent to "(.*?)"$/) do |email_address|
		  open_email email_address
		  expect(current_email).not_to be_nil
		end
		
		Then(/^the email should contain "(.*?)"$/) do |content|
		 expect(current_email.body).to have_content content
		end



