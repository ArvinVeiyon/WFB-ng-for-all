RX Ring
=======

rx_ring is a circular buffer, where we store packets, grouped by FEC blocks. It has two variables: *rx_ring_front* (index of the first allocated FEC block) and *alloc_size* -- number of allocated blocks.

When we receive a new packet it can belongs to:
1. New fec block - we need to allocate it in RX ring (do nothing if block was already processed)
2. Already existing fec block - we need to add it to them (do nothing if packet already processed)

When we allocate a new block we have following choices:
1. Add a new block to rx ring tail.
2. Override a block at rx ring head if rx ring is full.