<article>
<h1>Dead Reckoning</h1> 

I recently had a need for an absolute positioning system using only dead reckoning. At my disposal was a robot with two motors and encoders on each.  

Now, the motors that I used were simple DC motors of questionable quality. While we had built circuits to attempt to drive the two motors at the same speed, physical differences in the motors and circuits prevented this. A naive implementation of dead reckoning assumes that both motors move at the same rate. This is rarely (if ever) the case, and was certainly not the case here. Rather than trying to adjust voltages to match motor speeds, I opted to investigate how to track position with different motor speeds. 

<h2>Overview</h2>

Before diving into the specifics of computing how the robot's position changes over time, I'd like to take a moment to describe how one might implement code for this task. There are several steps, detailed in the list below:

   1. Sample the encoders to track wheel movement 
   2. Feed sampled positions into a function to compute changes in position 
   3. Apply these changes in position to the global positon of the robot

Let's address (1) first. In this step, we are just sampling how far the wheels move over a given period. This period is defined by some number of encoder counts, perhaps 25. It's also easy to find how far the wheel will move per encoder count, so the distance that the wheel moves is just the number of encoder counts, multiplied by the distance per encoder count. 

For (2), we feed in the sampled data from (1) to start calculating the change in the robot's position, and orientation. It is important to track both the position and orientation, because the equations that we will derive for the changes in position are relative to the robot's orientation. One could also say that we need to track the orientation because the changes are in the robot's local space, and cannot be directly applied to the robot's global position. This is not the same for tracking the orientation, or rotation. This can be directly applied to orientation of the robot because the changes in rotation are the same regardless of how the coordinate system is rotated. 

Finally, (3) involves updating the robot's position and orientation data. As mentioned in (2), this will require converting the changes in position relative to the robot's orientation into the global space.   

I am not going to talk about (1) because it is simple to implement. So let's start with (2). There are two ways in which a robot with two wheels can move. You can run both wheels in the same direction, or you can run the wheels in opposite directions.

<h2>Case 1: Linear Motion</h2>

Let's look at the case where both motors move in the same direction. If the motors don't move at the same rate, then we can assume that the robot will move in arcs. We assume this based on the additional assumption that the wheels do not slip. So each wheel will travel some (different) distance and the only way for an object to move with these assumptions is by moving in an arc. So, let's draw a digram.

<img src="images/arc1.svg" width="200" style="margin: auto"> 

Depcited in the figure is the case of the robot drifting to the right. In the figure we have some variables that need explanation. 

Knowns:
* $l_1$: distance read by the left encoder
* $l_2$: distance read by the right encoder
* $w$: width as measured to the ourside edges of each wheel

Unknowns: 
* $\theta$: the angle of the arc traced out by each wheel
* $r_1$: radius of the inner arc
* $r_2$: radius of the outer arc
* $\Delta \vec x_1$: change in position of the right (inner) wheel
* $\Delta \vec x_2$: change in position of the left (outer) wheel

What we seek is the change in position of the robot itself. We want to track this change in position relative to the robot's pivot point. This is simply the midpoint between both wheels. So the position of the pivot point is just the average of the two wheel positions, or the average of $\Delta\vec x_1$ and $\Delta\vec x_2$. This will also apply for Case 2. So we then need to find $\Delta\vec x_1$ and $\Delta\vec x_2$, and then compute the average to find the change in the position of the robot. 

If we assume $\theta$ in radians, we have:

$$\theta=\frac{l_1}{r_1}=\frac{l_2}{r_2}$$

Using the diagram above, we also see that $r_2=w+r_1$, so we then have:

$$\theta=\frac{l_1}{r_1}=\frac{l_2}{w+r_1}$$

This leaves us with just one unknown, $r_1$, so we can do some algebra this to find its value:

$$l_1(w+r_1)=l_2r_1\implies wl_1+l_1r_1=l_2r_1 \implies r_1=\frac{wl_1}{l_2-l_1}$$
>$$\implies r_1=\frac{w}{\frac{l_2}{l_1}-1}$$
>$$\implies r_2=r_1+w$$

As per described in step (2), we also need to track the orientation. We actually have two equations for $\theta$. We could pick either one for our $\theta$ calculation, or average them. I am going to average them, and I am going to assume that the value of $\theta$ is this average value for all subsequent calculations.  
>$$\theta=\frac{1}{2}(\frac{l_1}{r_1}+\frac{l_2}{r_2})$$

Using these values we can now compute the changes in position of each wheel.

>$$\Delta \vec x_1=r_1\begin{bmatrix}{1-cos(\theta)}\\{sin(\theta)}\end{bmatrix}$$
>$$\Delta \vec x_2=r_2\begin{bmatrix}{1-cos(\theta)}\\{sin(\theta)}\end{bmatrix}$$
>$$\Delta \vec x=\frac{1}{2}(\Delta \vec x_1 + \Delta \vec x_2)

As stated earlier, we want to take the average of $\Delta\vec x_1$ and $\Delta\vec x_2$ to compute $\Delta\vec x$, which is the change in position of the robot's pivot point reltaive to the robot's orientation.   

I am using the convension that the first row is the x component, and the second is the y. The $1-cos(\theta)$ comes from the fact that cosine starts at 1, while sine starts at 0. Since we want to know by how much the x position changes, we need to take 1 minus cosine. 

The equations above are only valid for when the robot is moving forwards, and drifting to the right. If the robot is moving to the left, we need to multiply the x component by -1, since the robot would be drifting in the -x direction. If the robot is moving backwards (and to the right), then we can simply multiply $\theta$ by -1 when calculating the changes in position. If the robot is moving backwards, and to the left, we multiply both the x component by -1, and $\theta$ by -1. We can determine left and right drift by the magnitudes of $l_1$ and $l_2$. If $l_1>l_2$ then the robot will drift left. If $l_1<l_2$, the robot will drift right. 

We also need to be carfull when calculating $\theta$. If $l_1<l_2$ we need to use a different equation for $\theta$:
>$$\theta=\frac{l_2}{r_1}=\frac{l_1}{r_2}$$

This keeps the assumption that $r_2$ is the outside radius, and $r_1$ is the inside radius. 

<h2>Case 2: Rotational Motion</h2>

In this case, we are assuming that one motor will move in the opposite direction as the other, thus turning the robot in place. In case 1, we already assumed that the wheels of the robot will move in arcs, and the same applies here. Once again, we draw a diagram. 
 

<img src="images/arc2.svg" width="250">

The figure depcits the robot turning to the right in place. We can see that when the speed of the two motors is not the same, the position of the robot's pivot point will actually change when turning in place. The values shown in the figure are the same as in Case 1. As before, we seek the change in position of the robot itself. This will again be the change in position of the robot's pivot point, which will be the average of $\Delta\vec x_1$ and $\Delta\vec x_2$. 

As before we have $\theta$:

$$\theta=\frac{l_1}{r_1}=\frac{l_2}{r_2}$$
From the diagram we see that $r_2=w-r_1$, and substituing in we have:

$$\frac{l_1}{r_1}=\frac{l_2}{w-r_1}\implies l_1(w-r_1)=r_1l_2\implies l_1w-l_1r_1=r_1l_2\implies r_1=\frac{l_1w}{l_1+l_2}$$
>$$\implies r_1=\frac{w}{\frac{l_2}{l_1}+1}$$
>$$\implies r_2=w-r_1$$

And as before we also have $\Delta\theta$:
>$$\theta=\frac{1}{2}(\frac{l_1}{r_1}+\frac{l_2}{r_2})$$

As with case 1, we now have all the information we need to solve for $\Delta x_1$ and $\Delta x_2$. 

>$$\Delta\vec x_1=r_1\begin{bmatrix}{cos(\theta)-1}\\{-sin(\theta)}\end{bmatrix}$$
>$$\Delta\vec x_2=r_2\begin{bmatrix}{1-cos(\theta)}\\{sin(\theta)}\end{bmatrix}$$
>$$\Delta\vec x=\frac{1}{2}(\Delta\vec x_1+\Delta\vec x_2)

As with case one, we use sines and cosines to solve for the change in position for each wheel. In this case, we see that the rigth wheel moves in the negative x and y direction, so we just multiple $\Delta\vec x_1$ by -1. We also have $\Delta\vec x$ as the average in the change in positions of each wheel, as before.  

Again, these equations are only valid when the robot is turning right, and $l_2>l_1$. If the robot is turning left, we need to multiply $\theta$ by -1 when calculating the changes in position. And as with Case 1, if $l_2<l_1$ we use the alternate equation for calculating $\theta$. This allows us to reduce the number of cases we have to deal with. For example, we could have the case that $l_2>l_1$, but we are turning to the left, which would imply negative movement in y, but positive in x. This would throw some more negatives into the works if dealt with using if/else, but using an alternative equation for $\theta$ cleans this up, while maintaining all the proper signs in all cases. 

<h2>Applying Changes</h2>

Now that we have derived formulas for the local changes in the robot's position, we need to convert these to changes in global position. 

Let's say we have a value $\theta_0$ that tracks the robot's orientation. To transform our values, we can simply use a rotation matrix, with $\theta_0$ as its parameter. So let's express the global change in position as $\Delta\vec x_g$:

>$$\Delta\vec x_g=\begin{bmatrix}{cos(\theta_0)}&{-sin(\theta_0)}\\{sin(\theta_0)}&{cos(\theta_0)}\end{bmatrix}\Delta\vec x$$

This applies to both Cases 1 and 2. Once we have computed this value, we can finally add this to whatever vector is tracking the robot's global position. 

To update the robot's rotation, all we need to do is add $\theta$ to our value $\theta_0$ after we have computed our global change. By convention, however, turning right means that $|\theta|$ should be subtracted, while turning left has $|\theta|$ added. 

<footer>
© 2019 Joseph Dunbar
</footer>
</article>