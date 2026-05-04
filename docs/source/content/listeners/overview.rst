📥 Listeners
==================

.. currentmodule:: pywa.types.sent_update

Ao lidar com atualizações, na maioria das vezes você solicita ao usuário uma entrada (por ex. uma resposta, texto, clique em botão, etc.). É aí que os listeners entram em cena.
Com listeners, você pode criar um handler `inline` que aguarda uma entrada específica do usuário e retorna o resultado.


Escuta
_________

Neste exemplo, criaremos um listener que aguarda o usuário enviar sua idade. O listener aguardará uma mensagem de texto com dígitos do usuário e então responderá com uma mensagem baseada na idade fornecida.

.. code-block:: python
    :linenos:
    :emphasize-lines: 8-10

    from pywa import WhatsApp, types, filters

    wa = WhatsApp(...)

    @wa.on_message(filters.command("start"))
    def start(client: WhatsApp, msg: types.Message):
        sent = msg.reply("Hello! How old are you?")
        age_reply: Message = sent.wait_for_reply(
            filters=filters.text & filters.new(lambda _, m: m.text.isdigit())
        )
        age = int(age_reply.text)
        if age < 18:
            age_reply.reply("You are too young to use this service.")
            # Handle the case when the user is too young
        else:
            age_reply.reply("Welcome! You can now use the service.")
            # Handle the case when the user is old enough

.. role:: python(code)
   :language: python

No exemplo acima, armazenamos a mensagem enviada na variável ``sent``. Em seguida, usamos o método :meth:`~SentMessage.wait_for_reply` para criar um listener que aguarda uma resposta do usuário. O listener aguardará uma mensagem que corresponda ao filtro :python:`filters.text & filters.new(lambda _, m: m.text.isdigit())`, o que significa que aguardará uma mensagem de texto contendo apenas dígitos.
Quando o usuário envia uma mensagem que corresponde ao filtro, o listener retorna a mensagem como um objeto :class:`~pywa.types.Message`, que armazenamos na variável ``age_reply``. Em seguida, convertemos o texto da mensagem para um inteiro e verificamos se o usuário tem idade suficiente para usar o serviço.


Cancelamento
_________

Agora, os listeners são bloqueantes. Isso significa que a execução do código será interrompida até que o listener retorne um resultado. No entanto, você pode cancelar o listener caso queira parar de aguardar uma resposta. Por exemplo, você pode adicionar um botão à mensagem que o usuário pode pressionar para cancelar o listener, ou definir um timeout para que o listener pare de aguardar após um determinado período de tempo.

.. code-block:: python
    :linenos:
    :emphasize-lines: 10-11

    from pywa import WhatsApp, types, filters

    wa = WhatsApp(...)

    @wa.on_message(filters.command("start"))
    def start(_: WhatsApp, msg: types.Message):
        sent = msg.reply("Hello! How old are you?", buttons=[types.Button(title="Cancel", callback_data="cancel")])
        age_reply = sent.wait_for_reply(
            filters=filters.text & filters.new(lambda _, m: m.text.isdigit()),
            cancelers=filters.callback_button & filters.matches("cancel"),
            timeout=60,
        )
        ...

No exemplo acima, adicionamos um botão à mensagem que o usuário pode pressionar para cancelar o listener. Também definimos um timeout de 60 segundos. Se o usuário pressionar o botão de cancelamento ou se o listener expirar, ele para de aguardar uma resposta e lança uma exceção.

Lidando com cancelamento e timeout
____________________________

Quando um listener é cancelado ou expira, ele lança uma exceção. Na maioria das vezes, você vai querer tratar essas exceções para oferecer uma melhor experiência ao usuário. PyWa fornece duas exceções para essa finalidade: :class:`~pywa.listeners.ListenerCanceled` e :class:`~pywa.listeners.ListenerTimeout`.
Você pode usar essas exceções para tratar os casos de cancelamento e timeout em seu código. Veja um exemplo:

.. code-block:: python
    :linenos:
    :emphasize-lines: 14-15, 17, 20

    from pywa import WhatsApp, types, filters

    wa = WhatsApp(...)


    @wa.on_message(filters.command("start"))
    def start(_: WhatsApp, msg: types.Message):
        try:
            age_reply = msg.reply(
                text="Hello! How old are you?",
                buttons=[types.Button(title="Cancel", callback_data="cancel")],
            ).wait_for_reply(
                filters=filters.text & filters.new(lambda _, m: m.text.isdigit()),
                cancelers=filters.callback_button & filters.matches("cancel"),
                timeout=60,
            )
        except types.ListenerCanceled:
            msg.reply("You canceled the operation by clicking the cancel button.")
            return
        except types.ListenerTimeout:
            msg.reply("You took too long to respond. Please try again later.")
            return
        ...


No exemplo acima, usamos um bloco try-except para tratar as exceções :class:`~pywa.listeners.ListenerCanceled` e :class:`~pywa.listeners.ListenerTimeout`. Se o usuário cancelar o listener clicando no botão de cancelamento, enviamos uma mensagem informando que a operação foi cancelada. Se o listener expirar, enviamos uma mensagem informando que o usuário demorou demais para responder.
Se o listener retornar um resultado, podemos continuar processando a entrada do usuário normalmente.


Listeners personalizados
_________________

.. currentmodule:: pywa.client

Você pode criar listeners personalizados usando o método bruto :meth:`WhatsApp.listen`. Esse método permite criar um listener que aguarda uma atualização específica e retorna o resultado quando ela é recebida.

Por exemplo, vamos criar um listener que aguarda outro usuário entrar no bot e se tornar um administrador. Criaremos um banco de dados simples para armazenar usuários e administradores, e então criaremos um listener que aguarda um usuário entrar no bot e o adiciona como administrador se ele ainda não estiver registrado.

.. code-block:: python
    :linenos:
    :emphasize-lines: 33-38

    from pywa import WhatsApp, types, filters, listeners

    wa = WhatsApp(...)

    class Database:
        def __init__(self):
            self._users = []
            self._admins = set()

        def add_user(self, user: str):
            if user not in self._users:
                self._users.append(user)

        def is_user_exists(self, user: str) -> bool:
            return user in self._users

        def add_admin(self, admin: str):
            if admin not in self._admins:
                self._admins.add(admin)
                self.add_user(admin)

        def is_admin(self, user: str) -> bool:
            return user in self._admins

    db = Database()
    db.add_admin("my_phone_number")  # Add your phone number as an default admin

    @wa.on_message(filters.command("add_admin") & filters.new(lambda _, msg: db.is_admin(msg.sender)))
    def add_admin(client: WhatsApp, msg: types.Message):
        _, new_admin_phone = msg.text.split(maxsplit=1)  # command is /add_admin <phone_number>
        if not db.is_user_exists(new_admin_phone):
            msg.reply("This user is not registered with the bot. Please ask them to enter the bot first.")
            new_chat: types.ChatOpened = client.listen(
                to=listeners.UserUpdateListenerIdentifier(
                    sender=new_admin_phone, recipient=client.phone_id
                ),
                filters=filters.chat_opened,
            )
            new_chat.reply(f"Hi {new_chat.from_user.name}, you have been added as an admin.")
            msg.reply(f"{new_chat.from_user.name} entered the bot, you are now an admin.")
        else:
            db.add_admin(new_admin_phone)
            msg.reply(f"{new_admin_phone} is now an admin.")

.. attention::

    Se o listener **não utilizou** a atualização (a atualização não correspondeu aos filtros nem aos canceladores), a atualização **será passada para os handlers**.
    Isso significa que a atualização pode ser processada por outros handlers registrados para lidar com o mesmo tipo de atualização.
    Esse comportamento mudou desde a versão ``3.0.0``: antes disso, quando a atualização não era utilizada pelo listener, ela era ignorada e não era passada para os handlers.

    Se você precisar impedir que a atualização seja passada para os handlers, chame o método :meth:`~pywa.types.base_update.BaseUpdate.stop_handling` na atualização dentro dos filtros ou canceladores (isso não afetará o comportamento do listener, apenas impedirá que a atualização seja passada para os handlers).


Atalhos
_________

PyWa fornece alguns atalhos para criar listeners ao enviar mensagens. Veja um exemplo:

.. code-block:: python
    :linenos:
    :emphasize-lines: 7

    from pywa import WhatsApp, types, filters

    wa = WhatsApp(...)

    @wa.on_message(filters.command("start"))
    def start(client: WhatsApp, msg: types.Message):
        age: types.Message = m.reply("Hello! How old are you?").wait_for_reply(filters.text)
        m.reply(f"You are {age.text} years old")

No exemplo acima, usamos o método :meth:`~pywa.types.sent_update.SentMessage.wait_for_reply` para criar um listener que aguarda uma resposta de texto do usuário.

.. code-block:: python
    :linenos:
    :emphasize-lines: 7

    from pywa import WhatsApp, types, filters

    wa = WhatsApp(...)

    @wa.on_message(filters.command("start"))
    def start(client: WhatsApp, msg: types.Message):
        msg.reply(f"Hello {msg.from_user.name}!").wait_until_delivered()
        msg.reply("How can I help you?")

No exemplo acima, usamos o método :meth:`~pywa.types.sent_update.SentMessage.wait_until_delivered` para criar um listener que aguarda até que a mensagem seja entregue ao usuário.

Outros atalhos estão disponíveis, como :meth:`~pywa.types.sent_update.SentMessage.wait_for_click`, :meth:`~pywa.types.sent_update.SentMessage.wait_for_selection`, :meth:`~pywa.types.sent_update.SentMessage.wait_until_read`, :meth:`~pywa.types.sent_update.SentVoiceMessage.wait_until_played`, e mais.

.. toctree::

    ./reference
