## Pure DPDK based switch, is there one ? 
ASFAIK there is no production-ready software switch that is built entirely using DPDK's libraries and frameworks.Developing such a switch would be a significant undertaking, as it would involve implementing many features (like VLANs, STP, LACP, etc.) that are typically found in software switches.

However DPDK is widely used for packet IO in other switching stacks like fd.io VPP, OVS-DPDK etc.

## DPDK's L2fwd sample app code walk through
Neverthless DPDK itslef provide a sample application to do the L2 forwarding.  

This application simply reads packets from an input port and forwards them to an output port.
The L2 Forwarding application is in the examples/l2fwd directory in the DPDK source tree. The main parts of the application are contained in the main.c file. Here's a high-level overview:

### Initialization

The main function (int main(int argc, char **argv)) starts by initializing the DPDK Environment Abstraction Layer (EAL) with rte_eal_init(). This is a necessary step for any DPDK application.

Command line arguments are parsed by the l2fwd_parse_args() function to configure the application. These arguments include the number of cores to use, the number of queues per port, and the forwarding delay.

### Port Configuration

The application then checks the number of available DPDK ports and configures them with l2fwd_configure_port(). Each port is set up with a receive (RX) and transmit (TX) queue, and the ports are started with rte_eth_dev_start().

### Launching the Data Processing Loop on Each Core

For each enabled core, the function rte_eal_remote_launch() is used to launch the function l2fwd_main_loop(). This function contains the main packet processing loop for the application.

### Main Loop

The l2fwd_main_loop() function is where the main packet processing happens. The function starts an infinite loop that:

  * Reads packets from the RX queue of the port with `rte_eth_rx_burst()`.
  * For each received packet, it swaps the source and destination MAC addresses.
  * It determines the output port (which is the other port for this application), and sends the packet out on the output port with `rte_eth_tx_burst()`.

### Cleanup

If the application is stopped, it cleans up by closing the ports with rte_eth_dev_stop() and rte_eth_dev_close().

This is a very high-level overview of the L2 Forwarding application. The actual code includes a lot more detail, including error checking and handling, statistics collection, and support for different packet processing modes (like single-core, multi-core, and multi-queue). I recommend checking out the main.c file in the examples/l2fwd directory of the DPDK source tree for more details.
