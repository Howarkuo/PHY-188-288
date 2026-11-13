# PHY-188-288

## 1. Monte carlo and usage in Life Cycle Analysis 
<img width="1473" height="866" alt="Screenshot 2026-11-12 at 8 53 16 PM" src="https://github.com/user-attachments/assets/828f5bb4-bc2b-4034-84fb-019388d1b2fc" />

Calculate one estimate per experiment
<img width="2048" height="1423" alt="image" src="https://github.com/user-attachments/assets/60aaa3cb-4a57-4eb1-b51e-d0136babef16" />


## Monte Carlo Estimation of $\pi$

Let $M$ be the number of experiments (`Nreps`) and $N$ be the number of random points in each experiment.
### 1. Write the quarter-circle area as an integral

Let $i$ index an experiment and $j$ index a point within that experiment.
In each experiment, draw $N$ independent points $(X_{ij}, Y_{ij})$
uniformly from the unit square $[0,1]\times[0,1]$.

Define the indicator function

$$
f(x,y) =
\begin{cases}
1, & x^2+y^2<1,\\
0, & x^2+y^2\geq 1.
\end{cases}
$$

Inside the quarter circle, $f(x,y)=1$, so integrating $f$ counts the
area of that region. Outside the quarter circle, $f(x,y)=0$, so that
region contributes zero. For a given $x$, the points inside the circle
have $0\leq y<\sqrt{1-x^2}$. Therefore,

$$
A = \int_0^1\int_0^1 f(x,y)\,dy\,dx
  = \int_0^1 \sqrt{1-x^2}\,dx
  = \frac{\pi}{4}.
$$

The unit square has area $1$. A uniform random point falls inside the
quarter circle with probability $A/1=A$. Since the indicator is $1$ for
a hit and $0$ otherwise, its expected value is also $A$.

### Simple form: indicator 

For point $j$ in experiment $i$, define the indicator

$$
I_{in} = 
\begin{cases} 
1, & X_{ij}^{2} + Y_{ij}^{2} < 1, \\ 
0, & X_{ij}^{2} + Y_{ij}^{2} \geq 1. 
\end{cases}
$$
### 2. Approximate the integral with random points

For point $j$ in experiment $i$, write $I_{ij}=f(X_{ij},Y_{ij})$.
Monte Carlo integration replaces the expected value with the average
of the $N$ sampled indicator values:

$$
\underbrace{\int_0^1\int_0^1 f(x,y)\,dy\,dx}_{\text{quarter-circle area}}
\approx
\underbrace{\frac{1}{N}\sum_{j=1}^N I_{ij}}_{\text{fraction of sampled points inside}}.
$$

The area factor is $1$ because the sampling domain is the unit square.
For a uniform sample over a rectangle of area $S$, multiply the sample
average by $S$ to estimate the integral over that rectangle.

| Python expression | Mathematical operation |
|---|---|
| `np.random.uniform(0, 1, (N, 1))` | Draw $N$ values for one coordinate. Two calls produce the $X$ and $Y$ coordinates. |
| `R = X**2 + Y**2` | Compute $X_{ij}^2+Y_{ij}^2$ for each point. |
| `R < 1` | Evaluate the $N$ indicators $I_{ij}$. |
| `(R < 1).sum()` | Count the hits, $\sum_{j=1}^N I_{ij}$. |
| `(R < 1).sum() / N` | Estimate the integral, $\widehat A_i$. |
| `4 * (R < 1).sum() / N` | Convert the area estimate into $\widehat{\pi}_i=4\widehat A_i$. |

For example, if $39$ out of $50$ points land inside the quarter circle,
the estimated integral is $39/50=0.78$. Multiplying by $4$ gives
$\widehat{\pi}_i=3.12$.

```python
import numpy as np


def CalcPi(N):
    X = np.random.uniform(0, 1, (N, 1))
    Y = np.random.uniform(0, 1, (N, 1))
    R = X**2 + Y**2

    area_estimate = (R < 1).sum() / N
    return 4 * area_estimate


def CalcPi2(N, Nreps=1000):
    PI = np.zeros(Nreps)

    for i in range(Nreps):
        PI[i] = CalcPi(N)

    return np.mean(PI), np.std(PI)
```

Each call to `CalcPi(N)` draws $N$ fresh points and approximates the
integral once. After `Nreps` calls, `PI` contains $M=\texttt{Nreps}$
estimates of $\pi$. Python stores these in slots `0` through `Nreps - 1`.


### Simple form


```python
PI[i] = 4 * (R < 1).sum() / N
```

implements the equation

$$
\widehat{\pi}_i = \frac{4}{N}\sum_{j=1}^{N} I_{ij}.
$$

The sum counts the points inside the quarter circle in experiment $i$. The array `PI = np.zeros(Nreps)` starts as $(0,\ldots,0)$. After the loop, it holds $(\widehat{\pi}_1,\ldots,\widehat{\pi}_M)$. The equations number experiments from $1$ to $M$, and Python indexes the corresponding array slots from `0` to `M - 1`.


### 3. Calculate the mean and standard deviation across experiments

`np.mean(PI)` calculates the mean of the $M$ estimates:

$$
\overline{\pi} = \frac{1}{M}\sum_{i=1}^{M}\widehat{\pi}_i.
$$

`np.std(PI)` calculates their standard deviation, using $M$ in the denominator by default:

$$
s = \sqrt{\frac{1}{M}\sum_{i=1}^{M}
\left(\widehat{\pi}_i - \overline{\pi}\right)^2}.
$$

For the three estimates $3.12$, $3.28$, and $2.96$,

$$
\overline{\pi} = \frac{3.12 + 3.28 + 2.96}{3} = 3.12,
$$

$$
s = \sqrt{\frac{0^2 + 0.16^2 + (-0.16)^2}{3}}
\approx 0.131.
$$
```bash
def CalcPi(N):
    X = np.random.uniform(0,1,(N,1))
    Y = np.random.uniform(0,1,(N,1))
    R = X**2 + Y**2
    PI = 4*(R< 1).sum()/N
Nreps = 1000
PI = np.zeros(Nreps)

for i in range(Nreps):
    # Generate N fresh points and calculate their squared distances.
    PI[i] = 4 * (R < 1).sum() / N

return np.mean(PI), np.std(PI)

p = np.pi / 4
e_theo = 4 * np.sqrt(p * (1 - p) / np.array(N))

```
## 1.1 Why the error in estimation shrinks?
![Uploading Screenshot 2026-11-12 at 8.52.54 PM.png…]()
<img width="1279" height="724" alt="Screenshot 2026-11-12 at 8 51 31 PM" src="https://github.com/user-attachments/assets/e1dffa27-3d9e-4a19-ab54-3b6788cced47" />

<img width="2048" height="1423" alt="image" src="https://github.com/user-attachments/assets/6b6ee4fd-df9c-4566-a2f8-76f1a6a51681" />

```
| Eq element |  Python code | Meaning |
|---|---|---|
| \(N_{\text{tot}}\) | `N` | Points per simulation |
| \(N_{\text{section}}\) | `(R < 1).sum()` | Hits in one simulation |
| Black dots | `np.mean(PI)` | Average estimate across repeated simulations |
| Black error bars | `np.std(PI)` | Observed run-to-run spread at that \(N\) |
| Red curves | `±4 * 0.4 / sqrt(N)` | Approximate theoretical spread of one estimate |
```
Increasing repetitions makes the measured spread more stable. Increasing \(N\) makes each individual estimate more precise.

## 1.2 MC usage case: Inputs as probability distributions 
. LCA uses Monte Carlo to see how uncertainty in the inputs affects uncertainty in the final environmental impact. Each uncertain input can be represented by a probability distribution, which describes the values it could take and how likely different ranges are. Inputs with known values can stay fixed.

Pseudocode for one crystallization route
Assume the functional unit is 1 kg of dried acetaminophen at the specified purity. The process model converts each draw into material, energy, and waste quantities per kg of product.



```
M = 10,000
impact = zeros(M)

FOR i = 0 TO M - 1:
    # Sample uncertain process inputs and emission factors
    yield, solvent_recovery = draw_process_inputs()
    EF_feed, EF_solvent, EF_power, EF_waste = draw_emission_factors()

    # Recalculate the inventory per kg of dried product
    feed_kg, fresh_solvent_kg, electricity_kWh, waste_kg =
        process_model(yield, solvent_recovery)

    # Calculate and store this run's footprint
    impact[i] = feed_kg          * EF_feed
              + fresh_solvent_kg * EF_solvent
              + electricity_kWh  * EF_power
              + waste_kg         * EF_waste
RETURN mean(impact),
       std(impact),
       percentile(impact, [2.5, 97.5])
FOR i = 0 TO M - 1:
    shared_grid_EF = draw_grid_emission_factor()

    impact_A[i] = calculate_footprint(route_A, shared_grid_EF)
    impact_B[i] = calculate_footprint(route_B, shared_grid_EF)
    difference[i] = impact_B[i] - impact_A[i]
fraction_A_lower = count(difference > 0) / M
```


```
| Output | Mean | Standard deviation | 95% simulation interval |
|---|---:|---:|---:|
| Route A | 6.06 | 0.75 | 4.74 to 7.62 |
| Route B | 5.29 | 0.58 | 4.24 to 6.49 |
| Paired difference \(B-A\) | −0.78 | 0.84 | −2.48 to +0.78 |
```
```
| Process input | Route A | Route B |
|---|---|---|
| Crystallization yield | 75%, 85%, 95% | 80%, 90%, 97% |
| Solvent recovery | 65%, 80%, 90% | 78%, 90%, 97% |
| Electricity per kg of feed | 1.0, 1.4, 1.8 kWh | 2.0, 2.7, 3.4 kWh |
```

<img width="1920" height="1040" alt="image" src="https://github.com/user-attachments/assets/9007af17-12cb-4996-ae80-b1f0817481ed" />

