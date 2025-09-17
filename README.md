# Assembly Pattern Recognition Guide

#### Learning Objectives
* Identify function boundaries and structure in x86-64 assembly code
* Recognize variable declaration and assignment patterns
* Understand control flow constructs like loops and conditionals
* Decode input/output operations for cout and cin statements
* Map assembly instructions to equivalent C++ code constructs
* Distinguish between program logic and compiler-generated boilerplate code

#### Function Structure

Every C++ function translates to assembly with a predictable structure. The function begins with a prologue that sets up the stack frame, followed by the actual program logic, and ends with an epilogue that cleans up and returns.

```assembly
push   %rbp          # Save caller's base pointer
mov    %rsp,%rbp     # Set up new stack frame
sub    $0x10,%rsp    # Allocate space for local variables
```

This prologue corresponds to the opening brace of a C++ function like `int main() {`. The epilogue follows the same pattern in reverse:

```assembly
mov    $0x0,%eax     # Set return value
leaveq               # Clean up stack frame
retq                 # Return to caller
```

This epilogue represents `return 0; }` in C++ code. Everything between the prologue and epilogue contains the actual program logic you need to convert.

#### Example: Complete C++ to Assembly Conversion

Let's examine how a simple C++ program converts to assembly:

```cpp
int main()
{
    int x = 10;
    cout << "x is " << x; 
    return 0;
}
```

The corresponding assembly code:

```assembly
main:
   0:   push   %rbp                    # Function start
   1:   mov    %rsp,%rbp               # Set up stack frame
   4:   sub    $0x10,%rsp              # Allocate local variables
   8:   movl   $0xa,-0x4(%rbp)         # int x = 10;
   f:   lea    0x0(%rip),%rsi          # Load "x is " string address
  16:   lea    0x0(%rip),%rdi          # Load cout object address
  1d:   callq  <operator<<>            # cout << "x is "
  22:   mov    %rax,%rdx               # Store cout return value
  25:   mov    -0x4(%rbp),%eax         # Load x into register
  28:   mov    %eax,%esi               # Prepare x for output
  2a:   mov    %rdx,%rdi               # Restore cout object
  2d:   callq  <operator<<>            # cout << x
  32:   mov    $0x0,%eax               # return 0;
  37:   leaveq                         # Clean up stack
  38:   retq                           # Return to caller
```

#### Variable Operations

Variable assignments in C++ become mov instructions in assembly. When you see `movl $0xa,-0x4(%rbp)`, this translates to `int x = 10;` in C++. The immediate value `$0xa` is 10 in hexadecimal, and `-0x4(%rbp)` represents the memory location where variable x is stored on the stack.

Loading variables for use in operations appears as `mov -0x4(%rbp),%eax`, which moves the variable's value into the eax register for further processing. The stack grows downward, so local variables appear at negative offsets from the base pointer rbp.

#### Assembly Instruction Reference

| Assembly Instruction | C++ Equivalent | Description |
|---------------------|----------------|-------------|
| `movl $10, -0x4(%rbp)` | `int x = 10;` | Store immediate value in variable |
| `mov -0x4(%rbp), %eax` | Load x for use | Move variable to register |
| `addl %eax, %ebx` | `b += a;` | Add two values |
| `subl $1, -0x4(%rbp)` | `x--;` | Decrement variable |
| `imull %eax, %ebx` | `b *= a;` | Multiply two values |
| `cmpl $0, %eax` | `if (x == 0)` | Compare value with zero |
| `je .L1` | `if (condition)` | Jump if equal |
| `jne .L1` | `if (!condition)` | Jump if not equal |
| `jl .L1` | `if (a < b)` | Jump if less than |
| `jg .L1` | `if (a > b)` | Jump if greater than |
| `jmp .L1` | `goto` or loop | Unconditional jump |
| `callq function` | `function();` | Function call |
| `push %rbp` | Function start | Save caller's frame |
| `leaveq` | Function end | Restore stack |
| `retq` | `return;` | Return to caller |

#### Control Flow

Loop structures in assembly consist of labels, comparison instructions, and conditional jumps. A typical loop pattern looks like:

```assembly
.L3:                      # Loop label
    cmpl   $0,-0x4(%rbp)  # Compare variable with 0
    jle    .L2            # Jump if less/equal (exit loop)
    # Loop body instructions
    subl   $1,-0x4(%rbp)  # Decrement counter
    jmp    .L3            # Jump back to loop start
.L2:                      # Loop exit
```

This assembly pattern represents a while loop in C++ that continues as long as a counter is greater than zero. The `cmpl` instruction performs the comparison, `jle` provides the conditional exit, and `jmp` creates the loop back to the beginning.

#### Input/Output Operations

Console output operations translate to function calls in assembly. String output appears as a sequence where the string address is loaded into a register, followed by a call to the cout operator:

```assembly
lea    0x0(%rip),%rsi    # Load string address
lea    0x0(%rip),%rdi    # Load cout object address  
callq  <operator<<>      # Call cout << operator
```

This pattern corresponds to `cout << "string";` in C++. Integer output follows a similar pattern but loads the variable value instead of a string address:

```assembly
mov    -0x4(%rbp),%eax   # Load variable into eax
mov    %eax,%esi         # Move to argument register
callq  <operator<<>      # Call cout << operator
```

#### Memory Management

The stack layout provides the foundation for understanding how variables are stored and accessed. The stack pointer rsp points to the current top of the stack, while the base pointer rbp provides a stable reference point for accessing local variables.

Local variables are stored at negative offsets from rbp, such as `-0x4(%rbp)` for the first variable, `-0x8(%rbp)` for the second, and so on. This addressing scheme allows the compiler to generate consistent code for accessing variables regardless of how the stack changes during execution.

#### Common Patterns

Certain assembly patterns appear frequently and map directly to C++ constructs. Data movement instructions like `movl $10, %eax` load immediate values, while `movl %eax, %ebx` copies values between registers. Stack-based variable access uses patterns like `movl -0x4(%rbp), %eax` to load variables.

Jump instructions provide the control flow mechanisms. Conditional jumps like `je` (jump if equal), `jne` (jump if not equal), `jl` (jump if less), and `jg` (jump if greater) correspond directly to if statement conditions in C++. Unconditional jumps using `jmp` represent goto statements or loop continuation.

Function calls appear as `callq` instructions with mangled C++ function names. Standard library operations like cout and cin have recognizable patterns even when the function names are mangled by the compiler.

#### Summary

Converting assembly to C++ requires recognizing these fundamental patterns and understanding the relationship between high-level constructs and low-level instructions. Focus on identifying function boundaries, variable operations, control flow structures, and input/output operations while ignoring compiler-generated boilerplate code. The key is to trace the logical flow of the program and map assembly instruction sequences to their equivalent C++ statements. With practice, these patterns become recognizable, making the conversion process systematic and reliable.
