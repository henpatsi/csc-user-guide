
## Mahti 

Mahti is an Bull Sequana XH2000 system from Atos. It consists of 1404 compute nodes each with two 64 core AMD processors and each node has 256 GB of memory. The storage conistst of a lustre filesystem with a capacity of 8.7 PB.

### Mahti compute node

Each node has two AMD Rome 7H12 CPUs, with 64 cores each. The CPUs are based on AMDs Zen 2 architecture, supporting the AVX2 vector instruction set. 

The cores in each CPU has two hardware threads, brining the total thread count for a node to 256 threads. Each core has 32 KiB of L1 data cache and 32 KiB of L1 instruction cache, the L2 cache is also private per core and each core has 512 KiB L2 cache. Each core has two FMA (fused multiply add) units, that operate on full 256 bit vectors, meaning operations in 8 single precision floats or 4 double precision floats can be carried out by each unit each clock cycle. Moving up from the core, 4 cores are grouped together into a core complex (CCX), within the CCX the cores share the same 16 MiB of L3 cache. Two of these CCX parts are then combined to form a compute die (CCD). For an in depth description of the Zen 2 core you can read further on it at WikiChip https://en.wikichip.org/wiki/amd/microarchitectures/zen_2

!["CCD configuration"](../img/mahti_ccd.png)  

Two core complexes are combined in the same compute die, each processor then consists of 8 compute dies and an additional I/O die that houses the memory controllers and PCI-e controller. Each node is then made up of two of these processors and one 200gbit HDR network adapter.

!["Node configuration"](../img/mahti_node.png)  

To make the massive amounts of cores more manageable and to provide slightly increased memory performance we run each CPU in NPS4 (NUMA per socket 4) mode that splits each CPU into 4 NUMA domains. Each NUMA domain gets two compute dies for a total of 16 cores, and two memory controllers with 32 GiB of memory in total. Core 0 runs threads 0 and 128, core 1 threads 1 and 129 and so on. The figure below shows how the threads are distributed over each of the cores and NUMA nodes.

!["NUMA configuration"](../img/mahti_numa.png)  

### Network

The interconnect is based on Mellanox HDR InfiniBand and each node is connected to the network with a single 200 Gbps HDR link. The network topology is a dragonfly+ topology. The topology consists of multiple groups of nodes, each of which is internally connected with a fat tree topology, these fat trees are then connected to each other using all to all links. 

!["Simplified dragonfly+ toppology"](../img/mahti_df_ex.png)  

In Mahti there are 234 nodes in each dragonfly group and the internal fat tree has a blocking factor of 1.7:1, with 20 or 18 nodes connected per leaf switch and each leaf switch has 12 links going to the spine switch in the group, all links are 200 Gbps links. There are in total 6 groups, and between the groups there is fully non-blocking all-to-all connectivity, with 5 200 Gbps links going from each spine switch to one spine switch in every other group. 

!["Mahti dragonfly+ toppology"](../img/mahti_df.png)

### Storage

Mahti has a 8.7 PB Lustre parallel storage system providing space for [home](disk.md#home-directory), 
[project](disk.md#projappl-directory) and [scratch](disk.md#scratch-directory) storages. 