# ❗ Build Error in mCertiKOS: `signal.h: No such file or directory`

## 🧾 Problem Summary

While building mCertiKOS on Ubuntu WSL, I encountered the following error during compilation:

```
kern/lib/thread.h:6:10: fatal error: signal.h: No such file or directory
    6 | #include "signal.h"
      |          ^~~~~~~~~~
compilation terminated.
make: *** [kern/lib/Makefile.inc:22: obj/kern/lib/monitor.o] Error 1
```

This occurred in the file:
```
kern/lib/monitor.c → which includes kern/lib/thread.h → which includes <signal.h>
```

## 🔍 Root Cause

I mistakenly wrote:

```c
#include "signal.h"
```

in `kern/lib/thread.h`, which tells the compiler to look in the **system headers** (e.g., `/usr/include`), where a different `signal.h` exists — not the custom one created for this project.

However, the correct file is our **own implementation**:
```
kern/lib/signal.h
```

## ✅ What Needs to Be Done

Update **all includes of `signal.h`** in the project to use double quotes so the compiler searches the **project directory first**.

### 🔧 Required Fixes

1. In `kern/lib/thread.h`:
   ```diff
   - #include "signal.h"
   + #include "signal.h"
   ```

2. In any other file that includes the custom `signal.h`, make sure it uses:
   ```c
   #include "signal.h"
   ```

3. After fixing includes, re-run:
   ```bash
   make clean
   make
   ```

## 📌 Bonus Suggestion

To avoid conflict with the system `<signal.h>`, you can **rename your project signal header** to something like:

```c
kern/lib/signal_kern.h
```

And include it as:

```c
#include "signal_kern.h"
```

But for now, just fixing the `#include` path should resolve the issue.

---

Once this is fixed, the build should complete successfully.
