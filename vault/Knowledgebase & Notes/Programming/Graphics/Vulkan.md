# High Level overview
1. Create Vulkan Instance
	1. For debug, enable validation layers. Also create a callback messenger if validation layers are enabled.
2. Create Window Surface
	1. Use GLFW to help create a window surface. Else, you'll need to touch the native OS's APIs to create a window surface (e.g. WIN32)
3. Pick Physical Device (Physical GPU)
	1. Either choose the first usable one, or score all available GPUs.
		1. Gotta check if certain queue families with certain functions (e.g. GRAPHICS) exist.
4. Create Logical Device
	1. Create device queues using VkDeviceQueueCreateInfo
		1.  Queue Families: 
	2. Specify devices features to use, if any.
	3. Create logical device, 
	4. After creation, query vkDevice for the queue family index for graphics and present queues (and any other queues), and save the pointer to a VkQueue object. Note: each VkQueue could have the same pointer as one another if the queue supports multiple functionalities.

# Queues and Queue Families
https://stackoverflow.com/a/55273688

Given command and output:
```cpp
vector<vk::QueueFamilyProperties> queue_families = device.getQueueFamilyProperties();
for(auto &q_family : queue_families)
{
    cout << "Queue number: "  + to_string(q_family.queueCount) << endl;
    cout << "Queue flags: " + to_string(q_family.queueFlags) << endl;
}

Queue number: 16
Queue flags: {Graphics | Compute | Transfer | SparseBinding}
Queue number: 1
Queue flags: {Transfer}
Queue number: 8
Queue flags: {Compute}
```

What this means:
- There are 3 queue families
- Each queue family contains 16 queues, 1 queue and 8 queues respectively per queue family

Extra details:
- Queues may directly correspond to actual hardware queues or maybe not. Making 2 VkQueues for graphics may or may not be executed concurrently or sequentially.
- Most GPUs only have 1 hardware graphics queue and multiple compute queues, although there may exist more VkQueues for them than there are hardware queues.