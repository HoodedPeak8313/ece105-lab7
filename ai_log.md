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

### Its answer (word for word)

### Right, partly right, or wrong? How do you know?

(Did it say why the number was exactly 0, or only that you cannot divide by 0? With its fix, does the
run still stop early?)

## Issue 5: Rastrigin's answer is wrong

### My prompt

### Its answer (word for word)

### Right, partly right, or wrong? How do you know?

(Did it agree that the swarm was stuck in a local minimum? What did your experiment show?)
