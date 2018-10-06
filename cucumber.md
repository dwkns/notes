add `factory-girl-rails` to Gemfile

To enable with cucumber add 
`World(FactoryGirl::Syntax::Methods)` into feaetures / support / env.rb 


Create `factories` folder in Features

create `modelname.rb` in factories

typical factory would be :

	FactoryGirl.define do 
 		factory :grill do
 	   	  name "default name" 
 	   	  description "default description"
 	 	end
	end

Then in your webstep something like

    Given(/^(\d+) Grills have been added$/) do |arg1|
      FactoryGirl.create(:grill, name: "The steakhouse")
      FactoryGirl.create(:grill, name: 'The chicken basket')
      FactoryGirl.create(:grill, name: "The scampi bucket")
    end
    
