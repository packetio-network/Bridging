# The Linux kernel bridge module 
This module is composed of several files, each implementing a different aspect of the bridge's functionality. Here is a brief overview of each of them:

    br.c: This file contains code for managing bridge devices. It includes functions for creating and deleting bridge devices, adding and removing ports from a bridge, and configuring various bridge parameters.

    br_device.c: This file implements the network device interface for bridge devices. It includes functions for opening and closing a bridge device, transmitting packets, and performing various other network device operations.

    br_fdb.c: This file implements the Forwarding Database (FDB) which the bridge uses to decide on which port to forward incoming frames. It provides mechanisms for adding, removing, and searching for entries in the FDB.

    br_forward.c: This file is responsible for forwarding packets. It contains functions that decide whether to forward a packet or not, and on which port to forward it.

    br_if.c: This file contains code for managing the interfaces (ports) that are part of a bridge. It includes functions for adding and deleting ports, enabling and disabling them, and configuring port parameters.

    br_input.c: This file handles incoming packets on a bridge device. It decides whether to forward a packet, send it to the local host, or drop it.

    br_ioctl.c: This file implements the ioctl system call interface for bridge devices. It allows userspace programs to configure the bridge by sending ioctl commands.

    br_stp.c: This file implements the Spanning Tree Protocol (STP), a network protocol designed to prevent loops in a network topology. It includes functions for enabling and disabling STP, sending and receiving STP packets, and performing the STP algorithm.

    br_stp_bpdu.c: This file handles BPDUs (Bridge Protocol Data Units), which are the packets used by the Spanning Tree Protocol. It includes functions for sending and receiving BPDUs.

    br_stp_if.c: This file contains code for managing the STP state of an interface. It includes functions for enabling and disabling STP on an interface, updating the STP state, and sending STP packets.

    br_stp_timer.c: This file implements the timers used by the Spanning Tree Protocol. It includes functions for starting, stopping, and resetting the various STP timers.

    br_sysfs_if.c: This file implements the sysfs interface for bridge ports. It allows userspace programs to view and configure port parameters through the sysfs filesystem.

    br_sysfs_br.c: This file implements the sysfs interface for bridge devices. It allows userspace programs to view and configure bridge parameters through the sysfs filesystem.

    br_netlink.c: This file implements the Netlink interface for the bridge module. Netlink is a Linux kernel interface used for communication between the kernel and userspace.

Each of these files plays a crucial role in the overall functioning of the Linux bridge module. They work together to implement a software bridge that can forward packets between different network segments, prevent network loops with the Spanning Tree Protocol, and provide a comprehensive interface for userspace programs to configure and manage the bridge.

Packet transmission (Tx) and reception (Rx) in the Linux kernel's bridge module largely involve the br_dev_xmit and br_handle_frame functions respectively, located in the br_device.c and br_input.c files.

## Packet Transmission (Tx)

When a packet is to be transmitted from a bridge, the br_dev_xmit function in br_device.c is called. Here's a simplified version:

  netdev_tx_t br_dev_xmit(struct sk_buff *skb, struct net_device *dev)
  {
      struct net_bridge_port *p;

      skb_reset_mac_header(skb);
      p = br_port_get_rcu(dev);
      skb_push(skb, ETH_HLEN);
      br_forward(p, skb, false);
      return NETDEV_TX_OK;
  }

Here's what's happening in this function:

    skb_reset_mac_header(skb): This resets the MAC header in the packet to the correct location.
    p = br_port_get_rcu(dev): This gets the bridge port associated with the network device from which the packet is being sent.
    skb_push(skb, ETH_HLEN): This adds an Ethernet header to the packet.
    br_forward(p, skb, false): This forwards the packet to its destination. The br_forward function is defined in br_forward.c and is responsible for delivering the packet to its correct destination port.

## Packet Reception (Rx)

When a packet is received on a bridge, the br_handle_frame function in br_input.c is called. Here's a simplified version:

  int br_handle_frame(struct sk_buff **pskb)
  {
      struct net_bridge_port *p;
      const unsigned char *dest;
      struct sk_buff *skb = *pskb;

      p = br_port_get_rcu(skb->dev);
      dest = eth_hdr(skb)->h_dest;

      if (unlikely(is_link_local(dest))) {
          if (br_handle_local_finish(skb) == RX_HANDLER_CONSUMED)
              return 0;
          goto handle_frame_local;
      }

      if (p->state == BR_STATE_FORWARDING) {
          if (br_should_route_hook(pskb))
              return 0;
          if (dest[0] & 1)
              return br_handle_frame_finish(p, skb);
          else
              return br_handle_frame_finish(p, skb);
      }

  handle_frame_local:
      return br_pass_frame_up(skb);
  }

Here's what's happening in this function:

    p = br_port_get_rcu(skb->dev): This gets the bridge port associated with the network device that received the packet.
    dest = eth_hdr(skb)->h_dest: This gets the destination MAC address from the Ethernet header of the packet.
    The is_link_local(dest) check is used to determine whether the destination MAC address is a link-local address. If it is, the packet is passed to br_handle_local_finish function for further processing. If the packet is consumed by br_handle_local_finish, the function returns.
    If the port is in the forwarding state (i.e., p->state == BR_STATE_FORWARDING), the function checks whether the packet should be routed. If it should, the function returns. Otherwise, it passes the packet to br_handle_frame_finish for further processing.
    If the destination is not a link-local address, the packet is passed to the br_pass_frame_up function, which sends the packet up to the higher layers of the network stack.
    
The real code contains more checks and logic to handle various scenarios and conditions. It's also worth noting that the Linux kernel networking stack, including the bridge module, uses the RCU (Read-Copy-Update) mechanism extensively for synchronization. This mechanism allows multiple threads to access shared data structures simultaneously without locking, which can significantly improve performance in multi-core systems.

## Frame Forwarding

The br_forward function in br_forward.c is called when a frame is to be forwarded to a specific port:

  void br_forward(const struct net_bridge_port *to, struct sk_buff *skb, bool local_rcv, bool local_orig)
  {
      struct net_device *dev = to->dev;

      if (should_deliver(to, skb)) {
          struct sk_buff *skb2;

          if (local_rcv) {
              skb2 = skb_clone(skb, GFP_ATOMIC);
              if (!skb2) {
                  kfree_skb(skb);
                  return;
              }
          } else {
              skb2 = skb;
          }

          skb->dev = dev;
          dev_queue_xmit(skb2);
      }
  }

Here's what's happening in this function:
    should_deliver(to, skb): This checks whether the frame should be delivered to the specified port. This can depend on several factors, such as the STP (Spanning Tree Protocol) state of the port, and whether the destination MAC address is in the port's FDB (Forwarding Database).
    If the frame should be delivered to the port, skb_clone is used to create a copy of the frame if local_rcv is true. This is because the original frame will be passed up to the local network stack.
    dev_queue_xmit(skb2): This queues the frame for transmission on the specified network device.

## Frame Handling

The br_handle_frame_finish function in br_input.c is called when a frame is to be handled by the bridge:

  int br_handle_frame_finish(struct net_bridge_port *p, struct sk_buff *skb)
  {
      struct net_bridge *br;
      unsigned char *dest;

      br = p->br;
      dest = eth_hdr(skb)->h_dest;

      if (unlikely(is_link_local(dest))) {
          return br_handle_local_finish(skb);
      }

      if (br->dev->flags & IFF_PROMISC) {
          struct sk_buff *skb2;

          skb2 = skb_clone(skb, GFP_ATOMIC);
          if (skb2) {
              br_pass_frame_up(skb2);
          }
      }

      if (dest[0] & 1) {
          return br_flood_deliver(br, skb);
      } else {
          return br_forward_deliver(br, skb);
      }
  }

Here's what's happening in this function:

    It starts by extracting the bridge device and the destination MAC address from the frame.
    If the destination MAC address is a link-local address, the frame is passed to br_handle_local_finish for further processing.
    If the bridge device is in promiscuous mode, a copy of the frame is created and passed up to the local network stack.
    If the destination MAC address is a multicast or broadcast address (i.e., dest[0] & 1), the frame is flooded to all ports by calling br_flood_deliver. Otherwise, the frame is delivered to a specific port by calling br_forward_deliver.    
    
## Frame Delivery

The br_forward_deliver function in br_forward.c is called when a frame is to be delivered to a specific port:

  void br_forward_deliver(const struct net_bridge *br, struct sk_buff *skb)
  {
      const unsigned char *dest = eth_hdr(skb)->h_dest;

      if (!compare_ether_addr(dest, br->dev->dev_addr))
          br_pass_frame_up(skb);
      else if (!compare_ether_addr(dest, br->bridge_addr))
          br_pass_frame_up(skb);
      else
          kfree_skb(skb);
  }

Here's what's happening in this function:

    It starts by extracting the destination MAC address from the frame.
    If the destination MAC address matches the MAC address of the bridge device or the bridge's group address, the frame is passed up to the local network stack by calling br_pass_frame_up.
    Otherwise, the frame is discarded by calling kfree_skb.

## Frame Flooding

The br_flood_deliver function in br_forward.c is called when a frame is to be flooded to all ports:

  void br_flood_deliver(struct net_bridge *br, struct sk_buff *skb, bool unicast)
  {
      struct net_bridge_port *p;
      struct hlist_node *h;
      unsigned int i;

      hlist_for_each_entry_rcu(p, h, &br->port_list, list) {
          if (should_deliver(p, skb)) {
              struct sk_buff *skb2;

              skb2 = skb_clone(skb, GFP_ATOMIC);
              if (skb2) {
                  br_forward(p, skb2, false);
              }
          }
      }

      kfree_skb(skb);
  }

Here's what's happening in this function:

    It starts by iterating over all ports in the bridge.
    For each port, it checks whether the frame should be delivered to that port by calling should_deliver.
    If the frame should be delivered to the port, a copy of the frame is created and forwarded to the port by calling br_forward.
    Finally, the original frame is discarded by calling kfree_skb.

These are the key functions involved in packet transmission and reception in the Linux kernel's bridge module. The actual code contains more checks and details, and there are many other functions involved in the process. The code also interacts with other parts of the Linux kernel networking stack, such as the neighbour (ARP) subsystem, the routing subsystem, and the network device driver layer.

To get a full understanding of the packet flow, you'll need to delve into the code and follow the path of a packet through the various functions and subsystems. Reading the Linux kernel documentation and source code is the best way to learn about these details.   
