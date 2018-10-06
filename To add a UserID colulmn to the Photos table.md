To add a UserID colulmn to the Photos table

rails g migration AddUserIdToPhotos user:(belongs_to||references)


to migrate both dev and test enviroments in rails.
alias rmg='rake db:migrate; rake db:migrate RAILS_ENV=test'



Test a one to many relationship
describe User do
  it {should have_many :photos}
end

describe Photo do
  it {should belong_to :user}
end

Shoda - validations and accociattions.

Adding a current user to a created photo
current_user is a helper method from devise. 

current_user.photos.create(params)

current_user.photos.build(params) # is like DBObject.new but allows accications.


To get a specific GEM from Githum
gem 'twitter-bootstrap-rails' , github: '', branch:''


