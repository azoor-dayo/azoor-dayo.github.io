---
tags: programming, gpu, cpp, graphics, opengl, vulkan, cuda
---

# OpenGL Programming
- 
# Vulkan Programming
- Vulkan Best Practices https://developer.nvidia.com/blog/vulkan-dos-donts/
- Cutting edge shit: Mesh Shaders https://developer.nvidia.com/blog/introduction-turing-mesh-shaders/
- ## Pipeline
	- Intro [Lecture Vulkan Intro.pdf](attachments/Lecture_Vulkan_Intro_1707281667185_0.pdf)
	- ![image.png|680](attachments/image_1707281439749_0.png)
	- ![image.png|683](attachments/image_1707281457801_0.png)
	- ![image.png|684](attachments/image_1707281471813_0.png)
	- 
# CUDA Programming
- HOST: The CPU
- DEVICE: The GPU
- |Function Signature|Executed on the:|Only callable from:|Remarks|
	|--|--|--|
	|\_\_device\_\_ void DeviceFunc()|device (gpu)|device (gpu)|Used together with __host__. Can only be called from another device func or a kernel/global func|
	|\_\_host\_\_ void HostFunc()|host (cpu)|host (cpu)|Used together with __device__, redundant if used alone|
	|\_\_global\_\_ void KernelFunc()|device (gpu)|host (cpu)|Is a kernel function. MUST return void. Called with the \<\<\<dim3\>\>\> semantics|
- ## Host Code
	- Example:
	- cpp code block below:
	```cpp
		void vecAdd(float* h_A, float* h_B, float* h_C, int n)
		{
		// d_A, d_B, d_C allocations and copies omitted 
		// Run ceil(n/256.0) blocks of 256 threads each
		vecAddKernel\<\<\<ceil(n/256.0),256\>\>\>(d_A, d_B, d_C, n);
		}
		```
- ## Device Code
	- 
- ## Kernel Code
	- All threads on a block executes the SAME kernel code.
- ## Compilation
	- sth sth NVCC and GCC or sth idk
	- has to do sth with \_\_host\_\_ and \_\_device\_\_
- ## Grids and Blocks and Threads and Warps
	- ### Heirarchy
		- Threads: Lowest level of compute. One thread = one running guy.
		- Warps: Usually groups of 32 threads. Warps are the smallest unit of execution a GPU can do. All threads in warps execute the same thing, but on different data.
		- Blocks: Groups of THREADS, not warps.
		- Grids: Groups of BLOCKS.
	- Grids are the BIG picture. There is a grid of BLOCKS. Each block contains THREADS.
	- Each level can be 1D, 2D or 3D. Denoted as dim(x,x,x).
	- x,y,z follow the top-left origin format. Like MS Excel.
	- x: COLUMN
	- y: ROW
	- z: DEPTH (idk go in or come out)
	- ![image.png](attachments/image_1705031096910_0.png)
		Example: 
		Grid of dim(2,2,1) blocks.
		each block has dim(2,2,4) threads.
		Total threads: (2\*2\*1)*(2\*2\*4) = 64 threads
	- Example of kernel code that uses grid/block in a 2D format:
		```cpp
		// Scale every pixel value by 2.0
		__global__ 
		void PictureKernel(float* d_Pin, float* d_Pout, int height, int width) {
		// Calculate the row # of the d_Pin and d_Pout element
		int Row = blockIdx.y*blockDim.y + threadIdx.y;
		
		// Calculate the column # of the d_Pin and d_Pout element
		int Col = blockIdx.x*blockDim.x + threadIdx.x;
		
		// each thread computes one element of d_Pout if in range
		if  {
			d_Pout[Row*width+Col] = 2.0*d_Pin[Row*width+Col];
		}
		}
		```
	- Example of configuring the grid and block dim sizes before launching the above kernel code:
		```cpp
		// assume that the picture is m × n, 
		// m pixels in y dimension and n pixels in x dimension
		// input d_Pin has been allocated on and copied to device
		// output d_Pout has been allocated on device
		dim3 DimGrid((n-1)/16 + 1, (m-1)/16+1, 1);
		dim3 DimBlock(16, 16, 1);
		PictureKernel\<\<\<DimGrid,DimBlock\>\>\>(d_Pin, d_Pout, m, n);
		```
	- ![image.png|555](attachments/image_1705032054961_0.png)
		_Using pre-defined block size (2D 16x16) to make a grid that can cover all pixels of an image. Some parts of some blocks may not be used, so bounds checking need to be considered_
- ## Thread Scheduling
	- Each GPU has a number of SMs (Streaming Multiprocessors).
	- Each SM can take up to a certain number of threads, depending on GPU. (Fermi takes 1536)
	- ALL SMs can only take up to 8 blocks at a time. Blocks are user-defined during kernel code execution.
		- ![image.png](attachments/image_1705035405002_0.png)
	- Something something warps are smallest unit of execution. Each warp does 32 threads, until newer GPUs change that number.
	- ### Optimizing SM and thread count
		- Try to squeeze the max amt of threads into an SM as possible, with the thread count limit + block count limit in consideration.
		- For Fermi gpu, should i use 8x8, 16x16 or 32x32 blocks? (Fermi: 1536 threads per SM)
		- 8x8
			- Threads: 8\*8= 64 threads per block
			- Could fit 1536 / 64 = 64 blocks
			- But max can fit 8 blocks
			- So each SM can hold 8 blocks, which is 512 threads out of 1536 threads utilized.
		- 16x16
			- Threads: 16\*16= 256 threads per block
			- Could fit 1536 / 256 = 6 blocks
			- All of 6 blocks can fit into an SM's limit of 8 blocks
			- So each SM can hold 6 blocks, which is 1536 threads out of 1536 threads utilized.
		- 32x32
			- Threads: 32\*32= 1024 threads per block
			- Could fit 1536 / 1024 = 1.5 blocks
			- All of 1.5 blocks can fit. But I can only choose 1 or 2 blocks to put in, so i can only put in max of 1 block.
			- So 1024 threads out of 1536 threads utilized.
- ## Memory
	- ![image.png](attachments/image_1705462227500_0.png)
		Overview of GPU memory layout (shared memory probably CUDA specific only)
		- \> There exists an "invisible" cache between the Global Memory and Block. Inaccessible to programmer.
	- Each thread takes a LONG time (maybe 50-100 cycles) to read memory
	- ![image.png](attachments/image_1705462375816_0.png)
		Declaring CUDA variables
	- ### Memory access technique
		- Works on the principle that:
		- ~~Threads that are going to do similar work are grouped together called "tiles".~~
			logseq.order-list-type:: number
		- GPU memory accessing is very slow, but bandwidth is pretty high. So a single, large memory transfer is better than many, small memory transfers
			logseq.order-list-type:: number
		- Data needs to be transferred from Global Memory to On-Chip memory, for each set of threads to do processing. On-Chip (Shared) memory can't take that much data at once. Each set of memory to load is called a "tile".
			logseq.order-list-type:: number
			- ![image.png](attachments/image_1705463747338_0.png)
		- ![image.png](attachments/image_1705463548810_0.png)
			- Load all data for all these threads, then wait for them to finish. Depends on the slowest thread. Do not load in data for next "tile" yet, or shit happens. This is called "Barrier Synchronization"
			- Once all threads are finished, then load in the next tile of data to be processed by the same set of threads.
- ## Warps
	- DISCLAIMER: DO NOT assume threads in warps OR warps themselves are ordered. THEY ARE UNORDERED.
		- Do not "do xxx only on specific warps" to try to avoid control divergence, because there is no way to accurately pinpoint warps.
		- For this, use __syncthreads() to let them all sync.
	- All threads in warp SHOULD take the same conditional branches and same number of iterations etc, for performance.
	- It is possible for some threads in a warp to take a different path
		- However, how the GPU does this is that it executes each path SYNCHRONOUSLY. So if we have a simple if-else branch and the threads in a warp goes down both paths, the GPU will have to do ONE path first, THEN the other path. So x2 the time.
		- With more branches, the time taken will increase, as long as ONE thread in the warp needs to go down a unique path.
	- Example: 
		- ![image.png|579](attachments/image_1705635552018_0.png)
		- First one: Checking if thread index is xxx. Since in each warp (unknown number of threads in a warp), some threads will be \<2 and some are \>2, there will be at least 1 warp that will take 2 paths. Hence there is a control divergence, not good.
		- Second one: Checking if block index is xxx instead. Since each block nicely fits a certain number of warps (i think??), there are no threads in the warp that have a different evaluation of blockid.x \> 2. Hence all warps only have 1 path of execution. Grape!
	- ### About warp and block sizes
		- According to this [SO Thread](https://stackoverflow.com/questions/31279505/conversion-from-block-dimensions-to-warps-in-cuda):
		- \> A blocksize being not a multiple of 32 will rounds up to the nearest multiple, even if you request fewer threads. See GPU Optimization Fundamentals presentation of Cliff Woolley from the NVIDIA Developer Technology Group has interesting hints about performance.
		- So, block size isn't fully controllable by the programmer. It defaults to a multiple of warp size.
	- ![image.png|508](attachments/image_1705636135948_0.png) 
		- ![image.png|504](attachments/image_1705636118339_0.png)
		1 out of 32 warps got control divergence. This is due to the boundary check, which is necessary.
		- ![image.png](attachments/image_1705636514141_0.png)
	- Perf impact of \>3% is calculated by 1/32
- ## Memory access: DRAM Bursting
	- TLDR: How much shit gets loaded at once from RAM
	- DRAM refers to DDR2/3/4 etc, or GDDR 3/4/5 etc
	- Accessing DRAM is slow af
	- The higher the DDR gen, the slower it gets (???)
		- ![image.png](attachments/image_1706240020979_0.png)
	- But each generation loads more stuff
		- ![image.png|695](attachments/image_1706240062451_0.png)
	- Optimized loading of data with fewer calls: 
		- ![image.png|560](attachments/image_1706240143354_0.png)
		- Conversely, not grape loading: 
			- ![image.png|543](attachments/image_1706240173662_0.png)
		- 
- ## Atomic Operations
	- TLDR: Acts like a semaphore, makes sure there's no data race.
	- Additional tip from memory access: If u have 4 "threads", and u want to process a block of data, don't split the block of data by 4. Instead, stagger the split like so: 
		- ![image.png|569](attachments/image_1706239647408_0.png)
		This ensures that every time all 4 threads need to do processing, only 1/4 of the chunk is loaded instead of the first bit of every chunk. (Which means all 4 chunks have to be loaded for processing to be done instead of just 1 chunk)
		- Example of strided access pattern: 
			- ![image.png|545](attachments/image_1706239738455_0.png)
		- assets:///C%3A/PERSONAL_FILES/LogseqNotes/assets/image_1706239647408_0.png
	- Atomic operations are SLOW, because they are "blocking". Nothing else can happen until the atomic operation finishes. This is made worse if there are multiple atomic ops in succession. Additionally, EACH memory access (read/write) takes about 1000 cycles to access the global DRAM! 
		- ![image.png|662](attachments/image_1706240348823_0.png)
	- Solution? Cache. If the atomic operation accesses something already in the cache, the latency drops.
		- Global DRAM: Slow. 1000-ish cycles.
		- L2 Cache: About 1/10 of the global DRAM latency (for Fermi).
			- All blocks can access. Free performance boost globally.
		- Shared Memory: Very fast.
			- But only on a per-block basis. Other blocks can't use this memory. Requires some work to implement in code.
	- ## Privatization
		- Idea: When doing atomic operations over many threads e.g. 100 threads, 10 blocks, that's 100 sequential operations that can't be parallelized. (Each atomic op, need to wait for ALL 100 threads to finish) Not grape, cos of memory access and wasted time waiting. So, we try to parallelize this bitch.
			- Firstly, instead of doing an atomic operation on the global memory level (the L2 cache level), you do it on the Shared memory level instead.
			- You create a shared buffer for each block, and then read the global data from global DRAM but store the results in Shared memory buffer. This way, all 10 blocks can run their atomic operations in parallel.
			- After all blocks are finished (using `__syncthreads()`, you then do the atomic addition of all these buffers in each block with another set of atomic operations.
			- Instead of all threads in every block trying to do atomic ops individually on the same data... 
				- ![image.png|350](attachments/image_1706241553845_0.png)
				We have each block do their own set of atomic ops on their own, then atomic add these buffers together on the 2nd pass 
				- ![image.png|353](attachments/image_1706241608388_0.png)
			- 
		- Pros and Cons:
			- PROS:
				- Yay parallel!
			- CONS:
				- Have to set up stuff to do this.. Not worth if amount of atomic ops is tiny.
				- Only works if the order of operation of every atomic op does not matter. If order matters, then this shit fails.
				- Block-based shared memory buffer should remain small enough to fit in the shared memory. So data that is too big is unusable.
- ## Parallel Programming Optimizations
	- ### Convolution
		- Basics of image/audio processing
		- takes a range/area of data in an array, and does processing on the data + the data around it
			- e.g. Gaussian blur or whatever
		- ![image.png|502](attachments/image_1706671942740_0.png)
			N: The data to do processing on
			M: The standard MASK to be applied to every computation
			P: Output after slapping all the Ns into M
		- ![image.png](attachments/image_1706672027691_0.png)
			If at the edge, there will be "ghost cells"
		- ![image.png|440](attachments/image_1706672140030_0.png)
			Small optimization: use a local register to store data instead of storing data straight back into the array.
			Instead of storing into out[xxx], don't do that cos accessing that guy takes quite a huge amount of cycles, esp in the double for loop
		- ## Tiled Convolution
			- [Lecture-Parallel Patterns.pptx](../assets/Lecture-Parallel_Patterns_1706676564137_0.pptx)
			- TLDR: Optimize the loading of elements into threads, minimize the amount of loading needed to be done.
			- Simplified example:
				- ![image.png|580](attachments/image_1709465360556_0.png)
				**Context**: For each of these elements, we take the neighboring 4 other elements to do processing, then plonk the value into that one element we are processing.
				**Note**: Each computation is denoted by T0, T1, etc. Could be done in parallel on diff threads, could be done in sequence. For now I assume is on different threads.
				**Observation**: When computing elements 0 to 4, the element 2 is loaded for all of them. Therefore, when we perform computations on elements 0 to 4, element 2 SHOULD be loaded (in shared memory) for all these threads. Then, element 2 don't need to be loaded anymore afterwards.
			- ### Tiled Convolution design
				- ![image.png|583](attachments/image_1709465766033_0.png)
				- Design 1:
					- ![image.png|427](attachments/image_1709474115211_0.png)
						Number of threads to work with == output (block) size. 
						When loading in the input cells into shared memory, each thread should load 1, but that won't be enough cos the side pieces has nobody to load them. So, during loading, some threads will have to load the outer cells too.
						
						After loading, there should be 1 thread for 1 output tile to do processing. Each thread dumps the output directly into their respective output cells. Done.
				- Design 2:
					- Similar to design 1.
						Number of threads to work with == input (block + a few more) size. 
						When loading in the input cells, is straight forward. 1 thread loads 1 cell.
						
						When processing, there should be 1 thread for 1 output tile to do processing. In fact, there's more than enough. The threads on the "outside" borders won't need to compute anything. Each thread writes to their respective output tiles (if it did processing) and that's it.
- # Other notes
	- https://stackoverflow.com/questions/2392250/understanding-cuda-grid-dimensions-block-dimensions-and-threads-organization-s blocks, threads and shit organization
	- what is warp https://softwareengineering.stackexchange.com/questions/369960/definition-and-usage-of-warp-in-parallel-gpu-programming
		- what is wavefront
	- If else in GPU, but gpu has no branch prediction. https://stackoverflow.com/questions/45734138/why-an-if-else-statement-in-gpus-code-will-cut-the-performance-in-half
	- https://developer.samsung.com/galaxy-gamedev/resources/articles/gpu-framebuffer.html gpu memory and tiling
