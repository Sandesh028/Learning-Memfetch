# Learning-Memfetch
# 🧠 Memfetch Lab — Student Walkthrough

## 🎯 Objective
Learn how to inspect process memory using `memfetch` and understand how data (static, stack, heap) is stored in memory even when not printed.

---

## 🧩 Programs Overview

| Program | Concept | Memory Area | Keyword to Find |
|----------|----------|-------------|-----------------|
| `prog1_static_fixed` | Static variable | `.data` segment | `TOP_SECRET` |
| `prog2_stack_fixed` | Local variable | Stack | `Alice` |
| `prog3_heap_fixed` | Dynamically allocated | Heap | `Confidential` |

---

## ⚙️ Step 1 — Compile the Programs

Run these commands inside your `lcamtuf-memfetch` folder:

```bash
gcc -Wall -O0 -fno-omit-frame-pointer -o prog1_static_fixed prog1_static_fixed.c
gcc -Wall -O0 -fno-omit-frame-pointer -o prog2_stack_fixed  prog2_stack_fixed.c
gcc -Wall -O0 -fno-omit-frame-pointer -o prog3_heap_fixed   prog3_heap_fixed.c
```

> 💡 **Tip:** `-O0` disables optimizations so that your compiler doesn’t remove unused variables.

---

## ▶️ Step 2 — Start the Programs

Run each in the background so they keep running:

```bash
./prog1_static_fixed &
./prog2_stack_fixed &
./prog3_heap_fixed &
```

Verify they’re running:

```bash
ps aux | grep prog
pidof prog1_static_fixed prog2_stack_fixed prog3_heap_fixed
```

You’ll see each program’s **PID** (Process ID).

---

## 🧠 Step 3 — Dump Memory with Memfetch

Use the PID of a program, e.g.:

```bash
sudo ./memfetch <PID>
```

This creates:
- `mfetch.lst` → memory map list
- `mem-XXX.bin` → individual dump files

Example snippet from `mfetch.lst`:

```
[004] mem-004.bin: 0x0000ffff9ce90000-0x0000ffff9d02f000 (1699840 bytes)
```

---

## 🔍 Step 4 — Search for Strings in Memory Dumps

### Static secret (.data)
```bash
strings mem-*.bin | grep -a TOP_SECRET
```

### Stack variable
```bash
strings mem-*.bin | grep -a Alice
```

### Heap secret
```bash
strings mem-*.bin | grep -a Confidential
```

Each keyword should appear in one of the files listed in `mfetch.lst`.

---

## 🧾 Step 5 — Interpret `mfetch.lst`

Use this command:
```bash
cat mfetch.lst
```

Then map each `mem-XXX.bin` file to its address range:

| Memory Region | Description | Typical Address Range |
|----------------|-------------|------------------------|
| `.data` | Global/static variables | 0x0000aaaa… |
| Heap | malloc() allocations | 0x0000ffff9c… |
| Stack | Local variables | 0x0000ffffa5… |

---

## 💡 Step 6 — Why Are Secrets Visible?

Even though the program never printed secrets, data remains in readable memory pages. `memfetch` copies those pages and `strings` extracts ASCII text.  
This demonstrates **data remanence** — sensitive data may remain in memory long after it’s used.

---

## 🧠 Step 7 — Optional Reflection Questions

1. Which `mem-*.bin` contained the static secret?  
2. Which region held the heap message?  
3. Why can we read memory from another process?  
4. How can a developer prevent this leakage?  

---

## ✅ Step 8 — Cleanup

After testing, stop the running programs:

```bash
pkill prog1_static_fixed
pkill prog2_stack_fixed
pkill prog3_heap_fixed
```

Or individually: `kill <PID>`

---

## 📘 Expected Results (Self-check)

| Program | File | Keyword Found | Region |
|----------|------|---------------|--------|
| prog1_static_fixed | mem-002.bin | `TOP_SECRET_KEY_12345` | Static (.data) |
| prog2_stack_fixed | mem-007.bin | `Alice` | Stack |
| prog3_heap_fixed | mem-004.bin | `Confidential message stored in heap!` | Heap |

---

## 🧰 Tips for Success
- Run `memfetch` with sudo for permission to read other process memory.  
- Compile with `-O0` to keep variables in memory.  
- Ensure process is running while dumping.  
- Use `grep -a` to find ASCII strings inside binary data.

---

## 🔐 Security Mitigation Discussion
**How to prevent leaks:**
- Zero memory after use (`memset_s`, `explicit_bzero`).  
- Avoid hardcoded secrets in binaries.  
- Use `mlock()` to keep sensitive pages from being swapped.  
- Use hardware-backed key stores or secure enclaves.  

---
