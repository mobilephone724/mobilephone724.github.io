- Each data file (heap or index) is divided into `postgres` disk blocks, the blocks are numbered sequentially, 0 to 0xFFFFFFFE.
- ```C
  typedef uint32 BlockNumber;
  #define InvalidBlockNumber		((BlockNumber) 0xFFFFFFFF)
  #define MaxBlockNumber			((BlockNumber) 0xFFFFFFFE)
  ```
- So one relation can have $32T$ storage size
	- $$4294967294( blocks) * 8(one block is 8K) / 1024 / 1024/ 1024 = 32T$$
-