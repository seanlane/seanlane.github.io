---
layout: post
title: Dynamically updating Matplotlib figures in Jupyter notebooks
commentIssueId: 8
tags: python, matplotlib, jupyter, dynamic plots
---

Updating matplotlib figures dynamically seems to be a bit of a hassle, but the code below seems to do the trick. This is an example that outputs a figure with multiple subplots, each with multiple plots. Oddly enough, at the time of writing the image will be smaller than the figure until the Jupyter cells stops running, but this can be fixed but generating the figure in one cell, and then updating the image in a subsequent cell [^1].

This code is run with the assumption that the following data file can be found in the working directory named `data.txt`:

[Sample TCL data](/files/2018/sample-tcl-data.txt)

```python
import numpy as np
import time
import matplotlib.pyplot as plt

%matplotlib notebook

def load_data():
    data = np.genfromtxt('data.txt', delimiter=',', skip_header=1)
    tm = data[:, 0]
    Q1 = data[:, 1]
    Q2 = data[:, 2]
    T1 = data[:, 3]
    T2 = data[:, 4]
    return (tm, Q1, Q2, T1, T2) 

(m_time, Q1s, Q2s, T1s, T2s) = load_data()

n = len(m_time)

labels = [
    [r'$T_1$ measured', r'$T_2$ measured'],
    # [r'$T_1 set point$', r'$T_2 set point$'],
    [r'$Q_1$', r'$Q_2$']
]

colors = [
    ['r:', 'b-'],
    ['r:', 'bx']
]

def plot_init(num_subplots=1, x_labels=None, y_labels=None):
    if not x_labels:
        x_labels = [None] * num_subplots
    if not y_labels:
        y_labels = [None] * num_subplots
    fig = plt.figure(figsize=(12,6), dpi=80)
    fig.subplots_adjust(hspace=.5)
    axes = []
    for i in range(1, num_subplots + 1):
        ax = plt.subplot(num_subplots, 1, i)
        ax.grid()
        if x_labels[i-1]:
            ax.set_xlabel(x_labels[i-1]) 
        if y_labels[i-1]:
            ax.set_ylabel(y_labels[i-1]) 
        axes.append(ax)
    return fig, axes

def plot_update(fig, axes, xs, ys, colors, labels):
    for i in range(len(axes)):
        for j in range(len(ys[i])):
            axes[i].plot(xs, ys[i][j], colors[i][j])
        axes[i].legend(labels=labels[i])
    fig.canvas.draw()


fig, axes = plot_init(
    num_subplots=2, 
    x_labels=[None, 'Time (sec)'],
    y_labels=['Temps (C)', 'Heaters']
)

for i in range(1, n):
    try:
        ys = [
            [T1s[:i], T2s[:i]],
            [Q1s[:i], Q2s[:i]]
        ]
        plot_update(fig, axes, m_time[:i], ys, colors, labels)
        time.sleep(0.2)
    except KeyboardInterrupt:
        break
```

[^1]: [Stack Overflow: Jupyter notebook matplotlib figures show up small until cell is completed](https://stackoverflow.com/questions/45384072/jupyter-notebook-matplotlib-figures-show-up-small-until-cell-is-completed)