#### Blockchain HowTo

Here we indicates the steps necessary to add a new blockchain to Diablo.
To make Diablo compatible with various blockchain implementations, we abstract away the main components of a
blockchain. The Diablo benchmark specification interacts with the resulting blockchain abstraction.
Let ```C``` be the set of clients.
A blockchain is modelled as a tuple ```(E,R,I)``` where ```E``` is the finite set of *endpoints* that act 
as blockchain nodes, ```R``` is a finite set of resources (e.g., account balance, smart contract state) 
maintained in the blockchain state, and $I$ is a potentially infinite set of interactions types 
(e.g. asset transfer, smart contract function invocation) between a client and the blockchain. 
An interaction event is denoted as a tuple ```(c,i,r,t)```.
The benchmark specification contains a function ```M``` mapping the Secondaries to the 
blockchain endpoints, a set ```P^R``` of resources needed for the test and the interactions.

To add a new blockchain, one has to implement at least one of these interaction types as well as 
4 functions that convert the benchmark specification to an executable test program: 
 * ```s.create_client(E)```,
 * ```create_resource(P^r)```,
 * ```encode(P^i, r, t)``` and 
 * ```trigger(i, r, t)```.
 
Since these functions are relatively fine grained, the implementations for the blockchains we test are small sized: 
between 1,000 and 1,200 LOC of Go.
