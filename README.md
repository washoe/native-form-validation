# native-form-validation
Native form validation proof of concept.

## Is native form validation usable?
Yes and no. Common validation requirements are covered by modern browsers, and the [Constraint Validation API](https://developer.mozilla.org/en-US/docs/Web/API/Constraint_validation) allows for custom validation. However, to implement common UX validation patterns requires some ingenuity.

What I'm attempting here is not to build a fully-rounded validation library - I'm pretty sure one would exist if it was that easy. I want to see how far we can lean on the browser's built in behaviour, and get a feel for how this approach compares with using an off-the-shelf validation library in terms of overall complexity.

[This article](https://www.quirksmode.org/blog/archives/2017/12/native_form_val.html) is very detailed but somewhat pessimistic. I'd like to think that it can be done, with a complexity that is at worst comparable to wrangling a 3rd party validation library with its own quirks.

## Is it worth the bother?
Conflicts between the JavaScript world of the framework and the HTML/CSS of the actual DOM have always felt like a kind of cognitive dissonance. The VDOM allows us precise control of the DOM, but also gives us the responsibility of ensuring it represents the framework's state accurately. The DOM is deep and not just visual.

## When to tell the user?

A common UX pattern is not to reveal validation errors until either:
- the user has finished interacting with an input (i.e. on blur), or
- the user attempts to submit the form data.

The browser will validate the inputs immediately, and reveal the results via the Constraint Validation api and the :valid and :invalid pseudo-selectors. This gives us all the information we need to convey to the user.

All that remains is to control when to convey this information, for which we do need some JavaScript to update the DOM with the appropriate error messages at the appropriate time. This could be vanilla, or any front-end framework. (I've gone with Vue here.)

As well as populating an .error element, we can also render the errors in the form control's aria-invalid attribute:

- Manipulating an attribute is more semantic than manipulating a class.
- We get the added benefit of enhanced a11y.
- There is a direct association between the errors and the form control.

## Next steps
### Cross-validation
Often we need to update a form control's validation rules according to the state of another form control - for example, when setting a date range, the end date needs to be after the start date.