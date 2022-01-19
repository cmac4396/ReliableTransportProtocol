# Reliable Transport 
*This project is for Networks and Distributed Systems (CS3700) at Northeastern University.*

***
Mouhamadou Sissoko and Candice Mac -- 11-06-21 --

This program is a reliable transport protocol that provides reliable datagram services on top of user datagram protocol

## Important Modules

* **sys**: used to write and read from standard in/out
* **socket**: used to establish connections for packet and message forwarding between neighbors.
* **json**: used to encode and decode messages received in JSON format.
* **time**: used to keep track of time in order to calculate the round trip time
* **datatime**: used to keep track of time of every action we do

## Key Features

## Sending Packets

### Sending The Next Packet

When we want to send the next packet, we first check if we've sent all the fragments from the packet. If we do then we
exit we break out of the "send_next_packet" function because the transmission of that packet has been complete. However,
if we haven't sent all the fragments then we exactly 1500 bytes of data from standard input and check if it has any
data. If it does, I create the msg, send the msg, record the current time, calculate the time needed for the packet to
time out(time that we sent the packet + 2 * Round Trip Time), and we store it in a list of dictionaries(packets_sent),
where all the keys are sequence numbers and the values are dictionaries:
[the message we are sending, the time we sent the packet, the packet timeout, and the number of retransmissions].
Finally, we increment the global sequence number, with the length of the new data that we just sent.

### Retransmitting Packets

When we want to retransmit a packet given a sequence number, we use the "retransmit_sequence" function. First we pull
the packet out of the list of dictionaries(packets_sent), using the inputted sequence number as the key, then we pull
the message that was sent with that packet, using "packet message" as the key. We then send the message that we pulled
out, record the current time, calculate the time needed for the packet to time out, and we replace it in a list of
dictionaries

### Sending The Initial Batch Of Packets

Before we start sending the initial batch of packets, we record the time of the latest packet received(It might be weird
because technically, we haven't received a packet, but we need some sort of initial time, so that's why we set it to the
time right before we send our initial packets). Now, we send <= WINDOW - 1 amount of packets. The reason why I added the
less than symbol is because the amount of fragments may be lower than the WINDOW - 1.

### Reading In Data and Sending More Packets

Now we are in a loop, where we constantly read and send packets, until the end of the file or we timeout. First we read
at most 1500 bytes of data, and if we do get data, we set the data equal to the "result", but if we don't we set the
result to None. We then check to see if we have data stored in result, and if we do we update the
latest_packet_reception_time to the current time, decode the data, check if the decoded data's SACK(This holds the
sequence number) is in the list of dictionaries, if so we know that it has been received, so we recompute the RTT and
delete it from the list of dictionaries(So list doesn't get too big and slow us down. However, if it hasn't been
received, we remove every sequence number less than the ack number of the sequence we just received because if we know
packets are in order(We implemented dynamic packet reordering), so we must've received all the packets before it. Now we
send out next batch of packets. Check if we've reached EOF, if we have then break out of the while loop

### Resending Packets If Needed

After we've finished reading in data and sending more packets, we have to check if we need to resend packets. In order
to do this, we first store the current time, we then check if the latest packet we've received has timed out and if it
has we time out and exit; Otherwise we iterate through all the remaining packets that we've sent and haven't removed,
and if the current time is greater than their packet timeout limit, we retransmit them

### Sending EOF Message

Finally, we send the EOF Message, once we've sent all the necessary packets and reached the end of the file.

## Receiving Packets

### Handling Received Packets Dynamically

For handling our packets and writing it to stdout in-order, we did it dynamically. We essentially, used the
handling_received_packets function and what it does is: Sort the packets based on sequence number, iterates through the
list of sorted packets and for each iteration it checks to see if that packet is equal to the current total data read
and if it is it writes it to stdout and increments the current total data read by the length of the data in that packet.
After it iterates through the list, it removes all the packets that it has written to stdout. This ensures that packets
are sent in the right order.

### Receiving Packets

We start listening for packets, if we receive nothing we time out, but if we receive packets, we decode it, and if the
packet's EOF flag is true, we send the final packet 5 times to ensure that 3700send gets it and we exit. If the EOF flag
isn't true, we check if we haven't received that same packet, check whether it's in order or out of order, log it, use
our handle_received_packet function, and finally send an ack back to the sender.

### Challenges

Most of our problems came from figuring out how to catch drop packets and retransmit them

### Testing

We tested using nettest and logging various things such as sequence numbers, length of the list of packets sent, and etc...


 
