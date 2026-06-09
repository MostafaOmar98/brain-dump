---
date: '2026-06-09T17:12:35Z'
draft: false
title: 'Probability for Software Engineers: The First Principles'
tags: ['probability', 'ai-journey']
math: true
---

Probability is one of the most intuitive concepts compared to other fields of mathematics. Humans reason about notions of *chance* and *uncertainty* in their daily lives. Words like *random*, *experiment*, *outcome*, and even *probability* are not uncommon in our natural language. Compare this to something like Calculus: *gradient*, *derivative*, *integrals*, *inflection* are not words we use, and are not concepts that we tend to reason about every day.

This familiarity makes probability approachable, but confusing. It's very easy to misinterpret formal probability writing. Also, some seemingly intuitive probability concepts may seem convoluted, or the formal language may seem to unnecessarily overcomplicate them. Questions that often arise due to a lack of rigorous understanding include:

* What is the difference between a probability distribution and a probability density/mass function?
* What's special about the well-known probability distributions: normal distribution, binomial distribution, etc.?
* What the hell is statistical significance? And what really is a $p$-value? Why do we say $p < 0.05$ represents statistical significance? What is this magic number?
* What are experiments, trials, random variables, and outcomes? Are these synonyms or different concepts? And how do they relate to each other?

I always followed an intuitive, hand-wavy approach to probability. It served me well for a long time. I was able to breeze through my undergrad courses. I have recently been going back to learning machine learning from first principles as I dive deeper into AI. The lack of rigor has limited my ability to understand fundamental algorithms, so I went back to probability's first principles.

This article will be the first in what will likely be a series where I explain the fundamentals of probability. I will present my mental model of the first principles, bringing the intuition closer to the formal definitions. I spent much more time coding than dealing with formal mathematics throughout my education, so I will express the mental model through code where useful. I will write in the language that has my favourite type system, TypeScript. Knowledge of TypeScript is not a prerequisite since I will explain the unfamiliar parts. Basic familiarity with probability concepts and code is useful, though.
## The Experiment: Where it all starts

At the very heart of the study of probability is the concept of an *experiment*, also known as a *random process*. It is a process that we can run to observe its output. Each run of this experiment/process is called a *trial*. The observed output of each trial is called an *outcome* (or an *observation*). This definition is very generic and is applicable to any kind of observable process. For example:

* **Roll a 6-sided die (Experiment):** Observe the number on the top of the die (Outcome).
* **Choose a random person (Experiment):** Observe their height (Outcome).
* **Send an HTTP request (Experiment):** Observe its error message, if any (Outcome).

There are a few important things to note about the nature of these experiments:

1. **The internal process is opaque:** We are not interested - at least for now - in studying the exact internals of the experiment that produced the observed outcomes. We acknowledge that the outcomes are stochastic and it might be impossible to figure out a deterministic algorithm that can reason about the mapping between a trial and its outcome. We are, however, interested in understanding the *distribution** of outcomes, and the insights that might follow from that.
2. **The population is a modeling choice:** We could choose that the experiment definition is *"Choosing a random person out of the whole world"*, or *"Choosing an employed person between the ages of 25-30 living in Australia"*. The population of choice will depend on the interest of the study.
3. **The observed outcome is a modeling choice:** There is an infinite number of observations we could make. For example, when rolling a 6-sided die, we don't have to observe the number on the top. We could observe the number of seconds it took before it became still, or the horizontal distance it moved from the original position it was thrown from. The outcome definition will depend on the interest of the study.

We can model an experiment in TypeScript code like this:
```typescript
// `Experiment` is a generic interface; it accepts a type parameter `T`.
// An `Experiment` implementation must implement a single method, `trial()`,
// which represents a single run of this experiment/random process and outputs
// an outcome of type T.
interface Experiment<T> {
	trial(): T;
}
```
That's it! that's all there is to an experiment. The actual outcome definition and the trial implementation will depend on the specific experiment. We can model a die roll experiment as such:
```typescript
// Example 1: Die roll experiment
type DieRollOutcome = 1 | 2 | 3 | 4 | 5 | 6;

class DieRollExperiment implements Experiment<DieRollOutcome> {
	trial(): DieRollOutcome {
		// The implementation is intentionally opaque.
		// Rolling a die is a stochastic, non-deterministic process. We do not know
		// the specific algorithm that produces the exact outcome, which is why
		// we are modeling this as a probability problem in the first place.
	}
}

const dieRollExperiment = new DieRollExperiment();

// Every log we are likely to see a different number
console.log(dieRollExperiment.trial());
console.log(dieRollExperiment.trial());
console.log(dieRollExperiment.trial());
```
TypeScript has a feature called *Union Types*, which we combined with *Literal Types* above. i.e., you can define a type as a union of specific literal values, like we did in `DieRollOutcome`. This is very useful to model finite sets.

An experiment where we choose a person and observe their height can be modeled as:
```typescript
type PersonHeightOutcome = number;

class PersonHeightExperiment implements Experiment<PersonHeightOutcome> {
	trial(): PersonHeightOutcome {
		// The internal implementation is opaque
	}
}
```
Interesting differences to note between the die roll and person height experiments are:
* In the die roll experiment, the set of all possible outcomes was finite. It was one of $\{1, 2, 3, 4, 5, 6\}$.
* In the person height case, the number of possibilities is infinite, since a person's height is a real number. We cannot capture the set of all possibilities ahead of time (unless we choose to round down to a fixed number of digits, which would make the set of possible values finite).

To continue, let's model our last experiment: sending an HTTP request and observing the returned error message, if any.
```typescript
type HTTPRequestErrorMessageOutcome = string | null;

class HTTPRequestExperiment implements Experiment<HTTPRequestErrorMessageOutcome> {
	trial(): HTTPRequestErrorMessageOutcome {
		// The internal implementation is opaque
	}
}
```
The takeaways of this experiment would be:
* There is no strict type requirement for the outcome. It could be a string, a number, or a list of values. The chosen outcome definition is a modeling choice representing which properties of interest we would like to observe.
* The set of possibilities here could be finite or infinite, depending on the implementation of the specific server (which is, again, opaque to us). For example, does the server return one of 5 fixed error messages, or does it return a hash in the error message that changes from time to time?

Up until now, we haven't really talked about probabilities! We just introduced a bit of rigor and formality to describing a process of interest - an experiment whose internals are opaque, but we'd still like to have a better understanding of its behavior. The actual understanding comes from modeling on top of said experiment: **The Probability Space**.

*\* We haven't formally introduced what a distribution is yet, but an intuitive understanding is sufficient for now.*

## Probability Space: The Formal Model

The word *probability* doesn't have a strict mathematical meaning on its own. Probability starts to make sense when it's referenced as a part of a **probability space**.

The probability space is a mathematical model for an experiment. Let's look at the formal definition from Wikipedia before we dive deeper:

> In probability theory, a probability space or a probability triple $(\Omega, \mathcal{F}, P)$ is a mathematical construct that provides a formal model of a random process or experiment.
>
> A probability space consists of three elements:
>
> * **A sample space, $\Omega$**: which is the set of all possible outcomes of a random process under consideration.
> * **An event space, $\mathcal{F}$**: which is a set of events, where an event is a subset of outcomes in the sample space.
> * **A probability function, $P$**: which assigns, to each event in the event space, a probability, which is a number between 0 and 1 (inclusive).

Let's untangle it one by one:

### The Sample Space

Hopefully, the definition reads clearly: it is the set of all possible outcomes of a random process. In TypeScript, that would be modeled like:
```typescript
type SampleSpace<T> = Set<T>;

type DieRollOutcome = 1 | 2 | 3 | 4 | 5 | 6;
const dieRollSampleSpace: SampleSpace<DieRollOutcome> = new Set([1, 2, 3, 4, 5, 6]);
```
In TypeScript, you can make types out of other types. Here we are essentially aliasing the type `Set` to be `SampleSpace` for clarity, and `T` should always be the outcome type definition.

**Important takeaway:** The sample space is heavily constrained by the physical reality of the experiment at hand, leaving less room for arbitrary modeling choices than other elements. i.e., we don't really get to choose what the possible outcomes of an experiment are without influencing the experiment's internals or constraining it some way or another.

### The Event Space

It can be confusing to intuitively differentiate between an *event* and an *outcome*. An outcome is a single result of a trial, a single instance of the sample space.

An event, however, is a set of outcomes. We can ask two interesting questions about any event:
* Has an event $e$ happened?
* What is the probability of an event $e$ happening?

An event is considered to have *happened* in a trial if the trial results in any of its constituent outcomes.

The *event space* ($\mathcal{F}$) is a set of events. i.e., it is a *set of sets*, where each set is a set of outcomes. These events can overlap or be disjoint. We get more modelling room in the Event Space than we do in the sample space. i.e., we can decide which base events we care about and consider that our event space. For an event space to be mathematically valid however, there are some constraints that need to be fulfilled (e.g., if an event is in the space, its complement must also be in the space). Also all outcomes in the defined events must belong to the sample space. For example, in discrete cases, there are $2^N$ different possible events you can define if the size of the sample space is $N$ outcomes.

```typescript
// NOTE: The SampleSpace and Event definitions are the same, but are 
// semantically different.
// A sample space must contain all possible outcomes of an experiment.
// An event must be a subset of a sample space.
// It would be convoluted to describe this in TypeScript, let's agree 
// to remember that for now.
type SampleSpace<T> = Set<T>;
type Event<T> = Set<T>;

type DieRollOutcome = 1 | 2 | 3 | 4 | 5 | 6;
class DieRollExperiment implements Experiment<DieRollOutcome> {
	trial(): DieRollOutcome {
		// Opaque implementation
	}
}

const dieRollSampleSpace: SampleSpace<DieRollOutcome> = new Set([1, 2, 3, 4, 5, 6]);

// Defining a bunch of events
const rollingATwo: Event<DieRollOutcome> = new Set([2]);
const rollingAnEvenNumber: Event<DieRollOutcome> = new Set([2, 4, 6]);
const rollingAPrimeNumber: Event<DieRollOutcome> = new Set([2, 3, 5]);

// Answers whether event `event` has happened if an outcome `outcome` has
// resulted from a trial.
function has_event_happened<T>(event: Event<T>, outcome: T) {
	return e.has(o);
}

const dieRollExperiment = new DieRollExperiment();
const outcome = dieRollExperiment.trial();

console.log(has_event_happened<DieRollOutcome>(rollingATwo, outcome));
console.log(has_event_happened<DieRollOutcome>(rollingAnEvenNumber, outcome));
console.log(has_event_happened<DieRollOutcome>(rollingAPrimeNumber, outcome));
```

In a case like a person height experiment, where the outcome is a continuous variable, each event can be an infinite set containing an infinite set of outcomes (e.g., a set of heights in the range $[150, 170]$ cm). We can still capture this model in TypeScript as a boolean function:

```typescript
type SampleSpace<T> = Set<T>;
type ContinuousVariableEvent<T> = (outcome: T) => boolean;

type PersonHeightOutcome = number;
class PersonHeightExperiment implements Experiment<PersonHeightOutcome> {
	trial(): PersonHeightOutcome {
		// Opaque implementation
	}
}

const oneSeventyCMOrHigher: ContinuousVariableEvent<PersonHeightOutcome> = 
	(height: PersonHeightOutcome) => height >= 170;

const betweenOneFiftyAndOneSeventy: ContinuousVariableEvent<PersonHeightOutcome> = 
	(height: PersonHeightOutcome) => height >= 150 && height <= 170;

function has_event_happened<T>(event: ContinuousVariableEvent<T>, outcome: T) {
	return e(o);
}

const personHeightExperiment = new PersonHeightExperiment();
const heightOutcome = personHeightExperiment.trial();

console.log(has_event_happened<PersonHeightOutcome>(oneSeventyCMOrHigher, heightOutcome));
console.log(has_event_happened<PersonHeightOutcome>(betweenOneFiftyAndOneSeventy, heightOutcome));
```

### The Probability Measure

We finally speak about probability! The familiar word probability refers to the output of a function called the *probability measure*, aka *probability function*. The probability measure only makes sense within a probability space, which in turn only makes sense when defined against an experiment - hence why it took ~2000 words before we could start defining *probability*.

To further stress this: the output of a probability measure is a probability value. 

> Reminder: A function maps each element of a set $X$ to exactly one element of a set $Y$. Here, the set $X$ is the event space (the set of events), and set $Y$ is the real numbers in the range $[0, 1]$.

The probability measure takes in an event and outputs a real number: the probability value of that event happening.

### Tying it together

Tying it all together, here is what a probability space looks like.

```typescript
// ProbabilitySpace is defined over an outcome of type T. The outcome of
// type T is an output of some experiment we're studying.

type SampleSpace<T> = Set<T>;
type Event<T> = Set<T>;

interface ProbabilitySpace<T> {
	sampleSpace: SampleSpace<T>;
	eventSpace: Set<Event<T>>;
	
	// This function is of most interest. Unlike `experiment.trial()`,
	// where we are not optimistic about understanding its 
	// internal implementation, probabilityMeasure() is a function 
	// whose behavior we really want to understand.
	probabilityMeasure(event: Event<T>): number;
}
```

Here is how we can implement the exact probability space for our die roll experiment:

```typescript
type DieRollOutcome = 1 | 2 | 3 | 4 | 5 | 6;

const dieRollSampleSpace: SampleSpace<DieRollOutcome> = new Set([1, 2, 3, 4, 5, 6]);

const rollingAnEvenNumber: Event<DieRollOutcome> = new Set([2, 4, 6]);
const rollingAnOddNumber: Event<DieRollOutcome> = new Set([1, 3, 5]);

const dieRollEventSpace = new Set([
	dieRollSampleSpace,
	rollingAnEvenNumber, 
	rollingAnOddNumber,
	new Set() // The empty set is always a valid event
]);

const dieRollProbabilitySpace: ProbabilitySpace<DieRollOutcome> = {
	sampleSpace: dieRollSampleSpace,
	eventSpace: dieRollEventSpace,
	probabilityMeasure: (event: Event<DieRollOutcome>): number => {
		// Implementation is opaque
	}
};
```

### Mathematical Constraints

There are some intuitive constraints for a probability space to be a valid mathematical model of an experiment. In formal probability theory, For example:
1. **Non-negativity:** You cannot have a negative chance of something happening. The lowest possible chance is exactly 0.
2. **Normalization:** *Something* has to happen. The probability of getting *any* outcome in the entire sample space is 100% (or 1).
3. **Additivity:** If two events cannot possibly happen at the same time (e.g., rolling a 2 and rolling an odd number), the chance of *either* of them happening is just the addition of their individual probabilities.

I will also not dive deeper into the formalities of the constraints for now.
## Everything else is an abstraction

The concepts introduced above are the absolute minimum needed to define probability and use it to model problems. Everything else is an abstraction.

Abstractions inherently omit details and dilute the internal meanings of the thing they are trying to abstract. This makes reasoning about abstractions more difficult. The goal of abstraction is to make things easier to deal with and build upon, rather than to reason about.

This is going to be a theme of everything that follows. We will be introducing an abstraction and building on top of it. Each abstraction will dilute the meaning of the underlying concept, making it harder to understand intuitively, but much easier to build on mathematically.

### Random Variables

The first abstraction: **Random Variables**. A random variable is - in fact - not just a variable. It is a function (the output of that function is a variable, but the function itself is called a random variable, which I don't find very intuitive). It maps the outcomes of an experiment to a real number.

```typescript
// A random variable is a function mapping an outcome T to a number
type RandomVariable<T> = (outcome: T) => number;
```
In the die roll example, it doesn't add much value, since the outcome might be equal to the mapped random variable. It might be useful however if we wanted to round down the observed person height.
```typescript
const dieRollRV: RandomVariable<DieRollOutcome> = (outcome) => outcome;

const personHeightRV: RandomVariable<PersonHeightOutcome> = (outcome) => {
	// Maybe we just want to abstract away the decimals
	return Math.floor(outcome); 
};
```

Random variables are extremely useful when transforming one space into another. For example, transforming HTTP errors into numbers, or transforming a complex number space into a simpler one.

```typescript
const httpErrorRV: RandomVariable<string | null> = (outcome) => {
	if (outcome === null) return 0; // Success
	if (outcome.startsWith('APPLICATION_ERROR')) return 1;
	if (outcome.startsWith('DATABASE_ERROR')) return 2;
	return 3;
};
```

Instead of focusing on the specific outcome types (strings, custom objects, etc.), we now deal exclusively with real numbers. We no longer need to reason about the specific outcome type `T`.

This is a powerful abstraction because dealing with real numbers is a lot easier than dealing with arbitrary types. You get all the useful mathematical properties of numbers (addition, averaging, variance, etc.) for free!

Take this experiment as an example: customer feedback. Each trial corresponds to one customer feedback response in one of 4 categories: $\{poor, okay, good, excellent\}$. If you use a random variable to map these responses to $\{0, 1, 2, 3\}$ and calculate an *expected value** of 2.5, it means the *average sentiment* is falling between good and excellent. This is something you wouldn't easily be able to calculate in the original space of strings!

*\* We haven't introduced expected value yet.*

### Probability Distribution

A probability distribution is very analogous to the probability measure we introduced in the probability space. The main difference is that a probability distribution works on a different domain (a different input).

The input to a probability measure was an Event (remember: a set of outcomes from the sample space of some type). The input to a probability distribution is a set of **real numbers**. So they represent the exact same concept, but in a numerical domain.

i.e., after applying a random variable to a probability space, you get an *induced probability space* with the following properties:

```typescript
// Remember our original space:
interface ProbabilitySpace<T> {
	sampleSpace: Set<T>;
	eventSpace: Set<Set<T>>;
	probabilityMeasure(event: Set<T>): number;
}

// NOTE how `T` doesn't appear at all in the induced probability space!
// It has been completely abstracted away.
interface InducedProbabilitySpace {
	// The sample space is implicitly all of R (real numbers)
	
	// The canonical input is no longer an event of T, but literature 
	// typically denotes a set of real numbers as a Borel set `B`
	probabilityDistribution(B: Set<number>): number;
}

// This function demonstrates exactly how the abstraction wraps the original space:
function getInducedProbabilitySpace<T>(
	probSpace: ProbabilitySpace<T>,
	rv: RandomVariable<T>
): InducedProbabilitySpace {
	
	return {
		probabilityDistribution: (B: Set<number>): number => {
			// 1. Look backwards: Find all original outcomes that the 
			// random variable maps into our target set of numbers 'B'
			const mappedEvent = new Set<T>();
			
			for (const outcome of probSpace.sampleSpace) {
				if (B.has(rv(outcome))) {
					mappedEvent.add(outcome);
				}
			}
			
			// 2. Ask the original probability measure how likely that 
			// mapped event is to occur.
			return probSpace.probabilityMeasure(mappedEvent);
		}
	};
}
```

We often hear "probability distribution" used in correspondence with specific, well-known ones: normal distribution, binomial distribution, Poisson distribution, etc. These are not inherent to the definition of a probability distribution itself.

Every induced probability space has its own probability distribution. Some of the shapes of the resulting probability distributions just happen to be extremely common in nature and mathematics. It is **NOT** a must that your probability distribution follows one of these known ones*, but it might resemble one in certain circumstances. If it does, then congrats - you get a lot of free statistical insights from the existing mathematical knowledge of that specific distribution.

One important thing to note here is that the `InducedProbabilitySpace` no longer has `T` in the definition, nor a manual sample space. It is completely independent of our original experiment definition, which makes it incredibly easy to build upon!

### Probability Mass Function (PMF)

If you notice, the probability distribution function mathematically accepts a *set* of real numbers. 

A probability mass function (aka *PMF*) is a simpler function that also outputs a probability value, but it accepts a *single* real number rather than a set. 

It's important to note that probability mass functions are only defined for **discrete random variables**. i.e., it answers the exact probability of the random variable taking a specific, exact value. Like a die roll being exactly equal to 5: 

$$ P(X = 5) $$

### Probability Density Function (PDF)

A probability density function (aka *PDF*) is very similar to a PMF. It accepts a single real number as well, but it outputs the *density* rather than a direct probability.

Since continuous random variables have an infinite number of possible values they could take, the actual probability of getting any *single exact value* is **zero**. Therefore, we are less interested in the probability of one value. We are instead interested in the probability of the random variable falling within a *range*. 

The probability of falling in a range is mathematically defined as the integral (the area under the curve) of the PDF over that specific range:

$$ P(a < X < b) = \int_{a}^{b} f_X(x) \, dx $$

## Stopping Here

So far, we have introduced: 
* **Experiment, Trial, and Outcome**
* **Probability Space:** Sample Space, Event Space, and Probability Measure
* **Random Variable**
* **Induced Probability Space:** Probability Distribution
* **Probability Mass Function (PMF)**
* **Probability Density Function (PDF)**

These are the absolute basics of probability. Everything else is built on top of these abstractions. The goal of this article was to help you get more comfortable with the formal language, differentiate between confusingly similar terms, and bring your intuition closer to the mathematical rigor. This foundational knowledge should make navigating complex probability topics much easier.

In the next article, I will introduce one very specific distribution: the normal distribution. I will also cover a very central theorem: the central limit theorem (no pun intended). It's a fascinating phenomenon that emerges naturally across countless independent processes. We will build on top of and reuse the abstractions we introduced today. Without them, we simply wouldn't be able to describe the more useful, advanced pieces of probability.