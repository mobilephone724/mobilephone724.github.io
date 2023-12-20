# wal record
	- Logically, a `WAL` record must contains the three parts below:
		- Which page to modify.(An unique ID, See [[block]]  and [[relfilelocator]] )
		  logseq.order-list-type:: number
		- The kind of `WAL`. Different kinds of `WAL` have different modification methods
		  logseq.order-list-type:: number
		- The way to modify the page
		  logseq.order-list-type:: number
	- Unique page ID
		- For heap pages:
			- A `relfilelocator` + `block number` can locate to a concrete page
			- But structure `registered_buffer` also add ((65805626-a222-4aa2-88e3-8a3dcfda498f)) here
				- DONE what's the reason #WAL
				  :LOGBOOK:
				  CLOCK: [2023-12-19 Tue 22:12:09]--[2023-12-19 Tue 22:12:09] =>  00:00:00
				  CLOCK: [2023-12-19 Tue 22:12:10]--[2023-12-19 Tue 22:12:10] =>  00:00:00
				  CLOCK: [2023-12-19 Tue 22:12:13]--[2023-12-19 Tue 22:12:14] =>  00:00:01
				  CLOCK: [2023-12-19 Tue 22:23:51]--[2023-12-19 Tue 22:23:51] =>  00:00:00
				  :END:
				- Record the changes of #fsm and #vm
		- See [[wal insert]] for how the unique id is record
		-
-
	-
-
-
-