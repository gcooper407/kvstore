# Distributed Key-Value Database
## High-Level Approach
At a high level, this program runs entirely within a single while loop. On 
each iteration of the loop, it check what status this replica is -- whether it is a 
leader, candidate, or follower. The program also has non-leaders continuously 
check if their timeout has been exceeded, and if so, start an election. 
Starting with followers, the program runs a loop checking for messages that have 
been received but not processed yet. The types of messages that a follower 
will process are as follows: gets and puts, a request vote, or an append 
entry (can mean either an appendEntry or a heartbeat).
For candidates and leaders, it is much the same, just with slightly 
different parameters and checks.
## Challenges
### Getting started
Due to the complex nature of the RAFT protocol, it was pretty difficult to get started.
This was mainly just due to understanding the complex RAFT protocol and 
devising a way to implement that in python. The following links guided a 
majority of the final implementation: [Rutgers](https://people.cs.rutgers.edu/~pxk/417/notes/raft.html) and 
[Raft Github](https://raft.github.io/slides/uiuc2016.pdf). The Rutgers 
write-up helped a lot to get a high level broad overview of the protocol 
while the RAFT GitHub link (provided in the assignment) helped with the 
finer implementation details (particularly pg.4)
### Timeout
Another issue that plagued development was that too many elections were going off. To 
solve this issue, we first increased the random range for timeouts from 0.15-0.
30 up to 0.50-0.65. This helped deal with some lag that may be caused by the 
run mock server that slowed down just a touch as to interfere with the regular 
running of the program. We then made it so whenever append entries were 
ready to go, the program would send them out immediately while heartbeats would be 
the only ones that followed the "just-in-time" method (being sent out right 
before the earliest timeout). We choose 0.45 for the "just-in-time".
### Dictator
We had an issue where when a follower becomes a leader, it doesn't reset 
its vote counter. This lead to issues such as when we had a network 
partition, a follower would continuously become a candidate and would keep 
on voting for itself. And thus this lead to that candidate basically 
electing itself, while also continuously increasing its term number, thus 
causing issues when the partitions were brought back together. Also, when 
choosing the legitimate leader, the program initially only checked term numbers, 
meaning that false candidate/leader who hosted many of its own elections 
would have an astronomically high term number and thus be seen as the "true" 
leader. So, we just added a few lines to reset vote counts and choose the 
rightful leader based off of both term numbers and number of log 
entries. 

## Interesting Features
One interesting feature of the program is that whenever it has an 
update to send out, it sends immediately, disregarding the "just-in-time" 
system. This greatly increased the program's response time while also limiting the 
number of heartbeats sent out. We also had a "fun" feature where we had a 
dictionary with each replica and the last time that the program sent something to 
that replica. This helped immensely with sending redundant heartbeats as it 
would now only send a heartbeat when the timeout for a particular replica 
was about to expire, not the entire thing as a whole. 

## Testing
After passing the initial milestone, we somewhat gun-hoed it and then 
fleshed out the rest of the program without stopping to test it much. That...
was a mistake. Afterward, we began to rewrite parts of our program and each 
time we would run the simple tests fairly regularly to ensure that the 
changes we were making did not break the program. Then later on, if running 
an advanced test succeeded, we could be fairly certain the core 
functionality of the kv store was working.
