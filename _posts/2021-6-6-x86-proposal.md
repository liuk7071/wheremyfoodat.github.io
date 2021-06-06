# Proposal to the x86 committee for a new long mode instruction

# Context
The [x86-64](https://en.wikipedia.org/wiki/X86-64) Instruction Set Architecture (ISA) is a 64-bit version of the x86 instruction set, first released in 1999, which is notable for introducing a new 64-bit operating mode (Also known as long mode). The x86-64 ISA (which will also be referred to as x64, x86_64, AMD64 or Intel 64 in this document) introduces new General Purpose Registers (GPRs), named r8-r15, new operations that work on 64-bit values, a new addressing mode for instructions that access memory, known as rip-relative addressing, which forms an address by applying displacements to the Instruction Pointer (The RIP register), and more

In addition, the x64 ISA also introduces 8 new Vector Registers (VRs), bumping the xmm (and consequently ymm/zmm) register count from 8 to 16.

These vector registers are accessible via operations introduced in the [Streaming SIMD Extensions (SSE)](https://en.wikipedia.org/wiki/Streaming_SIMD_Extensions) and the [Advanced Vector Extensions (AVX)](https://en.wikipedia.org/wiki/Advanced_Vector_Extensions) instruction set extensions, as well as their derivatives (SSE2, SSE3, SSSE3, SSE4, AVX2 and AVX-512). They're mainly used for [Single Instruction, Multiple Data (SIMD)](https://en.wikipedia.org/wiki/SIMD) operations, as well as floating-point operations, replacing and deprecating the old, slow and outdated [x87 FPU](https://en.wikipedia.org/wiki/X87)

The goal of this proposal is to suggest the addition of a new, potentially useful SIMD instruction, in a future extension to the x64 ISA. As is already known, the x64 ISA is still being embellished with new features and instructions for SIMD operations, most notably with the recent addition of the AVX-512 extension to commercial, non-enthusiast processor lines, such as the Cannon and Ice Lake processors.

# Our new instruction
Among the [instructions introduced in the SSE2 instruction set](https://en.wikipedia.org/wiki/X86_instruction_listings#SSE2_instructions) are the SSE2 SIMD Integer Instructions, which include the SSE2 MMX-like instructions extended to SSE registers. These include 4 instructions meant to perform bitwise operations on packed values. Those 4 instructions are [pand (Packed Logical And)](https://www.felixcloutier.com/x86/pand), [por (Packed Logical Or)](https://www.felixcloutier.com/x86/pand), [pxor (Packed Logical Xor)](https://www.felixcloutier.com/x86/pxor) and [pandn (Packed Logical And Not)](https://www.felixcloutier.com/x86/pandn).

This proposal aims to introduce a new instruction, `panda`, which stands for `packed logical and with accumulator`

# Panda
The panda instruction is unique due to being one of the few instructions that operate on a vector register and GPRs at the same time (in a similar manner to [`movd` and `movq`](https://www.felixcloutier.com/x86/movd:movq)). The 128-bit version of the panda instruction performs a logical AND between the lhs 128-bit operand (destination) with the source operand (which is always, implicitly, the register pair rdx:rax) and writes back the result to the destination operand

The 256-bit version of the panda instruction, vpanda, performs the same operation, except the source operand is (again, implicitly) the register quartet rsi:rdi:rdx:rax

# Addressing modes
```x86asm
panda xmm0 / m128
vpanda ymm0 / m256
```

# Example usage
```x86asm
xor edx, edx ; rdx = 0
mov rax, qword [rel g_mask] ; load low 64 bits of mask into rax 

movsd xmm3, qword [rel our_float_1] ; load a scalar double value into xmm3
panda xmm3 ; Slap a mask on the double because why not
or edx, 0x696969 ; Our mask is contained in 2 GPRs so it's very easy to change it

movdqa xmm2, xmmword [rel our_128bit_value] ; load a 128-bit value into xmm2
panda xmm2 ; AND with rdx:rax

xor esi, esi ; rsi = 0
xor edi, edi ; rdi = 0
vmovdqa ymm0, ymmword [rel our_256bit_value]
vpanda ymm0 ; We can do this with 128-bit values too!
vpanda ymmword [rel our_256bit_value] ; We can do it on the memory address as well!
```

# Use cases
There's certain cases where VR-GPR interoperability might be slightly helpful and fulfill a specific niche, and as we know this is x86, where if something has the slighest bit of use it gets added to the ISA (Or if you are a major corporation and are willing to pay enough)