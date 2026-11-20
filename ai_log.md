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

### Its answer (word for word)

### Right, partly right, or wrong? How do you know?

(Did it agree that the swarm was stuck in a local minimum? What did your experiment show?)
