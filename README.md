BERNARD
=======

Create powerful and flexible bots using BERNARD. Features (will) include:

- Represent conversations as finite-state machines
- Multi-platform (Facebook Messenger, WeChat, VK, Line, Twilio, ...)
- Flexible translation system. Handles language, plurals, gender and anything you'd like
- Run-time analysis through logging

BERNARD stands for "Bot Engine Responding Naturally At Requests Detection".

# Design goals

This framework does not exist yet. Here we'll list a few code snippets of how we want things to
look like from a developer standpoint.

## Story interface

Each state of the FSM will be represented by a class. The class will have as members/functions
all the tools you need to handle the message (the message itself, a way to reply, get the context,
get info about the user, ...).

```python
class BaseStory(Story):
    async def error(self):
        # Something went wrong while generating the response, that's a 500 error
        pass

    async def confused(self):
        # That's when the bot is confused and does not find anything to say
        pass

class MyStory(BaseStory):
    async def handle(self):
        # Produce the messages that this state is supposed to produce
        pass

    # MISSING = how do you do transitions?
```

## Transitions

TODO

## Intents DB

TODO

## Message layering

Simply put, messages are composed of content layers. Each platform will support only specific
combinations of layers. Like:

- Everybody handles a single text layer
- Facebook might allow for text + quick replies
- VK has text + image whereas Facebook has not
- and so on

```python
from bernard import layers

class MyStory(BaseStory):
    async def handle(self):
        self.reply(layers.Text('Hello'))

        with self.alternate() as msg:
            msg(
                layers.Card(card1),
                layers.Card(card2),
                layers.Card(card3),
            )

            msg(
                layers.Image(image1),
                layers.Image(image2),
                layers.Image(image3),
            )
```

Here, two ways to send a message:

1. Use `self.reply()` which allows only one alternative to be sent. Do this if you know that all
   the platforms you support will handle your layer(s) correctly
2. Use `self.alternate()` context manager in order to propose alternative sets of layers. By
   default, platforms will accept layers only if they fully support them, however you can use
   middlewares to transform layers into other layers so you can emulate features that do not exist
   (or that exist differently).
   It will choose the best-matching set of layers and send only this one.

## Configuration

We'll use Sanic's app configuration facilities. This lays two questions:

1. How is Sanic instantiated?
2. How do you access the app instance from anywhere in the code?

Configuration is composed of two things:

1. App-specific configuration (like the name of classes, list of middlewares, etc) that is commited
   and found in a Python file
2. Instance-specific configuration that comes from environment variables (like tokens and passwords)

## Middlewares

Middlewares allow to automatically propose alternatives to layer sets.

Default middleware are defined at the app level in the configuration, however each story can
customize its middlewares.

```python
class MyStory(BaseStory):
    def get_middlewares(self):
        m = super(MyStory, self).get_middlewares()
        m.append(MyOtherMiddleware())
        return m
```

TODO: define middleware interface

## Translation

Instead of just depending on the language, the translation system allows to make translations vary
according to an arbitrary number of criterion.

You can use translations this way:

```python
from bernard.i18n import translate as t
from bernard import layers

class MyStory(BaseStory):
    async def handle(self):
        # Simple case
        self.reply(layers.Text(t.HELLO)) # simple form
        self.reply(layers.Text(t('HELLO'))) # verbose form

        # With plural
        self.reply(layers.Text(t('PLURALIZE', 42)))

        # With substitution
        self.reply(layers.Text(t('SUBSTITUTE_ME', foo='bar')))
        self.reply(layers.Text(t('SUBSTITUTE_ME_PLURALIZE', 42, foo='bar')))
```

The idea is that you can make vary the translation according to lang and gender, by example. You
could get something like that:

| Key                           | Male              | Female             | Unknown             |
| ----------------------------- | ----------------- | ------------------ | ------------------- |
| SUBSTITUTE_ME_PLURALIZE\[0,1] | {name} le grand   | {name} la grande   | {name} en grandeur  | 
| SUBSTITUTE_ME_PLURALIZE\[2,]  | {name} les grands | {name} les grandes | {name} en grandeurs | 
| HELLO                         |                   |                    | Bonjour             |

Translated strings are not real strings but rather objects that you can render any time you'd like
using a context of your choosing. This will be done automatically by the layer-rendering functions
in the framework and other framework facilities.

The other option would be to have a magic Django `get_language()`-like function, but I don't like
magic and I feel it can be avoided here.

## Translations configuration

TODO

## Message templating

TODO

## Media handling

TODO

## Logging

TODO

## Single/Multi-user conversations

TODO

## Context

TODO

## Typing indications

TODO

## Pause between messages

TODO

## Send a message from a cron/external trigger

TODO
