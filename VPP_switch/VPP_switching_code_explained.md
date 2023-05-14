# The traffic switching code in VPP

## Input Node
  The journey starts with the input node, which is specific to the network interface driver in use. For example, in the case of DPDK devices, this would be the dpdk_input function located in dpdk/device/dpdk.c. This function is responsible for reading packets from the network interface and passing them to the next processing node.

## Ethernet Input Node
  Next, the packet arrives at the Ethernet input node. Here, the ethernet_input function in vnet/ethernet/input.c processes the Ethernet frame. If the Ethernet frame is identified as VLAN-tagged (0x8100 for single-tagged VLAN, 0x88A8 or 0x9100 for Q-in-Q or double-tagged VLAN), it will dispatch the frame to the l2-input node, marking it as VLAN-tagged.

## L2 Input Node
  The l2_input function in vnet/l2/l2_input.c is responsible for several tasks. If the packet is marked as VLAN-tagged, it extracts the VLAN ID from the VLAN tag and removes the tag from the frame. Then, it performs a MAC address lookup using the L2 FIB. However, the lookup is done in the context of the specific VLAN. This means that it uses the L2 FIB for the specific VLAN ID associated with the frame, allowing for separate MAC address tables for each VLAN. If the MAC address lookup is successful, the next node is set to l2-output.

## L2 Output Node
  Next, the l2_output function in vnet/l2/l2_output.c processes the packets that are destined for local interfaces. If the outgoing interface is a VLAN interface, the frame is retagged with the appropriate VLAN tag. The function also sets the correct source MAC address (the MAC address of the outgoing interface) in the Ethernet header of the packet.

## Interface Output Node
  Finally, in the interface output node, the packet is transmitted on the outgoing interface. This is also device-specific. For DPDK interfaces, the dpdk_interface_tx function in dpdk/device/dpdk.c handles this task, sending the packets to the DPDK library for transmission on the physical interface.
