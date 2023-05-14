# Why should a customer choose Linux Bridging based solution ?
Choosing between Linux kernel bridging and other technologies like FD.io VPP (Vector Packet Processing) or OVS-DPDK (Open vSwitch with Data Plane Development Kit) depends on several factors, including the specific requirements of the network, the complexity of the workloads, the need for performance, and the resources available for managing and operating the network. Here are a few considerations:

### Simplicity and Compatibility: 
Linux kernel bridging is built into the Linux kernel and is supported on all Linux distributions. This makes it a simple and straightforward solution for creating virtual networks on a Linux-based system. If your needs are simple and you're mainly concerned with compatibility and ease of use, Linux kernel bridging can be a good choice.

### Performance: 
If high performance is a critical requirement, you might want to consider solutions like FD.io VPP or OVS-DPDK. These technologies are designed to handle high-speed, high-volume packet processing more efficiently than traditional Linux kernel networking. VPP is designed to process packets quickly using vector processing, while OVS-DPDK uses DPDK to bypass the kernel and directly access the network hardware for faster packet processing. However, these solutions can be more complex to set up and manage, and they require more resources (e.g., dedicated CPU cores).

### Advanced Features: 
If you need advanced networking features like complex routing, fine-grained flow control, programmability, or integration with SDN (Software-Defined Networking) controllers, you might want to consider a more feature-rich solution like OVS. OVS supports OpenFlow and can be used with SDN controllers like OpenDaylight or ONOS, providing more flexibility and control over network traffic.

### Resources and Expertise: 
Implementing and managing solutions like FD.io VPP or OVS-DPDK can require more resources and expertise than using Linux kernel bridging. If your team has limited resources or is not familiar with these technologies, the simplicity and familiarity of Linux kernel bridging can be an advantage.
