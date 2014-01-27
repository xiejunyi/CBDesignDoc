## Overview

This doc is to help explain possible test scenarios for QE team to test the pause-and-resume XDCR in 3.0. 


## Pause and Resume XDCR

Today users cannnot pause and resume ongoing XDCR. Users can only delete XDCR completely and recreate a new one, in many cases it is an overkill because 
 
- it will trigger a lot of metadata operations after recreating XDCR;
- it will also trigger other overhead like deleting/recreating XDC replication doc and replicate it within cluster.



The new pause-and-resume XDCR will avoid these overhead. When XDCR is paused, 

- XDCR remembers the available tokens before pause;
- XDCR asks concurrency throttle to set the tokens to 0, thus no new vb replicator will be scheduled;
- For all active vb replicators, XDCR will close the changes manger queue to prevent changes reader from further reading mutations from disk or UPR;
- After all remaning mutations in chagnes queue are flushed, all vb replicators will go sleep;


When XDCR is resumed:

- XDCR will be restore the number of available tokens it remembers;
- XDCR asks concurrent throttle to change number of tokens from 0 to the value before pause;
- Sleeping vb replicators will be wakened up, and asking for token from concurrency throttle and resume replication.



## Test Scenarios

#### Basic functionality test:

- test pause/resume on UI;
- test pause/resume using REST API.

After pause, check
- all XDCR stats are flat, no XDCR activity should be seen on both sides.

After resume, check 
- number of active vb replicators recover to that before pause;
- XDCR stats resume;
- finally data should be sync-up on both sides. 

#### Advanced test:

- topology change on source side after XDCR is paused, but before it is resumed
- topology change on destination side after XDCR is paused, but before it is resumed

In both cases, errors could be seen after XDCR resumed. The XDCR behavior will be the same as regular topology change (vb replicators crash and recover).

### More to add?












 



