🔬 Filters
==========

.. currentmodule:: pywa.filters

Filtros são usados pelos handlers para decidir se uma atualização deve ser processada ou ignorada.

A biblioteca fornece vários filtros embutidos, disponíveis no módulo :mod:`pywa.filters`.

-----------------
Uso Básico
-----------------

.. code-block:: python
    :emphasize-lines: 5, 10

    from pywa import WhatsApp, types, filters

    wa = WhatsApp(...)

    @wa.on_message(filters.startswith("Hello", "Hi", ignore_case=True))
    def handle_hello(wa: WhatsApp, msg: types.Message):
        msg.react("👋")
        msg.reply(
            f"Hello {msg.from_user.name}!",
            buttons=[types.Button("Click me!", "click")]
        )

    @wa.on_callback(filters.matches("click"))
    def handle_click(wa: WhatsApp, clb: types.CallbackButton):
        clb.reply("You clicked me!")

-----------------
Combinando Filtros
-----------------

Filtros podem ser combinados com operadores lógicos:

- ``&`` → **and**
- ``|`` → **or**
- ``~`` → **not**

.. code-block:: python

    from pywa import filters

    # image with caption
    filters.image & filters.has_caption

    # text or image
    filters.text | filters.image

    # message must not contain "bad word"
    ~filters.contains("bad word")

.. tip::

   Todos os filtros de correspondência (:meth:`matches`, :meth:`contains` etc.) retornam ``True`` se **qualquer** uma das opções fornecidas corresponder.
   Então, em vez de escrever:

   .. code-block:: python

        filters.matches("hello") | filters.matches("hi")

   Você pode simplesmente escrever:

   .. code-block:: python

        filters.matches("hello", "hi")

-----------------
Filtros Personalizados
-----------------

Você pode definir seus próprios filtros escrevendo uma função que recebe o cliente e a atualização e retorna um booleano.

Se a função retornar ``True`` → o handler processará a atualização.
Se retornar ``False`` → a atualização será ignorada.

.. note::

   - Filtros personalizados devem ser encapsulados com :func:`pywa.filters.new`.
   - Você pode combinar filtros personalizados e embutidos usando operadores lógicos.
   - Funções assíncronas podem ser usadas como filtros **somente** com o cliente assíncrono.

.. code-block:: python
    :emphasize-lines: 3-4, 8, 13

    from pywa import WhatsApp, types, filters

    def without_xyz_filter(_: WhatsApp, msg: types.Message) -> bool:
        return msg.text and "xyz" not in msg.text

    wa = WhatsApp(...)

    @wa.on_message(filters.new(without_xyz_filter))
    def messages_without_xyz(wa: WhatsApp, msg: types.Message):
        msg.reply("You said something without xyz!")

    # Or with lambda:
    @wa.on_message(filters.new(lambda _, msg: msg.text and "xyz" not in msg.text))
    def messages_without_xyz(wa: WhatsApp, msg: types.Message):
        msg.reply("You said something without xyz!")

-----------------
Filtros Embutidos
-----------------

.. toctree::

    ./common_filters
    ./message_filters
    ./message_status_filters
