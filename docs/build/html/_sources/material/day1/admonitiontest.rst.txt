.. attention::
  attention

.. caution::
  caution

.. danger::
  danger

.. error::
  error

.. hint::
  hint

.. important::
  important

.. note::
  note

.. tip::
  tip

.. warning::
  warning


.. code-block:: r

  pop <- data.frame(x=d)

.. code-block:: python
   :emphasize-lines: 3,5

   def some_function():
       interesting = False
       print 'This line is highlighted.'
       print 'This one is not...'
       print '...but this one is.'

.. note::

  Your note should consist of one or more paragraphs, all indented
  so that they clearly belong to the note and not to the text or
  directive that follow 
  Many other directives are also supported, including: warning,
  versionadded, versionchanged, seealso, deprecated, rubric,
  centered, hlist, glossary, productionlist.

.. code-block:: c

   /* Or say "highlight::" once to set the language for all of the
      code blocks that follow it.  Options include ":linenos:",
      ":linenothreshold:", and ":emphasize-lines: 1,2,3". */

   char s[] = "You can also say 'python', 'ruby', ..., or 'guess'!";

.. literalinclude:: example.py
   :lines: 10-20
   :emphasize-lines: 15,16

.. module:: httplib

.. class:: Request

   Zero or more paragraphs of introductory material for the class.

   .. method:: send()

      Description of the send() method.

   .. attribute:: url

      Description of the url attribute.

   Many more members are possible than just method and attribute,
   and non-Python languages are supported too; see the Sphinx docs
   for more possibilities!

.. testcode::

   print 'The doctest extension supports code without >>> prompts!'

.. testoutput::

   The doctest extension supports code without >>> prompts!

.. _custom-label:
.. index:: single: paragraph, targeted paragraph, indexed paragraph

This paragraph can be targeted with :ref:`custom-label`, and will also
be the :index:`target` of several index entries!

.. index:: pair: copper, wire

This paragraph will be listed in the index under both “wire, copper”
and “copper, wire.”  See the Sphinx documentation for even more complex
ways of building index entries.

Many kinds of cross-reference can be used inside of a paragraph:

:ref:`custom-label`             :class:`~module.class`
:doc:`quick-sphinx`             :attr:`~module.class.method()`
:mod:`module`                   :attr: `~module.class.attribute`

(See the Sphinx “Inline Markup” chapter for MANY more examples!)