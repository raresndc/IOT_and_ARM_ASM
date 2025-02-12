# ARM Assembly Language Examples

This document provides various ARM Assembly examples alongside their corresponding C code, with explanations for each instruction. This serves as a guide to understanding basic ARM assembly operations, calling conventions, and interrupt handling.

---

## 1. Sum of Two Numbers

### C Code:
```c
int sum(int a, int b) {
    return a + b;
}
```

### ARM Assembly:
```assembly
.global sum                    // Make the function accessible externally
sum:                           // Function label
    ADD r0, r0, r1             // Add r0 (a) and r1 (b), store result in r0
    BX lr                      // Return from the function
```

**Explanation:**
- **Calling Conventions:**
  - First four parameters are passed in `r0`-`r3`.
  - The result is returned in `r0`.
- `ADD r0, r0, r1`: Adds `a` and `b`.
- `BX lr`: Branch back to the caller.

---

## 2. Simple Addition in ARM Assembly to C

### ARM Assembly:
```assembly
MOV r0, #5
MOV r1, #10
ADD r2, r0, r1
```

### Equivalent C Code:
```c
int a = 5;
int b = 10;
int c = a + b;
```

**Explanation:**
- `MOV r0, #5`: Load immediate value 5 into `r0`.
- `MOV r1, #10`: Load immediate value 10 into `r1`.
- `ADD r2, r0, r1`: Add `r0` and `r1`, store result in `r2`.

---

## 3. Conditional Addition

### ARM Assembly:
```assembly
CMP r0, #3        // Compare r0 with 3
ADDLT r0, r0, #1  // If r0 < 3, add 1 to r0
```

**Explanation:**
- `CMP r0, #3`: Compare the value in `r0` with 3.
- `ADDLT r0, r0, #1`: If `r0` is less than 3, increment it by 1.

---

## 4. Handling Interrupts in ARM Assembly

### ARM Assembly:
```assembly
.section .vector_table
.global _start

_start:
    LDR pc, =reset_handler
    .space 24               // Skip other vectors for simplicity
    LDR pc, =irq_handler    // IRQ handler at 0x00000018

.section .text
.global reset_handler

reset_handler:
    MOV r0, #0xD2           // Switch to IRQ mode
    MSR CPSR_c, r0
    LDR sp, =0x8000         // Set IRQ mode stack pointer

    MOV r0, #0xD3           // Switch back to supervisor mode
    MSR CPSR_c, r0
    BL setup_timer          // Setup timer interrupt

    MRS r0, CPSR            // Enable IRQs
    BIC r0, r0, #0x80
    MSR CPSR_c, r0

main_loop:
    B main_loop             // Infinite loop

setup_timer:
    LDR r0, =TIMER_BASE_ADDRESS
    MOV r1, #0x1            // Enable timer
    STR r1, [r0, #TIMER_CTRL_OFFSET]
    BX lr

irq_handler:
    SUB lr, lr, #4          // Adjust return address
    PUSH {r0-r12, lr}       // Save registers

    BL handle_timer_interrupt

    POP {r0-r12, pc}^       // Restore registers and return

handle_timer_interrupt:
    LDR r0, =TIMER_BASE_ADDRESS
    MOV r1, #0x1
    STR r1, [r0, #TIMER_INT_CLEAR_OFFSET]  // Clear interrupt flag
    BX lr
```

**Explanation:**
- **Vector Table:** Directs the processor to the appropriate handler.
- **Mode Switching:** Changes processor mode to handle interrupts.
- **IRQ Handling:** Saves context, processes the interrupt, and restores the context.

---

## 5. Compare Two Numbers and Store the Larger

### ARM Assembly:
```assembly
CMP r0, r1        // Compare r0 and r1
MOVGT r2, r0      // If r0 > r1, move r0 to r2
MOVLE r2, r1      // If r0 <= r1, move r1 to r2
```

**Explanation:**
- Compares two numbers and stores the larger in `r2`.

---

## 6. Subtraction

### C Code:
```c
int subtract(int a, int b) {
    return a - b;
}
```

### ARM Assembly:
```assembly
.global subtract
subtract:
    SUB r0, r0, r1   // r0 = a - b
    BX lr            // Return
```

---

## 7. Multiplication

### C Code:
```c
int multiply(int a, int b) {
    return a * b;
}
```

### ARM Assembly:
```assembly
.global multiply
multiply:
    MUL r0, r0, r1   // r0 = a * b
    BX lr            // Return
```

---

## 8. Division

### C Code:
```c
int divide(int a, int b) {
    return a / b;
}
```

### ARMv7 Assembly:
```assembly
.global divide
divide:
    SDIV r0, r0, r1  // r0 = a / b
    BX lr            // Return
```

---

## 9. Check if a Number is Even

### C Code:
```c
int is_even(int num) {
    return num % 2 == 0;
}
```

### ARM Assembly:
```assembly
.global is_even
is_even:
    AND r1, r0, #1    // Check least significant bit
    CMP r1, #0        // Compare with 0
    MOVEQ r0, #1      // If even, return 1
    MOVNE r0, #0      // If odd, return 0
    BX lr             // Return
```

---

## 10. Maximum of Two Numbers

### C Code:
```c
int max(int a, int b) {
    return (a > b) ? a : b;
}
```

### ARM Assembly:
```assembly
.global max
max:
    CMP r0, r1        // Compare a and b
    MOVGT r0, r0      // If a > b, keep a
    MOVLE r0, r1      // Else, move b to r0
    BX lr             // Return
```

---

## 11. Factorial Function

### C Code:
```c
int factorial(int n) {
    if (n == 0) return 1;
    else return n * factorial(n - 1);
}
```

### ARM Assembly:
```assembly
.global factorial
factorial:
    PUSH {lr}              // Save return address
    CMP r0, #0             // Compare n with 0
    BEQ base_case          // If n == 0, jump to base_case

    SUB r0, r0, #1         // n = n - 1
    BL factorial           // Recursive call
    MOV r1, r0             // Store result in r1
    LDR r0, [sp, #4]       // Load original n from stack
    MUL r0, r0, r1         // Multiply n * factorial(n - 1)
    POP {pc}               // Return

base_case:
    MOV r0, #1             // Return 1 if n == 0
    POP {pc}               // Return
```

---

## 12. Sum of an Array

### C Code:
```c
int sum_array(int arr[], int size) {
    int sum = 0;
    for (int i = 0; i < size; i++) {
        sum += arr[i];
    }
    return sum;
}
```

### ARM Assembly:
```assembly
.global sum_array
sum_array:
    PUSH {r4, r5, lr}       // Save registers

    MOV r4, r0              // r4 = arr
    MOV r5, #0              // r5 = sum
    MOV r2, #0              // r2 = i

loop:
    CMP r2, r1              // Compare i with size
    BGE end                 // Exit loop if i >= size

    LDR r3, [r4, r2, LSL #2] // Load arr[i]
    ADD r5, r5, r3          // sum += arr[i]

    ADD r2, r2, #1          // i++
    B loop                  // Repeat loop

end:
    MOV r0, r5              // Return sum
    POP {r4, r5, pc}        // Restore registers
```

---

## 13. Swap Two Numbers

### C Code:
```c
void swap(int *a, int *b) {
    int temp = *a;
    *a = *b;
    *b = temp;
}
```

### ARM Assembly:
```assembly
.global swap
swap:
    PUSH {lr}               // Save return address

    LDR r2, [r0]            // Load *a into r2
    LDR r3, [r1]            // Load *b into r3
    STR r3, [r0]            // *a = *b
    STR r2, [r1]            // *b = temp

    POP {pc}                // Return
```

---

## 14. Most Used ARM Assembly Instructions

### 1. **MOV** - Move/Load Immediate Value
**Usage:** Transfers data between registers or loads immediate values.
```assembly
MOV r0, #5   // Load immediate value 5 into r0
MOV r1, r0   // Copy value from r0 to r1
```

### 2. **ADD/SUB** - Addition and Subtraction
**Usage:** Performs arithmetic addition or subtraction.
```assembly
ADD r0, r0, r1   // r0 = r0 + r1
SUB r0, r0, #1   // r0 = r0 - 1
```

### 3. **MUL/SDIV** - Multiplication and Division
**Usage:** Multiplies or divides the contents of registers.
```assembly
MUL r0, r1, r2   // r0 = r1 * r2
SDIV r0, r1, r2  // r0 = r1 / r2
```

### 4. **CMP** - Compare
**Usage:** Compares two values and updates condition flags.
```assembly
CMP r0, r1       // Compare r0 with r1
```

### 5. **B/BL/BX** - Branch Instructions
**Usage:** Controls the flow of the program.
```assembly
B loop           // Unconditional branch to label 'loop'
BL function      // Branch to 'function' and link return address
BX lr            // Return from function
```

### 6. **LDR/STR** - Load and Store
**Usage:** Loads data from memory or stores data to memory.
```assembly
LDR r0, [r1]     // Load value from memory address in r1 into r0
STR r0, [r1]     // Store value from r0 into memory address in r1
```

### 7. **PUSH/POP** - Stack Operations
**Usage:** Saves and restores registers from the stack.
```assembly
PUSH {r0, lr}    // Save r0 and link register to stack
POP {r0, pc}     // Restore r0 and program counter (return)
```

### 8. **AND/ORR/EOR** - Bitwise Operations
**Usage:** Performs bitwise AND, OR, and XOR.
```assembly
AND r0, r1, #0xFF   // Bitwise AND r1 with 0xFF, store in r0
ORR r0, r1, r2      // Bitwise OR r1 with r2, store in r0
EOR r0, r1, r2      // Bitwise XOR r1 with r2, store in r0
```

---

This document provides a comprehensive overview of basic ARM assembly instructions and their practical applications in C programming. Each example illustrates fundamental operations and ARM-specific conventions to help you get started with ARM assembly programming.

