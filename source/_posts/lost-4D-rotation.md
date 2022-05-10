---
title: The Lost 4-Dimensional rotation 
date: 2022-05-09 09:29:41
tags: [math, geometry]
---
Occasionally, an article on higher spacetime or spatial dimensions pops up on Hackernews. The notion of 4-dimensions is a perennial favorite. From the last couple years:

* [What is the Fourth Dimension?](https://news.ycombinator.com/item?id=27329211)
* [The forth dimension: toward a geometry of higher reality](https://news.ycombinator.com/item?id=30368840)
* [A Spacetime Surprise: Time Isn’t Just Another Dimension](https://news.ycombinator.com/item?id=25109046)
 
Quaternions, an extension of complex numbers that are related to 3- and 4-dimensional rotations and important for computer graphics, are another perennial favorite:

* [Quaternions](https://news.ycombinator.com/item?id=29510237)
* [Rotations with Quaternions](https://news.ycombinator.com/item?id=27353263)
* [Let's Remove Quaternions from every 3D engine](https://news.ycombinator.com/item?id=29512302)

Sometimes these articles try to give an intuitive sense of higher-dimensional space through 3D and 4D analogy, eschewing a more formal mathematical approach. Here I'll do the opposite: show with a very little math that something that I find very unintuitive.

I always look for mention of a tricky kind of rotation that's not present in 2D or 3D: the 4D rotation about two axes.

## The lost 4D rotation

In four dimensions, it's possible to rotate around two _perpendicular_ axes at once. Imagine a car wheel spinning about its axle. In 4D, you could drive a second axle long-ways through the tire _and still drive the car smoothly down the road_.

## The 2D rotation

<figure>
  <img src="/images/rotations_2D_only.png" alt="2D, 3D, and 4D rotations."/>
  <figcaption style="text-align: center"><em>A 2D, 3D, and possible 4D rotation. The 3D rotation is not very different from the 2D rotation, but the 4D rotation admits new possibilities.</em></figcaption>
</figure>

The 2D rotation describes how a points on a plane change position when you rotate theta degrees around the origin. It can be written as a 2x2 matrix that transforms point (x, y) to point (x', y'). It's usually written like this:

{% mathjax %}
\mathbf{R_{2D}}(\theta)=\left(\begin{array}{cc} 
 \cos \theta  & -\sin \theta  \\
 \sin \theta  & \cos \theta 
\end{array}
\right)
{% endmathjax %}

You can check your intuition by thinking about rotating a point (1, 0) by ninety degrees. It should end up as (1, 0), right?

{% mathjax %}
\left(\begin{array}{c}
x' \\
y'
\end{array}
\right) =\left(\begin{array}{cc}
\cos \theta  & -\sin \theta  \\
\sin \theta  & \cos \theta
\end{array}
\right) \left(\begin{array}{c}
x \\
y
\end{array}
\right)=\left(\begin{array}{cc}
\cos 90  & -\sin 90  \\
\sin 90  & \cos 90 
\end{array}
\right) \left(\begin{array}{c}
1 \\
0
\end{array}
\right)=\left(\begin{array}{cc}
0  & -1  \\
1  & 0 
\end{array}
\right) \left(\begin{array}{c}
1 \\
0
\end{array}
\right)=\left(\begin{array}{c}
0 \\
1
\end{array}
\right)
{% endmathjax %}

## The 3D rotation

<figure>
  <img src="/images/rotations_2D3D.png" alt="2D, 3D, and 4D rotations."/>
  <figcaption style="text-align: center"><em>A 2D, 3D, and possible 4D rotation. The 3D rotation is not very different from the 2D rotation, but the 4D rotation admits new possibilities.</em></figcaption>
</figure>

If you understand the 2D rotation, the 3D rotation is easy. That's because a 3D rotation always has a fixed axis, like the axle of a car. So instead of a fixed point we have a fixed line.

In fact, the generic 3D rotation is a straightforward extension of the 2D rotation, with the extra dimension fixed.  This rotation sends a point (x, y, z) to (x', y', z'), and you see from the image above and the matrix below, the points on the z-axis don't move.

{% mathjax %}
\mathbf{R_{3D}}(\theta)=\left(\begin{array}{cc|c}
\cos \theta  & -\sin \theta & 0 \\
\sin \theta  & \cos \theta & 0 \\
\hline
0 & 0 & 1
\end{array}
\right)
{% endmathjax %}

## 4D rotations

{% mathjax %}
\mathbf{R_{4D}}(\theta)=\left(\begin{array}{ccc|c}
\cos \theta  & -\sin \theta & 0 & 0 \\
\sin \theta  & \cos \theta & 0 & 0\\
0 & 0 & 1 & 0 \\
\hline
0 & 0 & 0 & 1
\end{array}
\right)
{% endmathjax %}

<figure>
  <img src="/images/rotations.png" alt="2D, 3D, and 4D rotations."/>
  <figcaption style="text-align: center"><em>A 2D, 3D, and possible 4D rotation. The 3D rotation is not very different from the 2D rotation, but the 4D rotation admits new possibilities.</em></figcaption>
</figure>

{% mathjax %}
\mathbf{R_{4D}}(\theta,\phi)=
\left(
\begin{array}{cc|cc}
\cos \theta  & -\sin \theta & 0 & 0 \\
\sin \theta  & \cos \theta & 0 & 0\\
\hline
0 & 0 & \cos \phi  & -\sin \phi  \\
0 & 0 & \sin \phi  & \cos \phi
\end{array}
\right)
{% endmathjax %}