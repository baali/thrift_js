Overview
--------

Simple setup of cpp based thrift server for JSON protocol and HTTP
transport, and a javascript client to interact with it.

Usage
-----

- We use thrift to generate skeleton files for both server(cpp) and
  client side(js)::

  $ thrift –gen cpp shared.thrift
  $ thrift –gen cpp tutorial.thrift
  $ thrift –gen js shared.thrift
  $ thrift –gen js tutorial.thrift

- Compile all the files using make ::

  $ make
  
- To run server and log all the events ::

  $./CppServer

- To run client just open tutorial.html file in any browser::

Issues:
-------

- Handling "Preflighted requests" from chrome browser is still not
  neat and at times I am getting error message::

  "TThreadedServer client died: Could not refill buffer"
