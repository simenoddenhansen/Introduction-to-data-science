
# Q: Give me the rundown of an LCG and implement it using python and the Hull Dobell theorem
Linear Congruential Generators (LCG)1. The Purpose of LCGA Linear Congruential Generator is an algorithm that yields a sequence of pseudo-random numbers. Because computers are deterministic, they cannot generate truly random numbers without specialized hardware; instead, they use LCGs to simulate randomness for statistical experiments and simulations.The goal is to produce a sequence that is indistinguishable from a Uniform(0, 1) distribution. These uniform samples are the "raw material" used to generate more complex random variables (like Normal or Exponential) through the Inverse Transform Method.2. Mathematical DefinitionThe LCG is defined by the recursive formula:$$X_{n+1} = (aX_n + c) \mod m$$Where:$m$ is the modulus ($m > 0$).$a$ is the multiplier ($0 \le a < m$).$c$ is the increment ($0 \le c < m$).$X_0$ is the seed (the starting value).To get a number in the range $[0, 1)$, we calculate $U_n = \frac{X_n}{m}$.3. The Hull-Dobell Theorem (Theorem 6.8)To be considered a "good" generator, the LCG must have a full period. This means it should generate all $m$ possible values before the sequence repeats. According to the Hull-Dobell Theorem, an LCG has a full period $m$ if and only if:$c$ and $m$ are relatively prime (their only common divisor is 1).Every prime factor of $m$ divides $(a - 1)$.If 4 divides $m$, then 4 must also divide $(a - 1)$.4. Python ImplementationThe following code uses parameters ($m=2^{31}, a=1103515245, c=12345$) that satisfy the Hull-Dobell conditions.Pythonimport time

class LCG:
    """
    Implementation of a Linear Congruential Generator.
    Satisfies Hull-Dobell Theorem for a full period m.
    """
    def __init__(self, seed=None):
        # Parameters chosen for a full period m=2^31:
        self.m = 2**31
        self.a = 1103515245
        self.c = 12345
        
        # Hull-Dobell Check:
        # 1. c (12345) is odd, m (2^31) is a power of 2. They are relatively prime.
        # 2. Only prime factor of m is 2. (a-1) = 1103515244 is even (divisible by 2).
        # 3. m is divisible by 4. (a-1) = 1103515244 is also divisible by 4.
        
        if seed is None:
            self.state = int(time.time() * 1000) % self.m
        else:
            self.state = seed % self.m

    def next_int(self):
        """Update the state and return the next integer."""
        self.state = (self.a * self.state + self.c) % self.m
        return self.state

    def next_uniform(self):
        """Return the next pseudo-random number in [0, 1)."""
        return self.next_int() / self.m

# --- Testing the implementation ---
if __name__ == "__main__":
    rng = LCG(seed=123)
    print("First 5 samples from Uniform(0, 1):")
    for _ in range(5):
        print(f"{rng.next_uniform():.6f}")









# Q: Give me a short explanation of the point of LCG. Take into account the readings in order to explain it:

A **Linear Congruential Generator (LCG)** is a widely used algorithm designed to produce a sequence of numbers that behave like a random sample from a  distribution. While computers are inherently deterministic and cannot generate "true" randomness without external hardware, LCGs provide **pseudo-random** numbers that are sufficient for most statistical simulations.

The "point" or primary utility of an LCG can be broken down into three main functions:

### 1. Simulating Randomness Deterministically

The LCG uses a recursive mathematical formula to generate the next number in a sequence based on the current one:


* By choosing specific values for the multiplier (), the increment (), and the modulus (), the sequence appears to be random to an observer.


* This allows statisticians to mimic the behavior of random variables when an analytical solution is too complex.



### 2. Reproducibility via "Seeds"

One of the most important aspects of LCGs in research is that they are **reproducible**.

* The sequence is entirely determined by the initial value, called the **seed** ().


* If you share your seed and your LCG parameters with another researcher, they will generate the exact same "random" numbers, allowing them to verify your simulation results.



### 3. Foundation for Complex Distributions

The LCG typically produces integers between  and . By dividing these by , we get values  that are approximately .

* Once you have these uniform values, you can use techniques like the **Probability Integral Transform** to convert them into any other distribution (such as Normal, Exponential, or Binomial).


* Essentially, the LCG is the "engine" that powers almost all other random variable generation in statistical software.



---

Would you like me to show you how to use these uniform numbers to generate a different distribution, like an **Exponential** or **Normal** distribution?






# Q: Implement a Linear Congruential Generator where you tested out a good combination (a large $M$ with $a,b$ satisfying the Hull-Dobell (Thm 6.8)) of parameters. Follow the instructions in the code block.Do this in python
To implement a reliable Linear Congruential Generator (LCG), we must choose parameters that satisfy the **Hull-Dobell Theorem** to ensure the generator has a **full period** (it hits every integer between  and  before repeating).

### The Hull-Dobell Theorem (Thm 6.8)

According to the provided lecture notes, an LCG of the form  has a full period  if and only if:

1. c and m are relatively prime (their only common divisor is 1).


2. Every prime factor of m divides .


3. If 4 divides m, then 4 also divides .



### Implementation in Python

The following implementation uses parameters often found in common libraries (like the glibc generator), which are specifically chosen to satisfy these conditions with a large .

```python
import time

class LCG:
    def __init__(self, seed=None):
        # Parameters satisfying Hull-Dobell for a full period m:
        # m is a power of 2 (2^31), a-1 is a multiple of 4, c is odd.
        self.m = 2**31
        self.a = 1103515245
        self.c = 12345
        
        # Initialize seed with current time if not provided
        if seed is None:
            self.state = int(time.time()) % self.m
        else:
            self.state = seed % self.m

    def next_int(self):
        """Generates the next random integer in the sequence."""
        self.state = (self.a * self.state + self.c) % self.m
        return self.state

    def next_float(self):
        """Generates a pseudo-random float in the range [0, 1)."""
        # Dividing by m maps the integer sequence to the Uniform(0, 1) distribution
        return self.next_int() / self.m

# --- Testing the Implementation ---
# 1. Initialize the generator with a seed
generator = LCG(seed=42)

# 2. Generate a sequence of Uniform(0, 1) samples
print("First 5 pseudo-random numbers in [0, 1):")
for _ in range(5):
    print(f"{generator.next_float():.6f}")

# 3. Validation against Hull-Dobell conditions:
# Condition 1: c (12345) and m (2^31) are relatively prime (12345 is odd).
# Condition 2: The only prime factor of m is 2. (a-1) is 1103515244, which is even.
# Condition 3: m is divisible by 4. (a-1) is also divisible by 4 (1103515244 / 4 = 275878811).

```

### Why this is useful

The "point" of generating these uniform values is that they serve as the raw material for simulating more complex distributions. By applying the **Probability Integral Transform** (Inverse Transform Method), you can take these  values and transform them into samples from a Normal, Exponential, or any other distribution required for statistical analysis.

Would you like me to extend this code to generate samples for a specific distribution, such as the **Exponential** distribution?