# Validations in Controller Actions

Now that we know Rails automatically performs validations defined on models,
let's use this information to easily display validation errors to the user.

At this point, we'll be covering step two of the following flow:

1. User fills out the form and hits "Submit", transmitting the form data via
   a POST request.
2. **The controller sees that validations have failed, and re-renders the
   form.**
3. The view displays the errors to the user.


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

# Manually Checking Validation

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

The conventional pattern in Rails these days is to test the return value of the
CRUD method (like `@post.create` or `@post.save`) and decide what to do based
on that. But, for the sake of going slowly and understanding exactly what's
going on, we're going to split things up into a few steps before implementing it
the "easy" way:

```ruby
# app/controllers/posts_controller.rb

  def create
    @post = Post.new(post_params)

    if @post.valid?
      @post.save
      redirect_to post_path(@post)
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

In the next lessons, we'll learn how to use the error information in
`@post.errors` to display feedback to the user through our view.
