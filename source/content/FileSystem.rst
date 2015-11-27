How to unpack Android system.img
--------------------------------

.. code-block:: sh

   $ simg2img system.img sys.raw
   $ mkdir sys system
   $ mount -t ext4 -o loop sys.raw sys
   $ cp -fr sys/* system
   $ umount sys
   $ rm -fr sys sys.raw
