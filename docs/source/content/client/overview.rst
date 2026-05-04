🔌 Client
=========

.. currentmodule:: pywa.client

O cliente :class:`~WhatsApp` é o coração da biblioteca **pywa**.
Ele é responsável por gerenciar a comunicação com a WhatsApp Business API.

Suas **três principais responsabilidades** são:

1. **Enviar mensagens** — texto, mídia, localização, contatos, etc.
2. **Ouvir** — tratar mensagens recebidas, eventos e status.
3. **Gerenciar recursos** — templates, flows, perfis e outras configurações relacionadas ao negócio.

.. tip::
   :class: note

   O Pywa oferece **dois tipos de clients**:

   - **Síncrono** (`pywa`)
   - **Assíncrono** (`pywa_async`)

   Escolha o que melhor se adapta às necessidades da sua aplicação.

   .. code-block:: python
      :emphasize-lines: 1

      from pywa import WhatsApp, types
      wa = WhatsApp(...)

      @wa.on_message
      def on_message(_: WhatsApp, msg: types.Message):
          msg.reply("Hello!")

   .. code-block:: python
      :emphasize-lines: 1

      from pywa_async import WhatsApp, types
      wa = WhatsApp(...)

      @wa.on_message
      async def on_message(_: WhatsApp, msg: types.Message):
          await msg.reply("Hello!")

   Para uma verificação de tipos otimizada, certifique-se de que **todas** as suas importações venham do mesmo pacote — seja ``pywa`` ou ``pywa_async``.
.. autoclass:: WhatsApp()
   :members: __init__


Enviar Mensagens
----------------

O client permite enviar uma grande variedade de mensagens:

.. list-table::
   :widths: 40 60
   :header-rows: 1

   * - Method
     - Description
   * - :meth:`~WhatsApp.send_message`
     - Enviar uma mensagem de texto
   * - :meth:`~WhatsApp.send_image`
     - Enviar uma imagem
   * - :meth:`~WhatsApp.send_video`
     - Enviar um vídeo
   * - :meth:`~WhatsApp.send_audio`
     - Enviar um arquivo de áudio
   * - :meth:`~WhatsApp.send_voice`
     - Enviar uma mensagem de voz
   * - :meth:`~WhatsApp.send_document`
     - Enviar um documento
   * - :meth:`~WhatsApp.send_location`
     - Compartilhar uma localização
   * - :meth:`~WhatsApp.request_location`
     - Solicitar localização de um usuário
   * - :meth:`~WhatsApp.send_contact`
     - Enviar um ou mais contatos
   * - :meth:`~WhatsApp.send_sticker`
     - Enviar um sticker
   * - :meth:`~WhatsApp.send_template`
     - Enviar uma mensagem de template
   * - :meth:`~WhatsApp.send_catalog`
     - Enviar um catálogo de produtos
   * - :meth:`~WhatsApp.send_product`
     - Enviar um único produto
   * - :meth:`~WhatsApp.send_products`
     - Enviar múltiplos produtos
   * - :meth:`~WhatsApp.send_reaction`
     - Reagir a uma mensagem
   * - :meth:`~WhatsApp.remove_reaction`
     - Remover uma reação
   * - :meth:`~WhatsApp.mark_message_as_read`
     - Marcar uma mensagem como lida
   * - :meth:`~WhatsApp.indicate_typing`
     - Indicar status de digitação ao usuário


Tratar Atualizações
-------------------

Registre tratadores de eventos para ouvir atualizações:

.. list-table::
   :widths: 40 60
   :header-rows: 1

   * - Method
     - Description
   * - :meth:`~WhatsApp.on_message`
     - Tratar mensagens recebidas
   * - :meth:`~WhatsApp.on_callback_button`
     - Tratar cliques em botões de callback
   * - :meth:`~WhatsApp.on_callback_selection`
     - Tratar seleções de lista ou menu
   * - :meth:`~WhatsApp.on_message_status`
     - Rastrear status de entrega, leitura e falha de mensagens
   * - :meth:`~WhatsApp.on_chat_opened`
     - Detectar quando um usuário abre um chat
   * - :meth:`~WhatsApp.on_flow_request`
     - Tratar requisições de flow recebidas
   * - :meth:`~WhatsApp.on_flow_completion`
     - Tratar conclusões de flow
   * - :meth:`~WhatsApp.on_phone_number_change`
     - Tratar mudanças de número de telefone
   * - :meth:`~WhatsApp.on_identity_change`
     - Tratar mudanças de identidade
   * - :meth:`~WhatsApp.on_call_connect`
     - Tratar conexões de chamadas recebidas/efetuadas
   * - :meth:`~WhatsApp.on_call_terminate`
     - Tratar encerramentos de chamada
   * - :meth:`~WhatsApp.on_call_status`
     - Tratar atualizações de status de chamada
   * - :meth:`~WhatsApp.on_call_permission_update`
     - Tratar atualizações de permissão de chamada
   * - :meth:`~WhatsApp.on_user_marketing_preferences`
     - Tratar atualizações de preferências de marketing do usuário
   * - :meth:`~WhatsApp.on_template_status_update`
     - Tratar atualizações de status de template
   * - :meth:`~WhatsApp.on_template_category_update`
     - Tratar mudanças de categoria de template
   * - :meth:`~WhatsApp.on_template_quality_update`
     - Tratar mudanças de qualidade de template
   * - :meth:`~WhatsApp.on_template_components_update`
     - Tratar atualizações de componentes de template
   * - :meth:`~WhatsApp.on_raw_update`
     - Tratar atualizações brutas do WhatsApp
   * - :meth:`~WhatsApp.add_handlers`
     - Adicionar handlers dinamicamente via código
   * - :meth:`~WhatsApp.remove_handlers`
     - Remover handlers via código
   * - :meth:`~WhatsApp.remove_callbacks`
     - Remover handlers por callbacks
   * - :meth:`~WhatsApp.add_flow_request_handler`
     - Adicionar handlers de requisição de flow via código
   * - :meth:`~WhatsApp.load_handlers_modules`
     - Carregar handlers de módulos externos


Listening
---------

Você pode ouvir atualizações de usuários específicos:

.. list-table::
   :widths: 40 60
   :header-rows: 1

   * - Method
     - Description
   * - :meth:`~WhatsApp.listen`
     - Ouvir uma atualização de um usuário específico
   * - :meth:`~WhatsApp.stop_listening`
     - Parar de ouvir


Mídia
-----

Gerencie mídia com facilidade:

.. list-table::
   :widths: 40 60
   :header-rows: 1

   * - Method
     - Description
   * - :meth:`~WhatsApp.upload_media`
     - Fazer upload de mídia para os servidores do WhatsApp
   * - :meth:`~WhatsApp.download_media`
     - Baixar mídia
   * - :meth:`~WhatsApp.stream_media`
     - Transmitir mídia em stream
   * - :meth:`~WhatsApp.get_media_bytes`
     - Obter mídia como bytes
   * - :meth:`~WhatsApp.get_media_url`
     - Obter URL direta de mídia
   * - :meth:`~WhatsApp.delete_media`
     - Excluir mídia dos servidores do WhatsApp


Templates
---------

Crie, atualize e gerencie templates de mensagem:

.. list-table::
   :widths: 40 60
   :header-rows: 1

   * - Method
     - Description
   * - :meth:`~WhatsApp.create_template`
     - Criar um novo template
   * - :meth:`~WhatsApp.upsert_authentication_template`
     - Criar ou atualizar templates de autenticação em lote
   * - :meth:`~WhatsApp.get_templates`
     - Recuperar todos os templates
   * - :meth:`~WhatsApp.get_template`
     - Obter detalhes de um template específico
   * - :meth:`~WhatsApp.update_template`
     - Atualizar um template existente
   * - :meth:`~WhatsApp.delete_template`
     - Excluir um template
   * - :meth:`~WhatsApp.unpause_template`
     - Reativar um template pausado anteriormente
   * - :meth:`~WhatsApp.compare_templates`
     - Comparar dois templates
   * - :meth:`~WhatsApp.migrate_templates`
     - Migrar templates entre WABAs


Flows
-----

Gerencie flows via código:

.. list-table::
   :widths: 40 60
   :header-rows: 1

   * - Method
     - Description
   * - :meth:`~WhatsApp.create_flow`
     - Criar um flow
   * - :meth:`~WhatsApp.update_flow_metadata`
     - Atualizar metadados do flow (nome, categorias, endpoint, etc.)
   * - :meth:`~WhatsApp.update_flow_json`
     - Atualizar a definição JSON do flow
   * - :meth:`~WhatsApp.publish_flow`
     - Publicar um flow
   * - :meth:`~WhatsApp.delete_flow`
     - Excluir um flow
   * - :meth:`~WhatsApp.deprecate_flow`
     - Depreciar um flow
   * - :meth:`~WhatsApp.get_flow`
     - Obter detalhes de um flow
   * - :meth:`~WhatsApp.get_flows`
     - Listar todos os flows
   * - :meth:`~WhatsApp.get_flow_metrics`
     - Obter métricas de desempenho do flow
   * - :meth:`~WhatsApp.get_flow_assets`
     - Obter ativos do flow
   * - :meth:`~WhatsApp.migrate_flows`
     - Migrar flows entre WABAs


Business
--------

Gerencie a conta e o perfil do negócio:

.. list-table::
   :widths: 40 60
   :header-rows: 1

   * - Method
     - Description
   * - :meth:`~WhatsApp.get_business_account`
     - Obter detalhes da conta do negócio
   * - :meth:`~WhatsApp.get_business_profile`
     - Obter perfil do negócio
   * - :meth:`~WhatsApp.get_business_phone_numbers`
     - Obter todos os números de telefone do negócio
   * - :meth:`~WhatsApp.get_business_phone_number`
     - Obter um número de telefone específico do negócio
   * - :meth:`~WhatsApp.update_business_profile`
     - Atualizar detalhes do perfil (nome, descrição, foto, etc.)
   * - :meth:`~WhatsApp.update_display_name`
     - Atualizar o nome de exibição do número de telefone
   * - :meth:`~WhatsApp.update_conversational_automation`
     - Atualizar comandos e ice breakers
   * - :meth:`~WhatsApp.set_business_public_key`
     - Fazer upload da chave pública do negócio
   * - :meth:`~WhatsApp.get_business_phone_number_settings`
     - Obter configurações do número de telefone
   * - :meth:`~WhatsApp.update_business_phone_number_settings`
     - Atualizar configurações do número de telefone
   * - :meth:`~WhatsApp.register_phone_number`
     - Registrar um novo número de telefone
   * - :meth:`~WhatsApp.deregister_phone_number`
     - Cancelar o registro de um número de telefone


Gerenciar Usuários
------------------

.. list-table::
   :widths: 40 60
   :header-rows: 1

   * - Method
     - Description
   * - :meth:`~WhatsApp.block_users`
     - Bloquear usuários
   * - :meth:`~WhatsApp.unblock_users`
     - Desbloquear usuários
   * - :meth:`~WhatsApp.get_blocked_users`
     - Recuperar usuários bloqueados


QR Codes
--------

.. list-table::
   :widths: 40 60
   :header-rows: 1

   * - Method
     - Description
   * - :meth:`~WhatsApp.create_qr_code`
     - Criar um QR code
   * - :meth:`~WhatsApp.get_qr_code`
     - Obter detalhes de um QR code
   * - :meth:`~WhatsApp.get_qr_codes`
     - Listar todos os QR codes
   * - :meth:`~WhatsApp.update_qr_code`
     - Atualizar um QR code
   * - :meth:`~WhatsApp.delete_qr_code`
     - Excluir um QR code


Commerce
--------

.. list-table::
   :widths: 40 60
   :header-rows: 1

   * - Method
     - Description
   * - :meth:`~WhatsApp.get_commerce_settings`
     - Obter configurações de comércio
   * - :meth:`~WhatsApp.update_commerce_settings`
     - Atualizar configurações de comércio


Calls
-----

.. list-table::
   :widths: 40 60
   :header-rows: 1

   * - Method
     - Description
   * - :meth:`~WhatsApp.get_call_permissions`
     - Obter permissões de chamada
   * - :meth:`~WhatsApp.pre_accept_call`
     - Pré-aceitar uma chamada
   * - :meth:`~WhatsApp.accept_call`
     - Aceitar uma chamada
   * - :meth:`~WhatsApp.reject_call`
     - Rejeitar uma chamada
   * - :meth:`~WhatsApp.terminate_call`
     - Encerrar uma chamada


Server
------

Integre com eventos de webhook manualmente:

.. list-table::
   :widths: 40 60
   :header-rows: 1

   * - Method
     - Description
   * - :meth:`~WhatsApp.webhook_update_handler`
     - Tratar atualizações de webhook manualmente
   * - :meth:`~WhatsApp.webhook_challenge_handler`
     - Tratar o desafio de webhook manualmente
   * - :meth:`~WhatsApp.get_flow_request_handler`
     - Recuperar o handler de requisição de flow


Others
------

.. list-table::
   :widths: 40 60
   :header-rows: 1

   * - Method
     - Description
   * - :meth:`~WhatsApp.get_app_access_token`
     - Recuperar token de acesso do aplicativo
   * - :meth:`~WhatsApp.set_app_callback_url`
     - Definir a URL de callback do aplicativo
   * - :meth:`~WhatsApp.override_waba_callback_url`
     - Sobrescrever a URL de callback da WABA
   * - :meth:`~WhatsApp.delete_waba_callback_url`
     - Excluir a URL de callback da WABA
   * - :meth:`~WhatsApp.override_phone_callback_url`
     - Sobrescrever a URL de callback do telefone
   * - :meth:`~WhatsApp.delete_phone_callback_url`
     - Excluir a URL de callback do telefone


.. toctree::
   client_reference
   api_reference
