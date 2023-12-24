## background
- From [Official introduction ot Subtransactions](https://www.postgresql.org/docs/current/subxacts.html)
  #+BEGIN_QUOTE
  Subtransactions are started inside transactions, allowing large transactions to be broken into smaller units. Subtransactions can commit or abort without affecting their parent transactions, allowing parent transactions to continue. This allows errors to be handled more easily, which is a common application development pattern. The word subtransaction is often abbreviated as **subxact**.
  #+END_QUOTE
- ## high level view
- The main function:
  - From from `backend/access/transam/subtrans.c`
    * Tree like structure
    
    #+BEGIN_QUOTE
    The pg_subtrans manager is a pg_xact-like manager that stores the parent transaction Id for each transaction
    A main transaction has a parent of InvalidTransactionId, and each subtransaction has its immediate parent. **The tree can easily be walked from child to parent, but not in the opposite direction.**
    #+END_QUOTE
    
    * No restart or crash recovery restore. But this doesn't mean no disk storage, since we still consider that the `slru` buffer may be full.
    
    #+BEGIN_QUOTE
    we only need to remember pg_subtrans information for currently-open transactions. Thus, there is no need to preserve data over a crash and restart. During database startup, we simply force the currently-active page of SUBTRANS to zeroes.
    #+END_QUOTE
  -
- page organization
  - Thinking of [[clog]] , to tell the parent `Xid` of each `Xact`, just store a very large array like:
    ```
    xid parent_xid[XID_UPPER_LIMIT]
    ```
    In clog, two bits are needed to represent the the commit status of a `xid`. Similarly, `sizeof(xid)`(That is 32 bits) is need to represent its parent.
- ## lower level design
- `SubTransSetParent`
  - A notable optimal point is:
    ```C
    if (*ptr != parent)
    {
    	Assert(*ptr == InvalidTransactionId);
    	*ptr = parent;
    	SubTransCtl->shared->page_dirty[slotno] = true;
    }
    ```
    If there is no need to set the parent xid, we can avoid dirtying the buffer.
- `SubTransGetParent`
  id:: 6587fc49-7103-4bea-80a6-dcdf6d913ecb
  - A notable point is that not all meaningful `xid` are in current manger
    ```
    /* Bootstrap and frozen XIDs have no parent */
    if (!TransactionIdIsNormal(xid))
    	return InvalidTransactionId;
    ```
  - Another point is that is the assert.
    ```C
    Assert(TransactionIdFollowsOrEquals(xid, TransactionXmin));
    ```
    `TransactionXmin` is:
    #+BEGIN_QUOTE
    the oldest xmin of any snapshot in use in the current transaction (this is the same as MyProc->xmin).
    #+END_QUOTE
    Since the parent `xid` of a visible `xid`(from `xmin` or `xmax` of a tuple) is not always record in `subtrans` manager, we must forbid everyone from visiting a `subtrans` record that may not exist.
    - So all the three callers of this function always verify the input value:
      
      1. {{embed ((6588436e-0c99-4cd1-b1a7-fbddbfb25eea))}}
      2. {{embed ((65884437-33ff-48c0-b62f-88248615c4cc))}}
      3. ((6587fa8a-d02f-4b70-9995-66f1a0394080)) Just below
- `SubTransGetTopmostTransaction`
  id:: 6587fa8a-d02f-4b70-9995-66f1a0394080
  - Same as ((6587fc49-7103-4bea-80a6-dcdf6d913ecb))
    #+BEGIN_QUOTE
    Because we cannot look back further than TransactionXmin, it is possible that this function will lie and return an intermediate subtransaction ID. This is OK, because in practice we only care about detecting whether the topmost parent is still running or is part of a current snapshot's list of still-running transactions.
    #+END_QUOTE
- `StartupSUBTRANS`
  - Clear all the pages from `oldestActiveXID` to `nextXid`
    #+BEGIN_QUOTE
    `oldestActiveXID` is the oldest XID of any prepared transaction, or nextXid if there are none.
    #+END_QUOTE `oldestActiveXID` is
  - A notable point is that we may encounter page wraparound here
    ```C
    while (startPage != endPage)
    {
    	(void) ZeroSUBTRANSPage(startPage);
    	startPage++;
    	/* must account for wraparound */
    	if (startPage > TransactionIdToPage(MaxTransactionId))
    		startPage = 0;
    }
    ```