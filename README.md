# Validations in Controller Actions

Now that we know Rails automatically performs validations defined on models,
let's use this information to easily display validation errors to the user.

## A Note About Page Loads

You're probably used to validations happening almost instantaneously on websites
that you interact with on a daily basis. Every browser has a way of visually
indicating that a page load is happening; in Chrome, for example, the circular
refresh icon becomes an "X" (for "Stop Loading"), and reverts to refresh when
the load is complete.

A "page load" means that everything the browser was doing is thrown out, except
for cookies that are stored between requests. When you get validation feedback
*without* seeing a page load, that's JavaScript at work, sneakily performing
requests in the background without throwing away the current page.

We won't be using that advanced technique just yet. For now, we'll be performing
a full page load every time a form is submitted, which means there will be no
validation feedback for the user until they actually hit the "Submit" button.

# Objectives

After this lesson, you'll be able to...

- Define validations on a model
- Use the validation state of a model in a response conditional in an action
- Re-render a form with validation errors
- Validate a create action
- Validate an update action

# Form Helpers

This step will make heavy usage of `form_for`, the high-powered alternative to
`form_tag`. The biggest difference beteen these two helpers is that `form_for`
creates a form specifically **for** a model object. `form_for` is full of
convenient features.

A basic implementation looks like this:

```erb
<!-- app/views/posts/new.html.erb //-->

<%= form_for @post, url: {action: "create"} do |f| %>
  <%= f.text_field :title %>
  <%= f.text_area :content %>
  <%= f.submit "Create" %>
<% end %>
```

To wire up an empty form in our `new` view, we need to create a blank object:

```ruby
# app/controllers/posts_controller.rb

  def new
    @post = Post.new
  end
```

Up until this point, our `create` action has looked something like this:

```ruby
# app/controllers/posts_controller.rb

  def create
    @post = Post.create(post_params)

    redirect_to post_path(@post)
  end
```

However, we have two problems now:

1. If the post is invalid, there will be no `show` path to redirect to.
2. If we redirect, we start a new page load, which will lose all of the
   feedback from the validations.

# Re-Rendering With Errors

Remember from our last lesson how CRUD methods return `false` when validation
fails? We can use that to our advantage here and branch our actions based on the
result:

```ruby
# app/controllers/posts_controller.rb

  def create
    @post = Post.new(post_params)

    if @post.save
      redirect post_path(@post)
    else
      render :new
    end
  end
```

You can read more about this creative use of `render` in Section 2.2.2 of the
Rails Guide on [Layout and Rendering][layout_rendering].

[layout_rendering]: http://guides.rubyonrails.org/layouts_and_rendering.html#using-render

Remember: **redirects incur a new page load**. If we were to redirect after
validation failure, we would **lose** the instance of `@post` that has
validation feedback in its `errors` attribute.

Another way to differentiate redirects is this:

- If you hit Refresh after a redirect/page load, your browser resubmits the
  `GET` request without complaint.

- If you hit Refresh after rendering on a form submit, your browser gives you a
  popup to confirm that you want to resubmit form data with the `POST` request.

# Full Messages with Prepopulated Fields

Because of `form_for`, Rails will automatically prepopulate the `new` form with
the values the user entered on the previous page.

To get some extra verbosity, we can add the snippet from the previous lesson to
the top of the form:

```erb
<!-- app/views/posts/new.html.erb //-->

<% if @post.errors.any? %>
  <div id="error_explanation">
    <h2>
      <%= pluralize(@post.errors.count, "error") %>
      prohibited this post from being saved:
    </h2>

    <ul>
    <% @post.errors.full_messages.each do |msg| %>
      <li><%= msg %></li>
    <% end %>
    </ul>
  </div>
<% end %>
```
