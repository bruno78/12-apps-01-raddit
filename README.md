## Raddit App - A clone of the famous Reddit

This app was created under the tutorial 12 web apps in 12 weeks by Mackenzie Child (@mackenziechild). This is the number 1 of the series.


### Project Planning

* There will be a homepage which will be a list of links -[x]
* There will be a page for users sign in and sign up  -[x]
* There will be an area to add links -[x]
* Users will be able tod edit and delete own links -[x]
* Users will be able to vote up and down on each link -[x]
* Users will be able to comment on links -[x]
* Users will be able to edit own links (not included on tutorial) -[x]
* Users will be able to delete own links -[x]


#### The tutorial video
[Raddit](https://www.youtube.com/watch?v=7-1HCWbu7iU&index=1&list=PL23ZvcdS3XPLNdRYB_QyomQsShx59tpc-)

#### First steps

1. Rails new raddit
2. Initialize git

#### Creating Links Scaffold

3. Create a new branch on git: git checkout -b links_scaffold
4. rails g scaffold Link title:string url:string
5. rake db:migrate // to create the schema
6. rails s
7. http://localhost:3000/links
7.1. http://localhost:3000/links/new // where you can create a link

8. Since everything is working fine and there's nothing more to do with Posting the link and title feature. We can go back to master, merge and commit

9. git add .
10. git commit -am "finalizing Link scaffold"
11. git checkout master
12. git merge links_scaffold

#### Adding Users

13. Next: Creating users and their ability to sign in and sign out. We are going to use devise gem
14. git checkout -b add_users
15. add devise gem to gemfile
  * gem 'devise', '~> 3.3.0'
16. bundle install

17. restart the rails server to be able to recognize the devise gem
18. rails s
19. rails g devise:install // To install the devise generator

20. the devise gives you a list of instructions of what to do:

#### Some setup you must do manually if you haven't yet:

  1. Ensure you have defined default url options in your environments files. Here
     is an example of default_url_options appropriate for a development environment
     in config/environments/development.rb:
'''ruby
       config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }
'''
     In production, :host should be set to the actual host of your application.

  2. Ensure you have defined root_url to *something* in your config/routes.rb.
     For example:
                  root to: "home#index"

  3. Ensure you have flash messages in app/views/layouts/application.html.erb.
     For example:
'''html
       <p class="notice"><%= notice %></p>
       <p class="alert"><%= alert %></p>
'''
  4. If you are deploying on Heroku with Rails 3.2 only, you may want to set:

       config.assets.initialize_on_precompile = false

     On config/application.rb forcing your application to not access the DB
     or load models when precompiling your assets.

  5. You can copy Devise views (for customization) to your app by running:

            rails g devise:views
21. Number 1: DONE! (except the "I production part")
22. Number 2: root to: "links#index" // Now if you go to localhost:3000 it will show the Links controller page instead of the Rails default page.
23. Number 3:
          <% flash.each do |name, msg| %>
            <%= content_tag(:div, msg, class: "alert alert-#{name}") %>
          <%end%>
// Not using the page generator suggested for number 3. The reason for that is that we will be using bootstrap and the flash above will generate the alert class based on what alert type it is.
24. Number 4: will be skipped since we won't be uploading to Heroku

25. Number 5: rails g devise:views // We will be customizing the views.

26. rails g devise User // Now we can create the Users NEEDS TO BE SINGULAR!!!!

27. rake db:migrate

    // Issues... rake db:migrate failed!
    // Possible solution

    1) remove "devise_for :users" in routes.db
    2) run the command "rails destroy scaffold User"
    3) remove add_devise_to_users file in db/migrate
    3.1) add the new gem: "gem 'devise', '~> 3.4.0'
    3.2) bundle install
    4) re-run "rails generate devise User" to create a brand new devise_create_users file in db/migrate
    5) re-run "rake db:migrate"

28. restart server
29. rails s
30. http://localhost:3000/users/sign_up
31. rails c
32. User.count // To see how many users
33. @user = user.first // To see the first user
34. exit // to exit the database
35. Adding to applicationt.html.erb:
'''html
        <% if user_signed_in? %>
          <ul>
            <li><%= link_to 'Submit link', new_link_path %></li>
            <li><%= link_to 'Account', edit_user_registration_path %></li>
            <li><%= link_to 'Sign Out', destroy_user_session_path, :method => :delete %></li>
          </ul>
        <% else %>
          <ul>
            <li><%= link_to 'Sign up', new_user_registration_path %></li>
            <li><%= link_to 'Sign in', new_user_session_path %></li>
          </ul>
        <% end %>
'''
// That way you don't need to be going to a different page to sign out or account or if you are not logged in, sign up and login are all there.

36. Commented out the step 3 of Devise installation because it has already included.

37. Added "has_many :links" to models/user.rb
38. Added "belongs_to :user" to models/link
// 37 and 38 we now need to connect the relationship between user and links and vice-versa
39. rails c // To see that relationship in the Database
40. @link = Link.first // it will return the first link
41. @link.user // it will return "nil". That shows the relationship is working. If it didn't have the has_many :links at user.rb Model, it would give you an error.
42. exit
43. We need to add the user column to the link table. We will be using the user_id. rails g migration add_user_id_to_links user_id:integer:index
44. rake db:migrate
45. rails c   // to confirm if that works.
46. Link.connection
47. Link // you should see "user_id: integer" as part of the returned command
48. exit

49. We need to update the link controller so when users submit the link their user_id get to be assigned with that link. We can do that by updating the new and create method inside of the link controller.rb

50. remove @links = Link.new (def new) at links_controller.rb and replace for
@link = current_user.links.build and (def create) replace @links = Link.new(link_params) for @link = current_user.links.build(link_params)

51. Added some new link as signed in user

52. rails c // to see if the link was created with user_id
53. @link = Link.last // you should see user_id number.
54. @link.user // you should see the user attached to that link
55. @link.user.email  

Authentication. Where we have to fix the fact anyone can see the posts and edit and destroy

56. on links_controller.rb page we added:
before_filter :authenticate_user!, except: [:index, :show]

57. Only the signed in users can edit or destroy... but still "edit" and "destroy" options still showing on the index page.

58. on views/links/index.html.erb
'''html
          <% if link.user == current_user %>
            <td><%= link_to 'Edit', edit_link_path(link) %></td>
            <td><%= link_to 'Destroy', link, method: :delete, data: { confirm: 'Are you sure?' } %></td>
          <% end %>
'''
59. Rails c

Assigning those two first link to the first user:
@link = Link.first
@link.user = User.first
@link.save
@link = Link.find(2)
@link.user = User.first
@link.save

60. Reload the page

61. Removing New Link from the bottom of the page
index/views/index.html.erb
Remove <%= link_to 'New Link', new_link_path %> and </br> tag above
62. Rename "destroy" for "delete"
63. git commit
64. git checkout master
65. git merge add_users

#### Add bootstrap - Styling

66. git checkout -b add_bootstrap
67. gem 'bootstrap-sass', '~> 3.2.0.2' to .gemfile
68. bundle install
69. At app/assets/stylesheets/ rename application.css to application.css.scss for bootstrap to work and add (according with github bootstrap-sass):
  @import "bootstrap-sprockets";
  @import "bootstrap";
70. At javaScripts/application.js add
  //= require bootstrap-sprockets
71. delete scaffold.scss
72. Add styling on application.html.erb:
'''html
<!DOCTYPE html>
<html>
<head>
  <title>Raddit</title>
  <%= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track' => true %>
  <%= javascript_include_tag 'application', 'data-turbolinks-track' => true %>
  <%= csrf_meta_tags %>
</head>
<body>
	<header class="navbar navbar-default" role="navigation">
    <div class="navbar-inner">
      <div class="container">
        <div id="logo" class="navbar-brand"><%= link_to "Raddit", root_path %></div>
        <nav class="collapse navbar-collapse navbar-ex1-collapse">
        	<% if user_signed_in? %>
						<ul class="nav navbar-nav navbar-right">
							<li><%= link_to 'Submit link', new_link_path %></li>
							<li><%= link_to 'Account', edit_user_registration_path %></li>
							<li><%= link_to 'Sign out', destroy_user_session_path, :method => :delete %></li>
						</ul>
					<% else %>
						<ul class="nav navbar-nav pull-right">
							<li><%= link_to 'Sign up', new_user_registration_path %></li>
							<li><%= link_to 'Sign in', new_user_session_path %></li>
						</ul>
					<% end %>
        </nav>
      </div>
    </div>
  </header>

	<div id="main_content" class="container">
			<div id="content" class="col-md-9 center-block">
        <%= yield %>
			</div>
		</div>
	</div>
</body>
</html>
'''
73. Add styling on application.css.scss
'''scss
#logo {
	font-size: 26px;
	font-weight: 700;
	text-transform: uppercase;
	letter-spacing: -1px;
	padding: 15px 0;
	a {
		color: #2F363E;
	}
}

#main_content {
	#content {
		float: none;
	}
	padding-bottom: 100px;
	.link {
		padding: 2em 1em;
		border-bottom: 1px solid #e9e9e9;
		.title {

			a {
				color: #FF4500;
			}
		}
	}
	.comments_title {
		margin-top: 2em;
	}
	#comments {
		.comment {
			padding: 1em 0;
			border-top: 1px solid #E9E9E9;
			.lead {
				margin-bottom: 0;
			}
		}
	}
}
'''
74. Fixing index.html.erb. Delete everything and paste this:
'''html
<% @links.each do |link| %>
  <div class="link row clearfix">
    <h2>
      <%= link_to link.title, link %><br>
      <small class="author">Submitted <%= time_ago_in_words(link.created_at) %> by <%= link.user.email %></small>
    </h2>
  </div>
<% end %>
'''
74. b) add this to show.erb
'''html
  <div class="page-header">
    <h1><a href="<%= @link.url %>"><%= @link.title %></a><br> <small>Submitted by <%= @link.user.email %></small></h1>
  </div>

  <div class="btn-group">
  	<%= link_to 'Visit URL', @link.url, class: "btn btn-primary" %>
  </div>

  <% if @link.user == current_user -%>
  	<div class="btn-group">
  		<%= link_to 'Edit', edit_link_path(@link), class: "btn btn-default" %>
  		<%= link_to 'Delete', @link, method: :delete, data: { confirm: 'Are you sure?' }, class: "btn btn-default" %>
  	</div>
  <% end %>
'''
75. Changing the form enclosing them in form-group provided by bootstrap links/_form:
'''html
  <div class="form-group">
    <%= f.label :title %><br>
    <%= f.text_field :title, class: "form-control" %>
  </div>
  <div class="form-group">
    <%= f.label :url %><br>
    <%= f.text_field :url, class: "form-control" %>
  </div>
  <br>
  <div class="form-group">
    <%= f.submit "Submit", class: "btn btn-lg btn-primary" %>
  </div>
'''
76. Account page (views/devise/registrations/edit.html.erb) wrapping in a form-group class provided by bootstrap
'''thml
<h2>Edit <%= resource_name.to_s.humanize %></h2>

<%= form_for(resource, as: resource_name, url: registration_path(resource_name), html: { method: :put }) do |f| %>
  <%= devise_error_messages! %>

  <div class="panel panel-default">
    <div class="panel-body">

      <div class="form-inputs">

      <div class="form-group">
        <%= f.label :email %>
        <%= f.email_field :email, class: "form-control", :autofocus => true %>
      </div>

      <div class="form-group">
        <%= f.label :password %> <i>(leave blank if you don't want to change it)</i>
        <%= f.password_field :password, class: "form-control", :autocomplete => "off" %>
      </div>

      <div class="form-group">
        <%= f.label :current_password %> <i>(we need your current password to confirm your changes)</i>
        <%= f.password_field :current_password, class: "form-control" %>
      </div>

      </div>

      <div class="form-group">
        <%= f.submit "Update", class: "btn btn-primary" %>
      </div>

    <% end %>
  </div>
  <div class="panel-footer">

    <h3>Cancel my account</h3>

    <p>Unhappy? <%= button_to "Cancel my account", registration_path(resource_name), data: { confirm: "Are you sure?" }, method: :delete, class: "btn btn-default" %></p>

  </div>
'''
77. sign up page (views/devise/registrations/new.html.erb) wrapping in a form-group class provided by bootstrap
'''html
<h2>Sign up</h2>

<%= form_for(resource, as: resource_name, url: registration_path(resource_name)) do |f| %>
  <%= devise_error_messages! %>

	  <div class="form-group">
	  	<%= f.label :email %>
	  	<%= f.email_field :email, autofocus: true, class: "form-control", required: true %>
	  </div>

	  <div class="form-group">
	  	<%= f.label :password %>
	  	<%= f.password_field :password, class: "form-control", required: true %>
	  </div>

	  <div class="form-group">
	  	<%= f.label :password_confirmation %>
	  	<%= f.password_field :password_confirmation, class: "form-control", required: true %>
	  </div>

	  <div class="form-group">
	  	<%= f.submit "Sign up", class: "btn btn-lg btn-primary" %>
	  </div>
<% end %>

<%= render "devise/shared/links" %>
'''
78. sign up page (views/devise/sessions/new.html.erb) wrapping in a form-group class provided by bootstrap
'''html
<h2>Sign in</h2>

<%= form_for(resource, as: resource_name, url: session_path(resource_name)) do |f| %>

  <div class="form-group">
  	<%= f.label :email %>
  	<%= f.email_field :email, autofocus: true, class: "form-control", required: false %>
  </div>

  <div class="form-group">
  	<%= f.label :password %>
  	<%= f.password_field :password, class: "form-control", required: false %>
  </div>

  <div class="checkbox">
  	<label>
  	<%= f.check_box :remember_me, required: false, as: :boolean if devise_mapping.rememberable? %> Remember Me
		</label>
  </div>

  <div class="form-group">
  	<%= f.submit "Sign In", class: "btn btn-primary" %>
  </div>

<% end %>

<%= render "devise/shared/links" %>
'''
79. git commit and merge

#### Add Act as votable gem

80. git checkout -b add_act_as_votable

81. Add gem 'acts_as_votable', '~> 0.10.0' to .gemfile
82. bundle install
83. restart the server
84. rails g acts_as_votable:migration // creates the database migration if you go to db/migrate you will see the acts_as_votable database
85. rake db:migrate
86. models/link.rb add acts_as_votable
87. rails c
88. @user = User.first
89. @link = Link.first
90. @link.liked_by @user
91. @link.votes_for.size // You should see 1 and that means it's working
92. @link.save
93. Adding to routes by changing resources links to:
'''Ruby
    resources :links do
      member do
        put "like", to: "links#upvote"
        put "dislike", to: "links#downvote"
      end
    end
'''    
94. links_controller.rb we need to create a upvote and downvote method
'''Ruby
    def upvote
      @link = Link.find(params[:id])
      @link.upvote_by current_user
      redirect_to :back
    end

    def downvote
      @link = Link.find(params[:id])
      @link.downvote_from current_user
      redirect_to :back
    end
'''
95. Add the ability of upvote and downvote in the views at index.html.erb
right after the </h2>
'''html
    <div class="btn-group">
        <a class="btn btn-default btn-sm" href="<%= link.url %>">Visit Link</a>
        <%= link_to like_link_path(link), method: :put, class: "btn btn-default btn-sm" do %>
          <span class="glyphicon glyphicon-chevron-up"></span>
          Upvote
          <%= link.get_upvotes.size %>
        <% end %>
        <%= link_to dislike_link_path(link), method: :put, class: "btn btn-default btn-sm" do %>
          <span class="glyphicon glyphicon-chevron-down">
          Downvote
          <%= link.get_downvotes.size %>
        <% end %>
    </div>
'''
96. Adding on show.html.erb
'''html
    <div class="btn-group pull-right">
      <%= link_to like_link_path(@link), method: :put, class: "btn btn-default btn-sm" do %>
        <span class="glyphicon glyphicon-chevron-up"></span>
        Upvote
        <%= @link.get_upvotes.size %>
      <% end %>
      <%= link_to dislike_link_path(@link), method: :put, class: "btn btn-default btn-sm" do %>
        <span class="glyphicon glyphicon-chevron-down">
        Downvote
        <%= @link.get_downvotes.size %>
      <% end %>
    </div>
'''
97. git commit and merge branch

#### Adding Comment

98. git checkout -b add_comments
99. rails g scaffold Comment link_id:integer:index body:text user:references --skip-stylesheets (comment belongs to references and we are skipping the stylesheets because we don't want them)
100. rake db:migrate
101. add the latest: gem 'simple_form', '~> 3.2.1' to .gemfile
102. bundle install
103. restart the server
104. Add has_many :comments to link.rb at Models folder
105. Add belongs_to :user AND belongs_to :link to comment.rb at Models folder
106. Add resources to our comments at routes.rb after the first end of the loop:
    end
    resources :comments
    end
107. rake routes
108. erase index, show, new and edit methods from the comments_controller. We need to update the create method:
'''Ruby
    def create
       @link = Link.find(params[:link_id])
       @comment = @link.comments.new(comment_params)
       @comment.user = current_user

       respond_to do |format|
         if @comment.save
           format.html { redirect_to @link, notice: 'Comment was successfully created.' }
           format.json { render json: @link, status: :created, location: @link }
         else
           format.html { render action: "new" }
           format.json { render json: @comment.errors, status: :unprocessable_entity }
         end
       end
     end

     # DELETE /comments/1
     # DELETE /comments/1.json
     def destroy
       @comment.destroy
       respond_to do |format|
         format.html { redirect_to :back, notice: 'Comment was successfully destroyed.' }
         format.json { head :no_content }
       end
     end

     private
       # Use callbacks to share common setup or constraints between actions.
       def set_comment
         @comment = Comment.find(params[:id])
       end

       # Never trust parameters from the scary internet, only allow the white list through.
       def comment_params
         params.require(:comment).permit(:link_id, :body, :user_id)
       end
'''
109. Updating the show.html.erb at views/link. Adding this at the bottom of the page:
'''html
    <h3 class="comments_title">
      <%= @link.comments.count %> Comments
    </h3>

    <div id="comments">
      <%= render :partial => @link.comments %>
    </div>
    <%= simple_form_for [@link, Comment.new]  do |f| %>
      <div class="field">
        <%= f.text_area :body, class: "form-control" %>
      </div>
      <br>
      <%= f.submit "Add Comment", class: "btn btn-primary" %>
    <% end %>
'''
110. Create the partial under views/comments: _comment.html.erb
'''html
      <%= div_for(comment) do %>
      	<div class="comments_wrapper clearfix">
      		<div class="pull-left">
      			<p class="lead"><%= comment.body %></p>
      			<p><small>Submitted <strong><%= time_ago_in_words(comment.created_at) %> ago</strong> by <%= comment.user.email %></small></p>
      		</div>

      		<div class="btn-group pull-right">
      			<% if comment.user == current_user -%>
      				<%= link_to 'Destroy', comment, method: :delete, data: { confirm: 'Are you sure?' }, class: "btn btn-sm btn-default" %>
      			<% end %>
      		</div>
      	</div>
      <% end %>
'''
111. Git commit and merge!

#### Add User!

112. git checkout -b add_name_to_users
113. rails g migration add_name_to_users name:string
114. rake db:migrate
115. Now we need to add a input to a form field to accept that. views/devise/registrations/edit.html.erb add a name form right above the email:
'''html
      <div class="form-group">
        <%= f.label :name %>
        <%= f.text_field :name, class: "form-control", :autofocus => true %>
      </div>
'''
116. Because strong parameters on Rails 4 if you go to the account and try to save your name, it will save but it wont. show we gotta add code to application_controller.rb
'''Ruby
    before_filter :configure_permitted_parameters, if: :devise_controller?

    protected

    def configure_permitted_parameters
      devise_parameter_sanitizer.for(:sign_up) << :name
      devise_parameter_sanitizer.for(:account_update) << :name
    end
'''
117. views/links/index.html.erb change link.user.email for link.user.name
'''html
    <small class="author">Submitted <%= time_ago_in_words(link.created_at) %> by <%= link.user.name %></small>
'''
118. views/links/show.html.erb change link.user.email for link.user.name
'''html
    <h1><a href="<%= @link.url %>"><%= @link.title %></a><br> <small>Submitted by <%= @link.user.name %></small></h1>
'''
119. views/comments/_comments.html.erb change link.user.email for link.user.name
'''html
    </strong> by <%= comment.user.name %></small></p>
'''
120. Git merge, add and commit!
121. DONE!
