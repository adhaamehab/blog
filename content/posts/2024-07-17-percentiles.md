---
title: "Understanding Metrics Percentiles"
date: 2024-07-17
draft: false
description: "Explaining how to use percentiles to understand system performance."
summary: "Explaining how to use percentiles to understand system performance."
toc: true
readTime: true
autonumber: true
tags: ['performance', 'distributed-systems', 'statistics']
math: true
showTags: true
---

## Introduction

When working with production systems, we usually track metrics like **latency** and **throughput** to understand system performance. But, sometime (and I mean most of the time), those metrics can give a **false** sense of good performance.

Well, not the metrics themselves but how we reason about them and how they get used and visualized. 

In the amazing blog **[somethingsimilar: notes on distributed systems for the young bloods](https://www.somethingsimilar.com/2013/01/14/notes-on-distributed-systems-for-young-bloods/#metrics)**: _Jeff Hodges_ makes a point about metrics and percentiles:

> __Metrics are the only way to get your job done.__ Exposing metrics is the only way to bridge the gap between what you believe your system does in production and what it actually does.

> __Use percentiles, not averages.__ Percentiles are more accurate and informative than averages. Using a mean assumes that the metric under evaluation follows a bell curve.
> “Average latency” is a commonly reported metric, but I’ve never once seen a distributed system whose latency followed a bell curve.

This is a very valid point. Personally, I fell in this issue countless time early in my career, largely from a poor understanding of the statstics part of this and how to interpret system performance data.

## Imprecise Metric

Taking a hands-on approach on this, let's assume we just deployed a new API where we serve some calculations and are required to complete these calculations in under `25ms`. 

```python
[30, 15, 21, 18, 16, 40, 25, 35, 17, 18]
```

We can easily plot this as a line chart:

![img](https://github.com/user-attachments/assets/46c9d67a-d100-4b5d-ac7f-d147b66041fc)

We can also calculate the average latency with the following formula:

$$
\begin{align}
  avg &= \frac{sum(latency)}{count(latency)}
\end{align}
$$
$$
\begin{align}
avg &=  \frac{15 + 18 + 17 + 18 + 16 + 21 + 25 + 30 + 35 + 40}{10}
\end{align}
$$
$$
\begin{align}
avg &=  23.5 ms
\end{align}
$$

So, looking at the average for our API, it shows `23.5 ms`, which means our API is meeting expectations and everything is fine.

![img](https://github.com/user-attachments/assets/58a033a2-ec81-4c94-b8b9-1bd52eb3e0e9)

**But** this is misleading, and once we have a deeper look at the data we will realize we are not even close to meeting the expectations and only percentiles can give us the full picture.

## Using Percentiles to Understand Performance

### Calculating Percentiles
The concept of percentiles is straightforward. You sort your dataset in ascending order and determine which **[quantile](https://en.wikipedia.org/wiki/Quantile)** you are interested in.

This can be calcualted with these formulas


$$
\begin{align}
rank &=  \lfloor{Quantile * (length(Latency) + 1)}\rfloor 
\end{align}
$$

$$
\begin{align}
P_{Quantile} &= Latency_{rank}
\end{align}
$$

Usually in monitoring systems, we are interested in higher quantiles.

### P50, P75, P90, and P99

Sorting our dataset, it now looks like this:

![img](https://github.com/user-attachments/assets/51a97a73-08fb-462b-ba27-ee83a0c54190)

#### P50

The first percentile we'll calculate is the 50th quantile (median), which is the "middle" value in the sorted dataset.

Applying the numbers on the previous formula:

$$
\begin{align}
rank &=  \lfloor{0.5 * (10 + 1)}
floor 
\end{align}
$$

$$
\begin{align}
rank &=  5
\end{align}
$$

$$
\begin{align}
P50 &= Latency_{rank}
\end{align}
$$

$$
\begin{align}
P50 &= 18 ms
\end{align}
$$

![img](https://github.com/user-attachments/assets/4515ad1e-9fbe-4625-a4b6-8d2a77087327)

_So, the P50 shows us that 50% of our requests are meeting our 25ms latency threshold._

#### P75

Applying the same formula to calculate the 75th position:

$$
\begin{align}
rank &=  \lfloor{0.75 * (10 + 1)}
floor 
\end{align}
$$

$$
\begin{align}
rank &=  8
\end{align}
$$

$$
\begin{align}
P75 &= 30 ms
\end{align}
$$

![img](https://github.com/user-attachments/assets/60b397d3-4cac-46d4-9254-38d0e64f55d9)


This tells us that at least 25% of our requests are exceeding our threshold.

#### P90

By applying the same formula to calculate the P90, P99, and P99.9 quantiles, we gain more insights into our system's performance.

![img](https://github.com/user-attachments/assets/7326d3b3-e06d-4c9d-90bd-13dcf477461b)

The P90 shows us that we have 10% of our requests are taking at least 40ms which is nearly double of our average latency. 


### Full Picture

Putting together all the information we got gives us a comprehensive view of our system's performance:

- 50% of our requests are meeting our threshold.
- At least 25% of our requests are exceeding our threshold.
- 25% of requests are over 30ms.
- 10% of requests are nearly double our average latency.

With this information, we can reason about our system's performance and take concrete actions to improve it.

## Takeaway

The main takeaway is to use percentiles as much as possible. Generally speaking, when you are monitoring a system, you shouldn't rely on a single angle. The performance data you extract from the system might seem alright (this is the easy bit, right?), but how you represent, interpret, and visualize the data is what truly matters.

If you can't understand the full picture of a system, you will fall into the same trap as many others. Believing that your interpretation of how the system works matches how it actually works.

## Recommended Reads

- [Timescale blog on Prometheus metrics types](https://www.timescale.com/blog/four-types-prometheus-metrics-to-collect/)
- [Prometheus documentation](https://prometheus.io/docs/practices/histograms/)
- [How summary metrics work](https://grafana.com/blog/2022/03/01/how-summary-metrics-work-in-prometheus/)

