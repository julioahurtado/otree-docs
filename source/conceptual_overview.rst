.. _conceptual_overview:

Conceptual overview
===================

Sessions
--------

In oTree, a session is an event during which multiple participants take part in a series of tasks or games.
An example of a session would be:

"A number of participants will come to the lab and play a public goods game, followed by a
questionnaire. Participants get paid EUR 10.00 for showing up, plus their earnings from the
games."

Subsessions
-----------

A session is a series of subsessions;
subsessions are the "sections" or "modules" that constitute a session.
For example, if a session consists of a public goods game followed by a questionnaire,
the public goods game would be subsession 1, and the questionnaire would be subsession 2.
In turn, each subsession is a sequence of pages the user must navigate through.
For example, if you had a 4-page public goods game followed by a 2-page questionnaire:

.. figure:: _static/diagrams/session_subsession.png
    :align: center

If a game is repeated for multiple rounds, each round is a subsession.

Groups
------

Each subsession can be further divided into groups of players;
for example, you could have a subsession with 30 players, divided into 15 groups of 2 players each.
(Note: groups can be shuffled between subsessions.)


Object hierarchy
----------------

oTree's entities can be arranged into the following hierarchy::

    Session
        Subsession
            Group
                Player
                    Page


- A session is a series of subsessions
- A subsession contains multiple groups
- A group contains multiple players
- Each player proceeds through multiple pages

.. _participants_and_players:

Participant
-----------

In oTree, the terms "player" and "participant" have distinct meanings.
The relationship between participant and player is the same as the
relationship between session and subsession:

.. figure:: _static/diagrams/participant_player.png
    :align: center

A player is an instance of a participant in one particular subsession.
A player is like a temporary "role" played by a participant.
A participant can be player 2 in the first subsession, player 1 in the
next subsession, and so on.

.. _object_model:

What is "self"?
---------------

Short answer
~~~~~~~~~~~~

In Python, ``self`` is an instance of the class you're currently under.
If you are ever wondering what ``self`` means in a particular context,
scroll up until you see the name of the class.

For example, in this code, ``self`` is a group
(an instance of the ``Group`` class).

.. code-block:: python

    class Group(BaseGroup):

        def set_bonus(self):
            self.bonus = 100


Long answer
~~~~~~~~~~~

Here is an example that explains what ``self`` does.

Let's say you want to write a function that sets a player's payoff.
The argument to the function is a player object:

.. code-block:: python

    def set_bonus(player):
        player.bonus = 100

If you have not done object-oriented programming before, your first instinct
may be to define it as a function somewhere in your module:

.. code-block:: python

    class Subsession(BaseSubsession):
        ...

    class Group(BaseGroup):
        ...

    class Player(BasePlayer):
        ...

    def set_bonus(player):
        player.bonus = 100

However, in object-oriented programming, it's recommended to keep your code
organized by putting functions inside the class that they are related to.
So, we should do this:

.. code-block:: python

    class Player(BasePlayer):

        def set_bonus(player):
            player.bonus = 100

(Once we move it inside a class, we usually call it a "method" rather than a
"function", but the distinction is not that important.)

Now that it's under ``Player``, we should refer to it as ``Player.set_bonus``,
not just ``set_bonus``. So, if we have a player object, we can do this:

.. code-block:: python

    Player.set_bonus(some_player)

In Python, this can also be written in the shorter form:

    ``some_player.set_bonus()``

A little bit of magic takes place, and the ``some_player`` on the left
of the dot is automatically passed as the argument to ``set_bonus``,
so it can be omitted from the parentheses.

In fact, every method defined under ``Player`` must take an argument
which is a ``player`` object:

.. code-block:: python

    class Player(BasePlayer):

        def set_bonus(player):
            player.bonus = 100

        def do_stuff(player):
            ...

        def do_stuff2(player):
            ...

However, there is one problem with this code.
If I decide to rename my class from ``Player`` to ``Contestant``,
that means I would have to rename lots of variables everywhere in my class:

.. code-block:: python

    class Contestant(BaseContestant):

        def set_bonus(contestant):
            contestant.bonus = 100

        def do_stuff(contestant):
            ...

        def do_stuff2(contestant):
            ...

To avoid this inconvenience, in Python it's recommended to always name
a method's first argument ``self``, with the understanding that ``self``
is an instance of whatever class you are in:

So, this:

.. code-block:: python

    class Player(BasePlayer):

        def set_bonus(player):
            player.bonus = 100

Becomes this:

.. code-block:: python

    class Player(BasePlayer):

        def set_bonus(self):
            self.bonus = 100

We know ``self`` is a player, because we are in the ``Player``
class. The name ``self`` is just shorter and more convenient than ``player``.

This is similar to how people don't use their own names when they talk about themselves; they just
use pronouns like "me", "myself", and "I". So, ``self`` is basically a pronoun.

Here is a diagram of how you can refer to objects in the hierarchy within your code:

.. figure:: _static/diagrams/object_model_self.png
    :align: center

For example, if you are in a method on the ``Player`` class, you can
access the player's payoff with ``self.payoff`` (because ``self`` is the
player). But if you are inside a ``Page`` class in ``pages.py``, the
equivalent expression is ``self.player.payoff``,
which traverses the pointer from 'page' to 'player'.

Self: extended examples
-----------------------

Here are some code examples to illustrate:

.. code-block:: python

    class Session(...) # this class is defined in oTree-core
        def example(self):

            # current session object
            self

            self.config

            # child objects
            self.get_subsessions()
            self.get_participants()

    class Participant(...) # this class is defined in oTree-core
        def example(self):

            # current participant object
            self

            # parent objects
            self.session

            # child objects
            self.get_players()

in your ``models.py``

.. code-block:: python

    class Subsession(BaseSubsession):
        def example(self):

            # current subsession object
            self

            # parent objects
            self.session

            # child objects
            self.get_groups()
            self.get_players()

            # accessing previous Subsession objects
            self.in_previous_rounds()
            self.in_all_rounds()

    class Group(BaseGroup):
        def example(self):

            # current group object
            self

            # parent objects
            self.session
            self.subsession

            # child objects
            self.get_players()

    class Player(BasePlayer):

        def example(self):

            # current player object
            self

            # method you defined on the current object
            self.my_custom_method()

            # parent objects
            self.session
            self.subsession
            self.group
            self.participant

            self.session.config

            # accessing previous player objects
            self.in_previous_rounds()

            # equivalent to self.in_previous_rounds() + [self]
            self.in_all_rounds()

in your ``pages.py``

.. code-block:: python

    class MyPage(Page):
        def example(self):

            # current page object
            self

            # parent objects
            self.session
            self.subsession
            self.group
            self.player
            self.participant
            self.session.config



