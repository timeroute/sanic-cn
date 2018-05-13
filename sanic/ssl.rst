SSL 例子
-----------

可以传入 SSLContext:

.. code:: python

  import ssl
  context = ssl.create_default_context(purpose=ssl.Purpose.CLIENT_AUTH)
  context.load_cert_chain("/path/to/cert", keyfile="/path/to/keyfile")

  app.run(host="0.0.0.0", port=8443, ssl=context)

你也可以用字典传入一个证书和密钥的位置：


.. code:: python

  ssl = {'cert': "/path/to/cert", 'key': "/path/to/keyfile"}
  app.run(host="0.0.0.0", port=8443, ssl=ssl)
