Common Needs
============

This chapter collects solutions for requirements that will often crop
up once you start using Deform for real world applications.

.. contents:: :local:


.. _customizing-widget-templates:

Customizing widget templates
----------------------------

To override, add, or customize widget templates, see the chapter :doc:`templates`.


.. _complex-data-loading-changing-widgets-validating:

Complex data loading, changing widgets, or validating
-----------------------------------------------------

For complex loading of data, changing widgets, or validating beyond the basic declarative schema, you can use Colander's schema binding feature.
See the chapter in the Colander's documentation `Schema Binding <https://docs.pylonsproject.org/projects/colander/en/latest/binding.html>`_.


.. _changing_a_default_widget:

Changing the Default Widget Associated With a Field
---------------------------------------------------

Let's take another look at our familiar schema:

.. code-block:: python
   :linenos:

   import colander

   class Person(colander.MappingSchema):
       name = colander.SchemaNode(colander.String())
       age = colander.SchemaNode(colander.Integer(),
                                 validator=colander.Range(0, 200))

   class People(colander.SequenceSchema):
       person = Person()

   class Schema(colander.MappingSchema):
       people = People()

   schema = Schema()

This schema renders as a *sequence* of *mapping* objects.  Each
mapping has two leaf nodes in it: a *string* and an *integer*.  If you
play around with the demo at
`https://deformdemo.pylonsproject.org/sequence_of_mappings/
<https://deformdemo.pylonsproject.org/sequence_of_mappings/>`_ you will notice
that, although we do not actually specify a particular kind of widget
for each of these fields, a sensible default widget is used.  This is
true of each of the default types in :term:`Colander`.  Here is how
they are mapped by default.  In the following list, the schema type
which is the header uses the widget underneath it by default.

:class:`colander.Mapping`
   :class:`deform.widget.MappingWidget`

:class:`colander.Sequence`
    :class:`deform.widget.SequenceWidget`

:class:`colander.String`
    :class:`deform.widget.TextInputWidget`

:class:`colander.Integer`
    :class:`deform.widget.TextInputWidget`

:class:`colander.Float`
    :class:`deform.widget.TextInputWidget`

:class:`colander.Decimal`
    :class:`deform.widget.TextInputWidget`

:class:`colander.Boolean`
    :class:`deform.widget.CheckboxWidget`

:class:`colander.Date`
    :class:`deform.widget.DateInputWidget`

:class:`colander.DateTime`
    :class:`deform.widget.DateTimeInputWidget`

:class:`colander.Tuple`
    :class:`deform.widget.Widget`

:class:`colander.Time`
    :class:`deform.widget.TimeInputWidget`

:class:`colander.Money`
    :class:`deform.widget.MoneyInputWidget`

:class:`colander.Set`
    :class:`deform.widget.CheckboxChoiceWidget`


.. note::

   Not just any widget can be used with any schema type; the
   documentation for each widget usually indicates what type it can be
   used against successfully.  If all existing widgets provided by
   Deform are insufficient, you can use a custom widget.  See
   :ref:`writing_a_widget` for more information about writing 
   a custom widget.

If you are creating a schema that contains a type which is not in this
list, or if you would like to use a different widget for a particular
field, or you want to change the settings of the default widget
associated with the type, you need to associate the field with the
widget "by hand".  There are a number of ways to do so, as outlined in
the sections below.

As an argument to a :class:`colander.SchemaNode` constructor
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

As of Deform 0.8, you may specify the widget as part of the
schema:

.. code-block:: python
   :linenos:

   import colander

   from deform import Form
   from deform.widget import TextAreaWidget

   class Person(colander.MappingSchema):
       name = colander.SchemaNode(colander.String(),
                                  widget=TextAreaWidget())
       age = colander.SchemaNode(colander.Integer(),
                                 validator=colander.Range(0, 200))

   class People(colander.SequenceSchema):
       person = Person()

   class Schema(colander.MappingSchema):
       people = People()

   schema = Schema()

   myform = Form(schema, buttons=('submit',))

Note above that we passed a ``widget`` argument to the ``name`` schema
node in the ``Person`` class above.  When a schema containing a node
with a ``widget`` argument to a schema node is rendered by Deform, the
widget specified in the node constructor is used as the widget which
should be associated with that node's form rendering.  In this case,
we will use a :class:`deform.widget.TextAreaWidget` as the ``name``
widget.

.. note::

  Widget associations done in a schema are always overridden by
  explicit widget assignments performed via
  :meth:`deform.Field.__setitem__` and
  :meth:`deform.Field.set_widgets`.

Using dictionary access to change the widget
++++++++++++++++++++++++++++++++++++++++++++

After the :class:`deform.Form` constructor is called with the schema,
you can change the widget used for a particular field by using
dictionary access to get to the field in question.  A
:class:`deform.Form` is just another kind of :class:`deform.Field`, so
the method works for either kind of object.  For example:

.. code-block:: python
   :linenos:

   from deform import Form
   from deform.widget import TextInputWidget

   myform = Form(schema, buttons=('submit',))
   myform['people']['person']['name'].widget = TextInputWidget(size=10)

This associates the :class:`~colander.String` field named ``name``
in the rendered form with an explicitly created
:class:`deform.widget.TextInputWidget` by finding the ``name`` field
via a series of ``__getitem__`` calls through the field
structure, then by assigning an explicit ``widget`` attribute to the
``name`` field.

You might want to do this in order to pass a ``size``
argument to the explicit widget creation, indicating that the size of
the ``name`` input field should be 10em rather than the default.  

Although in the example above, we associated the ``name`` field with
the same type of widget as its default, we could have
associated the ``name`` field with a completely different widget using
the same pattern.  For example:

.. code-block:: python
   :linenos:

   from deform import Form
   from deform.widget import TextAreaWidget

   myform = Form(schema, buttons=('submit',))
   myform['people']['person']['name'].widget = TextAreaWidget()

The above renders an HTML ``textarea`` input element for the ``name``
field instead of an ``input type=text`` field.  This probably does not
make much sense for a field called ``name`` (names are not usually
multiline paragraphs), but it does let us demonstrate how different
widgets can be used for the same field.

Using the :meth:`deform.Field.set_widgets` method
+++++++++++++++++++++++++++++++++++++++++++++++++

Equivalently, you can also use the :meth:`deform.Field.set_widgets`
method to associate multiple widgets with multiple fields in a form.
For example:

.. code-block:: python
   :linenos:

   from deform import Form
   from deform.widget import TextAreaWidget

   myform = Form(schema, buttons=('submit',))
   myform.set_widgets({'people.person.name':TextAreaWidget(),
                       'people.person.age':TextAreaWidget()})

Each key in the dictionary passed to :meth:`deform.Field.set_widgets`
is a "dotted name" which resolves to a single field element.  Each
value in the dictionary is a widget instance.  See
:meth:`deform.Field.set_widgets` for more information about this
method and dotted name resolution, including special cases which
involve the "splat" (``*``) character and the empty string as a key
name.


.. _using-arbitrary-form-attributes:

Using arbitrary form attributes
-------------------------------

HTML5 introduced many attributes to HTML forms, most of which appear in the HTML ``<input>`` element as a ``type`` attribute.
For the full specification, see https://www.w3.org/TR/2017/REC-html52-20171214/sec-forms.html#sec-forms
For implementations and demos, the `Mozilla Developer Network <https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input>`_ is one useful resource.

Starting with Deform 2.0.7, all of the Deform widgets support arbitrary HTML5 form attributes.
They also support *any* arbitrary attribute, such as ``readonly`` or ``disabled``.
This is useful, for example, when you want to use any of the following new HTML5 input types.

- color
- date
- datetime
- datetime-local
- email
- month
- number
- range
- search
- tel
- time
- url
- week

You can also set placeholders, use multiple file uploads, and set some client-side validation requirements without JavaScript.

The following Python code will generate the subsequent HTML and rendered HTML5 number input.

.. code-block:: python

    from decimal import Decimal

    total_employee_hours = colander.SchemaNode(
        colander.Decimal(),
        widget=widget.TextInputWidget(
            attributes={
                "type": "number",
                "inputmode": "decimal",
                "step": "0.01",
                "min": "0",
                "max": "99.99",
            }
        ),
        validator=colander.Range(min=0, max=Decimal("99.99")),
        default=30.00,
        missing=colander.drop,
    )

.. code-block:: html

    <input
        name="total_employee_hours"
        value="30.00"
        id="total_employee_hours"
        class=" form-control "
        type="number"
        inputmode="decimal"
        step="0.01"
        min="0"
        max="99.99">

**Total employee hours**

.. raw:: html

    <input
        name="total_employee_hours"
        value="30.00"
        id="total_employee_hours"
        class=" form-control "
        type="number"
        inputmode="decimal"
        step="0.01"
        min="0"
        max="99.99">

.. versionadded:: 2.0.7

    Arbitrary form control attributes, providing support for HTML5 forms.


.. _using_readonly_in_html_form_control_attributes:

Using ``readonly`` in HTML form control attributes
--------------------------------------------------

.. note::

    Naming things is hard.
    In Deform an unfortunate naming decision was made for ``readonly`` when rendering a form without any form controls using a "readonly" template.
    Oops.
    Looking back, we ought to have named it ``viewonly``, ``static``, ``immutable``, or ``readable`` to avoid confusion with the HTML attribute `readonly <https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/readonly>`_.

The ``readonly`` HTML form control attribute makes the element not mutable, meaning the user cannot edit the control.
When ``"readonly": "readonly"`` is one of the items in a dict passed into the ``attributes`` option when creating a widget, the rendered widget both prevents the user from changing the value, and if the form does not pass validation after submitted then the field value will be displayed.

``readonly`` is supported by most form controls, but not all.
Deform adds some logic to add read-only support for a few of those form controls, as described below.

`View a demonstration of ``readonly`` in HTML form control attributes <https://deformdemo.pylonsproject.org/readonly_html/>`_.

``CheckboxWidget`` and ``CheckboxChoiceWidget``
    Due to the nature of how checkbox values are processed, the ``readonly`` attribute has no effect.
    To achieve a read-only behavior, pass in ``attributes={"onclick": "return false;"}``.
    This will render as an inline JavaScript ``onclick="return false;"`` for each checkbox item.

``MoneyInputWidget``
    The provided value will be displayed in the input and be not editable.

``RadioChoiceWidget``
    For the selected value it will render an attribute in the HTML as ``readonly="readonly"``, and for all other values as ``disabled="disabled"``.

``SelectWidget``
    For selection of single options only, the selected value will render an attribute in the HTML as ``readonly="readonly"``, and for all other values as ``disabled="disabled"``.
    Multiple selections, set by the ``multiple=True`` option, do not support the ``readonly`` attribute on the ``<select>`` element.
    For multiple selections, use the ``SelectizeWidget``.

``SelectizeWidget``
    For both single and multiple selections, the selected value or values will be rendered as selected, and the others will not be selectable.
    Selectize uses JavaScript to "lock" the form control.

``TextAreaWidget``
    The provided value will be displayed in the input and be not editable.

``TextInputWidget``
    The provided value will be displayed in the input and be not editable.

.. warning::

    Regardless of using ``readonly``, never trust user input or client-side only validation, and validate submitted data on the server side to ensure that values are not altered.

.. versionadded:: 2.0.15

    Enhanced ``readonly`` form control attribute.


.. _using_selectize_widget:

Using Selectize Widget
----------------------

The Selectize widget is based on the jQuery plugin `selectize.js <https://github.com/selectize/selectize.js>`_.

Configuration options of the Selectize widget can be passed in as a dict to the keyword argument ``selectize_options``.
These options are rendered as inline JavaScript in the HTML widget.
See the available `configuration options at the selectize.js project <https://github.com/selectize/selectize.js/blob/master/docs/usage.md>`_.

By default, Deform treats any options with a ``""`` value as normal by virtue of setting ``allowEmptyOption`` to ``True``.
This will render in HTML as ``<option value="">- Select -</option>``.
You can override this default value as follows.

.. code-block:: python

    widget=deform.widget.SelectizeWidget(
        values=choices,
        selectize_options={
            "allowEmptyOption": False,
        },
    )

Using the above pattern, you can configure the Selectize widget for all of its configuration options.

Try a basic `demonstration of the Selectize widget <https://deformdemo.pylonsproject.org/selectize/>`_.
Additional options are also demonstrated.

.. versionadded:: 2.0.15


.. _date-time-inputs:

Using Date, DateTime, and Time Inputs
-------------------------------------

The :class:`deform.widget.DateInputWidget`, :class:`deform.widget.DateTimeInputWidget`, and :class:`deform.widget.TimeInputWidget` inputs all use the jQuery plugin `pickadate <https://amsul.ca/pickadate.js/>`_.
This plugin is included with Deform in the directory ``static/pickadate``.

Arbitrary options may be passed into the widget as a Python object, which will be automatically converted to a JSON object by the widget.
These options are named ``date_options`` and ``time_options``.
This is useful to set a minimum or maximum date or time, and many other options.
For the complete options, see `date options <https://amsul.ca/pickadate.js/date/#options>`_ or `time options <https://amsul.ca/pickadate.js/time/#options>`_.

Use of these widgets is not a replacement for server-side validation of the field.
It is purely a UI affordance.
If the data must be checked at input time, a separate :term:`validator` should be attached to the related schema node.


.. _masked_input:

Using Text Input Masks
----------------------

The :class:`deform.widget.TextInputWidget` and
:class:`deform.widget.CheckedInputWidget` widgets allow for the use of
a fixed-length text input mask.  Use of a text input mask causes
placeholder text to be placed in the text field input, and restricts
the type and length of the characters input into the text field.

For example:

.. code-block:: python

    form['ssn'].widget = TextInputWidget(mask='999-99-9999')

When using a text input mask:

- ``a`` represents an alpha character (A-Z,a-z).
- ``9`` represents a numeric character (0-9).
- ``*`` represents an alphanumeric character (A-Z,a-z,0-9).

All other characters in the mask will be considered mask literals.

By default the placeholder text for non-literal characters in the
field will be ``_`` (the underscore character).  To change this for a
given input field, use the ``mask_placeholder`` argument to the
TextInputWidget:

.. code-block:: python

    form['date'].widget = TextInputWidget(mask='99/99/9999',
        mask_placeholder="-")

Example masks:

Date
    99/99/9999

North American Phone Number
    (999) 999-9999

United States Social Security Number
    999-99-9999

When this option is used, the :term:`jquery.maskedinput` library must
be loaded into the page serving the form for the mask argument to have
any effect.  A copy of this library is available in the
``static/scripts`` directory of the :mod:`deform` package itself.

See `https://deformdemo.pylonsproject.org/text_input_masks/
<https://deformdemo.pylonsproject.org/text_input_masks/>`_ for a working
example.

Use of a text input mask is not a replacement for server-side
validation of the field. It is purely a UI affordance.  If the data
must be checked at input time, a separate :term:`validator` should be
attached to the related schema node.


.. _autocomplete_input:

Using the AutocompleteInputWidget
---------------------------------

The :class:`deform.widget.AutocompleteInputWidget` widget allows for
client side autocompletion from provided choices in a text input
field. To use this you **MUST** ensure that :term:`jQuery` and the
:term:`jQuery UI` plugin are available to the page where the
:class:`deform.widget.AutocompleteInputWidget` widget is rendered.

For convenience a version of the :term:`jQuery UI` (which includes the
``autocomplete`` sublibrary) is included in the :mod:`deform` static
directory. Additionally, the :term:`jQuery UI` styles for the
selection box are also included in the :mod:`deform` ``static``
directory. See :ref:`serving_up_the_rendered_form` and
:ref:`get_widget_resources` for more information about using the 
included libraries for your application.

A very simple example of using
:class:`deform.widget.AutocompleteInputWidget` follows:

.. code-block:: python

   form['frozznobs'].widget = AutocompleteInputWidget(
                                values=['spam', 'eggs', 'bar', 'baz'])

Instead of a list of values, a URL can be provided to values:

.. code-block:: python

   form['frobsnozz'].widget = AutocompleteInputWidget(
                                values='http://example.com/someapi')

In the above case, a call to the URL should provide results in a JSON-compatible
format or JSONP-compatible response if on a different host than the
application.  Something like either of these structures in JSON are suitable.

.. code-block:: javascript

    // Items are used as both value and label
    ['item-one', 'item-two', 'item-three']

    // Separate values and labels
    [
        {'value': 'item-one', 'label': 'Item One'},
        {'value': 'item-two', 'label': 'Item Two'},
        {'value': 'item-three', 'label': 'Item Three'}
    ]

The autocomplete plugin will add a query string to the request URL with the
variable ``term`` which contains the user's input at that moment.  The server
may use this to filter the returned results.  

For more information, see https://api.jqueryui.com/autocomplete/#option-source — specifically, the section concerning the ``String`` type for the ``source`` option.

Some options for the :term:`jquery.autocomplete` plugin are mapped and
can be passed to the widget. See
:class:`deform.widget.AutocompleteInputWidget` for details regarding the
available options. Passing options looks like:

.. code-block:: python

    form['nobsfrozz'].widget = AutocompleteInputWidget(
                                values=['spam, 'eggs', 'bar', 'baz'],
                                min_length=1)

See `https://deformdemo.pylonsproject.org/autocomplete_input/
<https://deformdemo.pylonsproject.org/autocomplete_input/>`_ and
`https://deformdemo.pylonsproject.org/autocomplete_remote_input/
<https://deformdemo.pylonsproject.org/autocomplete_remote_input/>`_ for
working examples. A working example of a remote URL providing
completion data can be found at
`https://deformdemo.pylonsproject.org/autocomplete_input_values/
<https://deformdemo.pylonsproject.org/autocomplete_input_values/>`_.

Use of :class:`deform.widget.AutocompleteInputWidget` is not a
replacement for server-side validation of the field. It is purely a UI
affordance.  If the data must be checked at input time, a separate
:term:`validator` should be attached to the related schema node.

Creating a New Schema Type
--------------------------

Sometimes the default schema types offered by Colander may not be sufficient
to model all the structures in your application.  

If this happens, refer to the Colander documentation on
:ref:`colander:defining_a_new_type`.
