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

This document's sections will focus on two or, at most, three
mathematical spaces and the gimmicks that allow transformation from one
space to another.

# From song time to sequencer time {#sec:songtime2seqtime}

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
$t_{(s)}, t_{(d)}: \mathbb{R} \rightarrow \mathbb{R}$ that would
retrieve the song time after stops and delays (sequencer time) from the
song time. Additionally, we would like to have two inverse functions of
$t_{(s)}, t_{(d)}$, $t^{-1}_{(s)}$ for STOPS and $t^{-1}_{(d)}$ for
DELAYS so we are able to map from the sequencer time into the song time.

## Solution

To do so, let us imagine that we have a function
$f : \mathbb{R}\rightarrow \mathbb{R}$ that given a beat, it calculates
the song time. To simplyfy things further, imagine that this song is has
constant BPM of 60, so each second is worth 1 beat. Then $f(b) = b$,
being $b$ a beat.

Having this in mind, then we can write the piecewise functions
$$t_{(s)}(x) = t_{(d)}(x) = \begin{dcases}
            x\,, & \text{if $ x \leq 4 $}\,; \\
            4\,, & \text{if $4 <  x \leq 4+5 $}\,; \\
            x-5\,, & \text{if $4 + 5 <  x \leq 6 + 5 $}\,; \\
            6+5\,, & \text{if $6 + 5 <  x \leq 6 + 5+ 2 $}\,; \\
            x-2-5\,, & \text{if $ x > 6+5+2 $}\,; \\
        \end{dcases}
        \label{l}$$ that will map from song time to sequencer time for
the STOPS and DELAYS, respectively. A plot of this function can be see
in Figure [1](#fig:songtime2seqtime){reference-type="ref"
reference="fig:songtime2seqtime"}.

<figure id="fig:songtime2seqtime">

<figcaption>Plot of <span
class="math inline"><em>t</em><sub>(<em>d</em>)</sub></span>
</figcaption>
</figure>

We can also easily calculate the two different inverse functions to
model when notes at stops or delays should be tapped w.r.t. the song
time. On the one hand side, we write the function

$$t^{-1}_{(s)}(x) =\begin{dcases}
            x\,, &\text{if $ x \leq 4 $}\,;\\
            x+5\,,& \text{if $ 4 < x \leq 6 $}\,;\\
            x+5+2\,,& \text{if $ x > 6 $}\,;\\
        \end{dcases}
        \label{eq:tostops}$$ to map from the sequencer time into song
time for STOPS. Similarly, the function

$$t^{-1}_{(d)}(x) =\begin{dcases}
            x\,, &\text{if $ x < 4 $}\,;\\
            x+5\,,& \text{if $ 4 \leq x < 6 $}\,;\\
            x+5+2\,,& \text{if $ x \geq 6 $}\,;\\
        \end{dcases}
        \label{eq:todelays}$$ will map from the sequencer time into the
song time for DELAYS. Note that the signs at the conditions are slightly
different.

<figure id="fig:seqtime2songtime">

<figcaption>Plot of <span
class="math inline"><em>t</em><sup>−1</sup></span> </figcaption>
</figure>

## Formalization

Let
$\mathcal{T} =  \left\{\left( b_i^{(t)}, r_i \right)\right\}_{i=1}^{n}$
be a sequence of STOPS or DELAYS, where $r_i$ is the stop or delay
(measured in seconds) at beat $b_i^{(t)}$. Let
$f:\mathbb{B}\rightarrow \mathbb{S^{*}}$ be a function that retrieves
the second from the start of the song given a current beat, where
$\mathbb{B} = \mathbb{R}$, and $\mathbb{S^{*}} = \mathbb{R}$.

We define a new set
$$\mathcal{T'} = \left\{\left( c_i, r_i \right)\right\}_{i=1}^{n} = \left\{\left( f\left( b_i^{(t)}\right), r_i \right)\right\}_{i=1}^{n}
    \label{eq:tprimeset}$$ where $c_i$ is the second from the start of
the song of beat $b_i^{(t)}$. We define the functions
$t_{(s)}: \mathbb{D}\rightarrow \mathbb{S^{*}}$ and
$t_{(d)}: \mathbb{S}\rightarrow \mathbb{D}$
$$t_{(s)}(x) = t_{(d)}(x) = \begin{dcases}
        x - \sum_{j=1}^{i-1}r_j\,, & \text{if $ c_{i-1} + \sum_{j=1}^{i-1}r_j < x \leq c_i + \sum_{j=1}^{i-1}r_j\,, \quad \forall i=1,\dots,n$}\,;\\
        c_i\,, & \text{if $ c_i + \sum_{j=1}^{i-1}r_j < x \leq c_i + \sum_{j=1}^i r_j\,,\quad \forall i=1,\dots,n $}\,;\\
        x - \sum_{j=i}^n r_j & \text{if $ x > c_n + \sum_{j=1}^n r_j $}\,; 
    \end{dcases}
    \label{eq:t}$$ when $\mathcal{T'} \neq \emptyset$, where
$c_0 := -\infty$, that maps from the song time into the sequencer time.

We define the function
$t_{(s)}^{-1}: \mathbb{S^{*}}\rightarrow \mathbb{D}$
$$t_{(s)}^{-1}(x) = \begin{dcases}
        x\,, & \text{if $ x \leq c_1 $}\,;\\
        x+ \sum_{j=1}^{i}r_i\,, & \text{if $ c_i < x \leq c_{i+1}\,,\quad \forall i=1,\dots,n$}\,;\\
    \end{dcases}
    \label{eq:t-1s}$$ with $c_{n+1} := \infty$ which maps from the
sequencer time into the song time for the STOPS, if
$\mathcal{T'} \neq \emptyset$, and the function
$t_{(d)}^{-1}: \mathbb{D}\rightarrow \mathbb{S}$
$$t_{(d)}^{-1}(x) = \begin{dcases}
        x\,, & \text{if $ x < c_1 $}\,;\\
        x+ \sum_{j=1}^{i}r_i\,, & \text{if $ c_i \leq x < c_{i+1}\,,\quad \forall i=1,\dots,n$}\,;\\
    \end{dcases}
    \label{eq:t-1s}$$ that maps from the sequencer time into the song
time for the DELAYS, if $\mathcal{T'}\neq \emptyset$. If
$\mathcal{T'} = \emptyset$, then $t_{(s)}(x) = t_{(s)}^{-1}(x) = x$ and
$t_{(d)}(x) = t_{(d)}^{-1} (x) = x$, for STOPS and DELAYS respectively.
The final mapping from song time into sequencer time is the composition
of $t_{(d)}$ and $t_{(s)}$, $t_{(d)}\circ t_{(s)}$. Similarly, the
mapping from sequencer time into the song time is
$t_{(s)}^{-1}\circ 1_{(d)}^{-1}$.

# From sequencer time to beat

## Introduction {#sec:stepmania-definition-time2beat}

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

1.  From beat $- \infty$ to beat 8, the BPM is 120.

2.  From beat 8 to beat 13, the BPM is 180.

3.  From beat 13 to beat $+\infty$, the BPM is 60.

## Challenge

We want to find a function $f : \mathbb{R} \rightarrow \mathbb{R}$ that
retrieves the current beat given the a second in the sequencer time
space. This function is useful when a song is playing and we want to
know at what beat we are at if we know how much time has passed since
the start of the song. Also, as we will see later on, notes scroll at a
BPM rate, so if we can have the inverse function of $f$, $f^{-1}$, we
can sort of know when the steps should be tapped as well.

## Solution

First, let us convert BPMS to BPSS (Beats Per Second), since we are
going to provide the input in seconds instead of minutes. We can do so
by dividing the BPMS by 60, i.e.
$$\text{BPS}(x) = x \times \frac{\text{Beats}}{\text{Minute}} = x \times \frac{1 \times \text{Minute}}{60 \times \text{Seconds}} \frac{\text{Beats}}{\text{Minute}} = \frac{x}{60} \times \frac{\text{Beats}}{\text{Second}}\,. 
        \label{eq:bpm2bps}$$

Next, let us define a piecewise function
$f': \mathbb{R} \rightarrow \mathbb{R}$ that gives the current BPS given
the current Beat. Taking the `#BPMS` toy example from the previous
section, we get that $$f'(x) = \begin{dcases}
            2\,, & \text{if $x \leq 8\,;$}\\ 
            3\,, & \text{if $8 < x \leq 13\,;$}\\ 
            1\,, & \text{if $x > 13\,.$}\\ 
        \end{dcases}
        \label{eq:beat2bps}$$

In Figure [3](#fig:beat2bps){reference-type="ref"
reference="fig:beat2bps"} you can see the plot of $f'$ we just defined
in [\[eq:beat2bps\]](#eq:beat2bps){reference-type="eqref"
reference="eq:beat2bps"}.

<figure id="fig:beat2bps">

<figcaption>Plot of <span
class="math inline"><em>f</em>′</span></figcaption>
</figure>

Note that by using $f'$, we can get the BPS at any beat of the song.
This is great, but it does not quite solve our problem.

Next, we can calculate the SPB (Seconds Per Beat) by just inversing the
BPS, i.e. $$\text{SPB} = \frac{1}{\text{BPS}}\,,
    \label{eq:bps2spb}$$ and therefore we can define a function
$t: \mathbb{R} \rightarrow \mathbb{R}$

$$t(x) = x\times \text{SPB}
    \label{eq:beat2seconds}$$ that given a beat $x$ retrieves the
current second.

Let

$$f^{-1}(x) = \begin{dcases}
            \frac{x}{2}\,, & \text{if $x \leq 8\,;$}\\[1em]
            \frac{8}{2}+\frac{x-8}{3}\,, & \text{if $8 < x \leq 13\,;$}\\[1em]  
            \frac{8}{2}+\frac{5}{3}+x- 13\,, & \text{if $x > 13\,;$}\\ 
        \end{dcases}
        \label{eq:beat2second}$$ be the function that given a beat $x$
retrieves the current second. This function is the result of plugging
[\[eq:bps2spb\]](#eq:bps2spb){reference-type="eqref"
reference="eq:bps2spb"} and
[\[eq:beat2seconds\]](#eq:beat2seconds){reference-type="eqref"
reference="eq:beat2seconds"} into
[\[eq:beat2bps\]](#eq:beat2bps){reference-type="eqref"
reference="eq:beat2bps"}, and can be rewritten recursively as

$$f^{-1}(x) = \begin{dcases}
            \frac{x}{2}\,, & \text{if $x \leq 8\,;$}\\[1em]
            f^{-1}(8)+\frac{x-8}{3}\,, & \text{if $8 < x \leq 13\,;$}\\[1em]  
            f^{-1}(13) + x- 13\,, & \text{if $x > 13\,.$}\\ 
        \end{dcases}
        \label{eq:beat2second}$$

Figure [4](#fig:beat2second){reference-type="ref"
reference="fig:beat2second"} depicts the function $f^{-1}$.

<figure id="fig:beat2second">

<figcaption>Plot of <span
class="math inline"><em>f</em><sup>−1</sup></span></figcaption>
</figure>

As it turns out, the function $f$ that we are looking for is just the
inverse function of $f^{-1}$, thus

$$f(x) = \begin{dcases}
            2x\,, & \text{if $x \leq 4\,;$}\\[1em]
            (x-4)\times 3 + 8\,, & \text{if $4 < x \leq 5.6\,;$}\\[1em]  
            x-5.6 + 13\,, & \text{if $x > 5.6\,.$}\\ 
        \end{dcases}
        \label{eq:beat2second}$$

A plot of $f$ can be seen in Figure
[5](#fig:second2beat){reference-type="ref" reference="fig:second2beat"}.

<figure id="fig:second2beat">

<figcaption>Plot of <span
class="math inline"><em>f</em></span></figcaption>
</figure>

## Formalization

Let $\{\left( b_i, v_i \right)\}_{i=1}^{n}$ be a sequence of $n$ beat
signatures, where $v_i$ is the BPS value at beat $b_i$. Let
$f^{-1}: \mathbb{B} \rightarrow \mathbb{S^{*}}$ be a function that
provided a beat, returns the seconds in the sequencer time space. We
define this function as a $n$-step piecewise function
$$f^{-1}(x) = \begin{dcases}
            \frac{x}{v_1}\,, & \text{if $x \leq b_{2} $ }\,;\\
            f^{-1}(b_{i}) + \frac{x-b_{i}}{v_i}\,, & \text{if $b_{i} < x \leq b_{i+1}\,; \quad \forall i=2,\ldots,n $}\,; \\
        \end{dcases}
        \label{eq:beat2second}$$ where $b_1 := 0$, and
$b_{n+1} := \infty$.

Analogously, let $f: \mathbb{S^{*}} \rightarrow \mathbb{B}$ be a
function that provided a second in the sequencer time space, returns the
beat from the zero second. We define this function as a $n$-step
piecewise function $$f(x) = \begin{dcases}
            v_1x\,, & \text{if $x \leq f^{-1}(b_{2}) $ }\,;\\
            \left[x-f^{-1}(b_{i})\right]\times v_i + b_{i}\,, & \text{if $f^{-1}(b_{i}) < x \leq f^{-1}(b_{i+1})\,;\quad \forall i=2,\ldots,n$}\,. \\
        \end{dcases}
        \label{eq:beat2second}$$

# From beat to note position

## Introduction {#introduction-1}

There are a pair of stepmania definitions that influence the position
where notes should be placed on the screen. One of them is `#BPMS`,
which arguably is the rate at what notes travel upwards towards the
receptor w.r.t. to the music rhythm. As well see later on, we will
consider the BPMS as a speed function from which we can discover the
position of a note given the beat it should be tapped. As a toy example,
we will use the same definition as provided in Section
[2.1](#sec:stepmania-definition-time2beat){reference-type="ref"
reference="sec:stepmania-definition-time2beat"}.

However, there is another gimmick that plays a role in the note
positioning: `#SCROLLS`. Let's have a look at an example:

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

We would like to have a function $p: \mathbb{R} \rightarrow \mathbb{R}$
that given a beat, it retrieves the position w.r.t. the origin (or where
the receptor is) where a note at that beat should be drawn.

## Solution

Let us start by defining a function
$h: \mathbb{R} \rightarrow \mathbb{R}$ that retrieves the current BPS
(Beats per Second) from the current beat. Given the example, this
function is identical to that of
[\[eq:beat2bps\]](#eq:beat2bps){reference-type="eqref"
reference="eq:beat2bps"},

$$h(x) = \begin{dcases}
            2\,, & \text{if $x \leq 8\,;$}\\ 
            3\,, & \text{if $8 < x \leq 13\,;$}\\ 
            1\,, & \text{if $x > 13\,.$}\\ 
        \end{dcases}
        \label{eq:beat2bps-as-h}$$

You can see a plot of this function in Figure
[3](#fig:beat2bps){reference-type="ref" reference="fig:beat2bps"}. Now,
let us define a new function $g: \mathbb{R}\rightarrow \mathbb{R}$ that
given a beat, retrieves the effective BPS (i.e., BPS with applied
scrolls). For that matter, we just need to check out in what beats the
scroll is taking place, and change the BPS accordingly. For our toy
example the resultant $g$ function looks like this
$$g(x) = \begin{dcases}
            1\times 2\,, & \text{if $x \leq 4\,;$}\\ 
            0\times 2\,, & \text{if $4 < x \leq 8\,;$}\\ 
            0\times 3\,, & \text{if $8 < x \leq 10\,;$}\\ 
            2\times 3\,, & \text{if $10 < x \leq 13\,;$}\\ 
            2\times 1 \,, & \text{if $x > 13\,.$}\\ 
        \end{dcases}
        \label{eq:beat2effective-bps}$$

The function $g$ is depicted in Figure
[6](#fig:beat2effective-bps){reference-type="ref"
reference="fig:beat2effective-bps"}.

<figure id="fig:beat2effective-bps">

<figcaption>Plot of <span class="math inline"><em>g</em></span> (blue
line), Plot of <span class="math inline"><em>h</em></span> (dotted, red
line)</figcaption>
</figure>

Now, this is great! By asking $g$, now we have the effective BPS. Note
that $g$ is a function that models speed, i.e. the velocity that the
notes should move towards the receptor. To retrieve the position at each
beat, we can just consider that $\frac{dp}{dx} = g(x)$. Therefore, to
get $p$, we just have to take in integral of $g$ w.r.t. $x$,
$$p(x) = \int g(x) dx\,.
    \label{eq:beat2position}$$

For instance, if we want to know the position of a note at the beat 11,
it would only take to calculate
$$\text{position} = \int_{0}^{11} g(x) dx\,.
    \label{eq:beat2position-example}$$

You can see an example in Figure
[7](#fig:example-area){reference-type="ref"
reference="fig:example-area"}. As shown, the position at beat 11 is the
area under the curve (in light blue) from 0 to 11.

<figure id="fig:example-area">

<figcaption>The position at beat 11 will be the area underneeth the
function <span class="math inline"><em>g</em></span>.</figcaption>
</figure>

The resulting integral of $g(x)$ in our toy example is
$$p(x) = \begin{dcases}
            2x\,, & \text{if $x \leq 4\,;$}\\ 
            p(4) + 0\,, & \text{if $4 < x \leq 8\,;$}\\ 
            p(8) + 0\,, & \text{if $8 < x \leq 10\,;$}\\ 
            p(10) + (x-10)\times6 \,, & \text{if $10 < x \leq 13\,;$}\\ 
            p(13) + (x-13) \times 1 \,, & \text{if $x > 13\,,$}\\ 
        \end{dcases}
        \label{eq:beat2effective-bps}$$ and its plot can be seen in
Figure [8](#fig:position-example){reference-type="ref"
reference="fig:position-example"}.

<figure id="fig:position-example">

<figcaption>Plot of <span
class="math inline"><em>p</em></span></figcaption>
</figure>

## Formalization

Let
$$\left\{\left( b_{i}^{(b)},v_{i} \right)\right\}_{i=1}^{n} = \mathcal{B} = B^{(b)} \times V
        \label{eq:B}$$ be a sequence of $n$ beat signatures, where
$v_i \in V = \{v_j\}_{j=1}^{n}$ is the BPS value at beat
$b_i^{(b)} \in B^{(b)} = \{b_i^{(b)}\}_{j=1}^{n}$.

Let

$$\left\{ \left( b_{i}^{(s)},s_{i} \right) \right\} _{i=1}^{m} = \mathcal{S} = B^{(s)} \times S
        \label{eq:S}$$ be a sequence of $m$ scroll signatures, where
$s_i \in S = \{s_j\}_{j=1}^{m}$ is the scroll value at beat
$b_i^{(s)} \in B^{(s)}= \{b_j^{(s)}\}_{j=1}^{m}$.

Let
$$\left\{ \left( b_{i},z_{i} \right) \right\} _{i=1}^{n'} = \mathcal{Z} = B^{(b)} \cup B^{(s)}\times V \cup S
        \label{eq:Z}$$ be a sequence of $n'$ BPSs with applied scrolls,
where $z_i$ is the effective BPS at beat $b_i$ constructed from
$\mathcal{S}$ and $\mathcal{B}$ as
$$\mathcal{Z} = \bigcup_{i=1}^{m} \left\{\left( b_i^{(s)},h(b_i^{(s)})\times s_i \right)\right\}\bigcup_{j=1}^{n} 
        \begin{dcases}
            \left\{ \left( b_j^{(b)}, v_j\times s_i \right)\right\}\,, & \text{if $b_i^{(s)} < b_j^{(b)} < b_{i+1}^{(s)}$}\,; \\
            \emptyset\,, & \text{otherwise}\,;
        \end{dcases}
        \label{eq:Z-construction}$$ where
$h: \mathbb{R} \rightarrow \mathbb{R}$ is a function that given a beat,
returns the BPS for that beat, and $b_{m+1}^{(s)} := \infty$. We define
$$g(x) = \begin{dcases}
            z_1\,, & \text{if $ x \leq b_2 $}\,;\\
            z_i\,, & \text{if $ b_{i} < x \leq b_{i+1}\,; \quad \forall i=2,\dots,n'$}\,,
        \end{dcases}
        \label{eq:final-g}$$ as a function that retrieves the effective
BPS given a beat, and $b_{n'+1}:= \infty$. Finally, we define
$$p(x) = \int g(x) dx = \begin{dcases}
            xz_1\,, & \text{if $ x \leq b_2 $}\,;\\
            p(b_i) + (x-b_i)\times z_i\,, & \text{if $ b_{i} < x \leq b_{i+1}\,; \quad \forall i=2,\dots,n'$}\,,
        \end{dcases}
        \label{eq:position-final}$$ as the function that retrieves the
position given a beat.

# From beat to scroll

## Introduction {#introduction-2}

Another issue that we might have is to know how far upwards we should
scroll the notes from its original position given the current beat. If
we did not have any other gimmicks, this would be very easy to compute.
Let us suppose that the notes redendered in the screen are squares of
one unity of length and height, and that one beat is worth one distance
of separation, as shown in Figure
[9](#fig:notes-layout){reference-type="ref"
reference="fig:notes-layout"}.

<figure id="fig:notes-layout">
<embed src="beat2scroll.pdf" />
<figcaption>Two notes inside with their respective bounding boxes
separeted apart by one beat.</figcaption>
</figure>

In this set up, the amount of scroll that we have to apply for a given
beat $x$ is just $-x$ (if we were scrolling upwards, on the $y$ axis).

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

2.  Notes with beats in the range $[4, 4+2) = [4,6)$ will become fake
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

We would like to have a function $q: \mathbb{R}\rightarrow \mathbb{R}$
that calculates the scroll value for a given beat, as well as a function
$e: \mathbb{R}\rightarrow \mathbb{R}$ that calculates the speed factor
at a given beat.

## Solution

#### Warps

First, let us deal with the problem of the WARPS. We would need to
create a function that maps from beats to warped beats first, so we
could retrieve the real scroll value w.r.t. it. Actually, the function
that we are looking foris pretty much identical to that shown in Figure
[2](#fig:seqtime2songtime){reference-type="ref"
reference="fig:seqtime2songtime"} $$q(x) = \begin{dcases}
            x\,, &\text{if $ x \leq 4 $}\,;\\
            x + 2\,, &\text{if $ x > 4 $}\,.\\
        \end{dcases}
        \label{eq:warps-example}$$

You can see the function plotted in Figure
[10](#fig:warps-example){reference-type="ref"
reference="fig:warps-example"}.

<figure id="fig:warps-example">

<figcaption>Plot of <span
class="math inline"><em>q</em></span></figcaption>
</figure>

And that is that. Actually, the output of this function is already the
scroll value that we are looking for! It is inverse
$$q^{-1}(x) =  \begin{dcases}
        x\,, & \text{if $ x \leq 4 $}\,;\\
        4\,, & \text{if $ 4 < x \leq 4+2 $}\,;\\
        x-2\,, & \text{if $ x > 4+2$}\,;\\
    \end{dcases}
    \label{eq:warps-inverse}$$ also is very similar to how we calculated
$t_{(s)}^{-1}$ in Section
[1](#sec:songtime2seqtime){reference-type="ref"
reference="sec:songtime2seqtime"}. You can visualize it in Figure
[11](#fig:warps-example-inverse){reference-type="ref"
reference="fig:warps-example-inverse"}.

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
        1\,, & \text{if $ x \leq 6 $}\,;\\
        \frac{7-1}{1}(x-6)+1\,, & \text{if $ 6 < x \leq 6+1 $}\,;\\
        7\,, & \text{if $ x > 6+1 $}\,.
    \end{dcases}
    \label{eq:example-speeds}$$

<figure id="fig:speeds-example">

<figcaption>Plot of <span
class="math inline"><em>e</em></span></figcaption>
</figure>

## Formalization

#### Warps

Let
$\mathcal{W} =  \left\{\left( b_i^{(w)}, w_i \right)\right\}_{i=1}^{n}$
be a sequence of WARPS, where $w_i$ is the warp (measured in beats) at
beat $b_i^{(w)}$.

We define the function $q: \mathbb{B}\rightarrow \mathbb{T}$
$$q(x) = \begin{dcases}
        x\,, & \text{if $ x \leq b^{(w)}_1 $}\,;\\
        x+ \sum_{j=1}^{i}w_i\,, & \text{if $ b^{(w)}_i < x \leq b^{(w)}_{i+1}\,,\quad \forall i=1,\dots,n$}\,;\\
    \end{dcases}
    \label{eq:q-1}$$ with $b^{(w)}_{n+1} := \infty$ which maps from the
beat space into the translation space, if $\mathcal{W} \neq \emptyset$.

We define the function $q^{-1}: \mathbb{T}\rightarrow \mathbb{B}$
$$q^{-1}(x) = \begin{dcases}
        x - \sum_{j=1}^{i-1}w_j\,, & \text{if $ b^{(w)}_{i-1} + \sum_{j=1}^{i-1}w_j < x \leq b^{(w)}_i + \sum_{j=1}^{i-1}w_j\,, \quad \forall i=1,\dots,n$}\,;\\
        b^{(w)}_i\,, & \text{if $ b^{(w)}_i + \sum_{j=1}^{i-1}w_j < x \leq b^{(w)}_i + \sum_{j=1}^i w_j\,,\quad \forall i=1,\dots,n $}\,;\\
        x - \sum_{j=i}^n w_j & \text{if $ x > b^{(w)}_n + \sum_{j=1}^n w_j $}\,; 
    \end{dcases}
    \label{eq:q}$$ when $\mathcal{W} \neq \emptyset$, where
$b^{(w)}_0 := -\infty$, that maps from the translation space into the
beat space.

If $\mathcal{W} = \emptyset$, then $q(x) = q^{-1}(x) = x$.

#### Speeds

Let
$\mathcal{E} =  \left\{\left( b_i^{(e)}, s_i, p_i, t_i \right)\right\}_{i=1}^{n}$
be a sequence of SPEEDS, where $s_i$ is the speed change at beat
$b_i^{(e)}$ with a span transition of $p_i$ beats if $t_i = 0$, or $p_i$
seconds if $t_i=1$.

We define a new sequence
$\mathcal{E'} =  \left\{\left( b_i^{(e)}, s_i, p'_i \right)\right\}_{i=1}^{n}$
from $\mathcal{E}$ as $$\mathcal{E'} = \bigcup_{i=1}^{n} \begin{dcases}
        \left\{\left( b_i^{(e)}, s_i, p_i \right)\right\}\,, & \text{if $ t_i =0 $}\,;\\
        \left\{\left( b_i^{(e)}, s_i,g \left( p_i, b_i^{(e)} \right)  \right)\right\}\,, & \text{otherwise}\,,\\
    \end{dcases}
    \label{eq:eprime}$$ where
$$g(p,b) =\left( f \circ t_{(s)} \circ t_{(d)}  \right)\left(  \left( t_{(d)}^{-1} \circ  t_{(s)}^{-1} \circ f^{-1} \right)(b) + p \right)\,.
    \label{eq:iureio}$$

We define the function $e:\mathbb{B}\rightarrow \mathbb{E}$
$$e(x) = \begin{dcases}
        s_1\,, & \text{if $ x\leq b_1^{(e)} $}\,;\\
        \frac{s_i-s_{i-1}}{p'_i} \left( x-b_{i}^{(e)} \right)+s_{i-1} \,, & \text{if $ b_{i}^{(e)} < x \leq b_{i}^{(e)}+p'_i \land p'_i \neq 0\,,\quad \forall i=1,\dots,n$}\,;\\
        s_i\,, &\text{if $ b_{i}^{(e)}+p'_i < x \leq b_{i+1}^{(e)}\,,\quad \forall i=1,\dots,n$}\,,\\
    \end{dcases}
    \label{eq:e}$$ where $b_{n+1}^{(e)} := \infty$, and $s_0 := s_1$.

# Positioning and scrolling notes

Alright! Now we have formally defined everything we need to build a
sequencer using stepmania's notation. This section will show how to use
all functions defined formally in the sections above to determine where
to draw the notes in the screen at any given song time.

To ease the understanding, Table
[1](#tab:symbol-table){reference-type="ref"
reference="tab:symbol-table"} gathers all the functions that we are
going to need as well as a brief descripcion for each one of them.

::: {#tab:symbol-table}
  Symbol             Description
  ------------------ ----------------------------------------------------------------------------------------------------------------------------------------
  $\mathbb{S}$       Song time space (s) $\mathbb{S} = \mathbb{R}$.
  $\mathbb{D}$       Delayed time space (s) $\mathbb{D} = \mathbb{R}$.
  $\mathbb{S^{*}}$   Sequencer time space (s) $\mathbb{S^{*}} = \mathbb{R}$.
  $\mathbb{B}$       Beat space (beats) $\mathbb{B} = \mathbb{R}$.
  $\mathbb{P}$       Position space (units) $\mathbb{P} = \mathbb{R}$.
  $\mathbb{T}$       Translation space (units) $\mathbb{T} = \mathbb{R}$.
  $\mathbb{E}$       Speed space (speed factor units) $\mathbb{E} = \mathbb{R}$.
  $t_{(d)}$          Function $t_{(s)}: \mathbb{S}\rightarrow \mathbb{D}$ that maps from the song time space into the delayed time space.
  $t_{(d)}^{-1}$     Function $t_{(d)}^{-1}: \mathbb{D}\rightarrow \mathbb{S}$ that maps from the the delayed time space into the song time space.
  $t_{(s)}$          Function $t_{(s)}: \mathbb{D}\rightarrow \mathbb{S^{*}}$ that maps from the delayed time space into the sequencer time space.
  $t_{(s)}^{-1}$     Function $t_{(s)}^{-1}: \mathbb{S^{*}}\rightarrow \mathbb{D}$ that maps from the the sequencer time space into the delayed time space.
  $f$                Function $f: \mathbb{S^{*}}\rightarrow \mathbb{B}$ that maps from the sequencer time space into the beat space.
  $f^{-1}$           Function $f^{-1}: \mathbb{B}\rightarrow \mathbb{S^{*}}$ that maps from the beat space into the sequencer time space.
  $p$                Function $p: \mathbb{B}\rightarrow \mathbb{P}$ that maps from the beat space into the position space.
  $q$                Function $q: \mathbb{B}\rightarrow \mathbb{T}$ that maps from the beat space into the translation space.
  $q^{-1}$           Function $q^{-1}: \mathbb{T}\rightarrow \mathbb{B}$ that maps from the translation space into the beat space.
  $e$                Function $e: \mathbb{B}\rightarrow \mathbb{E}$ that maps from the beat space into the speed space.

  : Table of functions and spaces.
:::

We will take the following assumptions:

1.  We are modeling a rhythm game whose notes scroll upwards.

2.  The receptor is placed at the origin of the scrolling axis.

3.  The beat 0 has a position 0 in the scrolling axis.

Let $\mathcal{N} =  \left\{ \left( u_{i} \right) \right\}_{i=1}^{n}$ be
a sequence of notes where $u_{i}$ is the beat when the $i$-th note
should be tapped. We define a new set
$$\mathcal{N'} = \left\{ \left( u_{i}, v_i, w_i \right) \right\} = \bigcup_{\{u\}\in \mathcal{N}} \left\{ \left(u, g(u), p(u) \right) \right\}
        \label{eq:set}$$ where $g: \mathbb{T}\rightarrow \mathbb{S}$
$$g(x) = \left( t_{(d)}^{-1}\circ t_{(s)}^{-1}\circ f^{-1}\circ q^{-1} \right)(x)\,,
        \label{eq:final-g}$$ $v_i$ is the exact time where the $i$-th
note should be tapped, and $w_i$ is its relative position at beat 0 in
the scrolling axis. At a given moment in time $t$ w.r.t. to the begginig
of the song, the $i$-th note should be positioned at
$$\left[ w_i - \left( q\circ f\circ t_{(s)}\circ t_{(d)} \right)(t) \right] \times \left( e\circ q\circ f\circ t_{(s)}\circ t_{(d)} \right)(t)\,.
        \label{eq:final-position}$$
