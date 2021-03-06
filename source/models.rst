Models
======

``models.py`` is where you define your app's data models:

-  Subsession
-  Group
-  Player

A player is part of a group, which is part of a subsession.
See :ref:`conceptual_overview`.

Model fields: an example
------------------------

The main purpose of ``models.py`` is to define the columns of your
database tables. Let's say you want your experiment to generate data
that looks like this:

.. csv-table::
    :header-rows: 1

    name,age,is_student
    John,30,False
    Alice,22,True
    Bob,35,False
    ...

Here is how to define the above table structure:

.. code-block:: python

    class Player(BasePlayer):
        ...
        name = models.StringField()
        age = models.IntegerField()
        is_student = models.BooleanField()

When you run ``otree resetdb``, it will scan your ``models.py``
and create your database tables accordingly.
(Therefore, you need to run ``resetdb`` if you have added,
removed, or changed a field in ``models.py``.)

Model fields
~~~~~~~~~~~~

Here are the main field types:

-   ``BooleanField`` (for true/false and yes/no values)
-   ``CurrencyField`` for currency amounts; see :ref:`currency`.
-   ``IntegerField``
-   ``FloatField`` (for real numbers)
-   ``StringField`` (for text strings)
-   ``LongStringField`` (for long text strings)

``StringField`` and ``LongStringField`` are new (added January 2018).
See :ref:`v20` for more information.

Setting a field's initial/default value
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Any field you define will have the initial value of ``None``.
If you want to give it an initial value, you can use ``initial=``:

.. code-block:: python

    class Player(BasePlayer):
        some_number = models.IntegerField(initial=0)


min, max, choices
~~~~~~~~~~~~~~~~~

For info on how to set a field's ``min``, ``max``, or ``choices``,
see :ref:`form-validation`.

.. _constants:

Constants
---------

The ``Constants`` class is the recommended place to put your app's
parameters and constants that do not vary from player
to player.

Here are the required constants:

-   ``name_in_url``: the name used to identify your app in the
    participant's URL.

    For example, if you set it to ``public_goods``, a participant's URL might
    look like this:

    ``http://otree-demo.herokuapp.com/p/zuzepona/public_goods/Introduction/1/``

-  ``players_per_group`` (described in :ref:`groups`)

-  ``num_rounds`` (described in :ref:`rounds`)


Subsession
----------

Here is a list of attributes and methods for subsession objects.


session
~~~~~~~

The session this subsession belongs to.
See :ref:`object_model`.


round_number
~~~~~~~~~~~~

Gives the current round number.
Only relevant if the app has multiple rounds
(set in ``Constants.num_rounds``).
See :ref:`rounds`.

.. _creating_session:

creating_session
~~~~~~~~~~~~~~~~

This method is executed when the session is created:

.. figure:: _static/creating-session.png

``creating_session`` allows you to initialize the round,
by setting initial values on fields on players, groups, participants, or the subsession.
For example:

.. code-block:: python

    class Subsession(BaseSubsession):

        def creating_session(self):
            for p in self.get_players():
                p.some_field = some_value

More info on the section on :ref:`treatments <treatments>` and
:ref:`group shuffling <shuffling>`.

If your app has 1 round, ``creating_session`` will execute once.
If your app has N rounds, it will execute N times consecutively;
that is, once on each subsession instance.

.. note::
    This method does NOT run at the beginning of each round.
    For that, you should use a wait page with :ref:`after_all_players_arrive`.

.. _before_session_starts:

before_session_starts
~~~~~~~~~~~~~~~~~~~~~

``before_session_starts`` has been renamed to :ref:`creating_session`.
However, new versions of oTree still execute ``before_session_starts``,
for backwards compatibility.

group_randomly()
~~~~~~~~~~~~~~~~

See :ref:`shuffling`.

group_like_round()
~~~~~~~~~~~~~~~~~~

See :ref:`shuffling`.

get_group_matrix()
~~~~~~~~~~~~~~~~~~

See :ref:`shuffling`.

set_group_matrix()
~~~~~~~~~~~~~~~~~~

See :ref:`shuffling`.


get_groups()
~~~~~~~~~~~~

Returns a list of all the groups in the subsession.

get_players()
~~~~~~~~~~~~~

Returns a list of all the players in the subsession.

in_previous_rounds()
~~~~~~~~~~~~~~~~~~~~

See :ref:`in_rounds`.

in_all_rounds()
~~~~~~~~~~~~~~~

See :ref:`in_rounds`.

in_round(round_number)
~~~~~~~~~~~~~~~~~~~~~~

See :ref:`in_rounds`.

in_rounds(self, first, last)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

See :ref:`in_rounds`.



Group
-----

Here is a list of attributes and methods for group objects.

session/subsession
~~~~~~~~~~~~~~~~~~

The session/subsession this group belongs to.
See :ref:`object_model`.


get_players()
~~~~~~~~~~~~~

See :ref:`groups`.

get_player_by_role(role)
~~~~~~~~~~~~~~~~~~~~~~~~

See :ref:`groups`.

get_player_by_id(id_in_group)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

See :ref:`groups`.

set_players(players_list)
~~~~~~~~~~~~~~~~~~~~~~~~~

See :ref:`shuffling`.

in_previous_rounds()
~~~~~~~~~~~~~~~~~~~~

See :ref:`in_rounds`.

in_all_rounds()
~~~~~~~~~~~~~~~

See :ref:`in_rounds`.

in_round(round_number)
~~~~~~~~~~~~~~~~~~~~~~

See :ref:`in_rounds`.

in_rounds(self, first, last)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

See :ref:`in_rounds`.

Player
------

Here is a list of attributes and methods for player objects.

id_in_group
~~~~~~~~~~~
Integer starting from 1. In multiplayer games,
indicates whether this is player 1, player 2, etc.

payoff
~~~~~~
The player's payoff in this round. See :ref:`payoff`.

session/subsession/group/participant
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The session/subsession/group/participant this player belongs to.
See :ref:`object_model`.


get_others_in_group()
~~~~~~~~~~~~~~~~~~~~~

See :ref:`groups`.

get_others_in_subsession()
~~~~~~~~~~~~~~~~~~~~~~~~~~

See :ref:`groups`.

.. _role:

role()
~~~~~~
You can define this method to return a string label of the player's role,
usually depending on the player's ``id_in_group``.

For example::

    def role(self):
        if self.id_in_group == 1:
            return 'buyer'
        if self.id_in_group == 2:
            return 'seller'

Then you can use ``get_player_by_role('seller')`` to get player 2.
See :ref:`groups`.

Also, the player's role will be displayed in the oTree admin interface,
in the "results" tab.

in_previous_rounds()
~~~~~~~~~~~~~~~~~~~~

See :ref:`in_rounds`.

in_all_rounds()
~~~~~~~~~~~~~~~

See :ref:`in_rounds`.

in_round(round_number)
~~~~~~~~~~~~~~~~~~~~~~

See :ref:`in_rounds`.

in_rounds(self, first, last)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

See :ref:`in_rounds`.

Session
-------

num_participants
~~~~~~~~~~~~~~~~

The number of participants in the session.

config
~~~~~~

See :ref:`edit_config`
and :ref:`session_config_treatments`.

vars
~~~~

See :ref:`session_vars`.

Participant
-----------

vars
~~~~

See :ref:`vars`.

label
~~~~~

See :ref:`participant_label`.

id_in_session
~~~~~~~~~~~~~

The participant's ID in the session. This is the same as the player's
``id_in_subsession``.

payoff
~~~~~~

See :ref:`payoff`.

payoff_plus_participation_fee()
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

See :ref:`payoff`.


.. _how_otree_executes_code:

How oTree executes your code
----------------------------

Any code that is not inside a method
is basically *global* and *will only be executed once* --
when the server starts.

Some people write code mistakenly thinking that it will be re-executed for each
new session. For example, someone who wants to generate a random probability that a coin flip will
come up "heads" might do this in models.py:

.. code-block:: python

    class Constants(BaseConstants):
        heads_probability = random.random() # wrong

When the server starts, it loads models.py,
and executes the ``random.random()`` only once.
It will evaluate to some random number, for example "0.257291".
This means you have basically written this:

.. code-block:: python

    class Constants(BaseConstants):
        heads_probability = 0.257291

Because ``Constants`` is a global variable, that value 0.257291 will now be shared
by all players in all sessions.

For the same reason, this will not work either:

.. code-block:: python

    class Player(BasePlayer):

        heads_probability = models.FloatField(
            # wrong
            initial=random.random()
        )

The solution is to generate the random variables inside a method,
such as :ref:`creating_session`.

What's the difference between "Player" and "player"?
----------------------------------------------------

In your code, you should always use lower-case ``player``,
``group``, and ``subsession``. The only exception is defining the classes
in models.py, where you use ``class Player(BasePlayer)`` etc.

We use uppercase (e.g. ``Player``) when we are referring to the whole table
of players, and lowercase (``player``) when referring to a particular player,
i.e. a row in the table. In Python, ``Player`` is a class, and ``player``
is an instance of that class.

For example, in a template, to display a player's payoff,
we must use ``{{ player.payoff }}``, not ``{{ Player.payoff }}``.

However, for ``Constants``, we always use uppercase.
That's because ``Constants`` is not a database table with instances/rows,
because the constants are the same for all players.

How to make many fields
-----------------------

Let's say your app has many fields that are almost the same, such as:

.. code-block:: python

    class Player(BasePlayer):

        f1 = models.IntegerField(
            choices=[-1, 0, 1], widget=widgets.RadioSelect,
            blank=True, initial=0
        )
        f2 = models.IntegerField(
            choices=[-1, 0, 1], widget=widgets.RadioSelect,
            blank=True, initial=0
        )
        f3 = models.IntegerField(
            choices=[-1, 0, 1], widget=widgets.RadioSelect,
            blank=True, initial=0
        )
        f4 = models.IntegerField(
            choices=[-1, 0, 1], widget=widgets.RadioSelect,
            blank=True, initial=0
        )
        f5 = models.IntegerField(
            choices=[-1, 0, 1], widget=widgets.RadioSelect,
            blank=True, initial=0
        )
        f6 = models.IntegerField(
            choices=[-1, 0, 1], widget=widgets.RadioSelect,
            blank=True, initial=0
        )
        f7 = models.IntegerField(
            choices=[-1, 0, 1], widget=widgets.RadioSelect,
            blank=True, initial=0
        )
        f8 = models.IntegerField(
            choices=[-1, 0, 1], widget=widgets.RadioSelect,
            blank=True, initial=0
        )
        f9 = models.IntegerField(
            choices=[-1, 0, 1], widget=widgets.RadioSelect,
            blank=True, initial=0
        )
        f10 = models.IntegerField(
            choices=[-1, 0, 1], widget=widgets.RadioSelect,
            blank=True, initial=0
        )

        # etc...

This is quite complex; you should look for a way to simplify.

Are the fields all displayed on separate pages? If so, consider converting
this to a 10-round game with just one field. See the
`real effort <https://github.com/oTree-org/oTree/tree/master/real_effort>`__
sample game for an example of how to just have 1 page that gets looped over many rounds,
varying the question that gets displayed with each round.

If that's not possible, then put the repeated arguments into a dict in ``Constants``:

.. code-block:: python


    class Constants(BaseConstants):
        ...

        field_args = dict(
            choices=[-1, 0, 1], widget=widgets.RadioSelect, blank=True, initial=0
        )

Then use these arguments using dict unpacking (``**``):

.. code-block:: python

    class Player(BasePlayer):

        f1 = models.IntegerField(**Constants.field_args)
        f2 = models.IntegerField(**Constants.field_args)
        f3 = models.IntegerField(**Constants.field_args)
        # etc...
        f10 = models.IntegerField(**Constants.field_args)

You can also pass unique arguments in addition to the shared ones:

.. code-block:: python

    class Player(BasePlayer):

        f1 = models.IntegerField(label='What will you choose for 1?', **Constants.field_args)
        f2 = models.IntegerField(label='What will you choose for 2?', **Constants.field_args)
        f3 = models.IntegerField(label='What will you choose for 3?', **Constants.field_args)
        # etc...
        f10 = models.IntegerField(label='What will you choose for 10?', **Constants.field_args)
