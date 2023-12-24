- This is the commit log interface routines.
- ## low level design
  - `TransactionIdDidCommit`
    id:: 6588436e-0c99-4cd1-b1a7-fbddbfb25eea
    - A notable point is to call ((6587fc49-7103-4bea-80a6-dcdf6d913ecb))
      id:: 65884378-6b98-44b0-bd8f-e695abb68ca3
      ```C
      if (TransactionIdPrecedes(transactionId, TransactionXmin))
      	return false;
      parentXid = SubTransGetParent(transactionId);
      ```
      Must verify that `transactionId` logically is less than `TransactionXmin` first.
  - `TransactionIdDidAbort`
    id:: 65884437-33ff-48c0-b62f-88248615c4cc
    - A notable point is to call ((6587fc49-7103-4bea-80a6-dcdf6d913ecb))
      ```C
      if (TransactionIdPrecedes(transactionId, TransactionXmin))
      	return true;
      parentXid = SubTransGetParent(transactionId);
      ```
      Must verify that `transactionId` logically is less than `TransactionXmin` first.