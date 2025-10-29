# Setup (once)

```bash
pip install numpy matplotlib scipy
```

---

# Quick readings (15–20 min each, 1-slide summary)

* Raman basics (wavenumber, intensity, peaks).
* Fano line shape (why asymmetric peaks happen).
* Peak vs shoulder (use the first derivative to spot shoulders).
* AIC in one sentence (fit vs simplicity; lower is better).

Deliverable: one slide per topic (picture + 3–5 bullets).

---

# Tiny Python tasks (choose any 2–3 each)

## 1) Fano shape explorer (change q and Γ, see the shape)

```python
import numpy as np, matplotlib.pyplot as plt

def fano(w, I0, q, w0, gamma):
    x = (w - w0)/gamma
    return I0*((1 - (1/q)*x)**2)/(1 + x**2)

w = np.linspace(900, 1100, 800)
I0, w0 = 1.0, 1000.0

for q in [0.5, 1, 2, 10]:
    y = fano(w, I0, q, w0, gamma=20)
    plt.plot(w, y, label=f"q={q}")

plt.xlabel("Wavenumber (cm$^{-1}$)"); plt.ylabel("Intensity")
plt.legend(); plt.title("Fano: effect of q"); plt.show()
```

**What to hand in:** a figure and 3 bullets: how changing **q** flips/changes asymmetry.

---

## 2) Lorentz vs Fano (see the difference)

```python
import numpy as np, matplotlib.pyplot as plt

def lorentz(w, I0, w0, gamma):
    x = (w - w0)/gamma
    return I0/(1 + x**2)

def fano(w, I0, q, w0, gamma):
    x = (w - w0)/gamma
    return I0*((1 - (1/q)*x)**2)/(1 + x**2)

w = np.linspace(900, 1100, 800)
L = lorentz(w, 1.0, 1000.0, 20.0)
F = fano(w, 1.0, q=2.0, w0=1000.0, gamma=20.0)

plt.plot(w, L, label="Lorentz")
plt.plot(w, F, label="Fano (q=2)")
plt.xlabel("Wavenumber (cm$^{-1}$)"); plt.ylabel("Intensity")
plt.legend(); plt.title("Lorentz vs Fano"); plt.show()
```

**Deliverable:** 2 bullets describing how Fano differs when **q** is finite.

---

## 3) Shoulder via derivative (small nearby peak creates a shoulder)

```python
import numpy as np, matplotlib.pyplot as plt

def lorentz(w, I0, w0, g):
    return I0/(1 + ((w - w0)/g)**2)

w = np.linspace(950, 1050, 1200)
y = lorentz(w, 1.0, 1000.0, 10.0) + 0.3*lorentz(w, 1.0, 1018.0, 8.0)
dy = np.gradient(y, w)  # first derivative wrt w

plt.plot(w, y); plt.title("Signal with shoulder"); plt.xlabel("cm$^{-1}$"); plt.ylabel("I"); plt.show()
plt.plot(w, dy); plt.title("First derivative"); plt.xlabel("cm$^{-1}$"); plt.ylabel("dI/dw"); plt.show()
```

**Deliverable:** say where the shoulder sits vs the zero-crossings in the derivative.

---

## 4) AIC toy (choose 1 peak or 2 peaks; which is better?)

```python
import numpy as np, matplotlib.pyplot as plt

rng = np.random.default_rng(0)
def lorentz(w, I0, w0, g):
    return I0/(1 + ((w - w0)/g)**2)

w = np.linspace(950, 1050, 600)
y_true = lorentz(w, 1.0, 1000.0, 10.0) + 0.35*lorentz(w, 1.0, 1018.0, 8.0)
y = y_true + 0.02*rng.standard_normal(w.size)

# crude "fits by eye" (just plug numbers)
y1 = lorentz(w, 1.4, 1004.0, 15.0)                       # 1 peak (k≈3 params)
y2 = lorentz(w, 1.0, 1000.0, 10.0) + 0.35*lorentz(w, 1.0, 1018.0, 8.0)  # 2 peaks (k≈6)

def aic(yobs, yhat, k):
    n = yobs.size
    rss = np.sum((yobs - yhat)**2)
    return 2*k + n*np.log(rss/n)

print("AIC 1-peak:", aic(y, y1, k=3))
print("AIC 2-peak:", aic(y, y2, k=6))

plt.plot(w, y, label="data")
plt.plot(w, y1, label="1-peak guess")
plt.plot(w, y2, label="2-peak guess")
plt.legend(); plt.title("Toy AIC example"); plt.show()
```

**Deliverable:** the two AIC numbers and 1 sentence which model AIC prefers (lower is better).

---

## 5) Baseline intuition (remove slow background)

```python
import numpy as np, matplotlib.pyplot as plt

rng = np.random.default_rng(1)
w = np.linspace(800, 1800, 2000)
baseline = 0.3 + 0.0002*(w - 800) + 0.1*np.sin(2*np.pi*(w-800)/500)
peaks = (1/(1+((w-1200)/8)**2)) + 0.6/(1+((w-1500)/12)**2)
y = baseline + peaks + 0.02*rng.standard_normal(w.size)

# simple moving average baseline (window ~101 points)
k = 101
ker = np.ones(k)/k
y_base = np.convolve(y, ker, mode="same")
y_corr = y - y_base

plt.plot(w, y, label="raw"); plt.plot(w, y_base, label="baseline")
plt.legend(); plt.title("Baseline removal – step 1"); plt.show()

plt.plot(w, y_corr); plt.title("After baseline subtraction"); plt.show()
```

**Deliverable:** 2 bullets about how window size changes smoothness/peak preservation.

---

# What to send back (per person)

* 1–2 reading slides 
* Plots from the 2–3 Python tasks of your choice
* 5–7 bullets: “What I learned / questions I still have.”


