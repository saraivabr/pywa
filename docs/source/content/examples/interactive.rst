Enviando Mensagens Interativas
===============================


Enviar uma mensagem com teclado de seleção
------------------------------------------

- Você pode usar o teclado de seleção apenas com mensagens de texto.
- O número máximo de linhas de seção é 10.

.. sidebar:: 📱 Capturas de Tela

    .. tip:: Como a mensagem é exibida no WhatsApp:

        .. image:: ../../../../_static/examples/selection-message.webp
            :alt: Selection message example
            :width: 90%

    .. tip:: Como o teclado é exibido no WhatsApp:

        .. image:: ../../../../_static/examples/selection-keyboard.webp
            :alt: Selection keyboard example
            :width: 90%

.. code-block:: python
    :caption: Color selection message
    :linenos:

    from pywa import WhatsApp
    from pywa.types import SectionList, Section, SectionRow

    wa = WhatsApp(phone_id='972123456789', token='xxxxx')
    recipient = '972987654321'

    wa.send_message(
        to=recipient,
        header='Select your favorite color',
        text='Tap a button to select your favorite color:',
        footer='⚡ Powered by PyWa',
        buttons=SectionList(
            button_title='Colors',
            sections=[
                Section(
                    title='Popular Colors',
                    rows=[
                        SectionRow(
                            title='🟥 Red',
                            callback_data='color:red',
                            description='The color of blood',
                        ),
                        SectionRow(
                            title='🟩 Green',
                            callback_data='color:green',
                            description='The color of grass',
                        ),
                        SectionRow(
                            title='🟦 Blue',
                            callback_data='color:blue',
                            description='The color of the sky',
                        )
                    ],
                ),
                Section(
                    title='Other Colors',
                    rows=[
                        SectionRow(
                            title='🟧 Orange',
                            callback_data='color:orange',
                            description='The color of an orange',
                        ),
                        SectionRow(
                            title='🟪 Purple',
                            callback_data='color:purple',
                            description='The color of a grape',
                        ),
                        SectionRow(
                            title='🟨 Yellow',
                            callback_data='color:yellow',
                            description='The color of the sun',
                        )
                    ]
                )
            ]
        )
    )



Enviar uma mensagem com teclado de botões
-----------------------------------------

- Você pode adicionar até 3 botões a uma mensagem.

.. sidebar:: 📱 Capturas de Tela

    .. tip:: Como a mensagem é exibida no WhatsApp:

        .. image:: ../../../../_static/examples/buttons-message.webp
            :alt: Buttons message example
            :width: 90%


.. code-block:: python
    :caption: YouTube video info message
    :linenos:

    from pywa import WhatsApp
    from pywa.types import Button

    wa = WhatsApp(phone_id='972123456789', token='xxxxx')

    recipient = '972987654321'
    requested_vid_id = 'T9RRe4ZsSGw'

    wa.send_image(
        to=recipient,
        image=f'https://i.ytimg.com/vi/{requested_vid_id}/hqdefault.jpg',
        caption='Chandler Jokes | Friends • 2.9M views • 1 year ago',
        footer='⚡ Powered by PyWa',
        buttons=[
            Button(title='⬇️ Download', callback_data=f'dl:{requested_vid_id}'),
            Button(title='💬 Comments', callback_data=f'cmnts:{requested_vid_id}'),
            Button(title='🎬 Info', callback_data=f'info:{requested_vid_id}'),
        ]
    )
