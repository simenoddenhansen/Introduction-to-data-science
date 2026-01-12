# Probability Warmup: Problem Solving Guide

To ensure you can solve this probability problem in Python, I have curated a list of authoritative online sources. These cover the specific library functions you need (`scipy.stats`) and the probability theory required to handle the dependency between $N$ (known answers) and $Z$ (guessed answers).

## 1. Essential Online Sources

### For the Code Implementation (SciPy)

* **SciPy binom Documentation:** This is the primary reference for the `pmf` (Probability Mass Function) and `cdf` (Cumulative Distribution Function) methods. You will use `binom.pmf(k, n, p)` to calculate probabilities for both $N$ and $Z$.
* **NumPy sum and Array Basics:** You will likely need to sum over arrays to calculate the total probabilities for the denominators and numerators in your conditional formula.

### For the Mathematical Logic

* **Data 140: The Binomial Distribution:** This textbook resource explains the binomial distribution clearly, including how independent trials sum up. It is excellent for understanding why $Y$ (total score) is a sum of random variables.
* **Conditional Probability (DataCamp):** This tutorial reviews discrete probability distributions and the logic of $P(A|B)$, which is the core of your problem (calculating the probability of $N$ given a condition on $Y$).

## 2. How to Apply These Sources to Your Problem

The challenge in your problem is that $Z$ (guesses) depends on $N$ (knowns). You cannot simply add two independent binomials. Here is the logic you should implement in Python:

### Step 1: Understand the Joint Probability

The total score is $Y = N + Z$. The probability of a specific student outcome depends on both how many they knew ($N=n$) and how many they guessed ($Z=z$).

$$P(N=n, Y=y) = P(Y=y | N=n) \cdot P(N=n)$$

* **$P(N=n)$:** Use `binom.pmf(n, 20, 11/20)`.
* **$P(Y=y | N=n)$:** If a student knows $n$ answers, they must guess on the remaining $20-n$ questions. To get a total score of $y$, they need exactly $y-n$ correct guesses.
* **Formula:** `binom.pmf(y-n, 20-n, 0.5)`.

### Step 2: Construct a Joint Probability Matrix

Instead of trying to find a closed-form equation, use Python to build a table (a 2D NumPy array or nested loops):

1. Create a matrix where rows are $n$ (0 to 20) and columns are $y$ (0 to 20).
2. Fill each cell $(n, y)$ with the probability calculated in Step 1.
3. **Note:** If $y < n$, the probability is 0 (you cannot have a total score lower than the number of answers you already know).

### Step 3: Solve for the Thresholds

Once you have the matrix of probabilities $P(N=n, Y=y)$, you can solve Part 1 and Part 2 by summing specific regions of this matrix.

**For Part 1:** $P(N < 10 | Y \ge T)$

Use the conditional probability formula:

$$P(N < 10 | Y \ge T) = \frac{P(N < 10 \cap Y \ge T)}{P(Y \ge T)}$$

* **Numerator:** Sum the probabilities in your matrix where row index $< 10$ and column index $\ge T$.
* **Denominator:** Sum the probabilities in your matrix where column index $\ge T$ (regardless of $n$).

**For Part 2**

Loop through possible $T$ values and find the first one where $P(N \ge 10 | Y \ge T) \ge 0.90$.

* **Hint:** $P(N \ge 10 | Y \ge T) = 1 - P(N < 10 | Y \ge T)$.

## 3. Python Snippet Structure

```python
import numpy as np
from scipy.stats import binom

# Parameters
n_questions = 20
p_know = 11/20
p_guess = 0.5

# 1. Build Joint Distribution Matrix P(N=n, Y=y)
joint_prob = np.zeros((21, 21))

for n in range(21):
    prob_n = binom.pmf(n, n_questions, p_know)
    
    for y in range(n, 21): # y must be at least n
        # Correct guesses needed: (y - n)
        # Questions remaining to guess: (20 - n)
        prob_y_given_n = binom.pmf(y - n, n_questions - n, p_guess)
        
        joint_prob[n, y] = prob_n * prob_y_given_n

# 2. Calculate Conditional Probabilities for each T
problem11_probabilities = []
for T in range(21):
    # Total probability of passing: P(Y >= T)
    prob_pass = np.sum(joint_prob[:, T:])
    
    # Probability of being "unqualified" and passing: P(N < 10 AND Y >= T)
    prob_unqualified_pass = np.sum(joint_prob[:10, T:])
    
    if prob_pass > 0:
        val = prob_unqualified_pass / prob_pass
    else:
        val = 0 
    
    problem11_probabilities.append(val)

# 3. Solve Part 2: Smallest T where P(N >= 10 | Y >= T) >= 0.90
problem12_T = None
for T in range(21):
    prob_n_ge_10_given_pass = 1 - problem11_probabilities[T]
    if prob_n_ge_10_given_pass >= 0.90:
        problem12_T = T
        break