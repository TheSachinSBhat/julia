.. _man-documentation:

***************
 Documentation
***************

Julia enables package developers and users to document functions, types and
other objects easily via a built-in documentation system since Julia 0.4.

.. tip::

    This documentation system can also be used in Julia 0.3 via the
    `Docile.jl <https://github.com/MichaelHatherly/Docile.jl>`_ package; see
    the documentation for that package for more details.

The basic syntax is very simple: any string appearing at the top-level right
before an object (function, macro, type or instance) will be interpreted as
documenting it (these are called *docstrings*). Here is a very simple example:

.. doctest::

    "Tell whether there are too foo items in the array."
    foo(xs::Array) = ...

Documentation is interpreted as `Markdown <https://en.wikipedia.org/wiki/Markdown>`_,
so you can use indentation and code fences to delimit code examples from text.
Technically, any object can be associated with any other as metadata;
Markdown happens to be the default, but one can construct other string
macros and pass them to the ``@doc`` macro just as well.

Here is a more complex example, still using Markdown:

.. code-block:: julia

    """
        bar(x[, y])

    Compute the Bar index between `x` and `y`. If `y` is missing, compute
    the Bar index between all pairs of columns of `x`.

    # Examples
    ```julia
    julia> bar([1, 2], [1, 2])
    1
    ```
    """
    function bar(x, y) ...

As in the example above, we recommend following some simple conventions when writing
documentation:

1. Always show the signature of a function at the top of the documentation,
   with a four-space indent so that it is printed as Julia code.

   This can be identical to the signature present in the Julia code
   (like ``mean(x::AbstractArray)``), or a simplified form.
   Optional arguments should be represented with their default values (i.e. ``f(x, y=1)``)
   when possible, following the actual Julia syntax. Optional arguments which
   do not have a default value should be put in brackets (i.e. ``f(x[, y])`` and
   ``f(x[, y[, z]])``). An alternative solution is to use several lines: one without
   optional arguments, the other(s) with them. This solution can also be used to document
   several related methods of a given function. When a function accepts many keyword
   arguments, only include a ``<keyword arguments>`` placeholder in the signature (i.e.
   ``f(x; <keyword arguments>)``), and give the complete list under an ``# Arguments``
   section (see point 4 below).

2. Include a single one-line sentence describing what the function does or what the
   object represents after the simplified signature block. If needed, provide more details
   in a second paragraph, after a blank line.

   The one-line sentence should use the imperative form ("Do this", "Return that") instead
   of the third person (do not write "Returns the length...") when documenting functions.
   It should end with a period. If the meaning of a function cannot be summarized easily,
   splitting it into separate composable parts could be beneficial (this should not be
   taken as an absolute requirement for every single case though).

3. Do not repeat yourself.

   Since the function name is given by the signature, there is no need to
   start the documentation with "The function ``bar``...": go straight to the point.
   Similarly, if the signature specifies the types of the arguments, mentioning them
   in the description is redundant.

4. Only provide an argument list when really necessary.

   For simple functions, it is often clearer to mention the role of the arguments directly
   in the description of the function's purpose. An argument list would only repeat
   information already provided elsewhere. However, providing an argument list can be a good
   idea for complex functions with many arguments (in particular keyword arguments).
   In that case, insert it after the general description of the function, under
   an ``# Arguments`` header, with one ``*`` bullet for each argument. The list should
   mention the types and default values (if any) of the arguments::

       """
       ...
       # Arguments
       * `n::Integer`: the number of elements to compute.
       * `dim::Integer=1`: the dimensions along which to perform the computation.
       ...
       """

5. Group examples under an ``# Examples`` section and use `````julia`` blocks instead of
   standard text.

   Examples should consist of verbatim copies of the Julia REPL, including the ``julia>``
   prompt (see example above). This is used to allow running examples automatically and
   checking that their actual output is consistent with that presented in the
   documentation (a feature called *doctests*). This way, the code will be tested and
   examples won't get out of date without notice. An example::

       """
       Some nice documentation here.

       ```jldoctest
       julia> a = [1 2; 3 4]
       2×2 Array{Int64,2}:
        1  2
        3  4
       ```
       """

   You can then run ``make -C doc doctest`` to run all the doctests, which will ensure
   that your example works. Note that whitespace in your doctest is significant! The
   doctest will fail if you misalign the output of pretty-printing an array, for example.

6. Use backticks to identify code and equations.

   Julia identifiers and code excerpts should always appear between backticks `````
   to enable highlighting. Equations in the LaTeX syntax can be inserted between
   double backticks ``````. Use Unicode characters rather than their LaTeX escape sequence,
   i.e. ````α = 1```` rather than :samp:`\`\`\\\\alpha = 1\`\``.

7. Place the starting and ending ``"""`` characters on lines by themselves.

   That is, write::

       """
       ...

       ...
       """
       f(x, y) = ...

   rather than::

       """...

       ..."""
       f(x, y) = ...

   This makes it more clear where docstrings start and end.

8. Respect the line length limit used in the surrounding code.

   Docstrings are edited using the same tools as code. Therefore, the same conventions
   should apply. It it advised to add line breaks after 92 characters.

Accessing Documentation
-----------------------

Documentation can be accessed at the REPL or in IJulia by typing ``?``
followed by the name of a function or macro, and pressing ``Enter``. For
example,

.. doctest::

    ?fft
    ?@time
    ?r""

will bring up docs for the relevant function, macro or string macro
respectively. In `Juno <http://junolab.org>`_ using ``Ctrl-D`` will
bring up documentation for the object under the cursor.

Functions & Methods
-------------------

Functions in Julia may have multiple implementations, known as methods.
While it's good practice for generic functions to have a single purpose,
Julia allows methods to be documented individually if necessary.
In general, only the most generic method should be documented, or even the
function itself (i.e. the object created without any methods by
``function bar end``). Specific methods should only be documented if their
behaviour differs from the more generic ones. In any case, they should not
repeat the information provided elsewhere. For example:

.. doctest::

    """
    Multiplication operator. `x*y*z*...` calls this function with multiple
    arguments, i.e. `*(x,y,z...)`.
    """
    function *(x, y)
      # ... [implementation sold separately] ...
    end

    "When applied to strings, concatenates them."
    function *(x::AbstractString, y::AbstractString)
      # ... [insert secret sauce here] ...
    end

    help?>*
    Multiplication operator. `x*y*z*...` calls this function with multiple
    arguments, i.e. `*(x,y,z...)`.

    When applied to strings, concatenates them.

When retrieving documentation for a generic function, the metadata for
each method is concatenated with the ``catdoc`` function, which can of
course be overridden for custom types.

Advanced Usage
--------------

The ``@doc`` macro associates its first argument with its second in a
per-module dictionary called ``META``. By default, documentation is
expected to be written in Markdown, and the ``doc""`` string macro simply
creates an object representing the Markdown content. In the future it is
likely to do more advanced things such as allowing for relative image or
link paths.

When used for retrieving documentation, the ``@doc`` macro (or equally,
the ``doc`` function) will search all ``META`` dictionaries for metadata
relevant to the given object and return it. The returned object (some
Markdown content, for example) will by default display itself
intelligently. This design also makes it easy to use the doc system in a
programmatic way; for example, to re-use documentation between different
versions of a function:

.. doctest::

    @doc "..." foo!
    @doc (@doc foo!) foo

Or for use with Julia's metaprogramming functionality:

.. doctest::

    for (f, op) in ((:add, :+), (:subtract, :-), (:multiply, :*), (:divide, :/))
        @eval begin
            $f(a,b) = $op(a,b)
        end
    end
    @doc "`add(a,b)` adds `a` and `b` together" add
    @doc "`subtract(a,b)` subtracts `b` from `a`" subtract

Documentation written in non-toplevel blocks, such as ``if``, ``for``, and ``let``, are not
automatically added to the documentation system. ``@doc`` must be used in these cases. For
example:

.. code-block:: julia

    if VERSION > v"0.4"
        "..."
        f(x) = x
    end

will not add any documentation to ``f`` even when the condition is ``true`` and must instead
be written as:

.. code-block:: julia

    if VERSION > v"0.4"
        @doc "..." ->
        f(x) = x
    end

Syntax Guide
------------

A comprehensive overview of all documentable Julia syntax.

In the following examples ``"..."`` is used to illustrate an arbitrary docstring which may
be one of the follow four variants and contain arbitrary text:

.. code-block:: julia

    "..."

    doc"..."

    """
    ...
    """

    doc"""
    ...
    """

``@doc_str`` should only be used when the docstring contains ``$`` or ``\`` characters that
should not be parsed by Julia such as LaTeX syntax or Julia source code examples containing
interpolation.

Functions and Methods
~~~~~~~~~~~~~~~~~~~~~

.. code-block:: julia

    "..."
    function f end

    "..."
    f

Adds docstring ``"..."`` to ``Function`` ``f``. The first version is the preferred syntax,
however both are equivalent.

.. code-block:: julia

    "..."
    f(x) = x

    "..."
    function f(x)
        x
    end

    "..."
    f(x)

Adds docstring ``"..."`` to ``Method`` ``f(::Any)``.

.. code-block:: julia

    "..."
    f(x, y = 1) = x + y

Adds docstring ``"..."`` to two ``Method``\ s, namely ``f(::Any)`` and ``f(::Any, ::Any)``.

Macros
~~~~~~

.. code-block:: julia

    "..."
    macro m(x) end

Adds docstring ``"..."`` to the ``@m(::Any)`` macro definition.

.. code-block:: julia

    "..."
    :(@m)

Adds docstring ``"..."`` to the macro named ``@m``.

Types
~~~~~

.. code-block:: julia

    "..."
    abstract T1

    "..."
    type T2
        ...
    end

    "..."
    immutable T3
        ...
    end

Adds the docstring ``"..."`` to types ``T1``, ``T2``, and ``T3``.

.. code-block:: julia

    "..."
    type T
        "x"
        x
        "y"
        y
    end

Adds docstring ``"..."`` to type ``T``, ``"x"`` to field ``T.x`` and ``"y"`` to field
``T.y``. Also applicable to ``immutable`` types.

.. code-block:: julia

    "..."
    typealias A T

Adds docstring ``"..."`` to the ``Binding`` ``A``.

``Binding``\ s are used to store a reference to a particular ``Symbol`` in a ``Module``
without storing the referenced value itself.

Modules
~~~~~~~

.. code-block:: julia

    "..."
    module M end

    module M

    "..."
    M

    end

Adds docstring ``"..."`` to the ``Module`` ``M``. Adding the docstring above the ``Module``
is the preferred syntax, however both are equivalent.

.. code-block:: julia

    "..."
    baremodule M
    # ...
    end

    baremodule M

    import Base: @doc

    "..."
    f(x) = x

    end

Documenting a ``baremodule`` by placing a docstring above the expression automatically
imports ``@doc`` into the module. These imports must be done manually when the
module expression is not documented. Empty ``baremodule``\ s cannot be documented.

Global Variables
~~~~~~~~~~~~~~~~

.. code-block:: julia

    "..."
    const a = 1

    "..."
    b = 2

    "..."
    global c = 3

Adds docstring ``"..."`` to the ``Binding``\ s ``a``, ``b``, and ``c``.

.. code-block:: julia

    "..."
    sym

Adds docstring ``"..."`` to the value associated with ``sym``. Users should prefer
documenting ``sym`` at it's definition.

Multiple Objects
~~~~~~~~~~~~~~~~

.. code-block:: julia

    "..."
    a, b

Adds docstring ``"..."`` to ``a`` and ``b`` each of which should be a documentable
expression. This syntax is equivalent to

.. code-block:: julia

    "..."
    a

    "..."
    b

Any number of expressions many be documented together in this way. This syntax can be useful
when two functions are related, such as non-mutating and mutating versions ``f`` and ``f!``.

Macro-generated code
~~~~~~~~~~~~~~~~~~~~

.. code-block:: julia

    "..."
    @m expression

Adds docstring ``"..."`` to expression generated by expanding ``@m expression``. This allows
for expressions decorated with ``@inline``, ``@noinline``, ``@generated``, or any other
macro to be documented in the same way as undecorated expressions.

Macro authors should take note that only macros that generate a single expression will
automatically support docstrings. If a macro returns a block containing multiple
subexpressions then the subexpression that should be documented must be marked using the
:func:`@__doc__` macro.

The ``@enum`` macro makes use of ``@__doc__`` to allow for documenting ``Enum``\ s.
Examining it's definition should serve as an example of how to use ``@__doc__`` correctly.

.. function:: @__doc__(ex)

   .. Docstring generated from Julia source

   Low-level macro used to mark expressions returned by a macro that should be documented. If more than one expression is marked then the same docstring is applied to each expression.

   .. code-block:: julia

       macro example(f)
           quote
               $(f)() = 0
               @__doc__ $(f)(x) = 1
               $(f)(x, y) = 2
           end |> esc
       end

   ``@__doc__`` has no effect when a macro that uses it is not documented.

Markdown syntax
---------------

The following markdown syntax is supported in Julia.

Inline elements
~~~~~~~~~~~~~~~

Here "inline" refers to elements that can be found within blocks of text, i.e. paragraphs. These include the following elements.

Bold
^^^^

Surround words with two asterisks, ``**``, to display the enclosed text in boldface.

.. code-block:: text

   A paragraph containing a **bold** word.

Italics
^^^^^^^

Surround words with one asterisk, ``*``, to display the enclosed text in italics.

.. code-block:: text

   A paragraph containing an *emphasised* word.

Literals
^^^^^^^^

Surround text that should be displayed exactly as written with single backticks, ````` .

.. code-block:: text

   A paragraph containing a `literal` word.

Literals should be used when writing text that refers to names of variables, functions, or other parts of a Julia program.

.. tip::

    To include a backtick character within literal text use three backticks rather than one to enclose the text.

    .. code-block:: text

       A paragraph containing a ``` `backtick` character ```.

    By extension any odd number of backticks may be used to enclose a lesser number of backticks.

:math:`\LaTeX`
^^^^^^^^^^^^^^

Surround text that should be displayed as mathematics using :math:`\LaTeX` syntax with double backticks, `````` .

.. code-block:: text

   A paragraph containing some ``\LaTeX`` markup.

.. tip::

    As with literals in the previous section, if literal backticks need to be written within double backticks use an even number greater than two. Note that if a single literal backtick needs to be included within :math:`\LaTeX` markup then two enclosing backticks is sufficient.

Links
^^^^^

Links to either external or internal addresses can be written using the following syntax, where the text enclosed in square brackets, ``[ ]``, is the name of the link and the text enclosed in parentheses, ``( )``, is the URL.

.. code-block:: text

   A paragraph containing a link to [Julia](http://www.julialang.org).

Footnote references
^^^^^^^^^^^^^^^^^^^

Named and numbered footnote references can be written using the following syntax. A footnote name must be a single alphanumeric word containing no punctuation.

.. code-block:: text

   A paragraph containing a numbered footnote [^1] and a named one [^named].

.. note::

    The text associated with a footnote can be written anywhere within the same page as the footnote reference. The syntax used to define the footnote text is discussed in the `Footnotes`_ section below.

Toplevel elements
~~~~~~~~~~~~~~~~~

The following elements can be written either at the "toplevel" of a document or within another "toplevel" element.

Paragraphs
^^^^^^^^^^

A paragraph is a block of plain text, possibly containing any number of inline elements defined in the `Inline elements`_ section above, with one or more blank lines above and below it.

.. code-block:: text

   This is a paragraph.

   And this is *another* one containing some emphasised text.
   A new line, but still part of the same paragraph.

Headers
^^^^^^^

A document can be split up into different sections using headers. Headers use the following syntax:

.. code-block:: text

   # Level One
   ## Level Two
   ### Level Three
   #### Level Four
   ##### Level Five
   ###### Level Six

A header line can contain any inline syntax in the same way as a paragraph can.

.. tip::

    Try to avoid using too many levels of header within a single document. A heavily nested document may be indicative of a need to restructure it or split it into several pages covering separate topics.

Code blocks
^^^^^^^^^^^

Source code can be displayed as a literal block using an indent of four spaces as shown in the following example.

.. code-block:: text

   This is a paragraph.

       function func(x)
           # ...
       end

   Another paragraph.

Additionally, code blocks can be enclosed using triple backticks with an optional "language" to specify how a block of code should be highlighted.

.. code-block:: text

   A code block without a "language":

   ```
   function func(x)
       # ...
   end
   ```

   and another one with the "language" specified as `julia`:

   ```julia
   function func(x)
       # ...
   end
   ```

.. note::

    "Fenced" code blocks, as shown in the last example, should be prefered over indented code blocks since there is no way to specify what language an indented code block is written in.

Block quotes
^^^^^^^^^^^^

Text from external sources, such as quotations from books or websites, can be quoted using ``>`` characters prepended to each line of the quote as follows.

.. code-block:: text

   Here's a quote:

   > Julia is a high-level, high-performance dynamic programming language for
   > technical computing, with syntax that is familiar to users of other
   > technical computing environments.

Note that a single space must appear after the ``>`` character on each line. Quoted blocks may themselves contain other toplevel or inline elements.

Images
^^^^^^

The syntax for images is similar to the link syntax mentioned above. Prepending a ``!`` character to a link will display an image from the specified URL rather than a link to it.

.. code-block:: text

   ![alternative text](link/to/image.png)

Lists
^^^^^

Unordered lists can be written by prepending each item in a list with either ``*``, ``+``, or ``-``.

.. code-block:: text

   A list of items:

     * item one
     * item two
     * item three

Note the two spaces before each ``*`` and the single space after each one.

Lists can contain other nested toplevel elements such as lists, code blocks, or quoteblocks. A blank line should be left between each list item when including any toplevel elements within a list.

.. code-block:: text

   Another list:

     * item one

     * item two

       ```
       f(x) = x
       ```

     * And a sublist:

         + sub-item one
         + sub-item two

.. note::

    The contents of each item in the list must line up with the first line of the item. In the above example the fenced code block must be indented by four spaces to align with the ``i`` in ``item two``.

Ordered lists are written by replacing the "bullet" character, either ``*``, ``+``, or ``-``, with a positive integer followed by either ``.`` or ``)``.

.. code-block:: text

   Two ordered lists:

    1. item one
    2. item two
    3. item three


    5) item five
    6) item six
    7) item seven

An ordered list may start from a number other than one, as in the second list of the above example, where it is numbered from five. As with unordered lists, ordered lists can contain nested toplevel elements.

Display equations
^^^^^^^^^^^^^^^^^

Large :math:`\LaTeX` equations that do not fit inline within a paragraph may be written as display equations using a fenced code block with the "language" ``math`` as in the example below.

.. code-block:: text

   ```math
   f(a) = \frac{1}{2\pi}\int_{0}^{2\pi} (\alpha+R\cos(\theta))d\theta
   ```

Footnotes
^^^^^^^^^

This syntax is paired with the inline syntax for `Footnote references`_. Make sure to read that section as well.

Footnote text is defined using the following syntax, which is similar to footnote reference syntax, aside from the ``:`` character that is appended to the footnote label.

.. code-block:: text

   [^1]: Numbered footnote text.

   [^note]:

       Named footnote text containing several toplevel elements.

         * item one
         * item two
         * item three

       ```julia
       function func(x)
           # ...
       end
       ```

.. note::

    No checks are done during parsing to make sure that all footnote references have matching footnotes.

Horizontal rules
^^^^^^^^^^^^^^^^

The equivalent of an ``<hr>`` HTML tag can be written using the following syntax:

.. code-block:: text

   Text above the line.

   ---

   And text below the line.

Tables
^^^^^^

Basic tables can be written using the syntax described below. Note that markdown tables have limited features and cannot contain nested toplevel elements unlike other elements discussed above -- only inline elements are allowed. Tables must always contain a header row with column names. Cells cannot span multiple rows or columns of the table.

.. code-block:: text

   | Column One | Column Two | Column Three |
   |:---------- | ---------- |:------------:|
   | Row `1`    | Column `2` |              |
   | *Row* 2    | **Row** 2  | Column ``3`` |

.. note::

    As illustrated in the above example each column of ``|`` characters must be aligned vertically.

    A ``:`` character on either end of a column's header separator (the row containing ``-`` characters) specifies whether the row is left-aligned, right-aligned, or (when ``:`` appears on both ends) center-aligned. Providing no ``:`` characters will default to right-aligning the column.

Admonitions
^^^^^^^^^^^

Specially formatted blocks with titles such as "Notes", "Warning", or "Tips" are known as admonitions and are used when some part of a document needs special attention. They can be defined using the following ``!!!`` syntax:

.. code-block:: text

   !!! note

       This is the content of the note.

   !!! warning "Beware!"

       And this is another one.

       This warning admonition has a custom title: `"Beware!"`.

Admonitions, like most other toplevel elements, can contain other toplevel elements. When no title text, specified after the admonition type in double quotes, is included then the title used will be the type of the block, i.e. ``"Note"`` in the case of the ``note`` admonition.


Markdown Syntax Extensions
--------------------------

Julia's markdown supports interpolation in a very similar way to basic string
literals, with the difference that it will store the object itself in
the Markdown tree (as opposed to converting it to a string). When the
Markdown content is rendered the usual ``show`` methods will be
called, and these can be overridden as usual. This design allows the
Markdown to be extended with arbitrarily complex features (such as
references) without cluttering the basic syntax.

In principle, the Markdown parser itself can also be arbitrarily
extended by packages, or an entirely custom flavour of Markdown can be
used, but this should generally be unnecessary.

