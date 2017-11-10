# Lifeguard Paper Related Work

## Gossip without Trust/with Thresholds

### [ContagAlert: Using Contagion Theory for Adaptive, Distributed Alert Propagation](https://www.ideals.illinois.edu/bitstream/handle/2142/11071/ContagAlert%20Using%20Contagion%20Theory%20for%20Adaptive,%20Distributed%20Alert%20Propagation.pdf?sequence=2)

* propagate an alert while the attack is in progress,
while at the same time suppressing disruptive signals generated
by adversaries or false positives
* Not trust the other nodes
  * **Lifeguard extensing this to not trusting the local node?**

## Adaptive Gossip

### [Smart Gossip: An Adaptive Gossip-based Broadcasting Service for Sensor Networks](https://pdfs.semanticscholar.org/bd94/8c1688fb4cbc37370b2331625f0dd9d7e39b.pdf)
[Alternate](https://www.ideals.illinois.edu/bitstream/handle/2142/11189/Smart%20Gossip%20Infusing%20Adaptivity%20into%20Gossiping%20Protocols%20for%20Sensor%20Networks.pdf?sequence=2&origin=publication_detail)

* Indranil Gupta and 
* gossip percolation
* Smart Gossip
* Similarities
  * Nodes extract information from over-heard gossip messages, by applying simple rules.
  * Refine over time (i.e. adapt)
* **Section II.B - Adaptive Gossip Strategy**
  * Adaptive Neighbor: adjust own gossip probability in inverse proportion to number of neighbors.
  * **Adaptive Overheard**: adjust own gossip probability in inverse proportion to number of *duplicate* messages heard with previous/earlier sequence number.
* Dissimilarities
  * broadcast (promiscuous overhearing) based (optional?)
  * focus on determining parent v. child v. sibling relationship to each peer, to inform non-uniform probability for target selection for own gossip.
	
### [Efficient and Adaptive Epidemic-style Protocols for Reliable and Scalable Multicast](https://people.maths.bris.ac.uk/~maajg/indranil-tpds.pdf)

* Hierarchical gossip
* Adaptive dissemination framework
* Hierarchical Gossiping algorithm [12] uses a weighted target selection similar in
spirit to that of [27]. 
* adaptive to message loss rate?
  *  decentralized, local/partial view construction of tree (using hop numbers in recevied Tree messages)
  *  fall back from tree to gossip, if certain conditions not met. only in subtree regions affected. p.17 "Transition to Gossip Sub-Protocol"
  *  


### [Turning flash crowds into smart mobs with real-time stochastic detection and adaptive cooperative caching](https://www.ideals.illinois.edu/bitstream/handle/2142/10979/Turning%20Flash%20Crowds%20into%20Smart%20Mobs%20with%20Real-Time%20Stochastic%20Detection%20and%20Adaptive%20Cooperative%20Caching.pdf?sequence=2&isAllowed=y)

## Adaptive Wireless Sensor Networks

### [Cushion: Autonomically Adaptive Data Fusion in Wireless Sensor Networks](https://www.ideals.illinois.edu/bitstream/handle/2142/11094/Cushion%20Autonomically%20Adaptive%20Data%20Fusion%20in%20Wireless%20Sensor%20Networks.pdf?sequence=2&isAllowed=y)
* Sink adapts the upstream redundancy


# Considered But Not Relevant

### [REMAX: Reachability-Maximizing P2P Detection of Erroneous Readings in Wireless Sensor Networks](https://www.perform.illinois.edu/Papers/USAN_papers/16BAD01.pdf)
* Considered because co-authored by Indranil Gupta and appeared at DSN 2017.
* Includes a distributed fault detector, but for wireless sensors. Works by comparing the data values emitted by the local sensor to those of its neighbors. Not adaptive in the same way as Lifeguard.


## Sources

Indranil Gupta's papers

* As per [Google Scholar](https://scholar.google.com/citations?hl=en&user=Waaj7a8AAAAJ&view_op=list_works&sortby=pubdate) on 11/9/17, ordered by publication date.

## To Do
* Review Gupta's papers earlier than "A Case for Design Methodology Research"
* [8] P. Eugster, R. Guerraoui, S. Handurukande, A.-M. Kermarrec, and P. Kouznetsov. Lightweight Probabilistic Broadcast.
In Proc. 2001 Intnl. Conf. Dependable Systems and Networks (DSN ’01), pages 443–452, 2001. and other references from Efficient and Adaptive Epidemic-style Protocols for
Reliable and Scalable Multicast.
* [9] P. T. Eugster and R. Guerraoui. Probabilistic Multicast. In Proc. 2002 Intnl. Conf. Dependable Systems and Networks
(DSN ’02), pages 313–322, 2002
