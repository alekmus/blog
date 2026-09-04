---
layout: page
title: "Simulated annealing - Settling on good enough but very quickly"
permalink: /simulated_annealing
---
# Simulated annealing - Settling on good enough but very quickly

***What's in common with garbage disposal, delivery robots, and Victorian era computer science problems? Finding a short path powers real world industries from microchip manufacturing to logistics. Unfortunately the solution is not as straightforward as the problem, and that is where simulated annealing comes in.*** 

In computer science there’s a fairly famous problem; A travelling salesman wants to visit a set of cities. What is the shortest tour the salesman can take that visits them all? 

This is, perhaps unsurprisingly, known as the travelling salesman problem. It’s deceptively simple in both its description and solution. We’ll get back to how to actually solve it but let’s start with why we would be interested in something this basic or let alone write blog posts about the topic.

The travelling salesman problem looks, sounds and tastes like a toy problem for the data structures and algorithms course of a CS program. However, it has direct applications all over the place; microchip manufacturing, logistics and more. 

I personally got nerd sniped by the problem on two separate occasions recently. First when talking to some people from a delivery robot company and then by a company optimizing garbage disposal routes. The latter having arguably built their business on their solution to this exact problem. 

So clearly, the travelling salesman problem does not belong in a classroom, and its applications are used to create value in the real world where having a better solution translates into a competitive edge. Shorter routes mean saved gasoline, faster delivery times and more efficient microchips.  
   
That brings us back to the solution. The problem is old, some descriptions coming from the 1830’s. Someone must’ve come up with a way to solve it a long time ago. If you think that, you’d be… completely correct. There is a very simple algorithm that always returns the shortest route. 

But wait, what was that thing about businesses competing on solutions?  

### Finding the shortest route

Let’s look at the algorithm a bit. The idea behind it is simple, as said; try every solution and pick the best one. We’ll call this the “Brute Force” algorithm. 

My implementation for this post is a single threaded recursion based one written in C. Not terribly optimized but should be sufficiently performant. We are mostly looking at relative values here anyway, so speed doesn’t matter all that much.

For comparison, we’ll also introduce an “Ordered” algorithm. It’s an algorithm only by technicality as it builds the route by just picking the first city it sees as the next destination without considering the length of the route in any way. Think of it as the baseline value for just picking an arbitrary path and calling that good enough.

Now we are ready to look at the behaviour of the brute force algorithm. Let’s trickle some results for dramatic effect. 

No need to look at the tables too closely, I’ll walk you through the relevant parts.

|Cities|Ordered Time (ms)|Ordered Cost|Brute Force Time (ms)|Brute Force Cost|
|------|-----------------|------------|---------------------|----------------| 
|3     |           0.001 |3401.516195 |               0.002 |     3401.516195|  
|4     |           0.001 |3409.613069 |               0.002 |     3409.574239|  
|5     |           0.001 |6068.946325 |               0.003 |     3410.701741|  
|6     |           0.001 |6335.504049 |               0.014 |     3449.754019|

In the table above, we’re comparing the shortest routes found by the different algorithms using a variable number of Finnish cities. The data we'll be using.

[Finnish cities to visit](/media/simulated_annealing/tsp_finland.png)

The cost is the length of the route that visits every city and returns back to the starting city. It is calculated directly from the distance between the cities using the decimal form of the latitude and longitude coordinates. It’s hard to get a good intuitive meaning behind the cost figure. Just think “lower is better”, and we’ll be alright.

Looking at the numbers, we see that the brute force algorithm is an excellent, perfect and all-round beautiful thing. It runs in a couple millionths of a second and provides results where the cost is around half compared to just picking an arbitrary route. 

Choosing the first route you see obviously has the benefit that it’s always really fast, hence the low scores in the Ordered Time column. 

Great. Problem solved, blog post done. Let’s just run the algorithms on a couple of more cities so you can admire it some more.

|Cities|Ordered Time (ms)|Ordered Cost|Brute Force Time (ms)|Brute Force Cost|
|------|-----------------|------------|---------------------|----------------| 
|     7|            0.001| 6568.755159|                0.075|     3677.221508|  
|     8|            0.001| 6570.974834|                0.539|     3677.660308|  
|     9|            0.001| 6612.567148|                 4.66|     3682.507639|  
|    10|            0.001| 6679.096824|                43.42|     3682.586721|

Again, we’re seeing similar results. It’s fast and the cost is low. Even more, if you remember the algorithm’s definition from earlier, you might’ve realized it’s not only low but it is the actual lowest theoretical cost for this set of cities.

Curiously, the execution time has jumped a bit in the last two runs but we’re not worried. It’s still only 0.043 seconds. That’s plenty fast for anything we’re doing. Let’s run a couple of more, just to make sure.

|Cities|Ordered Time (ms)|Ordered Cost|Brute Force Time (ms)|Brute Force Cost|
|------|-----------------|------------|---------------------|----------------|  
|    11|            0.001| 6712.364764|              466.875|     3682.630639|  
|    12|            0.001| 8510.419084|             5500.562|     4087.138961|  
|    13|            0.001| 8514.021489|            70029.398|     4095.167815|  
|    14|            0.001| 9678.707211|           974797.909|     4102.337149|

Oh no…

### Destined to fail

Here lies the problem with the brute force algorithm, and the asterisk to my earlier statement about the solution being simple. Testing every possible option is easy and leaves you with the best possible route but it also takes a looooooong time. So long in fact, that it quickly becomes unfeasible to use this method in any practical application.

For five cities, we need to check  (5-1)\!2 \= 12 routes, assuming we're not checking symmetric routes twice. For 14 cities, there are roughly 3.1 billion routes to check. And if we were to try to find all possible routes in the entire dataset of 10 639 Finnish cities, there would be roughly 6.7×1038219 routes to check. 

For reference, the universe is approximately 13.8 billion years old which translates into 4.35 × 1026 nanoseconds. If we processed a path in one nanosecond, a thousandfold improvement to our current implementation, and had been running non-stop since the creation of the universe, we would have discovered basically zero percent of the possible routes.   
Even that “basically” is an overstatement, as the actual percentage is zero followed by 38 193 zeroes after the decimal point before the first significant number.

Okay, so you can’t just check all the paths. But what’s the alternative?

### Let's just try random stuff and hope it works (, and it does)

You could try to make the brute force algorithm more efficient with caching subpath costs, discarding symmetric paths or stop checking the routes when it’s clear that they cannot contain the best solution but that does not fundamentally solve the scalability issue. You still need to check a massive number of routes. It just allows you to skip checking some of the clearly bad ones.

There are better algorithms, such as dynamic programming approaches, that always give you the best possible route but even those scale exponentially. It is a lot better than our factorial scaling but not something we would exactly call efficient. 

One idea on how to approach this issue is to relax our expectations; What if, instead of trying to find the absolute best solution, we’d settle for a good one. One of the algorithms borne from this idea is simulated annealing.

Simulated annealing is also fairly straightforward in its approach; We start with an arbitrary tour and make a small random change to it. If the change results in a shorter tour, we keep the change, and if not, the change is either discarded or kept based on some probability. That probability of accepting worse solutions is determined by a so-called temperature parameter. 

It “cools” with each iteration making it more likely that we accept worse changes early on and dismiss them more readily later. There’s a whole science to the cooling scheduling which we are not touching here. We’ll just be lowering the temperature to 90% of its previous value in our tests. All this is repeated a predeterminate amount of times. Then we pick the best tour we saw during those iterations.

This raises an important question; what are those small random changes we’re making? In actuality there are not a lot of restrictions on what those can be. 

Simulated annealing can be viewed as a random walk on a search graph where each node represents a tour and each edge is one of those small changes. As long as the resulting search graph has a “sufficiently short” path between the starting tour and the best tour, anything goes with what the changes can be.  

We’re going to be lazy and be just moving around random length subtours in our tests but people do all kinds of things. I’ve seen implementations where individual nodes are moved, subtours are reversed, the whole matrix representation of the tour is transposed and more.

At first, the whole thing probably sounds like trying out random things and hoping for a better result. But that’s only mostly entirely correct.

### Getting better by not getting worse, except sometimes

If we forget for a second the whole accepting a worse route thing, each random small change improves the tour slightly or does nothing. Because of that, after each iteration, we either stay in place or take a small step, so to speak, downhill on the cost function towards a better result.   
![](/media/simulated_annealing/local_optima.png)
The reason we sometimes accept a worse result on top of this is due to the fact that the valley where we are walking downhill doesn’t necessarily have the absolute lowest bottom. 

If we were to walk downhill, refusing to ever step uphill, we would just end up at the bottom of our valley. This is great as long as our valley goes the deepest but if it doesn’t, we’ll end up in a so-called local optimum, best result in its own immediate proximity. By sometimes accepting a worse result, we can take a look in the neighbouring valley to see if it is deeper than ours, and hopefully finding the best possible result, a global optimum.

In practice the simulated annealing algorithm has a lot of desirable properties. The number of iterations is set beforehand and is not dependent on the number of cities. This allows you to for example set time budgets and decide exactly how long the algorithm should take. You can also stop the algorithm and continue searching for a better tour later without storing a massive state.

### Comparing results

With our simulated annealing implementation with a set number of 10 000 iterations, processing 10 cities takes 2.994 milliseconds and 1600 cities takes roughly two seconds, a huge improvement when compared to the brute force algorithm which took 16 minutes to process just 14 cities.  

So how much worse are the results if this doesn’t provide the shortest tour? Turns out not that much. 

|Cities|Simulated Annealing Cost|Brute Force Cost|Diff %  |  
|------|------------------------|----------------|--------| 
|    3 |             3401.516195| 	  3401.516195|       0|
|    4 |             3409.574239| 	  3409.574239|       0|
|    5 |             3411.426442| 	  3410.701741|0.021248|  
|    6 |              3457.17047|     3449.754019|0.214985|  
|    7 |             3911.235276| 	  3677.221508|6.363875|  
|    8 |             3682.431236| 	  3677.660308|0.129727|  
|    9 |              3683.89856| 	  3682.507639|0.037771|  
|    10|              3694.04086| 	  3682.586721|0.311035|  
|    11|             3694.098401| 	  3682.630639|0.311401|  
|    12|             4102.732663| 	  4087.138961|0.381531|  
|    13|     	     4099.471119|     4095.167815|0.105083|  
|    14|             4113.205272| 	  4102.337149|0.264925|

In the test set, the worst difference between the shortest distance and the simulated annealing result is 6.36% and the difference in the tour length being typically well under 0.5%. The 6.36% is an outlier, and I hadn’t seen a difference breaking even 1% before in this particular test set but it perfectly shows the tradeoff of simulated annealing. No matter how long you run it, you’re not guaranteed a great result. You’re just increasing the probability of getting one.

In the following graph you can see more behaviour as the city count goes up to 150\. The brute force results stop early because I’m not going to sit here waiting for 50 hours for runs to complete. 

![](/media/simulated_annealing/length_of_tour.png)

This graph is very indicative of the stochastic nature of simulated annealing. It’s fast, finds very good solutions but doesn’t always do it.

We don’t have the actual optimal tour to compare for the higher numbers of cities but the results are clearly better when compared to picking an arbitrary tour and close-ish to what we would expect the optimal tour to be.

There are issues though. See those peaks where the results are as bad as the ordered tour.

### What do you mean I can't just guess the perfect result?

The algorithm fails to find a better solution than the initial ordered tour we give it as a starting point. The reason behind this is in how we find neighbours for our tour to check if they are better or worse.

Remember when I said earlier that simulated annealing can be viewed as a random walk on a search graph of neighbouring tours, and how we're pre-setting the number of iterations to 10 000? What if there are no better solutions within 10 000 steps of the initial tour?
![](/media/simulated_annealing/local_optima_escape.png)
What if there are some but you first need to take 10 bad steps before you can reach that particular valley? Assuming there is an equal chance of getting a good and a bad tour, with our schedule there would be around 0.36% reaching that valley. This doesn't even take into account the iterations needed to get the bottom of it.

This behaviour is exacerbated by our implementation of the algorithm as we allow exploring for neighbouring tours by only moving around a subroute. Consider an optimal tour where the node order is 1, 5, 4, 3, 2, 1\. If we start from 1, 2, 3, 4, 5, 1\. We need 4 steps to reach it whereas if we'd allow reversing a random subpath, we'd only need a single step. By allowing more kinds of transitions, we can make the search graph more connected and mitigate the issue but it'll always be there. It just becomes harder to see as we model more complex concepts as the travelling salesman problem.  

In actual implementations these issues are mitigated with logic around the annealing schedule, iteration count or restarting runs but nonetheless they are a downside of the method itself which is why I left this behaviour in for demonstration reasons, definitely not laziness.

### Building on simulated annealing with machine learning

Like I said in the beginning, this method is widely used in industry and definitely works despite these downsides, but if you'd want to improve upon it, one approach would be to try to remove the randomness related to finding and escaping those local optima.

What if we could figure out if a step improves the tour before taking it? This is where derivatives and by extension machine learning come into the picture. If we know how steep the hills are, we can take better informed steps and even use the momentum from descending to the bottom to climb the opposite side of the valley to escape those local optima. 

Once I have more time to procrastinate on these, we'll be looking into some neural network reinforcement learning solutions resembling simulated annealing and hopefully some solutions where we don't even need to define the transitions so the neural nets can just figure out the whole thing by themselves.  
