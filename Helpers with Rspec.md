Spurious example of helper methods with RSpec tests in rails.

Open `spec/helpers/your_controller_helper_spec.rb`



Add in your test : 

	require 'spec_helper'

	describe YourHelper do

    	it "will return the days ago for the 29th" do
    	  date =  Date.parse('2013-11-29')
    	  expect( helper.how_long_ago date ).to eq "4 days ago"
   		end

    	it "will return the days ago for the 28th " do
      	  date =  Date.parse('2013-11-28')
      	  expect( helper.how_long_ago date ).to eq "5 days ago"
    	end 
	end
	

Create your helper in `app/helpers/your_controller_helper.rb`


	module YourHelper
      def how_long_ago date_visited, todays_date = DateTime.now
        days_ago = distance_of_time_in_words date_visited, todays_date
        "#{days_ago} ago" 
      end
    end

Call your helper from your app/views/your_view/show.html.erb


	<p>Posted : <%= how_long_ag(date_visited, DateTime.now)%> </p>
	