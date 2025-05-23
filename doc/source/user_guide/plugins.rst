I/O Plugin Infrastructure
-------------------------

.. note::
    The plugin infrastructure of ``skimage.io`` is deprecated since version
    0.25 and will be removed in version 0.27. Please use ``imageio`` or other
    I/O packages directly.

A plugin consists of two files, the source and the descriptor ``.ini``.  Let's
say we'd like to provide a plugin for ``imshow`` using ``matplotlib``.  We'll
call our plugin ``mpl``::

  skimage/io/_plugins/mpl.py
  skimage/io/_plugins/mpl.ini

The name of the ``.py`` and ``.ini`` files must correspond.  Inside the
``.ini`` file, we give the plugin meta-data::

  [mpl] <-- name of the plugin, may be anything
  description = Matplotlib image I/O plugin
  provides = imshow <-- a comma-separated list, one or more of
                        imshow, imsave, imread, _app_show

The "provides"-line lists all the functions provided by the plugin.  Since our
plugin provides ``imshow``, we have to define it inside ``mpl.py``::

  # This is mpl.py

  import matplotlib.pyplot as plt

  def imshow(img):
      plt.imshow(img)

Note that, by default, ``imshow`` is non-blocking, so a special function
``_app_show`` must be provided to block the GUI.  We can modify our plugin to
provide it as follows::

  [mpl]
  provides = imshow, _app_show

::

  # This is mpl.py

  import matplotlib.pyplot as plt

  def imshow(img):
      plt.imshow(img)

  def _app_show():
      plt.show()

Any plugin in the ``_plugins`` directory is automatically examined by
``skimage.io`` upon import.  You may list all the plugins on your
system::

  >>> import skimage as ski
  >>> ski.io.find_available_plugins()
  {'gtk': ['imshow'],
   'matplotlib': ['imshow', 'imread', 'imread_collection'],
   'pil': ['imread', 'imsave', 'imread_collection'],
   'test': ['imsave', 'imshow', 'imread', 'imread_collection'],}

or only those already loaded::

  >>> ski.io.find_available_plugins(loaded=True)
  {'matplotlib': ['imshow', 'imread', 'imread_collection'],
   'pil': ['imread', 'imsave', 'imread_collection']}

A plugin is loaded using the ``use_plugin`` command::

  >>> ski.io.use_plugin('pil') # Use all capabilities provided by PIL

or

::

  >>> ski.io.use_plugin('pil', 'imread') # Use only the imread capability of PIL

Note that, if more than one plugin provides certain functionality, the
last plugin loaded is used.

To query a plugin's capabilities, use ``plugin_info``::

  >>> ski.io.plugin_info('pil')
  >>>
  {'description': 'Image reading via the Python Imaging Library',
   'provides': 'imread, imsave'}
