- ```C
  typedef struct RelFileLocator
  {
  	Oid			spcOid;			/* tablespace */
  	Oid			dbOid;			/* database */
  	RelFileNumber relNumber;	/* relation */
  } RelFileLocator;
  ```
- `RelFileNumber` is the file name of the relation
  - Oid of a relation is logically existent.
  - RelFileNumber is physically existent, since operations like `vacuum full` and `truncate` will create new files with different file name
- `ForkNumber`
  id:: 65805626-a222-4aa2-88e3-8a3dcfda498f
  - It is the file "type" of a relation
    - The main data, fsm and vm are all buffered by the [[shared-buffers]] .
    - See ((6581a94f-7372-4aef-9531-962f2170ffec)) for detail
  - `MAIN_FORKNUM` : heap file
  - `FSM_FORKNUM` : free space manager file
  - `VISIBILITYMAP_FORKNUM` : VISIBILITYMAP file
  - `INIT_FORKNUM` : for  [unlogged table](https://www.postgresql.org/docs/current/sql-createtable.html#SQL-CREATETABLE-UNLOGGED)
    - {{embed [[unlogged-table]]}}
  -
-