---
layout: single
title:  "Automating FPL with Machine Learning and MPC - Part 2: Optimisation and Control"
date:   2025-09-15 12:00:00 +0000
categories: technical
---

<p style="text-align: center;">
<img src="/assets/images/fpl-ml-2.jpg" alt="My Photo" style="width:60%; border-radius:12px; margin: 20px 0;" />
</p>

Welcome back to part 2 of the blog post on how I am automating Fantasy Football this year. This probably marks the final technical blog post on my project, I will probably further update the blog further in the season, where we review some of the model's performance - so far there has been some interesting choices (a weird obsession with Bowen and Gundogan), and I'll try and explain the why, as well as the what with the progression of the season.

Without further ado, here is how we utilised the model we built in part 1 within a control framework. This will probably get a little bit maths heavy, purely because I feel there are less explanations on the internet as to how to formulate these optimisation problems, whereas every man and his dog has done an explanation on how LSTMs work.

---

#### **Preventing a Greedy Control Strategy**

---

In the first blog post, we briefly discussed training the model for longer time horizon predictions, to avoid taking 'greedy' options when utilising the control strategy. A greedy strategy would take the option that maximises expected points for that particular week. However fantasy football isn't won in just one week (I hope, as I say sat rooted at the bottom of all the leagues I am in currently).

We want our model to plan for the future over different game weeks. For example, if we had the choice between Bruno Fernandes and Martin Odegaard, who are similar players, with similar prices and over the course of a season might score similar points. If Bruno's next fixtures were Sunderland (who you'd *imagine* Man Utd might beat), Man City and Everton - one easy game and two tough games, his predicted xP might be 8.32 from the first game, but then only a measly 1.32 and then -1.41 from the next two games (probably due Jordan Pickford having that blokes number from the penalty spot). Then if Odegaard was to face Burnley, Leeds and Wolves, where he is predicted xPs of 6.43, 7.34 and 4.56; Odegaard's xP over the *horizon* is then the more attractive pick over Fernandes. However, there is no free lunch here: the further in the future we predict, the more complex and less accurate the model is with our current data.

---

#### **The Initial Optimisation Problem**

---

Picking our starting FPL squad is a relaxed optimisation problem compared to the receding horizon control problem (i.e. we have less constraints). This is because we don't have any dynamic decisions to make, so the static optimisation doesn't have to account for transition and decision making constraints.

We first introduce a binary decision variable, $y_{i}$ - this is if 1 if player $i$ is picked, and 0 otherwise. Here $i$ is an index that covers the set of all players in the database which we will denote as $I$, so $i$ could be Cole Palmer, Harrison Armstrong or even Luis Diaz, who is for some reason still kicking about on the database.

This then allows us to define our objective function over the set of $I$ players:

$$
\max \sum_{i \in I} y_{i} c_{i} (\mu_{i, \text{xP}} - r \sigma_{i, \text{xP}}) 
$$

Essentially, we sum over all the players, multiply each player by $c_{i}$, which is their chance of playing - this comes from the API and accounts for injuries, suspensions or if they are fuming they haven't been granted a transfer (a là Wissa or Isak), and then multiply this by their mean xP, $\mu_{xP}$, whilst accounting for the variance of their predicted points $\sigma_{xP}$, to some perceived risk level $r$. In this problem I set $r$ to 0.5, but the higher the value of $r$, the less risk, as the more punished a higher variance in the xP is. The optimiser here essentially 'switches on' certain players, and evaluates the overall expression, and then decides which players to pick based on the value when they are switched on.

We then have to satisfy constraints within our fantasy picks:

Firstly, we have to pick exactly 15 players in the squad, so:

$$
\sum_i^N y_i = 15
$$

{% raw %}
We also have constraints on the numbers of positions for each player. Firstly, we define the set of positions, 
$P = \{ GK, DEF, MID, FW \}$. 

We can then define the subset of each player for each position, 
$I_p = \{ i \in I : \text{pos}(i) = p \}$. 
{% endraw %}

For example, Marc Guehi would belong to $I_{\text{DEF}}$.

Then, we have equality constraints for each position:
$L_{GK} = 2, \; L_{DEF} = 5, \; L_{MID} = 5, \; L_{FW} = 3$

Finally, we can define the constraint over the set of positions, whereas before it was defined over all players:

$$
\sum_{i \in I_p} y_i = L_p \quad \forall p \in P
$$

{% raw %}
Similarly, we can define the set of clubs, 
$C = \{ \text{Everton}, \text{Arsenal}, \cdots \}$. 
Again, we have the subset of players belonging to each team, $I_c = \{ i \in I : \text{club}(i) = c \}$. 
{% endraw %}

As I write this, Marc Guehi *currently* belongs to $I_{\text{Crystal Palace}}$. 


As we can't have more than 3 players from the same club, this is an inequality constraint:
$$
\sum_{i \in I_c} y_i \leq 3 \quad \forall c \in C
$$

We also have a constraint on the budget, which we define as a further inequality constraint:

$$
\sum_{i \in I} y_i v_i \leq 100
$$

Here, $v_i$ is the value of player $i$.

When solving these optimisation problems, inequality constraints can either be *active* or *inactive*. Active means they are satisfied by equality, i.e. the cost of the squad is exactly 100, or we have exactly 3 Arsenal players in the squad. 
The full optimisation problem can then be written as:

{% raw %}
$$
\begin{aligned}
\max_{y_i \in \{0,1\}} \quad & \sum_{i \in I} y_i \, c_i \left( \mu_{i,\text{xP}} - r \, \sigma_{i,\text{xP}} \right) \\[1em]
\text{s.t.} \quad 
& \sum_{i \in I} y_i = 15 
&& \text{(squad size)} \\[0.5em]
& \sum_{i \in I_p} y_i = L_p \quad \forall p \in P 
&& \text{(position limits: }L_{GK}=2,\ L_{DEF}=5,\ L_{MID}=5,\ L_{FW}=3\text{)} \\[0.5em]
& \sum_{i \in I_c} y_i \leq 3 \quad \forall c \in C
&& \text{(team limit)} \\[0.5em]
& \sum_{i \in I} y_i v_i \leq 100
&& \text{(budget constraint)}
\end{aligned}
$$
{% endraw %}

This is all linear in our decision variables across the objective function and constraints, meaning this is an Integer Linear Program. This is a convex optimisation problem, which means all the solutions are globally optimal - i.e. our solver won't get stuck trying to pick substandard players!

We ended up with an initial squad of: 

1. Jordan Pickford — Everton — Goalkeeper — £5.5
2. Ezri Konsa  — Aston Villa — Defender — £4.5
3. Rúben Dias — Man City — Defender — £5.5
4. Aaron Wan-Bissaka — West Ham — Defender — £4.5
5. Dan Burn — Newcastle — Defender — £5.0
6. Bruno Fernandes — Man Utd — Midfielder — £9.0
7. Bryan Mbeumo — Man Utd — Midfielder — £8.0
8. Mohamed Salah — Liverpool — Midfielder — £14.5
9. Sandro Tonali — Newcastle — Midfielder — £5.5
10. Jarrod Bowen — West Ham — Forward — £8.0
11. Raúl Jiménez — Fulham — Forward — £6.5
12. Nick Pope — Newcastle — Goalkeeper — £5.0
13. İlkay Gündoğan — Man City — Midfielder — £6.4
14. Reece James — Chelsea — Defender — £5.5
15. Jørgen Strand Larsen — Wolves — Forward — £6.5

From this, we just picked the player with the highest predicted xP to be the captain (Bowen?!?!?!?), and then the top 11 players that fit into a squad as the starting XI. This seemed ok. Not what I would have chosen as my first choice, but my hypotheses for why this isn't perfect will come, when I have given the model more of a chance!

---

#### **Receding Horizon Model Predictive Control**

---

Now we have an initial starting point, I then made the optimisation dynamic. Receding horizon predictive control is one type of MPC strategy. Here, we have a model (our ML model), which will predict $T$ timesteps into the future, and we use an optimisation problem to decide which decisions to make to maximise an objective function over that time horizon, subject to constraints on that horizon. We then re-solve this problem at every time step, where we make updates to the control strategy; whilst attempting to plan-ahead.

When we make this dynamic, we have a few more decision variables - not just the players to pick. Firstly, our sets are not just defined over the set of players, $I$; but also the set of time horizons: $\mathcal{T} \in \{t, \cdots,  T\}$ - then the previous variable of the player being in the squad is also defined over time - i.e. $y_{i,t}$

We then add more binary decision variables: 
- $z_{i,t}$, this is 1 if player $i$ is in the starting XI at time $t$, and 0 otherwise.
- $c_{i,t}$, which is 1 if player $i$ is the captain at time $t$, and 0 otherwise.
- $s_{i,t}$, this is 1 if player $i$ has been sold at time $t$, and 0 otherwise.
- $b_{i,t}$, this is 1 if player $i$ has been bought at time $t$, and 0 otherwise.

This allows us to define our objective function, just with a summation over the time horizon as well (accounting for also taking a points hit for transfers):

$$
\max \sum_{t \in T} \sum_{i \in I} y_{i} c_{i} (\mu_{i,t, \text{xP}} - r \sigma_{i,t, \text{xP}}) - 4*p[t]
$$


Unfortunately, this means we now need to add loads of constraints to ensure the model does what we want it to do.

Firstly, the initial squad rule - i.e. at time 0, the indices of the selected players need to be the ones in our squad:

$ y_{i, 0} = 1$ if player $i$ is in the squad, 0 otherwise


We then need to define state transition rules - i.e. ensure there is continuity between the players depending on the actions to take each week.
Firstly, the squad evolution constraint - i.e. the players in the squad at time t, must be the players in the squad at time t-1, minus the ones we sold and plus the ones we have bought:

$$
y_{i, t} = y_{i, t-1} - s_{i,t} + b_{i, t}, \quad t>0
$$

The case at $t = 0$ is covered by the initial rule.

Then the budget evolution rule - i.e your bank balance is the balance prior accounting for transfers:

$$
\text{bank}_t = \text{bank}_{t-1} + v_i s_{i,t} - v_i b_{i,t}
$$

Then have constraints on the transfers made:

$$
\text{transfers_made}t = \sum{i \in I} b_{i,t}, \quad \forall t \in \mathcal{T}
$$

With logic to try and reduce number of paid transfers:

$$
\text{paid_transfers}_t \geq \text{transfers_made}_t - \text{free_transfers}_t, \quad \forall t \in \mathcal{T}
$$

Constraint on the evolution of free transfers too

$$
\text{free_transfers}t = 1 + \text{free_transfers}{t-1} - \text{transfers_made}_{t-1}, \quad t > 0, \quad \text{with } \text{free_transfers}_t \leq 2
$$

You have to buy and sell the same number of players - i.e. you can't just buy one player and go over the squad limit.

$$
\sum_{i \in I} s_{i,t} = \sum_{i \in I} b_{i,t}, \quad \forall t \in \mathcal{T}
$$

Then more logic regarding transfers, i.e. you cannot buy a player who is already in your squad:

$$
b_{i,t} + y_{i,t-1} \leq 1, \quad \forall i \in I, ; t > 0
$$

and can only sell players you currently own:

$$
s_{i,t} \leq y_{i,t-1}, \quad \forall i \in I, ; t > 0
$$


Finally, the same constraints as the static problem are translated to the dynamic setting. At any time step, the squad must contain exactly 15 players:

$$
\sum_{i \in I} y_{i,t} = 15, \quad \forall t \in \mathcal{T}
$$

with the correct positions:

$$
\sum_{i \in I_{\text{GK}}} y_{i,t} = 2, \quad
\sum_{i \in I_{\text{DEF}}} y_{i,t} = 5, \quad
\sum_{i \in I_{\text{MID}}} y_{i,t} = 5, \quad
\sum_{i \in I_{\text{FWD}}} y_{i,t} = 3
$$

and a team limit:

$$
\sum_{i \in I_{k}} y_{i,t} \leq 3, \quad \forall k \in \text{Teams}, ; \forall t \in \mathcal{T}
$$

As our model makes decisions on the captains and starting XI, we need to also define this. The starting XI must come from players within the squad:

$$
z_{i,t} \leq y_{i,t}, \quad \forall i \in I, ; t \in \mathcal{T}
$$

Must contain exactly 11 players:

$$
\sum_{i \in I} z_{i,t} = 11, \quad \forall t \in \mathcal{T}
$$

and again requires positions:

$$
\sum_{i \in I_{\text{GK}}} z_{i,t} = 1, \quad
\sum_{i \in I_{\text{DEF}}} z_{i,t} \geq 3, \quad
\sum_{i \in I_{\text{FWD}}} z_{i,t} \geq 1
$$

The captain must be in the starting XI:

$$
c_{i,t} \leq z_{i,t}, \quad \forall i \in I, ; t \in \mathcal{T}
$$

and can only pick one!

$$
\sum_{i \in I} c_{i,t} = 1, \quad \forall t \in \mathcal{T}
$$

This had lead us to a long final time-dependent optimisation problem - that I will not be retyping out. We solve this at every time step, to decide the transfers, starting XI and captain for the next week.

---

#### **Will this be any good?**

---

So finally, do I think all the hours of time I put into building the model, control strategy and writing up how I did it will pay dividends in terms of beating everyone?
Short answer: probably not.

To quote a visiting student in our group, Joao, "MPC is only as good as your model" - and he's right.

Currently the model isn't great. It doesn't have too much data, and is pretty narrow minded in terms of its picks. Although I incorporated some pretty basic uncertainty, it is pretty much a deterministic model. When it is translating how good a given player *might be* in the Premier League, it just picks a random value. It would be better to make almost everything stochastic - and with that, it would require a lot more data, and a lot more compute. I will probably devote some more time to trying to do this - but I also have to evade all the bot detections; but it will be interesting to see how good I can get this - and maybe one day, I'll be using it to make money on BJ88.html, or one of those other totally betting sites that sponsor Premier League teams (and probably have terribly priced odds!).