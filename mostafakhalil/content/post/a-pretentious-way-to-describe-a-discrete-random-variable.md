---
date: '2026-06-21T13:52:23+02:00'
title: 'A pretentious way to describe a discrete random variable'
math: true
tags: ['probability', 'ai-journey']
---

I was reading [*A Brief Introduction to Machine Learning for Engineers*](https://arxiv.org/abs/1709.02840?fbclid=IwY2xjawRtT7dleHRuA2FlbQIxMQBzcnRjBmFwcF9pZBAyMjIwMzkxNzg4MjAwODkyAAEeslp8XKxoQvZ44HU266uVgYKUj0CTClLPGbBAq_hAeq81VaPsEDPFVkDJ_yg_aem_rZru1T0GKO7oSLnoTyU-zA) because I wanted to build some rigor around Machine Learning theory. By the second chapter, I had already realized how much skill atrophy I'm facing. My math muscles are... a bit rusty I'd say.

On page 18 there was this beautiful formula that made me feel dumb. It describes a conditional probability distribution for a continuous random variable $t$.

$$t|x = x ∼ 0.8δ(t − x) + 0.2δ(t + x)$$

I did not know what that $δ$ meant. I was introduced to the [Dirac delta function](https://en.wikipedia.org/wiki/Dirac_delta_function). It has a very peculiar definition.

> The Dirac delta function, $\delta(x)$, is formally defined by two fundamental properties:
>
> $$
 \delta(x) = \begin{cases} 
 +\infty & \text{if } x = 0 \\ 
 0 & \text{if } x \neq 0 
 \end{cases}
 $$
> 
> Subject to the strict integral constraint over the entire real line:
> 
> $$
 \int_{-\infty}^{\infty} \delta(x) \, dx = 1
  $$

So if we take the two Dirac deltas in the above formula and expand them, $\delta(t - x)$ would expand to:
$$
\delta(t - x) = \begin{cases} 
+\infty & \text{if } t = x \\ 
0 & \text{if } t \neq x 
\end{cases}
$$

and $\delta(t + x)$ expands to:

$$
\delta(t + x) = \begin{cases} 
+\infty & \text{if } t = -x \\ 
0 & \text{if } t \neq -x 
\end{cases}
$$

And the integral constraint would be the same for both functions.

I immediately thought, that's just a discrete random variable, and this is a *pretentious* (or at least roundabout...) way to describe it! We could have just described the distribution's [PMF](/post/probability-for-software-engineers-the-first-principles/#probability-mass-function-pmf) as:

$$
p(t|x) = \begin{cases} 
0.8 & \text{if } t = x \\ 
0.2 & \text{if } t = -x \\ 
0 & \text{otherwise} 
\end{cases}
$$

And my biggest hype-up buddy, Gemini, agreed enthusiastically with me!

![](/images/posts/a-pretentious-way-to-describe-a-discrete-random-variable/you-hit-the-nail-on-the-head.png)


Apart from Gemini's desperate attempts to cure my imposter syndrome, describing a discrete random variable using the Dirac delta function is actually useful. You can now apply continuous random variable constructs on discrete random variables! i.e., the book just continued with calculating the expected value without doing discrete $\leftrightarrow$ continuous shenanigans.

$$
\begin{align*}
\mathbb{E}[t|x] &= \int_{-\infty}^{\infty} t \, p(t|x) \, dt \\
&= \int_{-\infty}^{\infty} t \left( 0.8\delta(t - x) + 0.2\delta(t + x) \right) dt \\
&= 0.8 \int_{-\infty}^{\infty} t \, \delta(t - x) \, dt + 0.2 \int_{-\infty}^{\infty} t \, \delta(t + x) \, dt \tag{1}
\end{align*}
$$

And there is a very convenient property for Dirac delta functions: [*Sifting*](https://math.stackexchange.com/questions/73010/proof-of-dirac-deltas-sifting-property). A translated Dirac delta multiplied by a function would integrate to the output of the function at the translated value.

$$
\int_{-\infty}^{\infty} f(t) \delta(t - x) dt = f(x)
$$

Substituting these evaluations back in $(1)$:

$$
\begin{aligned}
\mathbb{E}[t|x] &= 0.8(x) + 0.2(-x) \\
&= 0.8x - 0.2x \\
&= 0.6x
\end{aligned}
$$
 *(you can think that the function here is $f(t) = t$)*

I wondered how this is the first time I get to know about this. I was desperate for another imposter relief. Gemini didn't pick up on the signal for this one, but atleast it tried...

![](/images/posts/a-pretentious-way-to-describe-a-discrete-random-variable/yes-it-is-extremely-common.png)