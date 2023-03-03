This guide aims to assist anyone in creating a sequencer for a rhythm
game. A sequencer is a software that manages two crucial aspects of any
rhythm game:

1.  The timing of when the notes should be tapped concerning the
    beginning of the song.

2.  The placement of the notes at any given time.

In this guide, we will delve into these two aspects when introducing
gimmicks in the game. Gimmicks are alterations in these two aspects that
provide additional (usually aesthetic) possibilities to step makers.
Specifically, we will examine the StepMania approach to defining the
settings required to set up a sequencer. Each definition will be
analyzed in its own separate section, with examples provided, and
mathematical formalism that can be easily implemented in the programming
language of your choice at the end of each section.

This document’s sections will focus on two or, at most, three
mathematical spaces and the gimmicks that allow transformation from one
space to another.

# From song time to sequencer time

## Introduction

The `SSC` file provides two methods of artificially pausing the note
scrolling when the song is playing. Those gimmicks are referred as
`#STOPS` and `#DELAYS`. Although there is a subtle difference between
them, they are essentially identical w.r.t. the effect that produces.

One STOP definition might look as follows:

        #STOPS: 4=5,6=2;

Let us convert the definition into a JSON-like structure:

      {
        [
          beat: 4,
          stop: 5 
        ],
        [
          beat: 6,
          stop: 2 
        ]
      }

This `#STOPS` definition is telling us a couple of things.

1.  At beat 4, stop the scrolling for 5 seconds. Then resume.

2.  At beat 6, stop the scrolling for 2 seconds. Then resume.

Similarly, one DELAY defition could look like this:

        #DELAYS: 4=5,6=2;

And after converting it into the friendly structure:

      {
        [
          beat: 4,
          delay: 5 
        ],
        [
          beat: 6,
          delay: 2 
        ]
      }

The `#DELAYS` definition above is telling us essentially the same story
as the STOPS definition shown before, i.e.:

1.  At beat 4, stop the scrolling for 5 seconds. Then resume.

2.  At beat 6, stop the scrolling for 2 seconds. Then resume.

The difference between these two is that notes that lie exactly at the
stop/delay beat, will need to be tapped before and after the waiting
time for the stops and delays, respectively.

## Challenge

We would like to have a pair of functions
*t*<sub>(*s*)</sub>, *t*<sub>(*d*)</sub> : ℝ → ℝ that would retrieve the
song time after stops and delays (sequencer time) from the song time.
Additionally, we would like to have two inverse functions of
*t*<sub>(*s*)</sub>, *t*<sub>(*d*)</sub>,
*t*<sub>(*s*)</sub><sup>−1</sup> for STOPS and
*t*<sub>(*d*)</sub><sup>−1</sup> for DELAYS so we are able to map from
the sequencer time into the song time.

## Solution

To do so, let us imagine that we have a function *f* : ℝ → ℝ that given
a beat, it calculates the song time. To simplyfy things further, imagine
that this song is has constant BPM of 60, so each second is worth 1
beat. Then *f*(*b*) = *b*, being *b* a beat.

Having this in mind, then we can write the piecewise functions
$$t\_{(s)}(x) = t\_{(d)}(x) = \begin{dcases}
            x\\, & \text{if $ x \leq 4 $}\\; \\
            4\\, & \text{if $4 &lt;  x \leq 4+5 $}\\; \\
            x-5\\, & \text{if $4 + 5 &lt;  x \leq 6 + 5 $}\\; \\
            6+5\\, & \text{if $6 + 5 &lt;  x \leq 6 + 5+ 2 $}\\; \\
            x-2-5\\, & \text{if $ x &gt; 6+5+2 $}\\; \\
        \end{dcases}
        \label{l}$$
that will map from song time to sequencer time for the STOPS and DELAYS,
respectively. A plot of this function can be see in Figure
<a href="#fig:songtime2seqtime" data-reference-type="ref"
data-reference="fig:songtime2seqtime">1</a>.

<figure id="fig:songtime2seqtime">

<figcaption>Plot of <span
class="math inline"><em>t</em><sub>(<em>d</em>)</sub></span>
</figcaption>
</figure>

We can also easily calculate the two different inverse functions to
model when notes at stops or delays should be tapped w.r.t. the song
time. On the one hand side, we write the function

$$t^{-1}\_{(s)}(x) =\begin{dcases}
            x\\, &\text{if $ x \leq 4 $}\\;\\
            x+5\\,& \text{if $ 4 &lt; x \leq 6 $}\\;\\
            x+5+2\\,& \text{if $ x &gt; 6 $}\\;\\
        \end{dcases}
        \label{eq:tostops}$$
to map from the sequencer time into song time for STOPS. Similarly, the
function

$$t^{-1}\_{(d)}(x) =\begin{dcases}
            x\\, &\text{if $ x &lt; 4 $}\\;\\
            x+5\\,& \text{if $ 4 \leq x &lt; 6 $}\\;\\
            x+5+2\\,& \text{if $ x \geq 6 $}\\;\\
        \end{dcases}
        \label{eq:todelays}$$
will map from the sequencer time into the song time for DELAYS. Note
that the signs at the conditions are slightly different.

<figure id="fig:seqtime2songtime">

<figcaption>Plot of <span
class="math inline"><em>t</em><sup>−1</sup></span> </figcaption>
</figure>

## Formalization

Let
𝒯 = {(*b*<sub>*i*</sub><sup>(*t*)</sup>,*r*<sub>*i*</sub>)}<sub>*i* = 1</sub><sup>*n*</sup>
be a sequence of STOPS or DELAYS, where *r*<sub>*i*</sub> is the stop or
delay (measured in seconds) at beat *b*<sub>*i*</sub><sup>(*t*)</sup>.
Let *f* : 𝔹 → 𝕊<sup>\*</sup> be a function that retrieves the second
from the start of the song given a current beat, where 𝔹 = ℝ, and
𝕊<sup>\*</sup> = ℝ.

We define a new set
𝒯′ = {(*c*<sub>*i*</sub>,*r*<sub>*i*</sub>)}<sub>*i* = 1</sub><sup>*n*</sup> = {(*f*(*b*<sub>*i*</sub><sup>(*t*)</sup>),*r*<sub>*i*</sub>)}<sub>*i* = 1</sub><sup>*n*</sup>
where *c*<sub>*i*</sub> is the second from the start of the song of beat
*b*<sub>*i*</sub><sup>(*t*)</sup>. We define the functions
*t*<sub>(*s*)</sub> : 𝔻 → 𝕊<sup>\*</sup> and *t*<sub>(*d*)</sub> : 𝕊 → 𝔻
$$t\_{(s)}(x) = t\_{(d)}(x) = \begin{dcases}
        x - \sum\_{j=1}^{i-1}r\_j\\, & \text{if $ c\_{i-1} + \sum\_{j=1}^{i-1}r\_j &lt; x \leq c\_i + \sum\_{j=1}^{i-1}r\_j\\, \quad \forall i=1,\dots,n$}\\;\\
        c\_i\\, & \text{if $ c\_i + \sum\_{j=1}^{i-1}r\_j &lt; x \leq c\_i + \sum\_{j=1}^i r\_j\\,\quad \forall i=1,\dots,n $}\\;\\
        x - \sum\_{j=i}^n r\_j & \text{if $ x &gt; c\_n + \sum\_{j=1}^n r\_j $}\\; 
    \end{dcases}
    \label{eq:t}$$
when 𝒯′ ≠ ∅, where *c*<sub>0</sub> :=  − ∞, that maps from the song time
into the sequencer time.

We define the function
*t*<sub>(*s*)</sub><sup>−1</sup> : 𝕊<sup>\*</sup> → 𝔻
$$t\_{(s)}^{-1}(x) = \begin{dcases}
        x\\, & \text{if $ x \leq c\_1 $}\\;\\
        x+ \sum\_{j=1}^{i}r\_i\\, & \text{if $ c\_i &lt; x \leq c\_{i+1}\\,\quad \forall i=1,\dots,n$}\\;\\
    \end{dcases}
    \label{eq:t-1s}$$
with *c*<sub>*n* + 1</sub> := ∞ which maps from the sequencer time into
the song time for the STOPS, if 𝒯′ ≠ ∅, and the function
*t*<sub>(*d*)</sub><sup>−1</sup> : 𝔻 → 𝕊
$$t\_{(d)}^{-1}(x) = \begin{dcases}
        x\\, & \text{if $ x &lt; c\_1 $}\\;\\
        x+ \sum\_{j=1}^{i}r\_i\\, & \text{if $ c\_i \leq x &lt; c\_{i+1}\\,\quad \forall i=1,\dots,n$}\\;\\
    \end{dcases}
    \label{eq:t-1s}$$
that maps from the sequencer time into the song time for the DELAYS, if
𝒯′ ≠ ∅. If 𝒯′ = ∅, then
*t*<sub>(*s*)</sub>(*x*) = *t*<sub>(*s*)</sub><sup>−1</sup>(*x*) = *x*
and
*t*<sub>(*d*)</sub>(*x*) = *t*<sub>(*d*)</sub><sup>−1</sup>(*x*) = *x*,
for STOPS and DELAYS respectively. The final mapping from song time into
sequencer time is the composition of *t*<sub>(*d*)</sub> and
*t*<sub>(*s*)</sub>, *t*<sub>(*d*)</sub> ∘ *t*<sub>(*s*)</sub>.
Similarly, the mapping from sequencer time into the song time is
*t*<sub>(*s*)</sub><sup>−1</sup> ∘ 1<sub>(*d*)</sub><sup>−1</sup>.

# From sequencer time to beat

## Introduction

A `SSC` file gives a list of pairs which defines the bpms. The first
item in the pair is the target beat, and the second item is the desired
BPM from that beat on. Let us imagine we have a `SSC` file with the
following definition:

        #BPMS:0=120,8=70,13=200;     

Let us convert this cumbersome definition into a friendly structure:

      {
        [
          beat: 0,
          bpm: 120
        ],
        [
          beat: 8,
          bpm: 180 
        ],
        [
          beat: 13,
          bpm: 60 
        ]
      }

This `#BPMS` definition is telling us three things:

1.  From beat  − ∞ to beat 8, the BPM is 120.

2.  From beat 8 to beat 13, the BPM is 180.

3.  From beat 13 to beat  + ∞, the BPM is 60.

## Challenge

We want to find a function *f* : ℝ → ℝ that retrieves the current beat
given the a second in the sequencer time space. This function is useful
when a song is playing and we want to know at what beat we are at if we
know how much time has passed since the start of the song. Also, as we
will see later on, notes scroll at a BPM rate, so if we can have the
inverse function of *f*, *f*<sup>−1</sup>, we can sort of know when the
steps should be tapped as well.

## Solution

First, let us convert BPMS to BPSS (Beats Per Second), since we are
going to provide the input in seconds instead of minutes. We can do so
by dividing the BPMS by 60, i.e.
$$\text{BPS}(x) = x \times \frac{\text{Beats}}{\text{Minute}} = x \times \frac{1 \times \text{Minute}}{60 \times \text{Seconds}} \frac{\text{Beats}}{\text{Minute}} = \frac{x}{60} \times \frac{\text{Beats}}{\text{Second}}\\. 
        \label{eq:bpm2bps}$$

Next, let us define a piecewise function *f*′ : ℝ → ℝ that gives the
current BPS given the current Beat. Taking the `#BPMS` toy example from
the previous section, we get that
$$f'(x) = \begin{dcases}
            2\\, & \text{if $x \leq 8\\;$}\\ 
            3\\, & \text{if $8 &lt; x \leq 13\\;$}\\ 
            1\\, & \text{if $x &gt; 13\\.$}\\ 
        \end{dcases}
        \label{eq:beat2bps}$$

In Figure <a href="#fig:beat2bps" data-reference-type="ref"
data-reference="fig:beat2bps">3</a> you can see the plot of *f*′ we just
defined in <a href="#eq:beat2bps" data-reference-type="eqref"
data-reference="eq:beat2bps">[eq:beat2bps]</a>.

<figure id="fig:beat2bps">

<figcaption>Plot of <span
class="math inline"><em>f</em>′</span></figcaption>
</figure>

Note that by using *f*′, we can get the BPS at any beat of the song.
This is great, but it does not quite solve our problem.

Next, we can calculate the SPB (Seconds Per Beat) by just inversing the
BPS, i.e.
$$\text{SPB} = \frac{1}{\text{BPS}}\\,
    \label{eq:bps2spb}$$
and therefore we can define a function *t* : ℝ → ℝ

*t*(*x*) = *x* × SPB
that given a beat *x* retrieves the current second.

Let

$$f^{-1}(x) = \begin{dcases}
            \frac{x}{2}\\, & \text{if $x \leq 8\\;$}\\\[1em\]
            \frac{8}{2}+\frac{x-8}{3}\\, & \text{if $8 &lt; x \leq 13\\;$}\\\[1em\]  
            \frac{8}{2}+\frac{5}{3}+x- 13\\, & \text{if $x &gt; 13\\;$}\\ 
        \end{dcases}
        \label{eq:beat2second}$$
be the function that given a beat *x* retrieves the current second. This
function is the result of plugging
<a href="#eq:bps2spb" data-reference-type="eqref"
data-reference="eq:bps2spb">[eq:bps2spb]</a> and
<a href="#eq:beat2seconds" data-reference-type="eqref"
data-reference="eq:beat2seconds">[eq:beat2seconds]</a> into
<a href="#eq:beat2bps" data-reference-type="eqref"
data-reference="eq:beat2bps">[eq:beat2bps]</a>, and can be rewritten
recursively as

$$f^{-1}(x) = \begin{dcases}
            \frac{x}{2}\\, & \text{if $x \leq 8\\;$}\\\[1em\]
            f^{-1}(8)+\frac{x-8}{3}\\, & \text{if $8 &lt; x \leq 13\\;$}\\\[1em\]  
            f^{-1}(13) + x- 13\\, & \text{if $x &gt; 13\\.$}\\ 
        \end{dcases}
        \label{eq:beat2second}$$

Figure <a href="#fig:beat2second" data-reference-type="ref"
data-reference="fig:beat2second">4</a> depicts the function
*f*<sup>−1</sup>.

<figure id="fig:beat2second">

<figcaption>Plot of <span
class="math inline"><em>f</em><sup>−1</sup></span></figcaption>
</figure>

As it turns out, the function *f* that we are looking for is just the
inverse function of *f*<sup>−1</sup>, thus

$$f(x) = \begin{dcases}
            2x\\, & \text{if $x \leq 4\\;$}\\\[1em\]
            (x-4)\times 3 + 8\\, & \text{if $4 &lt; x \leq 5.6\\;$}\\\[1em\]  
            x-5.6 + 13\\, & \text{if $x &gt; 5.6\\.$}\\ 
        \end{dcases}
        \label{eq:beat2second}$$

A plot of *f* can be seen in Figure
<a href="#fig:second2beat" data-reference-type="ref"
data-reference="fig:second2beat">5</a>.

<figure id="fig:second2beat">

<figcaption>Plot of <span
class="math inline"><em>f</em></span></figcaption>
</figure>

## Formalization

Let
{(*b*<sub>*i*</sub>,*v*<sub>*i*</sub>)}<sub>*i* = 1</sub><sup>*n*</sup>
be a sequence of *n* beat signatures, where *v*<sub>*i*</sub> is the BPS
value at beat *b*<sub>*i*</sub>. Let
*f*<sup>−1</sup> : 𝔹 → 𝕊<sup>\*</sup> be a function that provided a
beat, returns the seconds in the sequencer time space. We define this
function as a *n*-step piecewise function
$$f^{-1}(x) = \begin{dcases}
            \frac{x}{v\_1}\\, & \text{if $x \leq b\_{2} $ }\\;\\
            f^{-1}(b\_{i}) + \frac{x-b\_{i}}{v\_i}\\, & \text{if $b\_{i} &lt; x \leq b\_{i+1}\\; \quad \forall i=2,\ldots,n $}\\; \\
        \end{dcases}
        \label{eq:beat2second}$$
where *b*<sub>1</sub> := 0, and *b*<sub>*n* + 1</sub> := ∞.

Analogously, let *f* : 𝕊<sup>\*</sup> → 𝔹 be a function that provided a
second in the sequencer time space, returns the beat from the zero
second. We define this function as a *n*-step piecewise function
$$f(x) = \begin{dcases}
            v\_1x\\, & \text{if $x \leq f^{-1}(b\_{2}) $ }\\;\\
            \left\[x-f^{-1}(b\_{i})\right\]\times v\_i + b\_{i}\\, & \text{if $f^{-1}(b\_{i}) &lt; x \leq f^{-1}(b\_{i+1})\\;\quad \forall i=2,\ldots,n$}\\. \\
        \end{dcases}
        \label{eq:beat2second}$$

# From beat to note position

## Introduction

There are a pair of stepmania definitions that influence the position
where notes should be placed on the screen. One of them is `#BPMS`,
which arguably is the rate at what notes travel upwards towards the
receptor w.r.t. to the music rhythm. As well see later on, we will
consider the BPMS as a speed function from which we can discover the
position of a note given the beat it should be tapped. As a toy example,
we will use the same definition as provided in Section
<a href="#sec:stepmania-definition-time2beat" data-reference-type="ref"
data-reference="sec:stepmania-definition-time2beat">2.1</a>.

However, there is another gimmick that plays a role in the note
positioning: `#SCROLLS`. Let’s have a look at an example:

        #SCROLLS:0=1,4=0,10=2;     

Again, let us convert this cumbersome definition into a friendly
structure:

      {
        [
          beat: 0,
          scroll: 1
        ],
        [
          beat: 4,
          scroll: 0 
        ],
        [
          beat: 10,
          scroll: 2 
        ]
      }

The scroll gimmick changes the effective BPM at a given beat by a rate
defined by the scroll value. Thus, in this example, the SCROLLS gimmick
is changing the BPMs as follows:

1.  From beat 0 to beat 4, the BPM is its value times 1.

2.  From beat 4 to beat 10, the BPM is its value times 0. (all the steps
    in between this beats, will have the same position)

3.  From beat 10 on, the BPM is its value times 2.

## Challenge

We would like to have a function *p* : ℝ → ℝ that given a beat, it
retrieves the position w.r.t. the origin (or where the receptor is)
where a note at that beat should be drawn.

## Solution

Let us start by defining a function *h* : ℝ → ℝ that retrieves the
current BPS (Beats per Second) from the current beat. Given the example,
this function is identical to that of
<a href="#eq:beat2bps" data-reference-type="eqref"
data-reference="eq:beat2bps">[eq:beat2bps]</a>,

$$h(x) = \begin{dcases}
            2\\, & \text{if $x \leq 8\\;$}\\ 
            3\\, & \text{if $8 &lt; x \leq 13\\;$}\\ 
            1\\, & \text{if $x &gt; 13\\.$}\\ 
        \end{dcases}
        \label{eq:beat2bps-as-h}$$

You can see a plot of this function in Figure
<a href="#fig:beat2bps" data-reference-type="ref"
data-reference="fig:beat2bps">3</a>. Now, let us define a new function
*g* : ℝ → ℝ that given a beat, retrieves the effective BPS (i.e., BPS
with applied scrolls). For that matter, we just need to check out in
what beats the scroll is taking place, and change the BPS accordingly.
For our toy example the resultant *g* function looks like this
$$g(x) = \begin{dcases}
            1\times 2\\, & \text{if $x \leq 4\\;$}\\ 
            0\times 2\\, & \text{if $4 &lt; x \leq 8\\;$}\\ 
            0\times 3\\, & \text{if $8 &lt; x \leq 10\\;$}\\ 
            2\times 3\\, & \text{if $10 &lt; x \leq 13\\;$}\\ 
            2\times 1 \\, & \text{if $x &gt; 13\\.$}\\ 
        \end{dcases}
        \label{eq:beat2effective-bps}$$

The function *g* is depicted in Figure
<a href="#fig:beat2effective-bps" data-reference-type="ref"
data-reference="fig:beat2effective-bps">6</a>.

<figure id="fig:beat2effective-bps">

<figcaption>Plot of <span class="math inline"><em>g</em></span> (blue
line), Plot of <span class="math inline"><em>h</em></span> (dotted, red
line)</figcaption>
</figure>

Now, this is great! By asking *g*, now we have the effective BPS. Note
that *g* is a function that models speed, i.e. the velocity that the
notes should move towards the receptor. To retrieve the position at each
beat, we can just consider that $\frac{dp}{dx} = g(x)$. Therefore, to
get *p*, we just have to take in integral of *g* w.r.t. *x*,
*p*(*x*) = ∫*g*(*x*)*d**x* .

For instance, if we want to know the position of a note at the beat 11,
it would only take to calculate
position = ∫<sub>0</sub><sup>11</sup>*g*(*x*)*d**x* .

You can see an example in Figure
<a href="#fig:example-area" data-reference-type="ref"
data-reference="fig:example-area">7</a>. As shown, the position at beat
11 is the area under the curve (in light blue) from 0 to 11.

<figure id="fig:example-area">

<figcaption>The position at beat 11 will be the area underneeth the
function <span class="math inline"><em>g</em></span>.</figcaption>
</figure>

The resulting integral of *g*(*x*) in our toy example is
$$p(x) = \begin{dcases}
            2x\\, & \text{if $x \leq 4\\;$}\\ 
            p(4) + 0\\, & \text{if $4 &lt; x \leq 8\\;$}\\ 
            p(8) + 0\\, & \text{if $8 &lt; x \leq 10\\;$}\\ 
            p(10) + (x-10)\times6 \\, & \text{if $10 &lt; x \leq 13\\;$}\\ 
            p(13) + (x-13) \times 1 \\, & \text{if $x &gt; 13\\,$}\\ 
        \end{dcases}
        \label{eq:beat2effective-bps}$$
and its plot can be seen in Figure
<a href="#fig:position-example" data-reference-type="ref"
data-reference="fig:position-example">8</a>.

<figure id="fig:position-example">

<figcaption>Plot of <span
class="math inline"><em>p</em></span></figcaption>
</figure>

## Formalization

Let
{(*b*<sub>*i*</sub><sup>(*b*)</sup>,*v*<sub>*i*</sub>)}<sub>*i* = 1</sub><sup>*n*</sup> = ℬ = *B*<sup>(*b*)</sup> × *V*
be a sequence of *n* beat signatures, where
*v*<sub>*i*</sub> ∈ *V* = {*v*<sub>*j*</sub>}<sub>*j* = 1</sub><sup>*n*</sup>
is the BPS value at beat
*b*<sub>*i*</sub><sup>(*b*)</sup> ∈ *B*<sup>(*b*)</sup> = {*b*<sub>*i*</sub><sup>(*b*)</sup>}<sub>*j* = 1</sub><sup>*n*</sup>.

Let

{(*b*<sub>*i*</sub><sup>(*s*)</sup>,*s*<sub>*i*</sub>)}<sub>*i* = 1</sub><sup>*m*</sup> = 𝒮 = *B*<sup>(*s*)</sup> × *S*
be a sequence of *m* scroll signatures, where
*s*<sub>*i*</sub> ∈ *S* = {*s*<sub>*j*</sub>}<sub>*j* = 1</sub><sup>*m*</sup>
is the scroll value at beat
*b*<sub>*i*</sub><sup>(*s*)</sup> ∈ *B*<sup>(*s*)</sup> = {*b*<sub>*j*</sub><sup>(*s*)</sup>}<sub>*j* = 1</sub><sup>*m*</sup>.

Let
{(*b*<sub>*i*</sub>,*z*<sub>*i*</sub>)}<sub>*i* = 1</sub><sup>*n*′</sup> = 𝒵 = *B*<sup>(*b*)</sup> ∪ *B*<sup>(*s*)</sup> × *V* ∪ *S*
be a sequence of *n*′ BPSs with applied scrolls, where *z*<sub>*i*</sub>
is the effective BPS at beat *b*<sub>*i*</sub> constructed from 𝒮 and ℬ
as
$$\mathcal{Z} = \bigcup\_{i=1}^{m} \left\\\left( b\_i^{(s)},h(b\_i^{(s)})\times s\_i \right)\right\\\bigcup\_{j=1}^{n} 
        \begin{dcases}
            \left\\ \left( b\_j^{(b)}, v\_j\times s\_i \right)\right\\\\, & \text{if $b\_i^{(s)} &lt; b\_j^{(b)} &lt; b\_{i+1}^{(s)}$}\\; \\
            \emptyset\\, & \text{otherwise}\\;
        \end{dcases}
        \label{eq:Z-construction}$$
where *h* : ℝ → ℝ is a function that given a beat, returns the BPS for
that beat, and *b*<sub>*m* + 1</sub><sup>(*s*)</sup> := ∞. We define
$$g(x) = \begin{dcases}
            z\_1\\, & \text{if $ x \leq b\_2 $}\\;\\
            z\_i\\, & \text{if $ b\_{i} &lt; x \leq b\_{i+1}\\; \quad \forall i=2,\dots,n'$}\\,
        \end{dcases}
        \label{eq:final-g}$$
as a function that retrieves the effective BPS given a beat, and
*b*<sub>*n*′ + 1</sub> := ∞. Finally, we define
$$p(x) = \int g(x) dx = \begin{dcases}
            xz\_1\\, & \text{if $ x \leq b\_2 $}\\;\\
            p(b\_i) + (x-b\_i)\times z\_i\\, & \text{if $ b\_{i} &lt; x \leq b\_{i+1}\\; \quad \forall i=2,\dots,n'$}\\,
        \end{dcases}
        \label{eq:position-final}$$
as the function that retrieves the position given a beat.

# From beat to scroll

## Introduction

Another issue that we might have is to know how far upwards we should
scroll the notes from its original position given the current beat. If
we did not have any other gimmicks, this would be very easy to compute.
Let us suppose that the notes redendered in the screen are squares of
one unity of length and height, and that one beat is worth one distance
of separation, as shown in Figure
<a href="#fig:notes-layout" data-reference-type="ref"
data-reference="fig:notes-layout">9</a>.

<figure id="fig:notes-layout">
<embed src="beat2scroll.pdf" />
<figcaption>Two notes inside with their respective bounding boxes
separeted apart by one beat.</figcaption>
</figure>

In this set up, the amount of scroll that we have to apply for a given
beat *x* is just  − *x* (if we were scrolling upwards, on the *y* axis).

However there are two gimmicks that modify this scroll function in two
different ways: `#WARPS` and `#SPEEDS`. Let us see and example to know
what these modifiers do. Suppose we have the following `#WARPS`
definition:

        #WARPS:4=2;     

It is equivalent to this one:

      {
        [
          beat: 4,
          warp: 2
        ]
      }

As it turns out, this gimmick is telling us the following information:

1.  At beat 4, warp over the next 2 following beats, i.e. skip two
    beats.

2.  Notes with beats in the range \[4, 4 + 2) = \[4, 6) will become fake
    notes.

On the other hand, suppose we have the following `#SPEEDS` definition:

        #SPEEDS:4=2=1=0,6=0.5=1=1;     

which is equivalent to this one:

      {
        [
          beat: 0,
          speed: 1,
          span: 0,
          type: 0
        ],
        [
          beat: 6,
          speed: 7 ,
          span: 1,
          type: 1
        ]
      }

Well, the information provided with this gimmick is the following:

1.  From beat 0 on, the separation between notes increases by a factor
    of 1, the scrolling speed also increases by a factor of 1, and
    because the `type` is `0`, this transition is smoothly applied in
    the span of 0 **beats**.

2.  From beat 6 on, the separation between notes increases by a factor
    of 7, the scrolling speed also increases by a factor of 7, and
    because the `type` is `1`, this transition is smoothly applied in
    the span of 1 **second**.

## Challenge

We would like to have a function *q* : ℝ → ℝ that calculates the scroll
value for a given beat, as well as a function *e* : ℝ → ℝ that
calculates the speed factor at a given beat.

## Solution

#### Warps

First, let us deal with the problem of the WARPS. We would need to
create a function that maps from beats to warped beats first, so we
could retrieve the real scroll value w.r.t. it. Actually, the function
that we are looking foris pretty much identical to that shown in Figure
<a href="#fig:seqtime2songtime" data-reference-type="ref"
data-reference="fig:seqtime2songtime">2</a>
$$q(x) = \begin{dcases}
            x\\, &\text{if $ x \leq 4 $}\\;\\
            x + 2\\, &\text{if $ x &gt; 4 $}\\.\\
        \end{dcases}
        \label{eq:warps-example}$$

You can see the function plotted in Figure
<a href="#fig:warps-example" data-reference-type="ref"
data-reference="fig:warps-example">10</a>.

<figure id="fig:warps-example">

<figcaption>Plot of <span
class="math inline"><em>q</em></span></figcaption>
</figure>

And that is that. Actually, the output of this function is already the
scroll value that we are looking for! It is inverse
$$q^{-1}(x) =  \begin{dcases}
        x\\, & \text{if $ x \leq 4 $}\\;\\
        4\\, & \text{if $ 4 &lt; x \leq 4+2 $}\\;\\
        x-2\\, & \text{if $ x &gt; 4+2$}\\;\\
    \end{dcases}
    \label{eq:warps-inverse}$$
also is very similar to how we calculated
*t*<sub>(*s*)</sub><sup>−1</sup> in Section
<a href="#sec:songtime2seqtime" data-reference-type="ref"
data-reference="sec:songtime2seqtime">1</a>. You can visualize it in
Figure <a href="#fig:warps-example-inverse" data-reference-type="ref"
data-reference="fig:warps-example-inverse">11</a>.

<figure id="fig:warps-example-inverse">

<figcaption>Plot of <span
class="math inline"><em>q</em><sup>−1</sup></span></figcaption>
</figure>

#### Speeds

The function that we need to come up for the speeds is simple as well.
However, it gets a bit complicated when dealing with speeds of `type` 1:
Since we are working at the beat space, we need somehow to convert a
span of seconds into a span of beats so we can perform operations in the
same units. We do so by mapping a given beat into the song time space,
then add the span of seconds, and then mapping back into the beat space.
To keep things simple, let us imagine that we are running a 60 BPM song
with no delays or stops. In this particular case, we can write
$$e(x) = \begin{dcases}
        1\\, & \text{if $ x \leq 6 $}\\;\\
        \frac{7-1}{1}(x-6)+1\\, & \text{if $ 6 &lt; x \leq 6+1 $}\\;\\
        7\\, & \text{if $ x &gt; 6+1 $}\\.
    \end{dcases}
    \label{eq:example-speeds}$$

<figure id="fig:speeds-example">

<figcaption>Plot of <span
class="math inline"><em>e</em></span></figcaption>
</figure>

## Formalization

#### Warps

Let
𝒲 = {(*b*<sub>*i*</sub><sup>(*w*)</sup>,*w*<sub>*i*</sub>)}<sub>*i* = 1</sub><sup>*n*</sup>
be a sequence of WARPS, where *w*<sub>*i*</sub> is the warp (measured in
beats) at beat *b*<sub>*i*</sub><sup>(*w*)</sup>.

We define the function *q* : 𝔹 → 𝕋
$$q(x) = \begin{dcases}
        x\\, & \text{if $ x \leq b^{(w)}\_1 $}\\;\\
        x+ \sum\_{j=1}^{i}w\_i\\, & \text{if $ b^{(w)}\_i &lt; x \leq b^{(w)}\_{i+1}\\,\quad \forall i=1,\dots,n$}\\;\\
    \end{dcases}
    \label{eq:q-1}$$
with *b*<sub>*n* + 1</sub><sup>(*w*)</sup> := ∞ which maps from the beat
space into the translation space, if 𝒲 ≠ ∅.

We define the function *q*<sup>−1</sup> : 𝕋 → 𝔹
$$q^{-1}(x) = \begin{dcases}
        x - \sum\_{j=1}^{i-1}w\_j\\, & \text{if $ b^{(w)}\_{i-1} + \sum\_{j=1}^{i-1}w\_j &lt; x \leq b^{(w)}\_i + \sum\_{j=1}^{i-1}w\_j\\, \quad \forall i=1,\dots,n$}\\;\\
        b^{(w)}\_i\\, & \text{if $ b^{(w)}\_i + \sum\_{j=1}^{i-1}w\_j &lt; x \leq b^{(w)}\_i + \sum\_{j=1}^i w\_j\\,\quad \forall i=1,\dots,n $}\\;\\
        x - \sum\_{j=i}^n w\_j & \text{if $ x &gt; b^{(w)}\_n + \sum\_{j=1}^n w\_j $}\\; 
    \end{dcases}
    \label{eq:q}$$
when 𝒲 ≠ ∅, where *b*<sub>0</sub><sup>(*w*)</sup> :=  − ∞, that maps
from the translation space into the beat space.

If 𝒲 = ∅, then *q*(*x*) = *q*<sup>−1</sup>(*x*) = *x*.

#### Speeds

Let
ℰ = {(*b*<sub>*i*</sub><sup>(*e*)</sup>,*s*<sub>*i*</sub>,*p*<sub>*i*</sub>,*t*<sub>*i*</sub>)}<sub>*i* = 1</sub><sup>*n*</sup>
be a sequence of SPEEDS, where *s*<sub>*i*</sub> is the speed change at
beat *b*<sub>*i*</sub><sup>(*e*)</sup> with a span transition of
*p*<sub>*i*</sub> beats if *t*<sub>*i*</sub> = 0, or *p*<sub>*i*</sub>
seconds if *t*<sub>*i*</sub> = 1.

We define a new sequence
ℰ′ = {(*b*<sub>*i*</sub><sup>(*e*)</sup>,*s*<sub>*i*</sub>,*p*′<sub>*i*</sub>)}<sub>*i* = 1</sub><sup>*n*</sup>
from ℰ as
$$\mathcal{E'} = \bigcup\_{i=1}^{n} \begin{dcases}
        \left\\\left( b\_i^{(e)}, s\_i, p\_i \right)\right\\\\, & \text{if $ t\_i =0 $}\\;\\
        \left\\\left( b\_i^{(e)}, s\_i,g \left( p\_i, b\_i^{(e)} \right)  \right)\right\\\\, & \text{otherwise}\\,\\
    \end{dcases}
    \label{eq:eprime}$$
where
*g*(*p*,*b*) = (*f*∘*t*<sub>(*s*)</sub>∘*t*<sub>(*d*)</sub>)((*t*<sub>(*d*)</sub><sup>−1</sup>∘*t*<sub>(*s*)</sub><sup>−1</sup>∘*f*<sup>−1</sup>)(*b*)+*p*) .

We define the function *e* : 𝔹 → 𝔼
$$e(x) = \begin{dcases}
        s\_1\\, & \text{if $ x\leq b\_1^{(e)} $}\\;\\
        \frac{s\_i-s\_{i-1}}{p'\_i} \left( x-b\_{i}^{(e)} \right)+s\_{i-1} \\, & \text{if $ b\_{i}^{(e)} &lt; x \leq b\_{i}^{(e)}+p'\_i \land p'\_i \neq 0\\,\quad \forall i=1,\dots,n$}\\;\\
        s\_i\\, &\text{if $ b\_{i}^{(e)}+p'\_i &lt; x \leq b\_{i+1}^{(e)}\\,\quad \forall i=1,\dots,n$}\\,\\
    \end{dcases}
    \label{eq:e}$$
where *b*<sub>*n* + 1</sub><sup>(*e*)</sup> := ∞, and
*s*<sub>0</sub> := *s*<sub>1</sub>.

# Positioning and scrolling notes

Alright! Now we have formally defined everything we need to build a
sequencer using stepmania’s notation. This section will show how to use
all functions defined formally in the sections above to determine where
to draw the notes in the screen at any given song time.

To ease the understanding, Table
<a href="#tab:symbol-table" data-reference-type="ref"
data-reference="tab:symbol-table">1</a> gathers all the functions that
we are going to need as well as a brief descripcion for each one of
them.

<table>
<caption>Table of functions and spaces.</caption>
<thead>
<tr class="header">
<th style="text-align: left;">Symbol</th>
<th style="text-align: left;">Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;"><span class="math inline">𝕊</span></td>
<td style="text-align: left;">Song time space (s) <span
class="math inline">𝕊 = ℝ</span>.</td>
</tr>
<tr class="even">
<td style="text-align: left;"><span class="math inline">𝔻</span></td>
<td style="text-align: left;">Delayed time space (s) <span
class="math inline">𝔻 = ℝ</span>.</td>
</tr>
<tr class="odd">
<td style="text-align: left;"><span
class="math inline">𝕊<sup>*</sup></span></td>
<td style="text-align: left;">Sequencer time space (s) <span
class="math inline">𝕊<sup>*</sup> = ℝ</span>.</td>
</tr>
<tr class="even">
<td style="text-align: left;"><span class="math inline">𝔹</span></td>
<td style="text-align: left;">Beat space (beats) <span
class="math inline">𝔹 = ℝ</span>.</td>
</tr>
<tr class="odd">
<td style="text-align: left;"><span class="math inline">ℙ</span></td>
<td style="text-align: left;">Position space (units) <span
class="math inline">ℙ = ℝ</span>.</td>
</tr>
<tr class="even">
<td style="text-align: left;"><span class="math inline">𝕋</span></td>
<td style="text-align: left;">Translation space (units) <span
class="math inline">𝕋 = ℝ</span>.</td>
</tr>
<tr class="odd">
<td style="text-align: left;"><span class="math inline">𝔼</span></td>
<td style="text-align: left;">Speed space (speed factor units) <span
class="math inline">𝔼 = ℝ</span>.</td>
</tr>
<tr class="even">
<td style="text-align: left;"><span
class="math inline"><em>t</em><sub>(<em>d</em>)</sub></span></td>
<td style="text-align: left;">Function <span
class="math inline"><em>t</em><sub>(<em>s</em>)</sub> : 𝕊 → 𝔻</span>
that maps from the song time space into the delayed time space.</td>
</tr>
<tr class="odd">
<td style="text-align: left;"><span
class="math inline"><em>t</em><sub>(<em>d</em>)</sub><sup>−1</sup></span></td>
<td style="text-align: left;">Function <span
class="math inline"><em>t</em><sub>(<em>d</em>)</sub><sup>−1</sup> : 𝔻 → 𝕊</span>
that maps from the the delayed time space into the song time space.</td>
</tr>
<tr class="even">
<td style="text-align: left;"><span
class="math inline"><em>t</em><sub>(<em>s</em>)</sub></span></td>
<td style="text-align: left;">Function <span
class="math inline"><em>t</em><sub>(<em>s</em>)</sub> : 𝔻 → 𝕊<sup>*</sup></span>
that maps from the delayed time space into the sequencer time
space.</td>
</tr>
<tr class="odd">
<td style="text-align: left;"><span
class="math inline"><em>t</em><sub>(<em>s</em>)</sub><sup>−1</sup></span></td>
<td style="text-align: left;">Function <span
class="math inline"><em>t</em><sub>(<em>s</em>)</sub><sup>−1</sup> : 𝕊<sup>*</sup> → 𝔻</span>
that maps from the the sequencer time space into the delayed time
space.</td>
</tr>
<tr class="even">
<td style="text-align: left;"><span
class="math inline"><em>f</em></span></td>
<td style="text-align: left;">Function <span
class="math inline"><em>f</em> : 𝕊<sup>*</sup> → 𝔹</span> that maps from
the sequencer time space into the beat space.</td>
</tr>
<tr class="odd">
<td style="text-align: left;"><span
class="math inline"><em>f</em><sup>−1</sup></span></td>
<td style="text-align: left;">Function <span
class="math inline"><em>f</em><sup>−1</sup> : 𝔹 → 𝕊<sup>*</sup></span>
that maps from the beat space into the sequencer time space.</td>
</tr>
<tr class="even">
<td style="text-align: left;"><span
class="math inline"><em>p</em></span></td>
<td style="text-align: left;">Function <span
class="math inline"><em>p</em> : 𝔹 → ℙ</span> that maps from the beat
space into the position space.</td>
</tr>
<tr class="odd">
<td style="text-align: left;"><span
class="math inline"><em>q</em></span></td>
<td style="text-align: left;">Function <span
class="math inline"><em>q</em> : 𝔹 → 𝕋</span> that maps from the beat
space into the translation space.</td>
</tr>
<tr class="even">
<td style="text-align: left;"><span
class="math inline"><em>q</em><sup>−1</sup></span></td>
<td style="text-align: left;">Function <span
class="math inline"><em>q</em><sup>−1</sup> : 𝕋 → 𝔹</span> that maps
from the translation space into the beat space.</td>
</tr>
<tr class="odd">
<td style="text-align: left;"><span
class="math inline"><em>e</em></span></td>
<td style="text-align: left;">Function <span
class="math inline"><em>e</em> : 𝔹 → 𝔼</span> that maps from the beat
space into the speed space.</td>
</tr>
</tbody>
</table>

Table of functions and spaces.

We will take the following assumptions:

1.  We are modeling a rhythm game whose notes scroll upwards.

2.  The receptor is placed at the origin of the scrolling axis.

3.  The beat 0 has a position 0 in the scrolling axis.

Let 𝒩 = {(*u*<sub>*i*</sub>)}<sub>*i* = 1</sub><sup>*n*</sup> be a
sequence of notes where *u*<sub>*i*</sub> is the beat when the *i*-th
note should be tapped. We define a new set
𝒩′ = {(*u*<sub>*i*</sub>,*v*<sub>*i*</sub>,*w*<sub>*i*</sub>)} = ⋃<sub>{*u*} ∈ 𝒩</sub>{(*u*,*g*(*u*),*p*(*u*))}
where *g* : 𝕋 → 𝕊
*g*(*x*) = (*t*<sub>(*d*)</sub><sup>−1</sup>∘*t*<sub>(*s*)</sub><sup>−1</sup>∘*f*<sup>−1</sup>∘*q*<sup>−1</sup>)(*x*) ,
*v*<sub>*i*</sub> is the exact time where the *i*-th note should be
tapped, and *w*<sub>*i*</sub> is its relative position at beat 0 in the
scrolling axis. At a given moment in time *t* w.r.t. to the begginig of
the song, the *i*-th note should be positioned at
\[*w*<sub>*i*</sub>−(*q*∘*f*∘*t*<sub>(*s*)</sub>∘*t*<sub>(*d*)</sub>)(*t*)\] × (*e*∘*q*∘*f*∘*t*<sub>(*s*)</sub>∘*t*<sub>(*d*)</sub>)(*t*) .
