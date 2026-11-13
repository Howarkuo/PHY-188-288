# PHY-188-288

## 1. Monte carlo and usage in Life Cycle Analysis 
Calculate one estimate per experiment
<img width="2048" height="1423" alt="image" src="https://github.com/user-attachments/assets/60aaa3cb-4a57-4eb1-b51e-d0136babef16" />


## Monte Carlo Estimation of $\pi$

Let $M$ be the number of experiments (`Nreps`) and $N$ be the number of random points in each experiment.

### 1. Define the indicator for each point

For point $j$ in experiment $i$, define the indicator

$$
I_{in} = 
\begin{cases} 
1, & X_{ij}^{2} + Y_{ij}^{2} < 1, \\ 
0, & X_{ij}^{2} + Y_{ij}^{2} \geq 1. 
\end{cases}
$$

### 2. Estimate $\pi$ in each experiment

The Python line

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
## 1.1 MC usage case: Inputs as probability distributions 
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
