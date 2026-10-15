
# Buffer Overflow and ASLR

## Project Overview

This project demonstrates how Address Space Layout Randomization (ASLR) affects memory addresses and how a buffer overflow can cause a vulnerable program to crash.

The lab was conducted in a Linux environment using C and Python. ASLR was enabled and disabled to compare stack pointer addresses across program executions. A vulnerable C program was then compiled and tested with a Python-generated payload, resulting in a segmentation fault.

The exercise highlights the risks associated with unsafe memory-handling functions and the importance of operating-system security protections.

## Lab Objectives & Tools

- **Objectives:** Observe ASLR behavior, disable and re-enable ASLR, create a vulnerable C program, generate a test payload using Python, and demonstrate a buffer overflow.
- **Tools & Environment:** Linux, GCC, C, Python 3, Linux kernel ASLR settings.

| Machine / Tool | Purpose | Configuration |
|---|---|---|
| Linux | Lab environment | Local system |
| GCC | Compile C programs | GNU Compiler Collection |
| C | Demonstrate vulnerable memory handling | `strcpy()` |
| Python 3 | Generate test payload | 116-character payload |
| ASLR | Memory address randomization | `/proc/sys/kernel/randomize_va_space` |


All testing was performed as part of a controlled lab exercise.

---

## Part 1 – Observing How ASLR Works

### 1. Enable ASLR

ASLR was enabled to observe how memory addresses changed between program executions.

```bash
echo 1 > /proc/sys/kernel/randomize_va_space
```

> *ASLR Enabled*

<img width="468" height="108" alt="image" src="https://github.com/user-attachments/assets/a2cfda9a-e887-4f29-be04-1d3d88effb6c" />

---

### 2. Create and Run the ESP Program

A C program named `esp.c` was created to display the stack pointer address. The program was compiled and executed multiple times.

```bash
gcc -o esp esp.c
./esp
./esp
./esp
```

Each execution produced a different stack pointer address, demonstrating how ASLR randomizes memory locations between program runs.

> *ESP Program with ASLR Enabled*

<img width="468" height="86" alt="image" src="https://github.com/user-attachments/assets/fa6b7e17-7f2b-420e-9e4b-053a703cfdf3" />
<img width="468" height="101" alt="image" src="https://github.com/user-attachments/assets/a559632d-8ffb-48a4-9593-6db2f4ef97df" />


---

## Part 2 – Disabling ASLR

### 3. Disable ASLR

ASLR was temporarily disabled to compare the memory layout with address randomization turned off.

```bash
echo 0 > /proc/sys/kernel/randomize_va_space
```

The ESP program was executed multiple times again.

**Observation:** The stack pointer address remained consistent across executions in the lab environment.

This comparison demonstrated the role of ASLR in making memory addresses less predictable.

> *ESP Program with ASLR Disabled*

<img width="432" height="118" alt="image" src="https://github.com/user-attachments/assets/84079db9-99f3-4b9b-8ffd-815133a6b635" />

---

## Part 3 – Creating the Vulnerable C Program

### 4. Write and Compile the Vulnerable Program

A C program named `bo1.c` was created using the `strcpy()` function to copy user input into a fixed-size buffer.

The program was compiled with the following command:

```bash
gcc -g -z execstack -no-pie -o bo1 bo1.c
```

The program was tested with a short input:

```bash
./bo1 A
```

Expected output:

```text
Done!
```

The program completed successfully with normal input.

The use of `strcpy()` creates a potential buffer overflow risk because the function does not automatically verify that the destination buffer can hold the supplied input. The compilation options also modify certain executable protections for this lab demonstration.

> *Vulnerable Program Compilation and Test*

<img width="430" height="139" alt="image" src="https://github.com/user-attachments/assets/59d2a9e2-9a6b-491a-8946-0f045a90a9b9" />
<img width="432" height="106" alt="image" src="https://github.com/user-attachments/assets/264b881e-30ed-4c7d-88ef-bdb746e66b2f" />


---

## Part 4 – Creating the Python Payload

### 5. Create the Python Script

A Python script named `b1` was created to generate a payload consisting of 116 capital letter `A` characters.

The script used the following shebang for Python 3 compatibility:

```python
#!/usr/bin/env python3
```

The script was made executable and run:

```bash
chmod 777 b1
./b1
```

The script printed 116 `A` characters.

> *Python Payload Generation*

<img width="468" height="80" alt="image" src="https://github.com/user-attachments/assets/a2744f0c-f185-4d0d-93c4-2fc15587423a" />
<img width="468" height="101" alt="image" src="https://github.com/user-attachments/assets/2ccd8053-05cf-4730-ac4e-307dd7a95a23" />


---

### 6. Save the Payload to a File

The generated output was redirected to a file named `e1`.

```bash
./b1 > e1
ls -l e1
```

The file contained the generated payload followed by a newline.

> *Payload File Creation*

<img width="468" height="95" alt="image" src="https://github.com/user-attachments/assets/9db2d8f0-bb53-4c9e-99ba-fec64e997896" />

---

## Part 5 – Performing the Buffer Overflow Test

### 7. Execute the Test

The vulnerable program was executed using the generated payload:

```bash
./bo1 $(cat e1)
```

**Result:** The program crashed with a segmentation fault.

The crash demonstrated that the oversized input caused invalid memory access in the vulnerable program. This is consistent with a buffer overflow, although a segmentation fault alone does not demonstrate arbitrary code execution.

> *Buffer Overflow Result*

<img width="468" height="96" alt="image" src="https://github.com/user-attachments/assets/7c05d24d-542d-44b0-9116-76a40dbdf7a0" />

---

## Part 6 – Restoring ASLR

After completing the experiment, ASLR was re-enabled to restore the system's memory randomization setting.

```bash
echo 1 > /proc/sys/kernel/randomize_va_space
```

Restoring security protections after testing is an important part of maintaining a secure lab environment.

> *ASLR Restored*

<img width="468" height="72" alt="image" src="https://github.com/user-attachments/assets/3b536db9-acc8-4422-90a9-fae6a41dddea" />

---

## Part 7 – Results & Analysis

### Lab Findings

| Test | Result |
|---|---|
| ASLR enabled | Stack pointer addresses changed between executions. |
| ASLR disabled | Stack pointer addresses remained consistent during the test. |
| Normal program input | Program displayed `Done!`. |
| Python payload generation | Generated 116 capital `A` characters. |
| Payload saved to file | Payload was saved in `e1` with a trailing newline. |
| Oversized input | Program crashed with a segmentation fault. |
| ASLR restoration | ASLR was re-enabled after the experiment. |

### Security Analysis

The lab demonstrated the relationship between memory layout randomization and the predictability of stack addresses. Disabling ASLR made the observed stack pointer address consistent across the tested executions.

The vulnerable program illustrated how unsafe memory-handling functions can cause unexpected behavior when supplied with oversized input.

The segmentation fault confirmed a crash, but did not establish that the test achieved arbitrary code execution or control of the instruction pointer.

### Security Recommendations

- Avoid unsafe functions such as `strcpy()` when handling data of unknown length.
- Validate input and check buffer boundaries before copying data.
- Use safer memory-handling functions and appropriate bounds checks.
- Keep ASLR enabled during normal system operation.
- Use compiler protections such as stack canaries and position-independent executables where appropriate.
- Test memory vulnerabilities only in authorized, isolated lab environments.
- Restore security settings after completing experiments.

---

## Results & Conclusion

The lab demonstrated how ASLR changes stack pointer addresses between executions and how disabling address randomization affected the observed memory layout.

A vulnerable C program was compiled and tested with a Python-generated payload containing 116 characters. The oversized input caused a segmentation fault, illustrating the potential consequences of improper memory handling.

Key takeaways:

- Observed the effects of enabling and disabling ASLR.
- Compiled and tested a vulnerable C program.
- Generated and saved a payload using Python.
- Demonstrated a program crash caused by oversized input.
- Identified secure coding practices and memory protections that help reduce buffer overflow risks.

This exercise reinforced the importance of secure programming, input validation, and defense-in-depth protections when developing and operating software.
