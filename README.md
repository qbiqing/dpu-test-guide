# Set up

Follow the Helium user manual to set up connection between the host and the DPU.

Go through sections 6 to 8.

Differences to note:

1. Under section 6.2 Host-side Driver Dependencies, step 4, the warning: `PCIe ACS override enabled` is not seen. Can ignore.

2. Under section 7.1 Enable Virtual Port on the Host, step 1, note the difference in root PCI bus address. You should observe that our address is `[0000:30]-+-02.0-[31]--+-00.0`.

In step 2, do not include `2>/dev/null` to view the output. Before `rmmod mgmt_net`, also run `rmmod mvmgmt0`.

Change the parameters of `modprobe pcie_ep` to the following: `modprobe pcie_ep host_sid=0x33000 pem_num=0 epf_num=0`. Note that the `host_sid` parameter is changed to `0x33000` to match the root PCI bus address `0000:30`.

Follow through the rest of the steps. You should be able to see some RX and TX packets when running `ifconfig mvmgmt0`.

# Running DPDK

## Using DPDK v19.11

Follow instructions on the user manual section 8. Use the file `Helium-DPDK19.11-Vx.yRz.tar.gz` in the shared folder.

You might get some Werrors when running `make` to compile DPDK on the host. If you do, you can go into the relevant directory and remove the CFLAG `-Werror` from the Makefile. If you get a wrong pointer type warning, you can use typecasting to cast the pointer into the correct type.

There is one error where the patch can be referenced [here](https://git.dpdk.org/dpdk/commit/?id=87efaea6376c8). You can apply the patch manually.

On the host, run `lspci | grep b203`. Observe the first two devices have the addresses `0000:31:02.0` and `0000:31:02.1`. Bind these two virtual ports to DPDK.

Correction for command to run `./build/app/testpmd`: change the flag `-I` to `-i`.


## Using newer versions of DPDK

NOTE: the host does not successfully receive packets in this mode.

On the host, run `lspci | grep b203`. Observe the first two devices have the addresses `0000:31:02.0` and `0000:31:02.1`. Bind these two virtual ports to DPDK.

```bash
sudo ./usertools/dpdk-devbind.py -b vfio-pci 0000:31:02.0 0000:31:02.1
```

Follow user manual instructions to enable hugepages.

Run testpmd with the following command adapted for the DPDK version we are using and port addresses.

```bash
sudo ./build/app/dpdk-testpmd -l 0,2,4 -a 0000:31:02.0 -a 0000:31:02.1 -- -i  --rxd=4096
```

You will see these two errors.

```bash
Error during enabling promiscuous mode for port 0: Operation not supported - ignore
Error during enabling promiscuous mode for port 1: Operation not supported - ignore
```

On the DPU, run `lspci | grep a0f7`. Observe that the PCI addresses are the same as written on the user manual. Follow the user manual instructions to bind the ports to DPDK.

If you encounter `Unknown device` error in the binding, add the following lines to `dpdk-devbind.py` (around lines 40-50):

```python
cavium_sdp = {'Class': '0b', 'Vendor': '177d', 'Device': 'a303',
              'SVendor': None, 'SDevice': None}
octeontx2_sdp = {'Class': '08', 'Vendor': '177d', 'Device': 'a0f6,a0f7','SVendor': None, 'SDevice': None}
```

and modify line 67 to
```python
network_devices = [network_class, cavium_pkx, avp_vnic, ifpga_class, cavium_sdp, octeontx2_sdp]
```

Furthermore, modify `otx2_mbox.h` line 93 to
```c
#define OTX2_MBOX_VERSION (0x0009)
```

Follow user manual instructions to enable hugepages. Run testpmd with the following command:

```bash
./build/app/dpdk-testpmd -l0,1-2 -w 0002:02:00.0 -w 0002:0f:00.2 -w 0002:0f:00.3 -- -i --portmask=0x6
```
