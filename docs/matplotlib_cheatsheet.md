# Matplotlib ML Cheat Sheet

```python
import matplotlib.pyplot as plt
import numpy as np
```

## Basic figure

```python
fig, ax = plt.subplots()   # fig is the canvas, ax is the plot area

ax.plot([1, 2, 3], [2, 4, 1])
plt.show()
```

Prefer the object-oriented `fig, ax` style over the stateful `plt.plot()` calls: it makes multiple plots and shared axes explicit.

```python
fig, ax = plt.subplots(figsize=(8, 4), dpi=120)
```

## Line plot

```python
ax.plot(x, y)                       # default line
ax.plot(x, y, color="tab:blue", linewidth=2)
ax.plot(x, y, linestyle="--", marker="o", markersize=4)
ax.plot(x, y, label="train")
```

Common markers: `"o"` circle, `"s"` square, `"^"` triangle, `"."` point. Line styles: `"-"`, `"--"`, `"-."`, `":"`.

## Scatter plot

```python
ax.scatter(x, y, s=20, alpha=0.6)
ax.scatter(x, y, c=labels, cmap="viridis")   # color by a third variable
ax.scatter(x, y, c="tab:orange", edgecolor="black")
```

`s` is the marker size, `alpha` the transparency (useful for overlapping points).

## Bar chart

```python
ax.bar(categories, values)
ax.barh(categories, values)          # horizontal bars
ax.bar(categories, values, color="tab:green", width=0.6)
```

## Histogram

```python
ax.hist(values, bins=30)
ax.hist(values, bins=30, density=True, alpha=0.7)  # density instead of counts
```

## Box plot

```python
ax.boxplot([group_a, group_b], labels=["A", "B"])
```

## Labels, titles, limits

```python
ax.set_title("Survival rate by sex")
ax.set_xlabel("Sex")
ax.set_ylabel("Survival rate")

ax.set_xlim(0, 10)
ax.set_ylim(0, 1)

ax.set_xticks([0, 1, 2])
ax.set_xticklabels(["low", "mid", "high"], rotation=45)
```

## Legend and grid

```python
ax.legend(loc="upper right")   # uses the label= arguments
ax.grid(True, alpha=0.3)
```

## Subplots

```python
fig, axes = plt.subplots(1, 2, figsize=(10, 4))
axes[0].plot(x, y)
axes[1].hist(values)
fig.tight_layout()
```

Shared axes:

```python
fig, axes = plt.subplots(2, 1, sharex=True)
```

## Multiple series on one axes

```python
fig, ax = plt.subplots()
ax.plot(x, train_loss, label="train")
ax.plot(x, val_loss, label="val")
ax.set_ylabel("loss")
ax.legend()
```

## Pandas integration

A Series or DataFrame can be passed straight to matplotlib:

```python
ax.plot(df["date"], df["sales"])
ax.scatter(df["age"], df["fare"], alpha=0.5)
ax.hist(df["age"].dropna(), bins=20)

df["survived"].value_counts().plot(kind="bar", ax=ax)
```

## Save a figure

```python
fig.savefig("plot.png", dpi=150, bbox_inches="tight")
fig.savefig("plot.pdf", bbox_inches="tight")   # vector format
```

Call `savefig` before `plt.show()`; after `show()` the figure may be cleared.

## Current axes shortcuts

When working in a notebook with a single plot, the stateful API is shorter:

```python
plt.plot(x, y)
plt.title("Title")
plt.xlabel("x")
plt.ylabel("y")
plt.show()
```

Mixing the two styles is fine, but do not call `plt.title()` after creating a `fig, ax` pair when you mean `ax.set_title()`.

## Style

```python
plt.style.use("seaborn-v0_8-whitegrid")   # list options: plt.style.available
```

Set a consistent default figure size:

```python
plt.rcParams["figure.figsize"] = (8, 4)
plt.rcParams["axes.grid"] = True
```
