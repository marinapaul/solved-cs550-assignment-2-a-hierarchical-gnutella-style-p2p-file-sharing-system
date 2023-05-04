Download Link: https://assignmentchef.com/product/solved-cs550-assignment-2-a-hierarchical-gnutella-style-p2p-file-sharing-system
<br>



In programming assignment 1, you implemented a Napster style file-sharing system where a central indexing server plays an important role. In this project, you are going to <strong>remove the central component and implement a pure distributed file-sharing system</strong>. An example of such a system is the well-known <strong>Gnutella network</strong>.

If you are not familiar with Gnutella, the following links provide some background about the technology. Pay more attention to its architecture and design goals rather than its protocol details, since we are not strictly implementing a full-featured Gnutella client, but a small subset of it.   Gnutella Protocol at <u>http://rfc-gnutella.sourceforge.net/developer/stable/index.html</u> <u>http://rfc-gnutella.sourceforge.net/Proposals/Ultrapeer/Ultrapeers.html</u>

In this programming assignment, you need to design <strong>a hierarchical Gnutella-style peer-to-peer (P2P) system</strong>.




Figure 1. A hierarchy topology for Gnutella network

The topology is shown above. A <strong>leaf-node</strong> keeps only one connection open, and that is to a super-peer. A <strong>super-peer</strong> acts as a proxy to the gnutella network for the client nodes connected to it. Each super-peer is like an index-server in PA1. All the leaf-nodes register the files in the connected super-peer first. Whenever a query request comes in from the connected leaf-node, the super-peer broadcasts the query to all its super-peer neighbors in addition to searching its local storage (check if this file exists in its connected leaf-nodes) and responds if necessary.

Below is a list of what you need to implement:

<ol>

 <li>First of all, we are not implementing the dynamic initialization of the network as in Gnutella. To keep things simple, assume that the structure of the P2P network is <strong>static</strong>. This means you don’t need to implement group membership messages (PING/PONG in</li>

</ol>

Gnutella). Your network is initialized statically using a config file that is read by each peer (including leaf-node and super-peer) at startup time.  After reading the config file, each leaf-node peer knows its connected super-peer, the super-peer knows the connected leaf nodes and neighbor super-peers.

<ol start="2">

 <li>Each leaf-node registers its files to the connected super-peer. (You can consider the superpeer as an index server).</li>

 <li>Having initialized the P2P network, a leaf-node searches for files by issuing a query. The query is sent to the connected super-peer. The super-peer initially checks whether this file exists in other leaf-nodes connected to the super-peer itself. If the file exists in other leafnodes, then a query-hit message is sent.</li>

 <li>In addition, the super-peer sends the query to its neighbor super-peer. Each neighbor looks up the specified file using a local index and responds with a <em>queryhit </em>message in the event of a hit. The <em>queryhit</em> message is propagated back to the original sender by following the reverse path of the query (the following paragraph describes how this can be done). Regardless of a hit or a miss, the super-peer also forwards the query to all of its super-peer neighbors.</li>

</ol>

To prevent query messages from being forwarded infinitely many times, each query message carries a <em>time-to-live (TTL)</em> value that is decremented at each hop from a superpeer to another super-peer. In addition, each query has a globally unique message ID (defined for our purposes as a [leaf-node ID, sequence number]). Each super-peer also keeps track of the message IDs for messages it has seen, in addition with the upstream peers (can be leaf-node or super-peer) where the messages are sent from. You can use an associative array that stores [message ID, upstream peer ID] pairs, where the upstream peer ID could be a peer’s IP address (and port number if necessary). Use your own heuristics to decide the size of this associative array your peer needs to maintain, and flush out old entries at appropriate times (in essence, you don’t want this buffer to grow indefinitely). The purpose of maintaining this data structure at each super-peer is two-fold, one is to prevent a super-peer from forwarding a message it already saw (and forwarded), and the second is to provide the reverse path for the <em>queryhit</em> message to propagate back to the original sender of the query. Note that the <em>queryhit</em> message MUST carry the same message ID as the corresponding query in order to be propagated back correctly.

Ideally <em>query</em> and <em>queryhit</em> messages can be propagated by using TCP socket connections, RPCs, or RMIs.

Assuming the above description, the message formats are defined as follows:

<ul>

 <li>query (message ID, TTL, file name)</li>

 <li>queryhit(message ID, TTL, file name, leaf-node IP, port number)</li>

</ul>

<ol start="5">

 <li>Routing of messages is by broadcast/back-propagation manner. Each super-peer maintains a list of its neighboring super-peers (picked statically by you). A query message from S is broadcasted to all of S’s neighbors and relayed by each receiver until its TTL value decreased to 0. A <em>queryhit</em> message is sent back to the original sender following the reverse path. The message ID is used for this purpose as described previously.</li>

 <li><em>Obtain</em> is achieved by sending a direct download request to the leaf-node in the <em>queryhit</em> message, this is pretty much the same as done in project one.

  <ul>

   <li>obtain (file name)</li>

  </ul></li>

</ol>

<strong>Other requirements:</strong>

<ul>

 <li>Use threads so your peer can serve multiple requests concurrently.</li>

 <li>No GUIs are required.</li>

</ul>

<ul>

 <li><strong>Evaluation and Measurement </strong></li>

</ul>

Deploy at least 10 super-peers. Each super-peer is connected to 1-3 leaf nodes. They can be setup on the same machine (different directories) or different machines. Each leaf-nodes has in its shared directory at least 10 text files of varying sizes (for example 1k, 2k, …, 10k). Make sure some files are replicated at more than one leaf-node sites (so your query will give you multiple results to select).  In addition, in the figure 1, there is an all-to-all connection to the super-peers. In this programming, assignment, you need to test two topologies for super-peers (initialize the topology by assigning neighbors for each super-peer):

<ul>

 <li><strong>An all-to-all topology (Figure 1) </strong></li>

 <li><strong>A linear topology</strong>.</li>

</ul>

Do a simple experiment to evaluate the behavior of your system. Compute the average response time per client query request by measuring the average response time seen by a client, since there may be multiple results for each query, measure the average among them. And repeat this measurement for 200 times and get the average. Use your own judgment/technique to decide when the last query result should come back. For example, define a cutoff time, waiting until that time and compute the result.  In addition, you can explore the response time from the leaf-node within the same super-peer and different super-peers.

Do the same calculation by changing system load, more specifically, do the same experiment where there’s only 1 client issuing queries, then 2 clients, 3 clients, and so on. Draw a plot after collecting all the data and justify your conclusion.

<u>You should compare the system performance between an all-to-all topology and linear topology</u> <u>under the same system size. In real-word, for this hierarchical architecture, linear topology for</u> <u>super-peers is not used. Please explain the reason.</u>