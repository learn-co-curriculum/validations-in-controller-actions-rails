# Validations in Controller Actions

## Objectives

  - Define validations on a model
  - Use the validation state of a model in a response conditional in an action
  - re-render a form with validation errors
  - validate a create action
  - validate an update action

## Notes

Really simple readme, given a form and a model with validations, what does a controller action that takes this validation into account look like?

We need to see if the object is valid on form submission, if so, save it, and redirect to the newly created resource.

if not, then we want to re-render the form with the invalid instance in scope so that we can use it to introspect on erorrs (more on that very soon, but we need to know what was wrong.) its important to try to remind the student that the @post from Post#new is long gone - every request happens indepedntly, we've recreated the essence of @post from Post#new when the form was submitted with Post.new(params[:post]). We can't redirect to new to reshow the form because then the invalid @post based on params[:post] is gone. Try to explain this idea. Why we render on invalid objects instead of redirecting.

if valid? vs if save on new  - explain the pattern, save returns false and triggers validation
if update - same with update

very common pattern.
