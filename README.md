# Distributed Systems Lab ­ Homework 7
## Chord and DHT (Part 1) ­ Theory (8 points)
Within this assignment the theoretical background of Chord networks should be investigated in detail.
Due to the inner organization of a Chord network, it is easy to realize a distributed hash table based on
top of it. Details can be found within this paper.
Within the next assignment sheet the theoretical principles will have to be put into practice.
### Assignments:
#### 1. Routing in Chord
Within a Chord network, objects (peers and data) are mapped to a linear address space spanning
the interval [0, 2
m
­1], where m is some constant (typically 128 or 160). The mapping is realized
via a hashing function (e.g. SHA­1). By considering the object with the address 0 to be adjacent
with the object 2
m
­1, a ring is formed (Chord Ring). For this example, consider m=5. Hence, our
network can contain up to 32 different objects.
a. Draw the Chord ring and place nodes on the addresses 1, 3, 7, 8, 12, 15, 19, 25 and 27.
Compute the Finger table of the nodes with the addresses 1 and 25 and illustrate those
within your drawing.
b. Peer 25 wants to send a message to the peer 19. Illustrate the necessary steps. Compute
the finger tables for the nodes encountered along the path.
c. Routing request/response message pairs may be realized within the chord ring using an
iterative or a recursive approach. In the iterative approach, the resolution of the target
node is conducted by the sending node. Therefore, it is asking the next node along the
routing path for the next step. The corresponding node checks its finger table and returns
the requested entry. The sender repeats the procedure by querying the retrieved node
for the next step until the target node has been found and the actual request/response
can be transferred. In the recursive approach, the actual request is send to the next node
along the path, which is further forwarding it. Illustrate both approaches by routing a
ping­pong request/response pair from node 25 to node 12.
d. In a (popular) variation of the recursive approach, the response message is directly send
back to the source (hence, it is not going backward through the recursive “call­stack”).
Can such a mechanism be implemented using RMI? (How?)
e. State one advantage of the iterative and one of the recursive approach.
f. What is the complexity of this routing algorithm – hence, in relation to the maximal
number of nodes within the network, how many steps are required to deliver a message
between two nodes in the worst case? Provide some informal prove.
(all: 3 points)

#### 2. Adding / Removing Nodes to/from a Chord Ring
In addition to the possibility of supporting an efficient routing mechanism between pairs of
nodes within the network, P2P systems also have to be capable of supporting efficient operations
for joining and leaving nodes. Within this assignment, the algorithm for joining and leaving the
network will be covered (for details see this paper).
Note, in addition to the finger tables, every node also maintains a reference to its immediate
successor (first finger table entry) and predecessor on the chord ring. This simplifies the join /
leave algorithms.

Joining the Network:
(one node at a time only – simplified version of the paper’s algorithm) Let n be the new node (n
is its address). When joining the network, the new node has to i) find its location within the ring,
ii) fill its finger table and finally iii) update the finger tables of other nodes which should now
point to the new node.

i. To join the network, the new node needs to know one node which is already connected.
The new node is using this node to conduct a lookup operation, searching for the node
succeeding the location n​, hence the location the new node should be added to. The new
node is now integrated into the ring by manipulating the predecessor / successor
references of the located node and its successor (as for linked lists)

ii. To fill the own finger table, the new node simply queries for the “ideal” addresses the
finger pointer should be pointing to (n + 2i­1) and stores the resulting node addresses.

iii. To update the finger pointers of other nodes, the corresponding nodes have to be
located. The following pseudo code fragment does the trick (remember, m is the number
of finger table entries)

    for(i=1tom){
        //findlastnodepwhosei­thfingermightben
        p=lookup_predecessor(n­2^(i­1));
        //updatefingerscounterclockwiseaslongasnecessary
        while(p.getFinger(i)>n){
            //updatei­thfinger
            p.setFinger(i,n);
            p=p.getPredecessor();
        }
    }
    
Leaving the Network:
In Chord, no active modifications are necessary when leaving the network. This property is
especially useful when facing sporadic failures of nodes. However, to ensure the validity of the
routing information, every node within the network has to verify/update its finger table entries
periodically (just like during the join operation). Further, the successor and predecessor links 
have to be kept valid. In case some node loses its successor, it can ask the node within the 2nd
entry of the finger table for its predecessor and iterate back until it finds its new predecessor.
When updating the successor reference, also the successor’s predecessor is updated. This way,
lost predecessors are recovered as well.

a. Simulate a node joining the network as given within 1a) at position 21.

b. Based on the state of 1a), assume node 12 is failing. Describe the necessary modifications
within the finger tables.

c. What are the values of the finger table and the successor / predecessor references of the
first node to “join” the network? (all: 3 points)

####  3. Building a DHT
The basic infrastructure of a Chord ring can be easily extend to support hash table features. By
definition, every node at address n​i
is responsible for all keys between the address of its
predecessor n​i­1 and its own address (n​i­1​, n​i​]. Therefore, the routing mechanism is slightly
extended such that messages can not only be routed to peers, but also to addresses between
peers (actually standard Chord feature). When routing a message to address k​, the message is
forwarded to the peer with the largest id ≤k, which finally passes on the message to its
successor, which is responsible to process the message.
When inserting an object into a DHT (based on Chord), a key k is generated (e.g. by computing a
hash value for the object) and the object is send to the node responsible for k​. This node has to
store the received value. Lookup / delete operations work the same way.
When nodes are joining the network, they accept the responsibility for a certain interval of the
address space. Therefore, after completing the join procedure, they also have to take over the
data stored within this interval from their successor, which does not involve any routing
operations. Leaving nodes have to hand of their maintained data to their successor within the
ring. Finally, to handle failing nodes, data has to be replicated within the DHT.
a. In the Chord ring of 1a) – perform an add operation for data element with the hash codes
16 and 21 triggered by the node n=1.
b. In the Chord ring of 1a) – when node 21 is joining the network, which interval will it have
to cover?
c. Describe a technique allowing to store data redundantly within the DHT such that it gets
more resilient against failures of
i. single, random nodes
ii. small intervals of the chord address space. (all: 2 point)
