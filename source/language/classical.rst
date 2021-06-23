Classical instructions
======================

We envision two levels of classical control: simple, low-level instructions embedded as
part of a quantum circuit and high-level external functions which perform more complex
classical computations. The low-level functions allow basic
computations on lower-level parallel control processors. These instructions are likely
to have known durations and many such instructions might be executed
within the qubit coherence time. The external, or ``extern``, functions execute
complex blocks of classical code that may be neither fast nor guaranteed to return. In order
to connect with classical compilation infrastucture, ``extern`` functions are defined outside of
OpenQASM. The compiler toolchain is expected to link ``extern`` functions when building an
executable. This strategy allows the programmer to use existing libraries without porting them into
OpenQASM. ``extern`` functions run on a global processor concurrently with operations on local
processors, if possible. ``extern`` functions can write to the global controller’s memory,
which may not be directly accessible by the local controllers.

Low-level classical instructions
--------------------------------

Generalities
~~~~~~~~~~~~

All types support the assignment operator ``=``. The left-hand-side (LHS) and
right-hand-side (RHS) of the assignment operator must be of the same
type. For real-time values assignment is by copy of the RHS value to the
assigned variable on the LHS.

.. code-block:: c

   int[32] a;
   int[32] b = 10; // Combined declaration and assignment

   a = b; // Assign b to a
   b = 0;
   a == b; // False
   a == 10; // True

Classical bits and registers
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Classical registers and bits support bitwise operators and the
corresponding assignment operators between registers of the same size:
and ``&``, or ``|``, xor ``^``. They support left shift ``<<`` and right shift ``>>`` by an unsigned
integer, and the corresponding assignment operators. The shift operators
shift bits off the end. They also support not ``~``, ``popcount`` [1]_, and left and
right circular shift, ``rotl`` and ``rotr``, respectively.

.. code-block:: c

   bit[8] a = "10001111";
   bit[8] b = "01110000";

   a << 1; // Bit shift left produces "00011110"
   rotl(a, 2) // Produces "00111110"
   a | b; // Produces "11111111"
   a & b; // Produces "00000000"

Comparison (Boolean) Instructions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Integers, fixed-point numbers, angles, bits, and classical registers can
be compared (:math:`>`, :math:`>=`, :math:`<`, :math:`<=`, :math:`==`,
:math:`!=`) and yield Boolean values. Boolean values support logical
operators: and ``&&``, or ``||``, not ``!``. The keyword ``in`` tests if an integer belongs to
an index set, for example ``i in {0,3}`` returns ``true`` if i equals 0 or 3 and ``false`` otherwise.

.. code-block:: c

   bool a = false;
   int[32] b = 1;
   fixed[8, 12] c = 1.05;
   angle[32] d = pi;
   float[32] e = pi;

   a == false; // True
   a == bool(b); // False
   c >= b; // True
   d == pi; // True
   // Susceptible to floating point casting errors
   e == float(d);

Integers
~~~~~~~~

Integer types support addition ``+``, subtraction ``-``, multiplication ``*``, integer division [2]_ ``/``
and modulo ``%``; the corresponding assignments ``+=``, ``-=``, ``*=``, ``/=`` and ``%=``; as well as
increment ``++`` and decrement ``--``.

.. code-block:: c

   int[32] a = 2;
   int[32] b = 3;

   a * b; // 6
   b / a; // 1
   b % a; // 1
   a += 4; // a == 6
   a++; // a == 7

Fixed-point numbers, floating-point numbers and angles
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Fixed-point, floating-point and angle types support addition, subtraction,
multiplication, and division and the corresponding assignment operators.

.. code-block:: c

   angle[20] a = pi / 2;
   angle[20] b = pi;
   a + b; // 3/2 * pi
   angle[10] c;
   c = angle(a + b); // cast to angle[10]

Complex numbers
~~~~~~~~~~~~~~~

Complex numbers support addition, subtraction, multiplication, and division and the corresponding
assignment operators.

.. code-block:: c

   complex[float[64]] a = 10.0 + 5.0im;
   complex[float[64]] b = -2.0 - 7.0im;
   complex[float[64]] c = a + b; // c = 8.0 - 2.0im
   complex[float[64]] d = a - b; // d = 12.0+12.0im;
   complex[float[64]] e = a*b; // e = 15.0-80.0im;
   complex[float[64]] f = a/b; // f = (-55.0+60.0im)/53.0

Evaluation order
~~~~~~~~~~~~~~~~

OpenQASM evaluates expressions from left to right.

   .. table:: [operator-precedence] operator precedence in OpenQASM ordered from highest precedence to lowest precedence. Higher precedence operators will be evaluated first.

      +----------------------------------------+------------------------------------+
      | Operator                               | Operator Types                     |
      +----------------------------------------+------------------------------------+
      | ``()``, ``[]``, ``++``, ``(type)(x)``  | Call, index, incrementors, cast    |
      +----------------------------------------+------------------------------------+
      | ``!``, ``-``, ``~``                    | Unary                              |
      +----------------------------------------+------------------------------------+
      | ``*``, ``/``, ``%``                    | Multiplicative                     |
      +----------------------------------------+------------------------------------+
      | ``+``, ``-``                           | Additive                           |
      +----------------------------------------+------------------------------------+
      | ``<<``, ``>>``                         | Bit Shift                          |
      +----------------------------------------+------------------------------------+
      | ``<``, ``<=``, ``>``, ``>=``           | Comparison                         |
      +----------------------------------------+------------------------------------+
      | ``!=``, ``==``                         | Equality                           |
      +----------------------------------------+------------------------------------+
      | ``&``                                  | Bitwise AND                        |
      +----------------------------------------+------------------------------------+
      | ``^``                                  | Bitwise XOR                        |
      +----------------------------------------+------------------------------------+
      | ``|``                                  | Bitwise OR                         |
      +----------------------------------------+------------------------------------+
      | ``&&``                                 | Logical AND                        |
      +----------------------------------------+------------------------------------+
      | ``||``                                 | Logical OR                         |
      +----------------------------------------+------------------------------------+


Looping and branching
~~~~~~~~~~~~~~~~~~~~~

The statement ``if ( bool ) { program }`` branches to program if the Boolean evaluates to true and
may optionally be followed by ``else { program }``.

.. code-block:: c

   bool target = false;
   qubit a;
   h a;
   bit output = measure qubit

   // example of branching
   if (target == output) {
      // do something
   } else {
      // do something else
   }

The statement ``for name in indexset { program }`` loops over integer values in the indexset, assigning them
to ``name``. The for loop body is not permitted to modify the loop variable of
the indexset.

.. code-block:: c

   int[32] b;
   // loop over a discrete set of values
   for i in {1, 5, 10} {
       b += i;
   } // b == 16

   // loop over every even integer from 0 to 20 using an indexset
   for i in [0:2:20] {
      // do something
   }

The statement ``while ( bool ) { program }`` executes program until the Boolean evaluates to
false [3]_. Variables in the loop condition statement may be modified
within the while loop body.

.. code-block:: c

   qubit q;
   bit result;

   int i = 0;
   // Keep applying hadamards and measuring a qubit
   // until 10, |1>s are measured
   while (i < 10) {
       h q;
       result = measure q;
       if (result) {
           i++;
       }
   }

A block ``{ program }`` can be exited with the statement ``break;``. The statement ``continue;`` can appear in
the body of a for or while loop. It returns control to the loop
condition. The statement ``end;`` terminates the program. In all of the
preceding, ``{ program }`` can also be replaced by a statement without the braces.

.. code-block:: c

   int[32] i = 0;

   while (i < 10) {
       i++;
       // continue to next loop iteration
       if (i == 2) {
           continue;
       }

       // some program

       // break out of loop
       if (i == 4) {
           break;
       }

       // more program
   }

Extern function calls
---------------------

``extern`` functions are declared by giving their signature using the
statement ``extern name(inputs) -> output;`` where ``inputs`` is a comma-separated list of type
names and ``output`` is a single type name. The parentheses may be omitted if there are no ``inputs``.

``extern`` functions can take of any number of arguments whose types correspond to the classical
types of OpenQASM. Inputs are passed by value. They can return zero or one value whose type
is any classical type in OpenQASM except real constants. If necessary,
multiple return values can be accommodated by concatenating registers.
The type and size of each argument must be known at compile time to
define data flow and enable scheduling. We do not address issues such as
how the ``extern`` functions are defined and registered.

``extern`` functions are invoked using the statement ``name(inputs);`` and the result may be
assigned to ``output`` as needed via an assignment operator (``=``, ``+=``, etc). ``inputs`` are
literals and ``output`` is a variable, corresponding to the types in the signature. The functions
are not required to be idempotent. They may change the state of the process providing the function.
In our computational model, ``extern`` functions may run concurrently with other classical and
quantum computations. That is, invoking an ``extern`` function will  *schedule* a classical
computation, but does not wait for that computation to terminate.

.. [1]
   ``popcount`` computes the Hamming weight of the input register.

.. [2]
   If multiplication and division instructions are not available in
   hardware, they can be implemented by expanding into other
   instructions.

.. [3]
   This clearly allows users to write code that does not terminate. We
   do not discuss implementation details here, but one possibility is to
   compile into target code that imposes iteration limits
