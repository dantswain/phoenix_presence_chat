# PhoenixChat

A demo using Phoenix Presence to show who is online in a chat application

Based on:
* http://work.stevegrossi.com/2016/07/11/building-a-chat-app-with-elixir-and-phoenix-presence/
* https://dockyard.com/blog/2016/03/25/what-makes-phoenix-presence-special-sneak-peek

# CRDT Notes

## Literature

* [Good old wikipedia](https://en.wikipedia.org/wiki/Conflict-free_replicated_data_type)
* [_The_ paper](https://hal.inria.fr/inria-00609399v1/document)
* [Summary paper](https://hal.inria.fr/inria-00555588/document)
* [Riak Glossary](http://docs.basho.com/riak/kv/2.1.4/learn/glossary/)
* [Alexander Songe Elixirconf 2015](https://www.youtube.com/watch?v=txD1tfyIIvY)
* [Building a Chat App with Elixir and Phoenix Presence](http://work.stevegrossi.com/2016/07/11/building-a-chat-app-with-elixir-and-phoenix-presence/)
* [Chris McCord explains ORSWOT and Phoenix Presence](https://www.youtube.com/watch?v=n338leKvqnA)
* [Marc Shapiro presentation](https://pages.lip6.fr/Marc.Shapiro/slides/eventual-consistency-Mysore-2011-02.pdf)
* [ORSWOT at bet365](http://www.erlang-factory.com/static/upload/media/1434558446558020erlanguserconference2015bet365michaelowen.pdf)

## Background

* CAP theorem
* Network partitioning

## Assumptions

* Non _byzantine_ systems - systems don't make inconsistent communications
* Crashed processes retain their memory on restart
* All updates eventually reach all nodes (but not necessarily in order)
* Fully connected communication graph

## Key ideas

* We have some global state X
* X can get updated from any node at any time
* How do we keep every node's representation of X valid?
* Eventual consistency:
    * Every node eventually reaches the same state once updates finish
    * May require arbitration to resolve conflicting merges
* Strong eventual consistency:
    * Eventual consistency without any arbitration
    * It it sufficient that each node gets the same updates
    
* Re: being a "solution" to the CAP theorem:

    The CAP theorem states that it is impossible to simultaneously ensure strong consistency (C), availability (A) and tolerate network partition (P). As, network faults unavoidably occur in a large-scale environment, a real system must sacrifice either consistency or availability. Availability is often the top priority in practice: does this mean giving up all consistency guarantees?
     
    No: SEC provides a solution. A SEC replica is always available for both reads and writes, independently of network conditions. Any communicating subset of replicas of a SEC object eventually converges, even if partitioned from the rest of the network. SEC is weaker than strong consistency but nonetheless provides the well-defined guarantee of strong eventual convergence.
     
    SEC provides an extreme form of fault tolerance, as a SEC object tolerates upp to n − 1 simultaneous crashes. Remarkably, SEC does not require to solve consensus.

Example of eventual consistency but not strong eventual consistency:

* Key-value store
* Suppose all nodes know that h['foo'] = 'bar'
* Node X
* An update occurs and h['foo'] should now be 'baz'
* Node X does not get the update
* Node X later comes back up and still thinks that h['foo'] = 'bar'
* This is a conflict
* Conflict can be resolved via consensus / quorum: a majority of nodes think h['foo'] = 'baz', so they win.

Strong eventual consistency can be accomplished with a key-value
store, but it requires additional information be stored.

* CvRDT vs CmRDT

* SEC and the CAP theorem
    * If a partition occurs, each partition is SEC
    * When the partition is resolved, the two groups will become
      consistent again

## Some math

Graph

Join Semilattice (or just semilattice in the literature)
* Lives in the world of Algebra
* Partially ordered set with a LUB operation
* "Partially ordered set" just means there is a <= operation
* LUB must satistfy:
    * Commutativity: x u y = y u x
    * Idempotency: x u x = x
    * Associativity: x u (y u z) = (x u y) u z
* Example: Real numbers and x u y == max(x, y)

Monotonic Semilattice Object
* Back in the world of distributed systems
* The merge operation is the LUB
* State is monotonically non-decreasing (x[t] <= x[t + 1])
