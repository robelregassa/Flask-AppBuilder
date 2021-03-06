Advanced Configuration
======================

Security
--------

To block or set the allowed permissions on a view, just set the *base_permissions* property with the base permissions

::

    class GroupModelView(ModelView):
        datamodel = SQLAModel(Group)
        base_permissions = ['can_add','can_delete']

With this initial config, the framework will only create 'can_add' and 'can_delete'
permissions on GroupModelView as the only allowed. So users and even the administrator
of the application will not have the possibility to add list or show permissions on Group table view.
Base available permission are: can_add, can_edit, can_delete, can_list, can_show. More detailed info on :doc:`security`

Base Filtering
--------------

To filter a views data, just set the *base_filter* property with your base filters. These will allways be applied first on any search.

It's very flexible, you can apply multiple filters with static values, or values based on a function you define. On this next example we are filtering a view by the logged in user and with column *name* starting with "a"

*base_filters* is a list of lists with 3 values [['column name',FilterClass,'filter value],...]

::

    def get_user():
        return g.user

    class MyView(ModelView):
        datamodel = SQLAModel(MyTable)
        base_filters = [['created_by', FilterEqualFunction, get_user],
                        ['name', FilterStartsWith, 'a']]


Default Order
-------------

Use a default order on your lists, this can be overridden by the user on the UI. Data structure ('col_name':'asc|desc')

::

    class MyView(ModelView):
        datamodel = SQLAModel(MyTable)
        base_order = ('my_col_to_be_ordered','asc')


Forms
-----

- You can create a custom query filter for all related columns like this::

    class ContactModelView(ModelView):
        datamodel = SQLAModel(Contact)
        add_form_query_rel_fields = [('group',
                    SQLAModel(Group, db.session),
                    [['name',FilterStartsWith,'W']]
                    )]


This will filter list combo on Contact's model related with Group model. The combo will be filtered with entries that start with W. You can define individual filters for add and edit. Take a look at the :doc:`api`
If you want to filter multiple related fields just add tuples to the list, remember you can add multiple filters for each field also, take a look at the *base_filter* property::

    class ContactModelView(ModelView):
        datamodel = SQLAModel(Contact)
        add_form_query_rel_fields = [('group',
                    SQLAModel(Group, db.session),
                    [['name',FilterStartsWith,'W']]
                    ),
                    ('gender',
                    SQLAModel(Gender, db.session),
                    [['name',FilterStartsWith,'M']]
                    )
        ]


- You can define your own Add, Edit forms to override the automatic form creation::

    class MyView(ModelView):
        datamodel = SQLAModel(MyModel)
        add_form = AddFormWTF


- You can define what columns will be included on Add or Edit forms,
for example if you have automatic fields like user or date, you can remove this from the Add Form::

    class MyView(ModelView):
        datamodel = SQLAModel(MyModel)
        add_columns = ['my_field1','my_field2']
        edit_columns = ['my_field1']

- You can contribute with any additional field that are not on a table/model,
for example a confirmation field::

    class ContactModelView(ModelView):
        datamodel = SQLAModel(Contact)
        add_form_extra_fields = {'extra': TextField(gettext('Extra Field'),
                        description=gettext('Extra Field description'),
                        widget=BS3TextFieldWidget())}


- You can contribute with your own additional form validations rules.
Remember the framework will automatically validate any field that is defined on the database
with *Not Null* (Required) or Unique constraints::

    class MyView(ModelView):
        datamodel = SQLAModel(MyModel)
        validators_columns = {'my_field1':[EqualTo('my_field2',
                                            message=gettext('fields must match'))
                                          ]
        }

Take a look at the :doc:`api`. Experiment with *add_form*, *edit_form*, *add_columns*, *edit_columns*, *validators_columns*, *add_form_extra_fields*, *edit_form_extra_fields*
