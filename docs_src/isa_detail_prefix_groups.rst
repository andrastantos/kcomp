.. _type_overrides_detail:

Type override (<type>)
-----------------------

*Instruction code*: 0xff** ...

*Exceptions*: None

*Type variants*: Yes

Description
~~~~~~~~~~~
Type override provides a way to momentarily change the apparent type of any of the operands of an instruction, without actually changing the stored type of the associated register. The override prefix changes the way types are interpreted in the immediately succeeding instruction. Any valid instruction can be preceded by a type override prefix. While in principle cascading prefix instructions is valid, this version of the ISA specifies only a single prefix instruction. The only conceivable cascading prefix sequence thus is multiple type overrides, which is invalid and raises an :code:`exc_unknown_inst` exception.

If either TYPE_A or TYPE_B is set to 0xf, the corresponding operand type is not overridden: the type from the register file is used during the subsequent operation.

Overriding the type of immediates is not supported. The corresponding field of the type override prefix is simply ignored for immediate operands.

In assembly, any appearance of an operand register can be prefixed with a type override in the form of :code:`(<type>)`. For instance::

    $r4 <- (INT32) $r6 * $r7
    $r4 <- (INT32) $r6 + (FP32) $r7
    if any (FP32) $r7 > 0 $pc <- $pc + 10

In all of these cases the type of the operand is overwritten to the given type. Type overrides cannot be provided to constants. For instance the following syntax is invalid::

    $r6 <- (FP32) $r12 + (FP32) 0x12345768

The result type that is written into the destination along with its value is the result type obtained after the type overrides.

.. note::
  *Exception behavior*: If a prefixed instruction throws an exception, $tpc points to the (first) prefix instruction after entering SCHEDULER mode. This allows the recovery code to decode and potentially retry the excepted instruction.

.. note::
  *Interrupt behavior*: If an interrupt is handled during the execution of a prefixed instruction, $tpc points to the (first) prefix instruction after entering SCHEDULER mode.

.. note::
  *Prefix concatenation*: Every processor implementation has a maximum instruction length it supports. In this version of the spec, it's 64 bits. If an instruction with all its prefixes exceeds this limit, the processor raises an :code:`exc_unknown_inst` exception, with $tpc pointing to the first prefix instruction. Without this provision it would be possible to create arbitrarily long instruction sequences in TASK mode. That in turn would prevent interrupts from being raised, effectively locking up the system (at least up to the point of exhausting the addressable RAM space).

