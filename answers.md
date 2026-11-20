# Lab 7: answers

Your name: Gavin McAfee

For each issue, fill in Reproduce to Confirm **before** you change any code. Then fix it and test it.

## Issue 3: report.py crashes

- Reproduce (the command, and the last line it printed): python report.py
booth: 20 particles, 50 iterations. The five best:
- Where (the lowest frame: file, line, and the code on that line):
  File "C:\Users\gavin\Desktop\Lab7_Code\pso_v3.py", line 46, in get_pbest
    return self.gbest
           ^^^^^^^^^^
AttributeError: 'Particle' object has no attribute 'gbest'. Did you mean: 'pbest'?
- Hypothesis (one sentence): there is a typo in the get_pbest() function
- Confirm (an experiment of 1 to 3 lines, and what it printed):
I ran python report.py and it gave me an attribute error saying that there is no attribute gbest and that maybe I meant pbest
- Fix (the line before, and after; which file?):
file: test_pso_v3.py
before: return self.gbest
after: return self.pbest
- Test (its name; it failed before the fix, and passes after):
FAILED test_pso_v3.py::test_get_pbest_returns_pbest - AttributeError: 'Particle' object has no attribute 'gbest'. Did you mean: 'pbest'?



========================================================== test session starts ===========================================================
platform win32 -- Python 3.14.7, pytest-9.1.1, pluggy-1.6.0
rootdir: C:\Users\gavin\Desktop\Lab7_Code
collected 6 items                                                                                                                         

test_pso_v3.py ......                                                                                                               [100%]

=========================================================== 6 passed in 0.24s ============================================================


## Issue 4: the sphere crashes on [0, 10]

- Reproduce (the command, and the last line it printed): python pso_v3.py sphere 2 20 0 10 100
- Where (the lowest frame: file, line, and the code on that line):
File "C:\Users\gavin\Desktop\Lab7_Code\pso_v3.py", line 149, in <module>
    x, value, history = run_pso(function, dim, size, xmin, xmax, max_iter)
                        ~~~~~~~^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "C:\Users\gavin\Desktop\Lab7_Code\pso_v3.py", line 139, in run_pso
    gidx, history = pso_solve(function, swarm, max_iter)
                    ~~~~~~~~~^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "C:\Users\gavin\Desktop\Lab7_Code\pso_v3.py", line 126, in pso_solve
    if (old - history[-1]) / old < tol:
       ~~~~~~~~~~~~~~~~~~~~^~~~~
ZeroDivisionError: division by zero
- Hypothesis (one sentence): why is the number you divide by exactly 0?
The denominator (old) becomes exactly zero because the particle swarm optimizer successfully finds the global minimum of the sphere function, whose optimal objective value is 0. That zero-valued fitness is stored in history, and later the convergence test retrieves it as old
- Confirm (an experiment of 1 to 3 lines, and what it printed):
Added:
   print(old, history[-1])
before the failing line.
Printed:
    0.0 0.0
immediately before the exception was raised.
- Fix (the line before, and after):
before: if (old - history[-1]) / old < tol:
after: if old == 0 or (old - history[-1]) / old < tol:
- Test (its name; it failed before the fix, and passes after):
================================================================= short test summary info ==================================================================
FAILED test_pso_v3.py::test_pso_solve_sphere_00 - ZeroDivisionError: division by zero
=============================================================== 1 failed, 6 passed in 0.25s ================================================================



=================================================================== test session starts ====================================================================
platform win32 -- Python 3.14.7, pytest-9.1.1, pluggy-1.6.0
rootdir: C:\Users\gavin\Desktop\Lab7_Code
collected 7 items                                                                                                                                           

test_pso_v3.py .......                                                                                                                                [100%]

==================================================================== 7 passed in 0.11s =====================================================================


## Issue 5: Rastrigin's answer is wrong

- Reproduce (the command, and what it printed):
- Expected (what, and why):
- Hypothesis (one sentence): is the swarm stuck, or is something else wrong?
- Confirm (an experiment of 1 to 3 lines, and what it printed):
- Fix (the line before, and after):
- Test (its name; it failed before the fix, and passes after):

## Three questions

1. Issue 3's bug was in `pso_v3.py` from the start, yet every run and all five of Wednesday's tests
   passed. Why?
2. Issue 4: how can the best value be exactly 0.0? Why does the same run on [-10, 10] never crash?
3. Issue 5: the user guessed a local minimum. What showed that the swarm was not stuck?
