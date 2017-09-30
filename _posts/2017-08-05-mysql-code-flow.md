mysqld_main -> mysqld_socket_acceptor->connection_event_loop() ->
Connection_acceptor.connection_event_loop() ->
Connection_handler_manager::process_new_connection(Channel_info* channel_info) ->   if (m_connection_handler->add_connection(channel_info)) -> do_command
