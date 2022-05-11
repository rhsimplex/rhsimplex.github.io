---
title: The Lost 4-Dimensional Rotation 
date: 2022-05-09 09:29:41
tags: [math, geometry]
---
Occasionally, an article about the Fourth Dimension pops up on Hacker News. For instance, [The Fourth dimension: Toward a geometry of higher reality (1984)](https://news.ycombinator.com/item?id=30368840) submitted February 2022.

In the spirit of the classic [Flatland](http://www.geom.uiuc.edu/~banchoff/Flatland/), such articles usually try to give an intuitive sense of higher-dimensional space through 2D or 3D analogy, eschewing a more mathematical approach. Here I'll try to do the opposite. I'll use a little math to show something I find very unintuitive about 4D that rarely comes up: that an object in 4D can spin around two perpendicular axes at the same time.

## No good analogy!

Here's how I might try to explain it intuitively: imagine a car wheel spinning about its axle. In 4D you could ram a second axle long-ways through the tire, start spinning the wheel around _that_ axis, and _still drive the car smoothly down the road_.

This analogy is not great -- we'll see why at the end -- so let's step through the dimensions and do some math to sort this out.

## The 2D rotation

<figure>
  <img src="/images/rotations_2D_only.png" alt="2D, 3D, and 4D rotations."/>
  <figcaption style="text-align: center"><em>The familiar 2D rotation in the xy plane.</em></figcaption>
</figure>

The 2D rotation describes how a points on a plane change position when you rotate around the origin. It can be written as a 2x2 matrix that transforms point (x, y) to point (x', y'). It's usually written like this:

{% mathjax %}
\mathbf{R_{2D}}(\theta)=\left(\begin{array}{cc} 
 \cos \theta  & -\sin \theta  \\
 \sin \theta  & \cos \theta 
\end{array}
\right)
{% endmathjax %}

You can check your intuition by thinking about rotating a point (1, 0) by ninety degrees counter-clockwise. It should end up as (0, 1): going from pointing right along the x-axis to pointing up along the y.

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
  <figcaption style="text-align: center"><em>The 3D rotation is not much more complicated. The fixed point at the origin becomes a fixed line along the z-axis.</em></figcaption>
</figure>

The 3D rotation is a simple extension of the 2D rotation, as long as we choose the axis of rotation to lie in the 3rd-dimension (the z-direction). The 3D rotation matrix below sends a point (x, y, z) to (x', y', z'), and you can see that it never moves the z-coordinate of any point.

{% mathjax %}
\mathbf{R_{3D}}(\theta)=\left(\begin{array}{ccc}
\cos \theta  & -\sin \theta & 0 \\
\sin \theta  & \cos \theta & 0 \\
0 & 0 & 1
\end{array}
\right)
{% endmathjax %}

Can you see this is just the 2D rotation with an extra dimension? If you're describing how a record spins on a turntable -- even in a 3D world -- you don't need to worry about the height of the record!

## 4D rotations

Now going to 4D, you could just extend the rotation matrix like we did for 2D -> 3D.

{% mathjax %}
\mathbf{R_{4D}}(\theta)=\left(\begin{array}{cccc}
\cos \theta  & -\sin \theta & 0 & 0 \\
\sin \theta  & \cos \theta & 0 & 0\\
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1
\end{array}
\right)
{% endmathjax %}

And indeed this is a valid 4D rotation. Instead of one fixed axis, it now has one fixed _plane_. If I split our 4D rotation matrix into quarters, you can see it in the bottom right-hand corner:

{% mathjax %}
\mathbf{R_{4D}}(\theta)=\left(\begin{array}{cc|cc}
\cos \theta  & -\sin \theta & 0 & 0 \\
\sin \theta  & \cos \theta & 0 & 0\\
\hline
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1
\end{array}
\right)
{% endmathjax %}

Since we have a whole free plane just doing nothing, couldn't we also shove a 2D rotation in there? Turns out we can, and the following is a perfectly valid 4D rotation:

{% mathjax %}
\mathbf{R_{4D}}(\theta,\phi)=
\left(
\begin{array}{cccc}
\cos \theta  & -\sin \theta & 0 & 0 \\
\sin \theta  & \cos \theta & 0 & 0\\
0 & 0 & \cos \phi  & -\sin \phi  \\
0 & 0 & \sin \phi  & \cos \phi
\end{array}
\right)
{% endmathjax %}

Now we have the totally new rotation! It has no precedent in 2 or 3 dimensions. We can now rotate in two different planes, through two different angles, at the same time.

<figure>
  <img src="/images/rotations.png" alt="2D, 3D, and 4D rotations."/>
  <figcaption style="text-align: center"><em>In the 4D rotation, you can rotate about two perpendicular planes. The axis of each rotation likes in the plane of the other. Here w denotes the new dimension.</em></figcaption>
</figure>

## Conclusion

So why was our analogy of a wheel with an extra axle stuck through it unsatisfying? Because not only are the axes perpendicular, the planes of rotation are as well. Since you can't have two perpendicular planes in 3D, our intuition -- and by extension the analogy -- fails.

If you want to read more about why I was getting into this back in 2008, feel free to jump to [Appendix C of my doctoral dissertation](https://hdl.handle.net/1813/33790) to learn about the character table of the [600 cell](https://en.wikipedia.org/wiki/600-cell).