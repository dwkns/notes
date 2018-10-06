###.erb forms without having to bind to an object.
Useful if your model does not yet exist, or you've not implimented the controller methods. 
 
		<p>You are about to email : </p>
		<%= form_for :any_symbol do |form| %>
			<%= form.label :param_tag, "label"%>
			<%= form.text_field :param_tag , placeholder:"gonads"  %>
			<%= form.submit 'Send email' %>
		<% end %>