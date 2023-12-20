## buffer tag
id:: 6581a94f-7372-4aef-9531-962f2170ffec
	- ```
	  BufferGetTag(Buffer buffer, RelFileLocator *rlocator, ForkNumber *forknum,
	  			 BlockNumber *blknum)
	  {
	  	bufHdr = GetLocalBufferDescriptor/GetBufferDescriptor
	      *rlocator = BufTagGetRelFileLocator(&bufHdr->tag);
	      *forknum = BufTagGetForkNum(&bufHdr->tag);
	      *blknum = bufHdr->tag.blockNum;
	  }
	  ```
-