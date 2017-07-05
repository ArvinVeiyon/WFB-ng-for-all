RX Ring
=======

rx_ring is a circular buffer, where we store packets, grouped by FEC blocks. It has two variables: *rx_ring_front* (index of the first allocated FEC block) and *alloc_size* -- number of allocated blocks.

When we receive a new packet it can belongs to:
1. New fec block - we need to allocate it in RX ring (do nothing if block was already processed)
2. Already existing fec block - we need to add it to them (do nothing if packet already processed)

When we allocate a new block we have following choices:
1. Add a new block to rx ring tail.
2. Override a block at rx ring head if rx ring is full.

block idx is rounded by modulo 256 - i.e.  0, 1, 2, ... 255, 0, 1 ...
For example:
1.  *rx_ring_front* == 5 and *alloc_size* == 2 and last last allocated block has *block_idx* == 132
2.  We got a packet with *block_idx* == 141. We need to allocate a (141 - 132) mod 256 = 9 new blocks. Such packet reordering can happens due to different latency in WiFi adapters if we use multiple adapters/antennas or packet reordering in adapter's internal queue.
3. This code allocates *new_blocks* in rx ring. It clears rx ring slots from previous data and write indexes of new blocks to rx ring slots. 