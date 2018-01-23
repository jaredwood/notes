---
layout: post
title: "GPS (Global Positioning System) Pseudorange"
modified:
categories: blog
excerpt: "Pseudorange is the receiver's measured apparent range to a satellite."
tags: [gps, pseudorange]
author: jared_wood
share: true
image:
  feature:
date: 2018-01-23
---

## GPS Pseudorange

GPS satellites are synchronized to GPS time, which is an absolute time similar to UTC. GPS satellites broadcast several messages. Within these messages is the time of transmission. Assuming a GPS receiver is synchronized to GPS time, when it receives a satellite message it can use the satellite transmission time to measure the apparent time duration for the message to reach the receiver. Factoring in the speed of light (the speed of the message) through the atmosphere, the distance \\(r\\) the message travels is then

\\[
r = \int_{t_0}^{t} c(\tau) d\tau
\\]
where \\( t_0 \\) is the true satellite transmission GPS time and \\( t \\) is the true receiver receive GPS time.

Unfortunately \\( t_0 \\) and \\( t \\) are not known. The satellite sends its measured clock time, which is
\\[
t_s = t_0 + \delta_s
\\]
where \\( \delta_s \\) is the satellite's GPS synchronization bias. Similarly the receiver reads its measured clock time, which is
\\[
t_r = t + \delta_r
\\]
where \\( \delta_r \\) is the receiver's GPS synchronization bias. An estimate of the satellite bias is sent in its message. However, the receiver bias is unknown.

Additionally \\( c(t) \\) is not known for the path of the message. But the speed of light in a vacuum is known. It is also known that \\( c(t) \\) will not vary too far from its constant speed in a vacuum \\( c_0 \\). The receiver then uses \\(c_0\\) \\(t_0\\) (in the message), and \\(t_r\\) (receiver's measured time) to compute a measured "apparent" distance, called pseudorange \\(\rho\\), which is

$$
\begin{align}
\rho &= \int_{t_s}^{t_r} c_0 d\tau \\
     &= c_0 (t_r - t_s) \\
\end{align}
$$

This is what the GPS receiver measures.

We now have a goal: How does the true distance relate to the pseudorange? First, notice that the speed will never be greater the its speed in a vacuum. The speed of light along the path of the message can then be written as
\\[
c(t) = c_0 - \delta_c(t)
\\]
The true distance is then

$$
\begin{align}
r &= \int_{t_0}^{t} c_0 - \delta_c(\tau) d\tau \\
  &= c_0 (t - t_0) - \int_{t_0}^{t} \delta_c(\tau) d\tau \\
  &= c_0 (t_r - \delta_r - (t_s - \delta_s)) - \int_{t_0}^{t} \delta_c(\tau) d\tau \\
  &= c_0 (t_r - t_s) - c_0 \delta_r + c_0 \delta_s - \int_{t_0}^{t} \delta_c(\tau) d\tau \\
  &= \rho - c_0 \delta_r + c_0 \delta_s - \int_{t_0}^{t} \delta_c(\tau) d\tau \\
\end{align}
$$

Note that I have been calling \\(r\\) and \\(\rho\\) distances rather than calling them ranges (as the symbols would imply). In fact the path of the message is not perfectly straight. But it is very close. Correction terms are typically applied to account for the variations in its path including its speed. So I will now call them ranges.

How should we evaluate the integral over \\(\delta_c(\tau)\\)? One approach would be to estimate \\(\delta_c(t)\\). But instead of estimating \\(\delta_c(\tau)\\) one would have to estimate \\(\delta_c(x, \tau)\\), where \\(x\\) is global position. Additionally the path of the message \\(x(\tau)\\) would need to be estimated. In reality this problem would require much more resource than a single receiver. So let's consider an alternative approach.

The integral over \\(\delta_c(\tau)\\) is an additive range. Let's attempt to account for it by replacing it with correction terms. To do this, replace it into two components according to the ionosphere and the troposphere. There have been many models developed to estimate these two components. In fact, parameters for one such ionosphere model are included in satellite messages. The troposphere is more difficult to estimate because weather affects it. However, we can now write the range as

$$
\begin{align}
r &= \rho - c_0 \delta_r + c_0 \delta_s - \int_{t_0}^{t} \delta_c(\tau) d\tau \\
  &= \rho - c_0 \delta_r + c_0 \delta_s - T - I - \epsilon \\
\end{align}
$$

where \\(\epsilon\\) accounts for model errors.

So how do we get the receiver position? Let's answer this for the case of using correction terms. Let's combine the corrections into the measured pseudorange.

$$
\begin{align}
r &= \rho - c_0 \delta_r + c_0 \delta_s - T - I - \epsilon \\
  &= (\rho + c_0 \delta_s - T - I) - c_0 \delta_r - \epsilon \\
  &= \rho_c - c_0 \delta_r - \epsilon \\
\end{align}
$$

where \\(\rho_c\\) is the corrected pseudorange measurement. \\(\rho_c\\) is known. Separating the known and unknown terms we get
\\[
\rho_c = r + c_0 \delta_r + \epsilon
\\]
And the geometric range is
\\[
r = \| x_s - x \|
\\]
where \\(x_s\\) is the satellite position at time of transmission and \\(x\\) is the receiver position at time of reception. So we have
\\[
\rho_c = \| x_s - x \| + c_0 \delta_r + \epsilon
\\]
where \\(\epsilon\\) is just a term to recognize unmodeled errors. Parameters to compute \\(x_s\\) are provided in the message. Our unknowns are then \\(x\\) and \\(\delta_r\\) and our target value is \\(\rho_c\\).

This equation is nonlinear. There are possible ways to estimate the unknowns. But first take a moment to consider how many independent samples are required. The answer is four, since there are four unknowns. But considering unmodeled errors it would be better to use as many samples as possible. Along these lines one approach is to linearize the equation about the solution point and do regression.

TODO
