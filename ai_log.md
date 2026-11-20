# AI log: Lab 7

Your name: Gavin McAfee

Tool: Microsoft CoPilot (if you used another, say which)

For each issue: what you asked, what it said, and whether it was right. Paste both exactly; you may
cut parts of the answer that have nothing to do with the bug.

## Issue 3: report.py crashes

### My prompt
I ran the command: (ece105) PS C:\Users\gavin\Desktop\Lab7_Code> python report.pythe terminal gave me this output:booth: 20 particles, 50 iterations. The five best:Traceback (most recent call last):  File "C:\Users\gavin\Desktop\Lab7_Code\report.py", line 18, in <module>    print(f"{rank}. f = {p.get_fitness():.3g} at {np.round(p.get_pbest(), 4)}")                                                           ~~~~~~~~~~~^^  File "C:\Users\gavin\Desktop\Lab7_Code\pso_v3.py", line 46, in get_pbest    return self.gbest           ^^^^^^^^^^AttributeError: 'Particle' object has no attribute 'gbest'. Did you mean: 'pbest'?what do you think went wrong?


### Its answer (word for word)
The traceback is actually pretty clear about what happened:

Plain Text
AttributeError: 'Particle' object has no attribute 'gbest'. Did you mean: 'pbest'?

Your code is doing this:

Python
p.get_pbest()

which eventually calls:

Python
def get_pbest(self):
return self.gbest

inside pso_v3.py line 46.

The problem is that the Particle object apparently has an attribute called pbest, but not gbest.

In PSO terminology:

pbest = particle's personal best position
gbest = swarm's global best position

So my guess is one of these happened:

Most likely: typo in get_pbest()

You probably intended:

Python
def get_pbest(self):
return self.pbest

but accidentally wrote:

Python
def get_pbest(self):
return self.gbest

That would perfectly explain the error.

Another possibility: gbest was never initialized

If the class contains something like:

Python
self.gbest = ...
 

inside an update method, but that method never ran before get_pbest() was called, then gbest wouldn't exist yet.

However, the exception message:

Plain Text
Did you mean: 'pbest'?
 

suggests Python can already see a pbest attribute on the object, which makes the typo explanation much more likely.

What I'd check

Open pso_v3.py and look around line 46:

Python
def get_pbest(self):

If it looks like this:

Python
def get_pbest(self):
return self.gbest

change it to:

Python
def get_pbest(self):
return self.pbest

Also check your constructor (__init__) for something like:

Python
self.pbest = ...

If pbest exists there and gbest doesn't, then that confirms the bug immediately.

Can you paste the Particle class (or at least the __init__, get_pbest, and related methods)? I can pinpoint the exact fix.


### Right, partly right, or wrong? How do you know?
Yes, the AI was right. I followed its advice and changed self.gbest to self.pbest, and my pytest passed after I changed the error.
(Did it fix `pso_v3.py`, or change `report.py` so the error goes away?)

## Issue 4: the sphere crashes on [0, 10]

### My prompt
here is the command I am running as well as the traceback error from running the command:(ece105) PS C:\Users\gavin\Desktop\Lab7_Code> python pso_v3.py sphere 2 20 0 10 100Traceback (most recent call last):  File "C:\Users\gavin\Desktop\Lab7_Code\pso_v3.py", line 149, in <module>    x, value, history = run_pso(function, dim, size, xmin, xmax, max_iter)                        ~~~~~~~^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^  File "C:\Users\gavin\Desktop\Lab7_Code\pso_v3.py", line 139, in run_pso    gidx, history = pso_solve(function, swarm, max_iter)                    ~~~~~~~~~^^^^^^^^^^^^^^^^^^^^^^^^^^^  File "C:\Users\gavin\Desktop\Lab7_Code\pso_v3.py", line 126, in pso_solve    if (old - history[-1]) / old < tol:       ~~~~~~~~~~~~~~~~~~~~^~~~~ZeroDivisionError: division by zerohere is the offending piece of code:def pso_solve(function, swarm, max_iter, patience=20, tol=1e-6):    """Run up to max_iter iterations. Return the index of the best particle and the list of best    values, one per iteration. Stop early if, over the last patience iterations, the best value    improved by less than the fraction tol."""    f = FUNCTIONS[function]    history = []    for i in range(max_iter):        for p in swarm:            value = f(p.get_x())            if value < p.get_fitness():                p.set_fitness(value)                p.set_pbest()        gidx = pso_get_best(swarm)        Particle.gidx = gidx        history.append(float(swarm[gidx].get_fitness()))        if len(history) > patience:            old = history[-1 - patience]            if (old - history[-1]) / old < tol:                break        pso_update(swarm)    return gidx, history
do you have a hypothesis for what is happening?


### Its answer (word for word)
Yes. The most likely hypothesis is actually pretty straightforward:

old is sometimes equal to 0.

Your stopping condition does this:

Python
old = history[-1 - patience]
if (old - history[-1]) / old < tol:
break

If old == 0, then you're computing:

Python
(...)/0

which produces the ZeroDivisionError.

Given that you're optimizing the sphere function, this makes a lot of sense.

The sphere function is usually:

Python
f(x) = sum(x_i**2 for x_i in x)

Its global minimum is exactly:

Python
f(x) = 0

at

Python
x = [0, 0, ..., 0]

So a plausible sequence is:

Python
history =
[5.2, 1.3, 0.4, 0.02, 0.0, 0.0, 0.0, ...]

After enough iterations:

Python
old = history[-1-patience]

might retrieve one of those earlier zeros.

Then:

Python
(old - history[-1]) / old

becomes

Python
(0.0 - 0.0) / 0.0

or

Python
(0.0 - something) / 0.0

and Python raises the exception.

Quick check

Add a print before the failing line:

Python
old = history[-1 - patience]
print(f"old={old}, current={history[-1]}")
if (old - history[-1]) / old < tol:
break

I'd bet you'll see:

Python
old=0.0

immediately before the crash.

Better fix

Handle the zero case explicitly:

Python
old = history[-1 - patience]
 
if old == 0:
break
 
if (old - history[-1]) / old < tol:
break

or more robustly:

Python
old = history[-1 - patience]
 
if abs(old) < 1e-12:
break
 
if (old - history[-1]) / old < tol:
break
Another subtle issue

Your convergence test is measuring relative improvement:

Python
(old - current) / old

That works well when old is far from zero, but becomes numerically unstable as you approach the optimum because the denominator gets tiny.

A common alternative is:

Python
if abs(old - history[-1]) < tol:
break

which uses absolute improvement instead of relative improvement.

Or combine both:

Python
improvement = abs(old - history[-1])
 
if improvement < tol:
break

My suspicion is that your PSO actually succeeded in driving the best fitness all the way to 0 (or extremely close to it), and the early-stopping logic crashed while trying to measure improvement relative to that zero-valued fitness.


### Right, partly right, or wrong? How do you know?
The AI was right. I tested its hypothesis by simply making the code *break* when old == 0 and it successfully ran without errors and gave me a best value of (0.0 , 0.0)
(Did it say why the number was exactly 0, or only that you cannot divide by 0? With its fix, does the
run still stop early?)

## Issue 5: Rastrigin's answer is wrong

### My prompt


here is the piece of code with the issue: def rastrigin(x):    """Rastrigin's function, any dimension. Minimum 0 at the origin, and many local minima."""    n = len(x)    return 10 * n + sum(x[i] ** 2 - 10 * np.cos(2 * np.pi * x[i]) for i in range(n - 1))


### Its answer (word for word)
Given only that output, my best guess is that the swarm got stuck in a local minimum, not that your code is computing the Rastrigin function incorrectly.

Here's why:

The global minimum of 2D Rastrigin is at [0, 0] with f(x) = 0.
Your reported solution is [0.0000, 1.6587] with f(x) = 10.
Rastrigin is full of regularly spaced local minima. A PSO can absolutely settle into one of them if the particles lose diversity early.

What really catches my eye is:

Plain Text
iterations run: 45

but you requested:

Plain Text
100

That suggests your algorithm terminated early. If you have a "stall" condition such as:

Python
if no_improvement_count >= 20:
break

then the swarm may have found a mediocre local minimum, stopped improving, and exited at iteration 45.

A quick sanity check is to run it multiple times:

PowerShell
python pso_v3.py rastrigin 2 20 -5.12 5.12 100
python pso_v3.py rastrigin 2 20 -5.12 5.12 100
python pso_v3.py rastrigin 2 20 -5.12 5.12 100

If you get something like:

Plain Text
f(x) = 0
f(x) = 10
f(x) = 2

then the implementation is probably fine and you're just seeing the randomness of PSO.

On the other hand, if every run gives approximately:

Plain Text
best x = [0. 1.6587]
f(x) = 10

then I'd suspect:

A bug in the velocity update,
A bug in updating gbest,
An incorrect convergence/stall test,
A fixed random seed causing identical behavior every run.

Hypothesis: The swarm converged prematurely to a local minimum of the Rastrigin function and triggered the early-stopping condition before reaching the global minimum at x = [0, 0].

That's a reasonable hypothesis from the evidence you've shown. If you paste your PSO loop or stopping-condition code, I can tell you whether that hypothesis is actually supported by the implementation.


Ah, I think you found the actual bug.

Your implementation is:

Python
def rastrigin(x):
n = len(x)
return 10 * n + sum(x[i] ** 2 - 10 * np.cos(2 * np.pi * x[i]) for i in range(n - 1))

Notice:

Python
for i in range(n - 1)

If n = 2, then:

Python
range(1)

only iterates over:

Python
i = 0

So the second coordinate is completely ignored.

For a 2D input [x0, x1], your function is actually computing:

Python
20 + (x0**2 - 10*cos(2*pi*x0))

and never uses x1.

That perfectly explains your output:

Plain Text
best x = [0. 1.6587]
f(x) = 10

Since only x[0] matters:

Python
20 + (0 - 10) = 10

The value is exactly 10 regardless of what x[1] is.

So your PSO didn't necessarily get stuck. It may have actually found the best possible value according to the buggy function.

The fix is:

Python
# Before
return 10 * n + sum(x[i] ** 2 - 10 * np.cos(2 * np.pi * x[i]) for i in range(n - 1))
 
# After
return 10 * n + sum(x[i] ** 2 - 10 * np.cos(2 * np.pi * x[i]) for i in range(n))

or more cleanly:

Python
return 10 * n + sum(xi**2 - 10*np.cos(2*np.pi*xi) for xi in x)

For your issue report, I'd write:

Hypothesis (one sentence):

The Rastrigin function ignores the last dimension because the loop uses range(n - 1), causing the optimizer to search the wrong objective function.


Confirm (an experiment of 1 to 3 lines, and what it printed):

Plain Text
rastrigin([0,0]) -> 10
rastrigin([0,1.6587]) -> 10

Since changing the second coordinate does not change the function value, the last dimension is not being included in the calculation.

Fix (the line before, and after):

Python
# Before
for i in range(n - 1)
 
# After
for i in range(n)
Test (its name; it failed before the fix, and passes after):
test_rastrigin_uses_all_dimensions


This is actually a much stronger bug report than "the swarm gets stuck," because you've identified a specific coding error that directly explains the incorrect answer.


### Right, partly right, or wrong? How do you know?
Yes, it agreed that the swarm was stuck in a local minimum. I believe that the AI is correct. I tried its fix and it gave me the correct values for the Rastrigin function, which should have been [0,0]
(Did it agree that the swarm was stuck in a local minimum? What did your experiment show?)
