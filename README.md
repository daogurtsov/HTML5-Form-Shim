# HTML5 Form Shim

* The project site: https://github.com/dsheiko/HTML5-Form-Shim
* The demo site: http://demo.dsheiko.com/html5formshim/

That is a jQuery plugin, which emulates HTML5 Form behavior on the legacy browsers. So it claims to solve both weaknesses of HTML5 Form: support on legacy browsers and attached-widget implementations heterogeneity among layout engines (browsers). If you care about design consistency of your application, you will make sure that UI components look more or less alike on different browsers. However you would hardly able to re-style built-in on-submit tooltips, date-picker, color-picker and others. With this plugin you may just force it to substitute browser native HTML5 Form API and, therefore, have all the attached UI components always in the same style.

### How to use
To enable the shim you simply have to include jQuery and this plugin.

    <script src="http://ajax.googleapis.com/ajax/libs/jquery/1.7.1/jquery.min.js" type="text/javascript"></script>
    <script src="./js/jquery.html5form.js" type="text/javascript"></script>


See details on how to use HTML5 form at http://www.html5rocks.com/en/tutorials/forms/html5forms/

### Styling

Take in the account that legacy browsers ignore CSS pseudo-classes :focus, :invalid, :valid, so use instead classes focus, invalid, valid.

NOTE: The examples below use CSS3 styles. If you want your forms not only behave, but look the same in all browsers, use simple styles

#### Example 1: Custom form submission validation

```
<form class="example1" data-custom-validation="true">
    <label for="name">Name:</label>
    <input type="text" placeholder="Name" required="true" name="name"  />
    <label for="email">Email:</label>
    <input type="email" placeholder="Email" autofocus="true" required="true" name="email" />
    <label for="name">Age:</label>
    <input type="number" required="true" name="age" min="18" max="100" />
    <label for="email">Promo code:</label>
    <input type="text" name="promocode" required="true" pattern="[A-Z]{3}[0-9]{4}"
title="Promo code consist of 3 uppercase letters followed by 4 digits." />
    <label for="email">Tel:</label>
     <input type="tel" name="tel" required="true" pattern="^\+(?:[0-9] ?){6,14}[0-9]$"
title="Please enter valid tel." />
    <button type="submit">Submit</button>
</form>
```

This form shows custom tooltips as on old browsers as well as on those supporting HTML5.

To make the form showing your custom tooltip on submission validation, define data-custom-validation property on the
form element.

The plugin will use $.setCustomValidityCallback to display validation messages. You can override this callback with
your own function:

```
   $.setCustomInputTypeValidator = function( type, msg, validatorCb, initCb ) {
         Control[ type ] = function() {
                return {
                    "__extends__" : AbstractControl,
                    "__init__": function() {
                        initCb && initCb.apply( this.boundingBox, [ this ] );
                    },
                    validateValue: function() {
                        validatorCb.apply( this.boundingBox, [ this ] ) ||
                            this.throwValidationException( "customError", msg );
                        return this.logger;
                    }
                };
            };
     };
```

#### Example 2: Showing validation messages on custom elements
```
<form class="example2" data-custom-validation="true">
    <label for="email">Email:</label>
    <input id="f2email" type="email" placeholder="Email" required="required" name="email" />
    <div class="help-inline">
        <div data-validation-message-for="f2email"></div>
    </div>
    <label for="name">Age:</label>
    <input id="f2age" type="number" required="required" name="age" min="18" max="100" />
    <div class="help-inline">
        <div data-validation-message-for="f2age"></div>
    </div>
    <button class="btn btn-inverse btn-large" type="submit">Submit</button>
</form>
```
This form forced to custom validation by attribute data-custom-validation. So whatever browser you use it displays validation messages on the elements marked as data-validation-message-for="<input-id>"

#### Example 3: Custom input type validation

```
<form class="example3">
    <label for="name">Zipcode (custom input type):</label>
    <input type="zipcode" placeholder="Zipcode" required="required" name="zipcode"  />
    <button class="btn btn-inverse btn-large" type="submit">Submit</button>
</form>
```

You can easily define your own input validator by using $.setCustomInputTypeValidator. It receives following arguments:

    (string) Input type (here Zip for <input type="zip">)
    (string) Input value validation message
    (string) Validation callback (returns boolean)

```
$.setCustomInputTypeValidator( "Zip", "Please enter a valid zip code", function() {
    var pattern = /^[0-9]{6,8}$/;
    return pattern.test( $( this ).val() );
});
```
Mark that the input element is available within the callback as the context (this).


#### Example 4: Attaching 3rd-party widgets to form elements
```
<form class="example4">

    <label for="color">Color:</label>
    <input type="color" name="color" value="#d4251b" />

    <label for="date">Date:</label>
    <input type="date" name="date" />

    <button class="btn btn-inverse btn-large" type="submit">Submit</button>
</form>
```
You can attach to any input element a custom handler. Thus you can enrich inputs of such types as color, date, datetime, week, moth, time, range with corresponding widgets (e.g. how it's implemented in latest Chrome/Opera releases).

In the example below you can find color input using colorPicker jQuery-plugin and date input using datePicker jQuery-plugin

```
// Custom color input handler example
$.setCustomInputTypeValidator( "Color", "Please enter a valid hex color", function() {
    var pattern = /^#(?:[0-9a-f]{3}){1,2}$/i;
    return pattern.test( $( this ).val() );
}, function( control ) {
    control.isShimRequired() &&
    control.degrade().boundingBox.ColorPicker({
        onChange: function( hsb, hex, rgb ) {
            control.boundingBox.val( "#" + hex );
        },
        onBeforeShow: function () {
            $( this ).ColorPickerSetColor( this.value );
        }
    });
});

// Custom date input handler example
$.setCustomInputTypeValidator( "Date", "Please enter a valid date", function() {
    var pattern = /^\d{2,4}(-|\/)\d{2,4}(-|\/)\d{2,4}$/i;
    return !$( this ).val() || pattern.test( $( this ).val() );
}, function( control ) {
    control.isShimRequired() &&
    control.degrade().boundingBox.datePicker({
        createButton: false,
        clickInput: true,
        startDate: '01/01/1970'
    })
        .bind( 'dateSelected', function() {
            control.validateValue();
        });
});
```
As you see here last argument of $.setCustomInputTypeValidator is initialization callback. It injects dependency to input control handler (control). From which you can access the element control.boundingBox(the same as $(this) ) and methods such as control.isShimRequired() (indicates if if the input type supported) and control.degrade() (switch the type to text, which prevents collision with browser-specific element treatment)

#### Example 5: Custom oninput callback
```
<form class="example5"  data-custom-validation="true">
    <label for="email">Email:</label>
    <input type="email" placeholder="Email" required="required" name="email" />
    <label for="password">Password:</label>
    <input type="password" required="required" name="password" />
    <label for="confirm">Password confirmation:</label>
    <input type="password" required="required" name="confirm" />
    <button class="btn btn-inverse btn-large" type="submit">Submit</button>
</form>
```

HTML5 introduces a new event oninput, which can be handled to perform additional validation tests on a field. For example, making registration form you can define a handler which checks by XMLHttpRequest if the given email already exists in DB. Here an example for a cross-browser oninput handler:

```
$('form.example3 input[name=confirm]').on("oninput", function () {
    var input = $(this);
    if (input.val() != $('form.example3 input[name=password]').val()) {
    input.setCustomValidity('The two passwords must match.');
    } else {
    // input is valid -- reset the error message
    input.setCustomValidity('');
    }
});
```