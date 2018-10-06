To use a graphical element in a form with Rails, use a hidden field. 

Assuming that you are using 
[RateIt](http://rateit.codeplex.com) and `/assets/jquery.rateit.js` is linked in the header. Then in your view you can have something like :

	<%= form_for @your_controller do |form| %>
	  <%= form.label :rating, 'Your rating' %>
	  <span class="rateit" id="rateit" ></span><span id='rating-value'></span>
	  <%= form.hidden_field :rating,  class: "rateit" %>
	  <%= form.submit 'Add review' %>
    <% end %>
    
And the required Javascript
  
	<script type="text/javascript">
      //rated is effectivly a click on the starts.
      $("#rateit").bind('rated', function (event, value) { 
        $(".rateit").val(value);
        $("#rating-value").html(" " + value + "/5");
      });
    </script>

`$(".rateit").val(value);` sets the value attribute of the hidden form field when it is clicked. The submit then works as expected.