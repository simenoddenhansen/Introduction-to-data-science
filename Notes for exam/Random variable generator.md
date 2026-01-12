# Exam vB, PROBLEM 2

**Maximum Points = 8**

## Random variable generation and transformation

The purpose of this problem is to show that you can implement your own sampler.  
This will be built in the following three steps:

1. **[2p]** Implement a Linear Congruential Generator where you tested out a good combination  
   (a large \( M \) with \( a, b \) satisfying the Hull–Dobell (Thm 6.8)) of parameters.  
   Follow the instructions in the code block.

2. **[2p]** Using a generator, construct random numbers from the uniform \([0, 1]\) distribution.

3. **[4p]** Using a uniform \([0, 1]\) random generator, generate samples from

   \[
   p_0(x) = \frac{\pi}{2} \lvert \sin(2\pi x) \rvert, \qquad x \in [0,1].
   \]

Using the **Accept–Reject sampler** (Algorithm 1 in TFDS notes) with sampling density given by the uniform \([0,1]\) distribution.


# Random Variable Generation and Transformation: A Complete Guide

This problem asks you to build a random number generation pipeline from scratch. You start with a raw mathematical generator (LCG), refine it into a standard Uniform $[0,1]$ generator, and finally use that to sample from a complex custom distribution using Rejection Sampling.

## 1. How to Think About This Problem

### Part 1: The Engine (Linear Congruential Generator)
Computers are deterministic; they cannot generate "true" randomness. Instead, we use **Pseudo-Random Number Generators (PRNGs)**. The Linear Congruential Generator (LCG) is one of the oldest and simplest algorithms.

* **The Logic:** It uses a recurrence relation. You take the last number, multiply it by a huge number, add another number, and take the remainder (modulus). This "scrambles" the bits predictably but chaotically enough to look random.
* **The Formula:** $X_{n+1} = (aX_n + c) \mod m$
* **Key Insight:** The "quality" of the randomness depends entirely on the choice of $a$, $c$, and $m$. The parameters provided in your problem satisfy the **Hull-Dobell Theorem**, which guarantees the generator will cycle through every possible number before repeating.

### Part 2: Standardization (Uniform Distribution)
The LCG produces integers between $0$ and $m-1$. Most statistical formulas, however, expect a probability $U \in [0, 1]$.

* **The Logic:** To map the integers $0, \dots, m-1$ to the continuous range $[0, 1]$, you simply divide by the modulus (or period).
* **The Transformation:** $U_i = \frac{X_i}{m}$

### Part 3: Sculpting the Distribution (Accept-Reject Sampling)
Now you have a source of Uniform noise ($U$), but you need samples from a specific "wavy" distribution $p_0(x) = \frac{\pi}{2}|\sin(2\pi x)|$.

* **The Logic:** Imagine throwing darts at a rectangular board that encloses your desired function curve.
    1. Pick a random horizontal spot (Proposal $X$).
    2. Pick a random vertical height (Check $U$).
    3. If the dart lands **under** the curve ($U \leq p_0(X)$), you keep (accept) the $X$.
    4. If it lands **above**, you throw it away (reject).
* **The Math:** We need a constant $M$ such that our target density $f(x) \leq M \cdot g(x)$, where $g(x)$ is our proposal (Uniform, so $g(x)=1$).
    * Max value of $|\sin(2\pi x)|$ is 1.
    * Max value of $p_0(x)$ is $\frac{\pi}{2}$.
    * Therefore, $M = \frac{\pi}{2}$.
    * **Acceptance Condition:** Accept $x$ if $u < \frac{f(x)}{M \cdot g(x)}$.
    * Simplifying: $u < \frac{\frac{\pi}{2}|\sin(2\pi x)|}{\frac{\pi}{2} \cdot 1} \implies u < |\sin(2\pi x)|$.

## 2. Essential Online Sources

* **Linear Congruential Generators:**
    * [Wikipedia: Linear Congruential Generator](https://en.wikipedia.org/wiki/Linear_congruential_generator) - Excellent overview of the recurrence relation and parameters.
    * [Computer Science Guide: Random Number Generation](https://www.geeksforgeeks.org/linear-congruential-method-for-generating-pseudo-random-numbers/) - A simpler breakdown of the LCG code logic.

* **Rejection Sampling (Accept-Reject):**
    * [Columbia University: Rejection Sampling Intuition](http://stat.columbia.edu/~blei/fogm/2016/lectures/rejection_sampling.pdf) - A clear visual guide on "throwing darts" under the curve.
    * [Introduction to Probability (DataCamp)](https://www.datacamp.com/tutorial/probability-distributions-python) - Python-centric guide to distributions.

## 3. Python Implementation

Here is the code to fill in the `XXX` sections of your screenshots.

### Step 1: `problem2_LCG`
Implementation of the recurrence relation.

```python
def problem2_LCG(size=None, seed=0):
    """
    A linear congruential generator that generates pseudo random numbers according to size.
    """
    m = 2**31
    a = 1103515245
    c = 12345

    out = []
    x = seed  # Start with the seed (u0)
    
    for _ in range(size):
        # Apply the LCG formula: X_{n+1} = (a * X_n + c) % m
        x = (a * x + c) % m
        out.append(x)

    return out
```

### Step 2: `problem2_uniform`
Normalization of the integer output to the $[0, 1]$ interval.

```python
def problem2_uniform(generator=None, period=1, size=None, seed=0):
    """
    Takes a generator and produces samples from the uniform [0,1] distribution.
    """
    # 1. Get the raw integers from the provided LCG generator
    raw_integers = generator(size=size, seed=seed)
    
    # 2. Transform integers to [0, 1] by dividing by the period (m)
    out = [x / period for x in raw_integers]
    
    return out
```

### Step 3: `problem2_accept_reject`
Implementation of the Rejection Sampling algorithm.

*Note: Because we reject some samples, we need to generate more candidate numbers than the requested size. A common safe multiplier is 3x or 4x the requested size.*

```python
import numpy as np

def problem2_accept_reject(uniformGenerator=None, size=None, seed=0):
    """
    Produces samples from (pi/2)*abs(sin(x*2*pi)) using Accept-Reject.
    """
    out = []
    current_seed = seed
    
    # We loop until we have collected enough samples ('size')
    while len(out) < size:
        # We need pairs of numbers: one for the candidate X, one for the vertical check U.
        # We generate a batch to be efficient. 
        # Safety buffer: Generate 4x what we still need to ensure we get enough.
        needed = size - len(out)
        batch_size = needed * 4
        
        # Generate random numbers from the uniform generator
        # We assume uniformGenerator returns values in [0, 1]
        random_batch = uniformGenerator(size=batch_size * 2, seed=current_seed)
        
        # Update seed for next iteration to ensure different numbers if we loop again
        current_seed += 1 
        
        # Process the batch in pairs
        # We take steps of 2: index i is Candidate X, index i+1 is Check Value U
        for i in range(0, len(random_batch) - 1, 2):
            if len(out) >= size:
                break
            
            x_candidate = random_batch[i]
            u_check = random_batch[i+1]
            
            # Acceptance Condition:
            # u < f(x) / (M * g(x))
            # Here: u < |sin(2 * pi * x)|
            if u_check <= np.abs(np.sin(2 * np.pi * x_candidate)):
                out.append(x_candidate)
                
    return out
```