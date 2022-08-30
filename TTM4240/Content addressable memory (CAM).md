### CAM application; router address lookup
Hardware search engines that are much faster than algorithmic approaches for search-intensive applications

CAMs are composed of conventional semiconductor memory (usually SRAM) with added comparison circuitry that enable a search operation to complete in a single clock cycle. The two most common search-intensive tasks that use CAMs are packet forwarding and packet classification in Internet routers. I introduce CAM architecture and circuits by first describing the application of address lookup in Internet routers. Then we describe how to implement this lookup function with CAM.

![[Pasted image 20220829105455.png]]
	*Checks if packets are matching the address. If an adress is matching more than one in the table, if will chose the longest-prefix matching*

IPv6 adresses will make these list a lot more bigger in the future

Almost all algorithmic approaches are too slow to keep up with projected routing requirements. In contrast, CAMs use hardware to complete a search in a single cycle, resulting in constant *O(1)* time complexity. This is accomplished by adding comparison circuitry to every cell of hardware memory. The result is a fast, massively parallel lookup engine. The strength of CAMs over algorithmic approaches is their high search throughput. The current bottleneck is the large power consumption due to the large amount of comparison circuitry activated in parallel. Reducing the power consumption is a key aim of current CAM research.

##### Content-addressable memory
A CAM search operation begins with precharging all matchlines high, putting them all temporarily in the match state. Next, the search line drivers broadcast the search data, 01101 in the figure, onto the search lines. Then each CAM core cell compares its stored bit against the bit on its corresponding search lines. Cells with matching data do not affect the matchline but cells with a mismatch pull down the matchline. Cells storing an X operate as if a match has occurred. The aggregate result is that matchlines are pulled down for any word that has at least one mismatch. All other matchlines remain activated (precharged high). In the figure, the two middle matchlines remain activated, indicating a match, while the other matchlines discharge to ground, indicating a mismatch. Last, the encoder generates the search address location of the matching data.
![[Pasted image 20220829105606.png]]
	*In the example, the encoder selects numerically the smallest numbered matchline of the two activated matchlines, generating the match address 01. This match address is used as the input address to a RAM that contains a list of output ports as depicted in Figure 2. This CAM/RAM system is a complete implementation of an address lookup engine. The match address output of the CAM is in fact a pointer used to retrieve associated data from the RAM. In this case the associated data is the output port. The CAM/RAM search can be viewed as a dictionary lookup where the search data is the word to be queried and the RAM contains the word definitions. With this sketch of CAM operation, we now look at the comparison circuitry in the CAM core cells.*

### CAM circuits
![[Pasted image 20220829105704.png]]
	*Displays a conventional SRAM core cell that stores data using positive feedback in back-to-back inverters. Two access transistors connect the bitlines, bl and /bl (we use the prefix / to denote the logical complement in the text and we use an overbar in the figures), to the storage nodes under control of the wordline, wl. Data can be read from the cell or written into the cell through the bitlines. We use this differential cell as the storage for building CAM cells.*

![[Pasted image 20220829110222.png]]
	*Depicts a conventional binary CAM (BCAM) cell with the matchline denoted ml and the differential search lines denoted sl and /sl. The figure also lists the truth value, T, stored in the cell based on the values of d and d. Read and write access circuitry is omitted for clarity in this figure and subsequent CAM core cell figures. For a binary CAM, we store a single bit differentially. The comparison circuitry attached to the storage cell performs a comparison between the data on the search lines (sl and /sl) and the data in the binary cell with an XNOR operation (ml = ! (d XOR sl)). A mismatch in a cell creates a path to ground from the matchline through one of the series transistor pairs. A match of d and sl disconnects the matchline from ground.*

![[Pasted image 20220829110252.png]]
	*Shows a ternary CAM (TCAM) cell. The TCAM cell stores an extra state compared to the binary CAM, the don't care state, labeled X, which necessitates two independent bits of storage. When a don't care is stored in the cell, a match occurs for that bit regardless of the search data. The figure shows that the TCAM cell stores X when d0 = d1 = 0. The state d0= d1= 1 is undefined and is never used.*

![[Pasted image 20220829110307.png]]
If mismatch on the bits it will discharge the path