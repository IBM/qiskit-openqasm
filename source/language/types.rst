.. role:: raw-latex(raw)
   :format: latex
..

Types and Casting
=================

Generalities
------------

Variable identifiers must begin with a letter [A-Za-z], an underscore or an element from the Unicode character categories
Lu/Ll/Lt/Lm/Lo/Nl :cite:`noauthorUnicodeNodate`. The set of permissible
continuation characters consists of all members of the aforementioned character
sets with the addition of decimal numerals [0-9]. Variable identifiers may not
override a reserved identifier.

In addition to being assigned values within a program, all of the classical
types can be initialized on declaration. Any classical variable or Boolean that is not explicitly
initialized is undefined. Classical types can be mutually cast to one another using the typename.

Declaration and initialization must be done one variable at a time for both quantum and classical
types. Comma seperated declaration/initialization (``int x, y, z``) is NOT allowed for any type. For
example, to declare a set of qubits one must do

.. code-block:: c

   qubit q0;
   qubit q1;
   qubit q2;

and to declare a set of classical variables

.. code-block:: c

   int[32] x;
   float[32] y = 5.5;
   bit[3] c;
   bool my_bool = false;

We use the notation ``s:m:f`` to denote the width and precision of fixed point numbers,
where ``s`` is the number of sign bits, ``m`` is the number of integer bits, and ``f`` is the
number of fractional bits. It is necessary to specify low-level
classical representations since OpenQASM operates at the intersection of
gates/analog control and digital feedback and we need to be able to
explicitly transform types to cross these boundaries. Classical types
are scoped to the braces within which they are declared.

Quantum types
-------------

Qubits
~~~~~~

There is a quantum bit (``qubit``) type that is interpreted as a reference to a
two-level subsystem of a quantum state. The statement ``qubit name;``
declares a reference to a quantum bit. These qubits are referred 
to as "virtual qubits" (in distinction to "physical qubits" on 
actual hardware; see below). The statement ``qubit[size] name;`` 
declares a quantum register with ``size`` qubits.
Sizes must always be constant positive integers. The label ``name[j]`` 
refers to a qubit of this register, where
:math:`j\in \{0,1,\dots,\mathrm{size}(\mathrm{name})-1\}` is an integer.
Quantum registers are static arrays of qubits 
that cannot be dynamically resized. 

The keyword ``qreg`` is included
for backwards compatibility and will be removed in the future. 

Qubits are initially in an undefined state. A quantum ``reset`` operation is one
way to initialize qubit states.

All qubits are global variables.
Qubits cannot be declared within gates or subroutines. This simplifies OpenQASM
significantly since there is no need for quantum memory management.
However, it also means that users or compiler have to explicitly manage
the quantum memory.

Physical Qubits
~~~~~~~~~~~~~~~

While program qubits can be named, hardware qubits are referenced only
by the syntax ``$[NUM]``. For an ``n`` qubit system, we have physical qubit
references given by ``$0``, ``$1``, ..., ``$n-1``. These qubit types are
used in lower parts of the compilation stack when emitting physical
circuits. Physical qubits must not be declared and they are, as all the qubits, global variables.

.. code-block:: c
   :force:

   // Declare a qubit
   qubit gamma;
   // Declare a qubit with a Unicode name
   qubit γ;
   // Declare a qubit register with 20 qubits
   qubit[20] qubit_array;
   // CNOT gate between physical qubits 0 and 1
   CX $0, $1;

Classical types
---------------

Classical bits and registers
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

There is a classical bit type that takes values 0 or 1. Classical
registers are static arrays of bits. The classical registers model part
of the controller state that is exposed within the OpenQASM program. The
statement ``bit name;`` declares a classical bit, and or ``bit[size] name;`` declares a register of
``size`` bits. The label ``name[j]`` refers to a bit of this register, where :math:`j\in
\{0,1,\dots,\mathrm{size}(\mathrm{name})-1\}` is an integer.

Bit registers may also be declared as ``creg name[size]``. This is included for backwards
compatibility and may be removed in the future.

For convenience, classical registers can be assigned a text string
containing zeros and ones of the same length as the size of the
register. It is interpreted to assign each bit of the register to
corresponding value 0 or 1 in the string, where the least-significant
bit is on the right.

.. code-block:: c

   // Declare a register of 20 bits
   bit[20] bit_array;
   // Declare and assign a rgister of bits with decimal value of 15
   bit[8] name = "00001111";

Integers
~~~~~~~~

There are n-bit signed and unsigned integers. The statements ``int[size] name;`` and ``uint[size] name;`` declare
signed 1:n-1:0 and unsigned 0:n:0 integers of the given size. The sizes
are always explicitly part of the type; there is no implicit width for
classical types in OpenQASM. Because register indices are integers, they
can be cast from classical registers containing measurement outcomes and
may only be known at run time. An n-bit classical register containing
bits can also be reinterpreted as an integer, and these types can be
mutually cast to one another using the type name, e.g. ``int[16](c)``. As noted, this
conversion will be done assuming little-endian bit ordering. The example
below demonstrates how to declare, assign and cast integer types amongst
one another.

.. code-block:: c

   // Declare a 32-bit unsigned integer
   uint[32] my_uint = 10;
   // Declare a 16 bit signed integer
   int[16] my_int;
   my_int = int[16](my_uint);

Signed fixed-point numbers
~~~~~~~~~~~~~~~~~~~~~~~~~~

There are ``1:m:f`` fixed-point numbers with ``m`` integer bits, ``f`` fractional bits, and 1
sign bit. The statement ``fixed[m, f] name;`` declares a ``1:m:f`` fixed-point number.

.. code-block:: c

   // Declare a 32-bit fixed point number.
   // The number is signed, has 7 integer bits
   // and 24 fractional bits. The decimal number
   // assignment is implicitly cast.
   fixed[7, 24] my_fixed = -7.0625;

Floating point numbers
~~~~~~~~~~~~~~~~~~~~~~

IEEE 754 floating point registers may be declared with ``float[size] name;``, where ``float[64]`` would
indicate a standard double-precision float. Note that some hardware
vendors may not support manipulating these values at run-time.

.. code-block:: c
   :force:

   // Declare a single-precision 32-bit float
   float[32] my_float = π;

Fixed-point angles
~~~~~~~~~~~~~~~~~~

Fixed-point angles are interpreted as 2π times a 0:0:n
fixed-point number. This represents angles in the interval
:math:`[0,2\pi)` up to an error :math:`\epsilon\leq \pi/2^{n-1}` modulo
2π. The statement ``angle[size] name;`` declares an n-bit angle. OpenQASM3
introduces this specialized type because of the ubiquity of this angle
representation in phase estimation circuits and numerically controlled
oscillators found in hardware platform. Note that defining gate
parameters with ``angle`` types may be necessary for those parameters to be
compatible with run-time values on some platforms.

.. code-block:: c
   :force:

   // Declare an angle with 20 bits of precision and assign it a value of π/2
   angle[20] my_angle = π / 2;
   float[32] float_pi = π;
   // equivalent to pi_by_2 up to rounding errors
   angle[20](float_pi / 2);

Boolean types
~~~~~~~~~~~~~

There is a Boolean type ``bool name;`` that takes values ``true`` or ``false``. Qubit measurement results
can be converted from a classical ``bit`` type to a Boolean using ``bool(c)``, where 1 will
be true and 0 will be false.

.. code-block:: c

   bit my_bit = 0;
   bool my_bool;
   // Assign a cast bit to a boolean
   my_bool = bool(my_bit);

Real constants
~~~~~~~~~~~~~~

To support mathematical expressions, there are immutable real constants
that are represented as double precision floating point numbers. On
declaration, they take their assigned value and cannot be redefined
within the same scope. These are constructed using an in-fix notation
and scientific calculator features such as scientific notation, real
arithmetic, logarithmic, trigonometric, and exponential functions
including ``sqrt``, ``floor``, ``ceiling``, ``log``, ``pow``, ``div``, ``mod`` and the built-in constant π. The
statement ``const name = expression;`` defines a new constant. The expression on the right hand side
has a similar syntax as OpenQASM 2 parameter expressions; however,
previously defined constants can be referenced in later variable
declarations. Real constants are compile-time constants, allowing the
compiler to do constant folding and other such optimizations. Scientific
calculator-like operations on run-time values require extern function
calls as described later and are not available by default. Real
constants can be cast to other types. Casting attempts to preserve the
semantics, but information can be lost, since variables have fixed
precision. Unlike casting from other types, implicit casts from real
constants are permitted.

A standard set of built-in constants which are included in the default
namespace are listed in table `1 <#tab:real-constants>`__.

.. code-block:: c
   :force:

   // Declare a constant
   const my_const = 1234;
   // Scientific notation is supported
   const another_const = 1e2;
   // Constant expressions are supported
   const pi_by_2 = π / 2;
   // Constants may be cast to real-time values
   float[32] pi_by_2_val = float(pi_by_2)

.. container::
   :name: tab:real-constants

   .. table:: [tab:real-constants] Built-in real constants in OpenQASM3.

      +-------------------------------+--------------+--------------+---------------------+
      | Constant                      | Alphanumeric | Unicode      | Approximate Base 10 |
      +-------------------------------+--------------+--------------+---------------------+
      | Pi                            | pi           | π            | 3.1415926535...     |
      +-------------------------------+--------------+--------------+---------------------+
      | Tau                           | tau          | τ            | 6.283185...         |
      +-------------------------------+--------------+--------------+---------------------+
      | Euler’s number                | euler        | ℇ            | 2.7182818284...     |
      +-------------------------------+--------------+--------------+---------------------+

Note that `e` is a valid identifier. `e/E` are also used in scientifier notation where appropriate.
Types related to timing
-----------------------

Duration
~~~~~~~~

We introduce a ``duration`` type to express timing.
Durations are positive numbers with a unit of time. ``ns, μs, ms, s`` are used for SI time
units. ``dt`` is a backend-dependent unit equivalent to one waveform sample on
the backend. ``durationof()`` is an intrinsic function used to reference the duration of a calibrated gate.

.. code-block:: c

   duration one_second = 1000ms;
   duration thousand_cycles = 1000dt;
   duration c = durationof({x $3);

``duration`` is further discussed in :any:`duration-and-stretch`

Stretch
~~~~~~~

We further introduce a ``stretch`` type which is a sub-type of ``duration``. ``stretch`` types
have variable non-negative duration that is permitted to grow as necessary
to satisfy constraints. Stretch variables are resolved at compile time
into target-appropriate durations that satisfy a user’s specified design
intent.

``stretch`` is further discussed in :any:`duration-and-stretch`

Aliasing
--------

The ``let`` keyword allows quantum bits and registers to be referred to by
another name as long as the alias is in scope.

.. code-block:: c

  qubit[5] q;
  // myreg[0] refers to the qubit q[1]
  let myreg = q[1:4];


Register concatenation and slicing
----------------------------------

Two or more registers of the same type (i.e. classical or quantum) can
be concatenated to form a register of the same type whose size is the
sum of the sizes of the individual registers. The concatenated register
is a reference to the bits or qubits of the original registers. The
statement ``a || b`` denotes the concatenation of registers ``a`` and ``b``. A register cannot
be concatenated with any part of itself.

Classical and quantum registers can be indexed in a way that selects a
subset of (qu)bits, i.e. by an index set. A register so indexed is
interpreted as a register of the same type but with a different size.
The register slice is a reference to the original register. A register
cannot be indexed by an empty index set.

An index set can be specified by a single unsigned integer, a
comma-separated list of unsigned integers ``a,b,c,…``, or a range. A
range is written as ``a:b`` or ``a:c:b`` where ``a``, ``b``, and ``c`` are integers (signed or unsigned).
The range corresponds to the set :math:`\{a, a+c, a+2c, \dots, a+mc\}`
where :math:`m` is the largest integer such that :math:`a+mc\leq b` if
:math:`c>0` and :math:`a+mc\geq b` if :math:`c<0`. If :math:`a=b` then
the range corresponds to :math:`\{a\}`. Otherwise, the range is the
empty set. If :math:`c` is not given, it is assumed to be one, and
:math:`c` cannot be zero. Note the index sets can be defined by
variables whose values may only be known at run time.

.. code-block:: c

   qubit[2] one;
   qubit[10] two;
   // Aliased register of twelve qubits
   let concatenated = one || two;
   // First qubit in aliased qubit array
   let first = concatenated[0];
   // Last qubit in aliased qubit array
   let last = concatenated[-1];
   // Qubits zero, three and five
   let qubit_selection = two[0, 3, 5];
   // First six qubits in aliased qubit array
   let sliced = concatenated[0:6];
   // Every second qubit
   let every_second = concatenated[0:2:12];
   // Using negative ranges to take the last 3 elements
   let last_three = two[-4:-1];
   // Concatenate two alias in another one
   let both = sliced || last_three;
