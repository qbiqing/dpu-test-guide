# Set up

Follow the Helium user manual to set up connection between the host and the DPU.

Run through sections 6 to 8.

Differences to note:

1. Under section 6.2 Host-side Driver Dependencies, step 4, the warning: `PCIe ACS override enabled` is not seen. Can ignore.

2. Under section 7.1 Enable Virtual Port on the Host, step 1, note the difference in root PCI bus address. You should observe that our address is `[0000:30]-+-02.0-[31]--+-00.0`.

In step 2, do not include `2>/dev/null` to view the output. Before `rmmod mgmt_net`, also run `rmmod mvmgmt0`.

Change the parameters of `modprobe pcie_ep` to the following: `modprobe pcie_ep host_sid=0x33000 pem_num=0 epf_num=0` to match the bus address.

Follow through the rest of the steps. You should be able to see some RX and TX packets when running `ifconfig mvmgmt0`.

