- tag::WAL
- ## Overview
  This chapter explains the content of `clog` #SLRU_IMPLEMENT
  
  `clog`(commit log), records the commit status of each transaction. The log exists both in memory mannaged by `slru` buffer and disk for durability. The commit status can be the four kinds below:
  ```C
  #define TRANSACTION_STATUS_IN_PROGRESS		0x00
  #define TRANSACTION_STATUS_COMMITTED		0x01
  #define TRANSACTION_STATUS_ABORTED			0x02
  #define TRANSACTION_STATUS_SUB_COMMITTED	0x03
  ```
- ## In-Disk Representation
  Thinking that the commit status of each transaction composites an array `clog[]` and `clog[xid]` records the status, we can easily store the array to disk by the `slru`.
  
  The status of one transaction needs two bits to represent:
  ```C
  #define CLOG_BITS_PER_XACT	2
  #define CLOG_XACTS_PER_BYTE 4
  #define CLOG_XACTS_PER_PAGE (BLCKSZ * CLOG_XACTS_PER_BYTE)
  #define CLOG_XACT_BITMASK	((1 << CLOG_BITS_PER_XACT) - 1)
  ```
  
  So we can get the xid's index and offset in page and byte.
  ```C
  #define TransactionIdToPage(xid)	((xid) / (TransactionId) CLOG_XACTS_PER_PAGE)
  #define TransactionIdToPgIndex(xid) ((xid) % (TransactionId) CLOG_XACTS_PER_PAGE)
  #define TransactionIdToByte(xid)	(TransactionIdToPgIndex(xid) / CLOG_XACTS_PER_BYTE)
  #define TransactionIdToBIndex(xid)	((xid) % (TransactionId) CLOG_XACTS_PER_BYTE)
  ```
  
  Thinking of that one slru segment contains 32 pages, so we name the clog file as `0000`(contains xid in [0, 32 * CLOG_XACTS_PER_PAGE - 1]), `0001`(contains xid in [32 * CLOG_XACTS_PER_PAGE, 32 * CLOG_XACTS_PER_PAGE * 2 - 1]) and so on. Because four hex numbers can represent $16^4=2^{12}$ files with $2^{12} \times 32 \times 8192 \times 4 = 2^{32}$ transactions' status(a int32 size)
  
  Attension, such simple mapping means that the pages in clog file don't have page headers. So we can't record `LSN`, `checksum` in each page. The lack of `LSN` means the changes of clog page wouldn't be recorded in `WAL` but clog doesn't need it indeed.
- ## Extend And Truncate
  - ### high level design
    - During the process of generating a new `xid`, we make sure that the slru page exists.
      * If it's the first xid of the page, we allocate a new page in clog buffer.
      * Also generate a WAL to record the birth of the page.
      * If not, the page must exist in memory or flushed into disk. So it's for slru
      layer to manage such situation.  Keep in mind that the general self-increment xid does't begin at zero:
      ```C
      #define FirstNormalTransactionId	((TransactionId) 3)
      ```
      so:
      * During bootstrap, initialize the first clog page
      * During extend new pages, be careful about the `FirstNormalTransactionId`,
      since it is not the first xid in page representation but the first general one.
      
      The above behaviors indicate that although a clog segment at most occupies 256K space, it doesn't have such size just after initialization. We extend 8K pages one by one during the xid increment.
      
      Since at most half of `uint32` xids can be in use, it's natural to clean up out of date clog files. Different from extending a page, we always delete a whole page. So once we promote the `frozenxid`, we try to find some clog files to delete:
      1. The judgement whether there is a file can be deleted is completed in slru layer(a loop to scan the directory), but clog layer supports a hook to judge one file.
      2. Advance the oldest clog xid in shared memory
      3. Generate a clog truncate WAL record
      4. Real truncate. Complemented in slru layer.
      
      Details of the two kind WAL record will be shown later.
  - ### WAL level details #wal
    - For the latter one,
    - It's a matter of course to generate a WAL record., since we must guarantee the clog to be recovery-safe. But some details deserve a glance:
    - For extending a new page, it makes no difference that we flush the WAL record now or later. Since once we want to set status in a non-existent page during recovery, we can padding a new empty page. This trick doesn't affect the page usage.
      logseq.order-list-type:: number
    - For deleting a clog segment, we have no chance to remedy the lost of clogs, and the disaster means a lot of tuple can be accessed at all. So regardless of the synchronous commit level, we must ensure the WAL record has flushed into disk before really delete the segments.
      logseq.order-list-type:: number
    -
    -
- ## Set And Get
  - ### high level design
    - For now, if you are not interested in the details, the following two points are enough
      1. The pair of operations wouldn't generate any WAL record
      2. They are done during the commit or abort procedure.
    - `TransactionIdSetTreeStatus`
      id:: 6589833b-83a9-47e9-b307-cbffb3a5b40b
      #+BEGIN_QUOTE
      Record the final state of transaction entries in the commit log for a transaction and its subtransaction tree.
      #+END_QUOTE
      (See [[subtransaction]] for backgrounds)
      - The key point is that the whole operation must be atomic.
        - The reason is that, this function doesn't acquire the exclusive lock in the beginning, so despite crash-recovery, we must still ensure that other running `xacts` can get the right state of the transaction.
        - In the commit case,  atomicity is limited by whether all the `subxids` are in the same CLOG page as xid.  
          * If they all are, then the lock will be grabbed only once, and the status will be set to committed directly.  
          * Otherwise, we must:
          #+BEGIN_IMPORTANT
            1. set sub-committed all subxids that are not on the same page as the main xid
            2. atomically set committed the main xid and the subxids on the same page
            3. go over the first bunch again and set them committed
          #+END_IMPORTANT
          Note that it's only the `commit` situation. For `abort`  situation, just setting the top xid seems enough
          #+BEGIN_NOTE
          When a top-level transaction with an xid commits, all of its subcommitted child subtransactions are also persistently recorded as committed in the pg_xact subdirectory. If the top-level transaction aborts, all its subtransactions are also aborted, even if they were subcommitted.
          #+END_NOTE
        - For **example**
          TransactionId t commits and has subxids t1, t2, t3, t4. t is on page p1, t1 is also on p1, t2 and t3 are on p2, t4 is on p3
          * update pages2-3: 
            * page2: set t2,t3 as sub-committed
          * update page1: 
            * page1: set t,t1 as committed
          * update pages2-3: 
            * page2: set t2,t3 as committed
            * page3: set t4 as committed
      - Another notable point is that this function supports both sync and async commit, where the later one uses the group commit technique.
      - utils:
        - `set_status_by_pages`
          - #+BEGIN_QUOTE
            set the status for a bunch of transactions, chunking in the separate CLOG pages involved
            #+END_QUOTE
        - `TransactionIdSetPageStatus`
          - #+BEGIN_QUOTE
            Record the final state of transaction entries in the commit log for all entries on a single page.  **Atomic only on this page**.
            #+END_QUOTE
          - Note that the exclusive lock will be hold only in this function.
    - `TransactionIdGetStatus`
      collapsed:: true
      - #+BEGIN_QUOTE
        Aside from the actual commit status, this function returns  an LSN that is late enough to be able to guarantee that if we flush up to that LSN then we will have flushed the transaction's commit record to disk.
        #+END_QUOTE
        See ((658994e9-09a2-4b11-a960-f08770e30d01)) for detail
  - ### WAL level details #wal
    - It's unbelievable but tricky that ((6589833b-83a9-47e9-b307-cbffb3a5b40b)) wouldn't generate a WAL record. But since only the transactions that changes the content data(some hint flags are exception, such as tuple infomask, but that's unimportant) will have a xid(and then record on clog segment). During the replay of such transactions' commit(or abort? (#TODO will aborted transactions be replay in recovery?)) WAL record, we can redo the clog by the way.
    - Clog also supports group lsn to enhance the efficiency
      id:: 658994e9-09a2-4b11-a960-f08770e30d01
      - {{embed ((6589991b-a89e-4b6f-991a-3e75cbc6538b))}}