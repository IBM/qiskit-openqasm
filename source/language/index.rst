.. _sec:spec:

Language
========

OpenQASM refers to the extended language we now describe, specifically
OpenQASM Version 3.0.

The human-readable form of OpenQASM is a simple, case-sensitive textual language.
Statements are separated by semicolons and whitespace is ignored.

In other respects, OpenQASM possesses a dual nature as an assembly language and
as a hardware description language

Appendix `[app:summary] <#app:summary>`__ summarizes the
language statements, Appendix `[app:grammar] <#app:grammar>`__ specifies
the grammar, and Appendix `[app:semantics] <#app:semantics>`__ gives formal
semantics.

.. toctree::

   comments
   types
   gates
   insts
   classical
   subroutines
   directives
   delays
   pulses
   openpulse
