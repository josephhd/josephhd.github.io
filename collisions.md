<article>
<h1>Solving Collisions Without Rotations</h1>
As of late, I have been interested in rigidbody physics simulation. Here, I look at the case of resolving collisions without rotations. This does not cover collision detection, as it is an entirely different topic with many of its own complexities.   

<h2>Newton's Second and Third Laws</h2>
Let's take a look at Newton's Second Law. 

$$\vec F=m\frac{d\vec v}{dt}$$

In a collision, we have a force acting on an object over a time, denoted $t$. So let's integrate $\vec F$ over this time ($\Delta$ denotes the change in a quantity). 

$$\int_0^t\vec Fdt=\int_0^tm\frac{d\vec v}{dt}dt=m\int_0^td\vec v=m(\vec v(t)-\vec v(0))=m\Delta\vec v$$

And we must remember that $\vec v$ is a function of time, so we get the result above. If we recall that momentum is $\vec p=m\vec v$, then we see that $\int_0^t\vec Fdt=\Delta\vec p.$ And so we see that a force applied over time yields a change in momentum. 

Let's work with this result for a bit. Newton's Third Law states that $\vec F_a=-\vec F_b$ for two objects (a and b) in collision. At the start, we stated that a collision occurs over a time $t$. So if we integrate both sides of this equation, we get the following result.

>$$\Delta\vec p_a=-\Delta\vec p_b$$
>$$\implies m_a(\vec v_a-\vec u_a)=-m_b(\vec v_b-\vec u_b)$$
>*where $u$ and $v$ denote the pre-collision and post-collision velocities, and $m$ is mass*

We can rearrange this and see that that the total change in momentum must be zero, i.e. momentum is conserved. 

We can further manipulate this equation to see that momentum is conserved along any direction of our choosing. Simply taking the dot product of both sides with a unit vector $\vec n$ yields

$$m_a(\vec v_a-\vec u_a)\cdot \vec n=-m_b(\vec v_b-\vec u_b)\cdot \vec n$$

This does something interesting for us. It allows us to reduce the number of dimensions in our problem to just one. If we were to try and solve for the post collision velocities using the previous form, we would have four, or six unknowns depending on what dimension our collision lives in. Of course we don't get this totally for free. We still need to know what direction the collision occured, so that we can apply the appropriate change in momenta, but we are assuming that we do know this.

So let's interpret what this equation tells us. It tells us that the change in momentum of two objects in collision is equal and opposite along some direction $\vec n$. Let's write this more simply using *impulses*. An impulse is what we've already derrived: a change in momentum. We will use $j$ to denote the magnitude of the impulse in the direction of the collision, $\vec n$.

$$j=-j$$

Using this form, we can write new equations for the change in velocity of the respective objects a and b. 

>$$\vec v_a=\vec u_a+\frac{j}{m_a}\vec n$$
>$$\vec v_b=\vec u_b-\frac{j}{m_b}\vec n$$

Here we are incorporating what we derrived above. We have that $j$ is a change in momentum, so $j/m$ yields a change in velocity (since again $p=mv$). We then multiply by our collision normal $\vec n$ to recover the direction in which the momentum of each object changes. 

<h2>Coefficient of Restitution</h2>
In real collisions we see that energy is lost. Just bounce a tennis ball, and watch as it eventually comes to rest on the floor. Conveniently, a quantity already exists to describe this loss of energy, called the coefficient of resitution, denoted $\epsilon$.

$$\epsilon=\frac{v_b-v_a}{u_a-u_b}$$
$$0\leq \epsilon\leq 1$$

If $\epsilon=1$ the collision is lossless, and if $\epsilon=0$ all energy is lost.

Note here that all these quantities are scalars, not vectors as before. The post and pre-velocities used in this equation are relative to some collision normal ($\vec n$), so we could rewrite this equation as 

$$\epsilon =\frac{(\vec v_b-\vec v_a)\cdot \vec n}{(\vec u_a-\vec u_b)\cdot \vec n}$$

Which we can rearrange into

$$\epsilon (\vec u_a-\vec u_b)\cdot \vec n=(\vec v_b-\vec v_a)\cdot \vec n \implies (\vec v_a-\vec v_b)\cdot \vec n=-\epsilon (\vec u_a-\vec u_b)\cdot \vec n$$

This equation gives us a relation for the velocities of each object pre and post colliison, but along *any* arbitrary direction $\vec n$. 

<h2>Putting it all Together</h2> 

To recap we have three equations at our disposal.

>$$(\vec v_a-\vec v_b)\cdot \vec n=-\epsilon (\vec u_a-\vec u_b)\cdot \vec n$$
>$$\vec v_a=\vec u_a+\frac{j}{m_a}\vec n$$
>$$\vec v_b=\vec u_b-\frac{j}{m_b}\vec n$$

To solve our collision, we want to solve for the magnitude of the impulse $j$. As shown previously, applying the impulse $j$ in the direction of the collision $\vec n$ will result in the changes in momenta that we are after. 

All that's left to do is to plug in and solve for $j$. 

$$(\vec v_a-\vec v_b)\cdot \vec n=-\epsilon (\vec u_a-\vec u_b)\cdot \vec n$$ 

Substituting $v_a$ and $v_b$:

$$(\vec u_a+\frac{j}{m_a}\vec n-(\vec u_b-\frac{j}{m_b}\vec n))\cdot \vec n=-\epsilon (\vec u_a-\vec u_b)\cdot \vec n$$
$$\implies(\vec u_a-\vec u_b+j\vec n(\frac{1}{m_a}+\frac{1}{m_b}))\cdot \vec n=-\epsilon (\vec u_a-\vec u_b)\cdot \vec n$$

Note here that $||\vec n||=1\implies \vec n\cdot \vec n = 1$

$$\implies (\vec u_a-\vec u_b)\cdot n+j(\frac{1}{m_a}+\frac{1}{m_b})=-\epsilon (\vec u_a-\vec u_b)\cdot \vec n$$
$$\implies j(\frac{1}{m_a}+\frac{1}{m_b})=-\epsilon (\vec u_a-\vec u_b)\cdot \vec n-(\vec u_a-\vec u_b)\cdot n$$
$$\implies j(\frac{1}{m_a}+\frac{1}{m_b})=-(1+\epsilon)(\vec u_a-\vec u_b)\cdot n$$

And so we arive at our answer:
>$$j=\frac{-(1+\epsilon)}{\frac{1}{m_a}+\frac{1}{m_b}}(\vec u_a-\vec u_b)\cdot n$$

<footer>
© 2019 Joseph Dunbar
</footer>
</article>