## Pure DPDK switch
Choosing to build or use a switch entirely based on DPDK libraries and framework depends on several factors. Here are a few scenarios where it might be beneficial:

#### Performance: 
  DPDK can provide high performance due to its user-space implementation and bypassing of the kernel network stack. If you have a use case that requires high-speed packet processing (for example, in a high-frequency trading system or a telecom network function), a DPDK-based switch might be a good choice.

#### Customization: 
  If you need specific features or behaviors that aren't provided by existing switch software, creating your own switch with DPDK allows you to implement those custom features.

#### Learning and Research: 
  If you're studying network programming or doing research on network performance, building a switch with DPDK can be a great learning experience.

#### Control over the Code Base: 
  If you want complete control over the switch code base for reasons like security or stability, creating your own switch with DPDK can provide that.

However, building a switch entirely on DPDK is a non-trivial task, and there are some important considerations to keep in mind:
#### Development and Maintenance Costs: 
  Building and maintaining a full-featured switch software is a significant undertaking. It requires a deep understanding of networking and the DPDK library.
#### Lack of Higher-Level Features: 
  DPDK provides low-level packet processing capabilities. Implementing higher-level switch features like Spanning Tree Protocol (STP), VLAN, Multicast, ACLs etc., would need to be done manually.
#### Community Support: 
  Established software switches like OVS or FD.io VPP have large communities and extensive documentation to support them. A custom DPDK-based switch wouldn't have the same level of community support.
#### Integration with Existing Infrastructure: 
  Established switch software often has better integration with network management and orchestration tools. A custom DPDK-based switch might require additional work to integrate with these tools.

So, while there are scenarios where a DPDK-based switch can be beneficial, it's important to weigh these benefits against the potential costs and challenges.
