# Various test cases with VPA and its observations

This document explains the various test cases that we tested VPA on, divided into memory and cpu utilization. The following are the observations made for each of these.

## When memory is under utilized

![Memory UnderUtilization](/images/Memory_UnderUtilization.png)

This is memory under-utilization scenario. Plot shows memory of the pod on y-axis with time on x-axis. Top red line is the allocated memory, where both request and limit is 570mb. Bottom green line is the memory usage of application which is 300 initially and further reduced to 130mb. After few hours, memory request dropped by 20mb. This will further reduce with time based on history, but not instantly. 


## When memory is over utilized 

![Memory OverUtilization](/images/MemoryOverUtilization.png)

In memory over-utilization scenario, green line shows usage, which is higher than blue line which is requested memory. After some time of over-utilization, request increased from 260 to 350mb. 

## CPU - On and Off scenario

This scenario is to simulate the workload of a application where there is high CPU usage in day time and no load in the night time. To test this, we created a cron job that sends 150mc of cpu every hour in the day time. Initial cpu request was 25mc. With workload, cpu request changed to 182mc and then to 203mc CPU. In the night time when there was no load on CPU, we observed that request values drop gradually in steps as in the graph below, but not instantly. 

![CPU On and Off scenario](/images/CPU_OnOff_NightTime.png)


