# TODOs 
* read kernel log at boot and find warnings
* flash partitioning, kernel non-aligned? wasting space?
* Pinctl configuration
* write some blog posts
* bring up LEDs
* bring up GPIO/reset button
* bring up Ethernet
* bring up wifi
* bring up LTE
* SNMP config

# Hardware Recon
I found photos from the FCC report https://fccid.io/2AG32EG7035L96
I took the thing apart to find the UART header, follow traces, photograph the back of the board w/ flash chip
Flsah is MX25L25635F, found data sheet https://media.digikey.com/pdf/Data%20Sheets/Macronix/MX25L25635F.pdf

# Getting firmware image
Baicells site is an option, but for OD04 CPE they emailed me a link to a newer one than is on the site when I complained SNMP wasn't working properly.

6d04d132aad2f80cffa2982696e97a63  BaiCE_BM_2.5.26.2_NA.bin

# Extracting firmware image
Use `binwalk -Me <image>`. It may require some additonal utilities to be installed to extract the file system.

# Getting the device tree
Device tree blob was embedded in the firmware image, extracted into a file binwalk named 40.

```
$ binwalk 40

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
2072          0x818           Flattened device tree, size: 8989 bytes, version: 17
9926          0x26C6          mcrypt 2.2 encrypted data, algorithm: blowfish-448, mode: CBC, keymode: 8bit
864934        0xD32A6         PGP RSA encrypted session key - keyid: 524D1 6050C80 RSA (Encrypt or Sign) 1024b
2966548       0x2D4414        Linux kernel version 3.18.2
3100632       0x2F4FD8        xz compressed data
3126020       0x2FB304        Unix path: /lib/firmware/updates/3.18.29
3140228       0x2FEA84        Unix path: /sys/firmware/devicetree/base
3186022       0x309D66        Neighborly text, "neighbor %.2x%.2x.%pM lost rename link %s to %s"
3336192       0x32E800        CRC32 polynomial table, little endian
3589088       0x36C3E0        ASCII cpio archive (SVR4 with no CRC), file name: "dev", file name length: "0x00000004", file size: "0x00000000"
3589204       0x36C454        ASCII cpio archive (SVR4 with no CRC), file name: "dev/console", file name length: "0x0000000C", file size: "0x00000000"
3589328       0x36C4D0        ASCII cpio archive (SVR4 with no CRC), file name: "root", file name length: "0x00000005", file size: "0x00000000"
3589444       0x36C544        ASCII cpio archive (SVR4 with no CRC), file name: "TRAILER!!!", file name length: "0x0000000B", file size: "0x00000000"
```

Found some instructions about extracting dts from firmware image https://unix.stackexchange.com/questions/265890/is-it-possible-to-get-the-information-for-a-device-tree-using-sys-of-a-running

```
$ dd if=40 of=tree.dtb bs=1 count=8989 skip=2072
8989+0 records in
8989+0 records out
8989 bytes (9.0 kB, 8.8 KiB) copied, 0.0533368 s, 169 kB/s
```
```
dtc -f -I dtb -O dts tree.dtb                 
<stdout>: Warning (unit_address_vs_reg): /cpus/cpu@0: node has a unit name, but no reg or ranges property
<stdout>: Warning (unit_address_vs_reg): /cpuintc@0: node has a unit name, but no reg or ranges property
<stdout>: Warning (unit_address_vs_reg): /ethernet@10100000/port@0: node has a unit name, but no reg or ranges property
<stdout>: Warning (unit_address_vs_reg): /pcie@10140000/pcie-bridge: node has a reg or ranges property, but no unit name
<stdout>: Warning (pci_bridge): /pcie@10140000/pcie-bridge: node name is not "pci" or "pcie"
<stdout>: Warning (pci_bridge): /pcie@10140000/pcie-bridge: missing ranges for PCI bridge (or not a bridge)
<stdout>: Warning (unit_address_format): Failed prerequisite 'pci_bridge'
<stdout>: Warning (pci_device_reg): Failed prerequisite 'pci_bridge'
<stdout>: Warning (pci_device_bus_num): Failed prerequisite 'pci_bridge'
<stdout>: Warning (i2c_bus_bridge): /pinctrl/i2c: incorrect #address-cells for I2C bus
<stdout>: Warning (i2c_bus_bridge): /pinctrl/i2c: incorrect #size-cells for I2C bus
<stdout>: Warning (i2c_bus_reg): Failed prerequisite 'i2c_bus_bridge'
<stdout>: Warning (spi_bus_bridge): /palmbus@10000000/spi@b00: incorrect #size-cells for SPI bus
<stdout>: Warning (spi_bus_bridge): /pinctrl/spi: incorrect #address-cells for SPI bus
<stdout>: Warning (spi_bus_bridge): /pinctrl/spi: incorrect #size-cells for SPI bus
<stdout>: Warning (spi_bus_reg): Failed prerequisite 'spi_bus_bridge'
<stdout>: Warning (avoid_unnecessary_addr_size): /gpio-keys-polled: unnecessary #address-cells/#size-cells without "ranges" or child "reg" property
<stdout>: Warning (gpios_property): /palmbus@10000000/gpio@600:ralink,num-gpios: Could not get phandle node for (cell 0)
<stdout>: Warning (gpios_property): /palmbus@10000000/gpio@638: Missing property '#gpio-cells' in node /pinctrl/pcie or bad phandle (referred from ralink,num-gpios[0])
<stdout>: Warning (gpios_property): /palmbus@10000000/gpio@660:ralink,num-gpios: Could not get phandle node for (cell 0)
<stdout>: Warning (gpios_property): /palmbus@10000000/gpio@688: Missing property '#gpio-cells' in node /palmbus@10000000/intc@200 or bad phandle (referred from ralink,num-gpios[0])
<stdout>: Warning (interrupt_provider): /palmbus@10000000/intc@200: Missing #address-cells in interrupt provider
/dts-v1/;

/ {
	#address-cells = <0x01>;
	#size-cells = <0x01>;
	compatible = "ralink,mt7620a-eval-board\0ralink,mt7620a-soc";
	model = "BaiCPE-EG7035E";

	cpus {

		cpu@0 {
			compatible = "mips,mips24KEc";
		};
	};

	chosen {
		bootargs = "console=ttyS0,57600";
	};

	cpuintc@0 {
		#address-cells = <0x00>;
		#interrupt-cells = <0x01>;
		interrupt-controller;
		compatible = "mti,cpu-interrupt-controller";
		linux,phandle = <0x03>;
		phandle = <0x03>;
	};

	palmbus@10000000 {
		compatible = "palmbus";
		reg = <0x10000000 0x200000>;
		ranges = <0x00 0x10000000 0x1fffff>;
		#address-cells = <0x01>;
		#size-cells = <0x01>;

		sysc@0 {
			compatible = "ralink,mt7620a-sysc\0ralink,rt3050-sysc";
			reg = <0x00 0x100>;
		};

		timer@100 {
			compatible = "ralink,mt7620a-timer\0ralink,rt2880-timer";
			reg = <0x100 0x20>;
			interrupt-parent = <0x01>;
			interrupts = <0x01>;
		};

		watchdog@120 {
			compatible = "ralink,mt7620a-wdt\0ralink,rt2880-wdt";
			reg = <0x120 0x10>;
			resets = <0x02 0x08>;
			reset-names = "wdt";
			interrupt-parent = <0x01>;
			interrupts = <0x01>;
		};

		intc@200 {
			compatible = "ralink,mt7620a-intc\0ralink,rt2880-intc";
			reg = <0x200 0x100>;
			resets = <0x02 0x13>;
			reset-names = "intc";
			interrupt-controller;
			#interrupt-cells = <0x01>;
			interrupt-parent = <0x03>;
			interrupts = <0x02>;
			linux,phandle = <0x01>;
			phandle = <0x01>;
		};

		memc@300 {
			compatible = "ralink,mt7620a-memc\0ralink,rt3050-memc";
			reg = <0x300 0x100>;
			resets = <0x02 0x14>;
			reset-names = "mc";
			interrupt-parent = <0x01>;
			interrupts = <0x03>;
		};

		uart@500 {
			compatible = "ralink,mt7620a-uart\0ralink,rt2880-uart\0ns16550a";
			reg = <0x500 0x100>;
			resets = <0x02 0x0c>;
			reset-names = "uart";
			interrupt-parent = <0x01>;
			interrupts = <0x05>;
			reg-shift = <0x02>;
			status = "disabled";
		};

		gpio@600 {
			compatible = "ralink,mt7620a-gpio\0ralink,rt2880-gpio";
			reg = <0x600 0x34>;
			resets = <0x02 0x0d>;
			reset-names = "pio";
			interrupt-parent = <0x01>;
			interrupts = <0x06>;
			gpio-controller;
			#gpio-cells = <0x02>;
			ralink,gpio-base = <0x00>;
			ralink,num-gpios = <0x18>;
			ralink,register-map = [00 04 08 0c 20 24 28 2c 30 34];
		};

		gpio@638 {
			compatible = "ralink,mt7620a-gpio\0ralink,rt2880-gpio";
			reg = <0x638 0x24>;
			interrupt-parent = <0x01>;
			interrupts = <0x06>;
			gpio-controller;
			#gpio-cells = <0x02>;
			ralink,gpio-base = <0x18>;
			ralink,num-gpios = <0x10>;
			ralink,register-map = [00 04 08 0c 10 14 18 1c 20 24];
			status = "disabled";
		};

		gpio@660 {
			compatible = "ralink,mt7620a-gpio\0ralink,rt2880-gpio";
			reg = <0x660 0x24>;
			interrupt-parent = <0x01>;
			interrupts = <0x06>;
			gpio-controller;
			#gpio-cells = <0x02>;
			ralink,gpio-base = <0x28>;
			ralink,num-gpios = <0x20>;
			ralink,register-map = [00 04 08 0c 10 14 18 1c 20 24];
			linux,phandle = <0x11>;
			phandle = <0x11>;
		};

		gpio@688 {
			compatible = "ralink,mt7620a-gpio\0ralink,rt2880-gpio";
			reg = <0x688 0x24>;
			interrupt-parent = <0x01>;
			interrupts = <0x06>;
			gpio-controller;
			#gpio-cells = <0x02>;
			ralink,gpio-base = <0x48>;
			ralink,num-gpios = <0x01>;
			ralink,register-map = [00 04 08 0c 10 14 18 1c 20 24];
			status = "disabled";
		};

		i2c@900 {
			compatible = "link,mt7620a-i2c\0ralink,rt2880-i2c";
			reg = <0x900 0x100>;
			resets = <0x02 0x10>;
			reset-names = "i2c";
			#address-cells = <0x01>;
			#size-cells = <0x00>;
			status = "disabled";
			pinctrl-names = "default";
			pinctrl-0 = <0x04>;
		};

		i2s@a00 {
			compatible = "ralink,mt7620a-i2s";
			reg = <0xa00 0x100>;
			resets = <0x02 0x11>;
			reset-names = "i2s";
			interrupt-parent = <0x01>;
			interrupts = <0x0a>;
			dmas = <0x05 0x04 0x05 0x05>;
			dma-names = "tx\0rx";
			status = "disabled";
		};

		spi@b00 {
			compatible = "ralink,mt7620a-spi\0ralink,rt2880-spi";
			reg = <0xb00 0x100>;
			resets = <0x02 0x12>;
			reset-names = "spi";
			#address-cells = <0x01>;
			#size-cells = <0x01>;
			status = "okay";
			pinctrl-names = "default";
			pinctrl-0 = <0x06>;

			m25p80@0 {
				#address-cells = <0x01>;
				#size-cells = <0x01>;
				compatible = "mx25l25635e";
				reg = <0x00 0x00>;
				linux,modalias = "m25p80\0mx25l25635e";
				spi-max-frequency = <0x1e8480>;

				partition@0 {
					label = "u-boot";
					reg = <0x00 0x20000>;
				};

				partition@20000 {
					label = "u-boot-env";
					reg = <0x20000 0x20000>;
					read-only;
				};

				partition@40000 {
					label = "factory";
					reg = <0x40000 0x10000>;
					read-only;
					linux,phandle = <0x0c>;
					phandle = <0x0c>;
				};

				partition@50000 {
					label = "firmware";
					reg = <0x50000 0xeb0000>;
				};

				partition@f00000 {
					label = "log";
					reg = <0xf00000 0x100000>;
				};

				partition@1050000 {
					label = "image2";
					reg = <0x1050000 0xfa0000>;
				};

				partition@1ff0000 {
					label = "user-data";
					reg = <0x1ff0000 0x10000>;
				};

				partition@1000000 {
					label = "customization";
					reg = <0x1000000 0x50000>;
				};
			};
		};

		uartlite@c00 {
			compatible = "ralink,mt7620a-uart\0ralink,rt2880-uart\0ns16550a";
			reg = <0xc00 0x100>;
			resets = <0x02 0x13>;
			reset-names = "uartl";
			interrupt-parent = <0x01>;
			interrupts = <0x0c>;
			reg-shift = <0x02>;
			pinctrl-names = "default";
			pinctrl-0 = <0x07>;
		};

		systick@d00 {
			compatible = "ralink,mt7620a-systick\0ralink,cevt-systick";
			reg = <0xd00 0x10>;
			resets = <0x02 0x1c>;
			reset-names = "intc";
			interrupt-parent = <0x03>;
			interrupts = <0x07>;
		};

		pcm@2000 {
			compatible = "ralink,mt7620a-pcm";
			reg = <0x2000 0x800>;
			resets = <0x02 0x0b>;
			reset-names = "pcm";
			interrupt-parent = <0x01>;
			interrupts = <0x04>;
			status = "disabled";
		};

		gdma@2800 {
			compatible = "ralink,mt7620a-gdma\0ralink,rt2880-gdma";
			reg = <0x2800 0x800>;
			resets = <0x02 0x0e>;
			reset-names = "dma";
			interrupt-parent = <0x01>;
			interrupts = <0x07>;
			#dma-cells = <0x01>;
			#dma-channels = <0x10>;
			#dma-requests = <0x10>;
			status = "disabled";
			linux,phandle = <0x05>;
			phandle = <0x05>;
		};
	};

	pinctrl {
		compatible = "ralink,rt2880-pinmux";
		pinctrl-names = "default";
		pinctrl-0 = <0x08>;

		pinctrl0 {
			linux,phandle = <0x08>;
			phandle = <0x08>;

			gpio {
				ralink,group = "i2c\0uartf";
				ralink,function = "gpio";
			};
		};

		pcm_i2s {

			pcm_i2s {
				ralink,group = "uartf";
				ralink,function = "pcm i2s";
			};
		};

		uartf_gpio {

			uartf_gpio {
				ralink,group = "uartf";
				ralink,function = "gpio uartf";
			};
		};

		spi {
			linux,phandle = <0x06>;
			phandle = <0x06>;

			spi {
				ralink,group = "spi";
				ralink,function = "spi";
			};
		};

		i2c {
			linux,phandle = <0x04>;
			phandle = <0x04>;

			i2c {
				ralink,group = "i2c";
				ralink,function = "i2c";
			};
		};

		uartlite {
			linux,phandle = <0x07>;
			phandle = <0x07>;

			uart {
				ralink,group = "uartlite";
				ralink,function = "uartlite";
			};
		};

		mdio {
			linux,phandle = <0x0b>;
			phandle = <0x0b>;

			mdio {
				ralink,group = "mdio";
				ralink,function = "mdio";
			};
		};

		ephy {

			ephy {
				ralink,group = "ephy";
				ralink,function = "ephy";
			};
		};

		wled {

			wled {
				ralink,group = "wled";
				ralink,function = "wled";
			};
		};

		rgmii1 {
			linux,phandle = <0x09>;
			phandle = <0x09>;

			rgmii1 {
				ralink,group = "rgmii1";
				ralink,function = "rgmii1";
			};
		};

		rgmii2 {
			linux,phandle = <0x0a>;
			phandle = <0x0a>;

			rgmii2 {
				ralink,group = "rgmii2";
				ralink,function = "rgmii2";
			};
		};

		pcie {
			linux,phandle = <0x10>;
			phandle = <0x10>;

			pcie {
				ralink,group = "pcie";
				ralink,function = "pcie rst";
			};
		};
	};

	rstctrl {
		compatible = "ralink,mt7620a-reset\0ralink,rt2880-reset";
		#reset-cells = <0x01>;
		linux,phandle = <0x02>;
		phandle = <0x02>;
	};

	usbphy {
		compatible = "ralink,mt7620a-usbphy";
		#phy-cells = <0x01>;
		resets = <0x02 0x16 0x02 0x19>;
		reset-names = "host\0device";
		linux,phandle = <0x0f>;
		phandle = <0x0f>;
	};

	ethernet@10100000 {
		compatible = "ralink,mt7620a-eth";
		reg = <0x10100000 0x2710>;
		#address-cells = <0x01>;
		#size-cells = <0x00>;
		interrupt-parent = <0x03>;
		interrupts = <0x05>;
		resets = <0x02 0x15 0x02 0x17>;
		reset-names = "fe\0esw";
		status = "okay";
		pinctrl-names = "default";
		pinctrl-0 = <0x09 0x0a 0x0b>;
		ralink,port-map = "llllw";
		mtd-mac-address = <0x0c 0xb4>;

		port@4 {
			compatible = "ralink,mt7620a-gsw-port\0ralink,eth-port";
			reg = <0x04>;
		};

		port@5 {
			compatible = "ralink,mt7620a-gsw-port\0ralink,eth-port";
			reg = <0x05>;
			status = "okay";
			phy-mode = "rgmii";
			phy-handle = <0x0d>;
		};

		mdio-bus {
			#address-cells = <0x01>;
			#size-cells = <0x00>;
			status = "okay";

			ethernet-phy@0 {
				reg = <0x00>;
				linux,phandle = <0x0e>;
				phandle = <0x0e>;
			};

			ethernet-phy@5 {
				phy-mode = "rgmii";
				reg = <0x05>;
				linux,phandle = <0x0d>;
				phandle = <0x0d>;
			};
		};

		port@0 {
			status = "okay";
			phy-handle = <0x0e>;
		};
	};

	gsw@10110000 {
		compatible = "ralink,mt7620a-gsw";
		reg = <0x10110000 0x1f40>;
		resets = <0x02 0x17>;
		reset-names = "esw";
		interrupt-parent = <0x01>;
		interrupts = <0x11>;
		ralink,port4 = "ephy";
	};

	sdhci@10130000 {
		compatible = "ralink,mt7620-sdhci";
		reg = <0x10130000 0xfa0>;
		interrupt-parent = <0x01>;
		interrupts = <0x0e>;
		status = "okay";
	};

	ehci@101c0000 {
		compatible = "ralink,rt3xxx-ehci";
		reg = <0x101c0000 0x1000>;
		interrupt-parent = <0x01>;
		interrupts = <0x12>;
		phys = <0x0f 0x01>;
		phy-names = "usb";
		status = "okay";
	};

	ohci@101c1000 {
		compatible = "ralink,rt3xxx-ohci";
		reg = <0x101c1000 0x1000>;
		interrupt-parent = <0x01>;
		interrupts = <0x12>;
		phys = <0x0f 0x01>;
		phy-names = "usb";
		status = "okay";
	};

	pcie@10140000 {
		compatible = "mediatek,mt7620-pci";
		reg = <0x10140000 0x100 0x10142000 0x100>;
		#address-cells = <0x03>;
		#size-cells = <0x02>;
		resets = <0x02 0x1a>;
		reset-names = "pcie0";
		interrupt-parent = <0x03>;
		interrupts = <0x04>;
		pinctrl-names = "default";
		pinctrl-0 = <0x10>;
		device_type = "pci";
		bus-range = <0x00 0xff>;
		ranges = <0x2000000 0x00 0x00 0x20000000 0x00 0x10000000 0x1000000 0x00 0x00 0x10160000 0x00 0x10000>;
		status = "okay";

		pcie-bridge {
			reg = <0x00 0x00 0x00 0x00 0x00>;
			#address-cells = <0x03>;
			#size-cells = <0x02>;
			device_type = "pci";
		};
	};

	wmac@10180000 {
		compatible = "ralink,rt7620-wmac\0ralink,rt2880-wmac";
		reg = <0x10180000 0x9c40>;
		interrupt-parent = <0x03>;
		interrupts = <0x06>;
		ralink,mtd-eeprom = <0x0c 0xe000>;
		ralink,eeprom = "soc_wmac.eeprom";
	};

	gpio-keys-polled {
		compatible = "gpio-keys";
		#address-cells = <0x01>;
		#size-cells = <0x00>;
		poll-interval = <0x14>;

		reset {
			label = "reset";
			gpios = <0x11 0x18 0x01>;
			linux,code = <0x198>;
		};
	};
};
```

# Building image
First `make download`, then `make menuconfig` and select my board. Then `make -j7` to build the world. Build products are in `bin/targets/ramips/mt7620/`.

# Flashing initial image
I used the Baicells webui (192.168.150.1 u:admin p:admin) to flash the `openwrt-ramips-mt7620-baicells-od04-squashfs-sysupgrade.bin` file I built. Board did not boot again after :(

# Debugging boot
I connected 3.3V FTDI cable to the UART header on the PCB and ran `sudo modprobe usbserial`, `sudo modprobe ftdi_sio` to get it acting as a ttyUSB.

Then used `screen` to attach to the device. Tried a few bauds, cycling power to CPE each time, until I found the correct one that didn't put garbage on screen.

`screen /dev/ttyUSB0 57600` produces this boot output:

```
U-Boot 1.1.4 (Sep  8 2021 - 09:37:10)

Board: Ralink APSoC DRAM:  64 MB
relocate_code Pointer at: 83fb8000
enable ephy clock...done. rf reg 29 = 5
SSC disabled.
spi_wait_nsec: 29
spi device id: c2 20 19 c2 20 (2019c220)
find flash: MX25L25635E
raspi_read: from:20000 len:1000
raspi_read: from:20000 len:1000
============================================
Baicells UBoot Version: 4.3.0.2
--------------------------------------------
ASIC 7620_MP (Port5<->None)
DRAM component: 512 Mbits DDR, width 16
DRAM bus: 16 bit
Total memory: 64 MBytes
Flash component: 32 MBytes NOR Flash
Date:Sep  8 2021  Time:09:37:10
============================================
icache: sets:512, ways:4, linesz:32 ,total:65536
dcache: sets:256, ways:4, linesz:32 ,total:32768

 ##### The CPU freq = 580 MHZ ####
 estimate memory size =64 Mbytes
raspi_read: from:1ff0000 len:80
raspi_read: from:1ff0000 len:80
raspi_read: from:50000 len:40
raspi_read: from:1050000 len:40

=================================================
Check image validation:
Image1 Header Magic Number --> OK
Image2 Header Magic Number --> OK
Image1 Header Checksum --> OK
Image2 Header Checksum --> OK
raspi_read: from:50000 len:54a084
crc_tmp = 2493080170 crchdr1 = 2493080170 image1_len = 5546116
Image1 Data Checksum --> OK
raspi_read: from:1050000 len:ae89ec
crc_tmp = 3738783975 crchdr2 = 3738783975 image2_len = 11438572
Image2 Data Checksum --> OK

Image1: OK Image2: OK  current bootpart:bootPart=master

=================================================

Please choose the operation:
   1: Load system code to SDRAM via TFTP.
   2: Load system code then write to Flash via TFTP.
   3: Boot system code via Flash (default).
   4: Entr boot command line interface.
   7: Load Boot Loader code then write to Flash via Serial.
   9: Load Boot Loader code then write to Flash via TFTP.                                                                                                                                                          0
raspi_read: from:1ff0000 len:80

3: System Boot system code via Flash.
## Booting image1 at bc050000 ...
raspi_read: from:50000 len:40
   Image Name:   MIPS OpenWrt Linux-5.15.142
   Image Type:   MIPS Linux Kernel Image (lzma compressed)
   Data Size:    2283638 Bytes =  2.2 MB
   Load Address: 80000000
   Entry Point:  80000000
raspi_read: from:50040 len:22d876
   Verifying Checksum ... OK
   Uncompressing Kernel Image ... OK
No initrd
## Transferring control to Linux (at address 80000000) ...
raspi_read: from:1ff0000 len:80
bootpart is bootPart=master 1
cmdline= bootPart=master bootver=4.3.0.2
## Giving linux memsize in MB, 64

Starting kernel ...

[    0.000000] Linux version 5.15.142 (cody@meh) (mipsel-openwrt-linux-musl-gcc (OpenWrt GCC 12.3.0 r24664-85f59c8e27) 12.3.0, GNU ld (GNU Binutils) 2.40.0) #0 Fri Dec 15 15:13:02 2023
[    0.000000] Board has DDR2
[    0.000000] Analog PMU set to hw control
[    0.000000] Digital PMU set to hw control
[    0.000000] SoC Type: MediaTek MT7620A ver:2 eco:6
[    0.000000] printk: bootconsole [early0] enabled
[    0.000000] CPU0 revision is: 00019650 (MIPS 24KEc)
[    0.000000] MIPS: machine is BaiCPE-EG7035E
[    0.000000] Initrd not found or empty - disabling initrd
[    0.000000] Primary instruction cache 64kB, VIPT, 4-way, linesize 32 bytes.
[    0.000000] Primary data cache 32kB, 4-way, PIPT, no aliases, linesize 32 bytes
[    0.000000] Zone ranges:
[    0.000000]   Normal   [mem 0x0000000000000000-0x0000000003ffffff]
[    0.000000] Movable zone start for each node
[    0.000000] Early memory node ranges
[    0.000000]   node   0: [mem 0x0000000000000000-0x0000000003ffffff]
[    0.000000] Initmem setup node 0 [mem 0x0000000000000000-0x0000000003ffffff]
[    0.000000] Built 1 zonelists, mobility grouping on.  Total pages: 16240
[    0.000000] Kernel command line: console=ttyS0,57600 rootfstype=squashfs,jffs2
[    0.000000] Dentry cache hash table entries: 8192 (order: 3, 32768 bytes, linear)
[    0.000000] Inode-cache hash table entries: 4096 (order: 2, 16384 bytes, linear)
[    0.000000] Writing ErrCtl register=0000c6aa
[    0.000000] Readback ErrCtl register=0000c6aa
[    0.000000] mem auto-init: stack:off, heap alloc:off, heap free:off
[    0.000000] Memory: 56088K/65536K available (5518K kernel code, 605K rwdata, 1196K rodata, 1184K init, 215K bss, 9448K reserved, 0K cma-reserved)
[    0.000000] SLUB: HWalign=32, Order=0-3, MinObjects=0, CPUs=1, Nodes=1
[    0.000000] NR_IRQS: 256
[    0.000000] Could not get sysc syscon regmap
[    0.000000] Kernel panic - not syncing: unable to get CPU clock, err=-517
[    0.000000] Rebooting in 1 seconds..
[    0.000000] Reboot failed -- System halted
```

Yay, uboot is still working. Boo, kernel is not booting.

Searching online, found https://lore.kernel.org/linux-clk/20230617052435.359177-1-sergio.paracuellos@gmail.com/T/ Sounds like maybe recent changes. Gut feeling is the dts I extracted needs elaboration to work with new code for syscon.

# DTS again
Comparing vanilla mt7620a.dtsi from openwrt with one extracted from the CPE. mt7620a.dtsi seems like a cleaner base version of the one on the CPE. CPE has customizations for eg. SPI flash, pinctl...

I try removing most of the stuff that's also in mt7620a.dtsi from the CPE .dts file and build again. Use `tftp` to send the image over and write to flash memory

My new DTS file attempt
```
// SPDX-License-Identifier: GPL-2.0-or-later OR MIT

#include "mt7620a.dtsi"

#include <dt-bindings/gpio/gpio.h>
#include <dt-bindings/input/input.h>
#include <dt-bindings/leds/common.h>

/ {
    compatible = "baicells,eg7035", "ralink,mt7620a-soc";
    model = "Baicells EG7035";

};

// TODO leds
// TODO keys, keys polled
// TODO gpio

    // keys {
    //  compatible = "gpio-keys-polled";
    //  poll-interval = <20>;

    //  reset {
    //      label = "reset";
    //      gpios = <0x11 0x18 0x01>;
    //      linux,code = <0x198>;
    //  };

    // };

&spi0 {
    status = "okay";
    pinctrl-names = "default";
    // TODO pinctrl-0 = <&spi_pins>, <&spi_cs1_pins>;

    m25p80@0 {
        compatible = "jedec,spi-nor";
        reg = <0>;
        spi-max-frequency = <2000000>;

        partitions {
            compatible = "fixed-partitions";
            #address-cells = <1>;
            #size-cells = <1>;

            partition@0 {
                label = "u-boot";
                reg = <0x00 0x20000>;
                read-only;
            };

            partition@20000 {
                label = "u-boot-env";
                reg = <0x20000 0x20000>;
                read-only;
            };

            partition@40000 {
                // TODO is MAC in here? IMEI? What else?
                label = "factory";
                reg = <0x40000 0x10000>;
                read-only;
                linux,phandle = <0x0c>;
                phandle = <0x0c>;
            };

            partition@50000 {
                label = "firmware";
                reg = <0x50000 0xeb0000>;
            };

            partition@f00000 {
                label = "log";
                reg = <0xf00000 0x100000>;
            };

            partition@1050000 {
                label = "image2";
                reg = <0x1050000 0xfa0000>;
            };

            partition@1ff0000 {
                label = "user-data";
                reg = <0x1ff0000 0x10000>;
            };

            partition@1000000 {
                label = "customization";
                reg = <0x1000000 0x50000>;
            };
        };
    };
};
```

# Build the image, then tftp boot
`$ mkdir /tmp/tftp` to make directory for serving images out of
`$ sudo dnsmasq --port=0 --enable-tftp --tftp-root=/tmp/tftp --tftp-no-blocksize --user=root --group=root` to start tftp server locally
`$ cp bin/targets/ramips/mt7620/openwrt-ramips-mt7620-baicells-eg7035-squashfs-sysupgrade.bin /tmp/tftp/openwrt-ramips-mt7620-mt7620a_eg7035e-squashfs-sysupgrade.bin` to put upgrade file in tftp dir where uboot expects it

log of upgrading image:
```
Please choose the operation:
   1: Load system code to SDRAM via TFTP.
   2: Load system code then write to Flash via TFTP.
   3: Boot system code via Flash (default).
   4: Entr boot command line interface.
   7: Load Boot Loader code then write to Flash via Serial.
   9: Load Boot Loader code then write to Flash via TFTP.

You choosed 2
                                                                                                                                                                                                                                                                       0
raspi_read: from:40028 len:6


2: System Load Linux Kernel then write to Flash via TFTP.
 Warning!! Erase Linux in Flash then burn new one. Are you sure?(Y/N)
 Please Input new ones /or Ctrl-C to discard
        Input device IP (192.168.150.1) ==:192.168.150.1
        Input server IP (192.168.150.154) ==:192.168.150.154
        Input Linux Kernel filename (openwrt-ramips-mt7620-mt7620a_eg7035e-squashfs-sysupgrade.bin) ==:openwrt-ramips-mt7620-mt7620a_eg7035e-squashfs-sysupgrade.bin

 netboot_common, argc= 3

 NetTxPacket = 0x83FE5680

 KSEG1ADDR(NetTxPacket) = 0xA3FE5680

 NetLoop,call eth_halt !

 NetLoop,call eth_init !
Trying Eth0 (10/100-M)

 Waitting for RX_DMA_BUSY status Start... done


 ETH_STATE_ACTIVE!!
TFTP from server 192.168.150.154; our IP address is 192.168.150.1
Filename 'openwrt-ramips-mt7620-mt7620a_eg7035e-squashfs-sysupgrade.bin'.

 TIMEOUT_COUNT=10,Load address: 0x80100000
Loading: Got ARP REPLY, set server/gtwy eth addr (e8:6a:64:39:cd:62)
Got it
#################################################################
         #################################################################
         #################################################################
         #################################################################
         #################################################################
         #################################################################
         #################################################################
         #################################################################
         #################################################################
         #################################################################
         #################################################################
         #################################################################
         #################################################################
         #########################################Got ARP REQUEST, return our IP
########################
         #################################################################
         #################################################################
         #################################################
done
Bytes transferred = 5571377 (550331 hex)
NetBootFileXferSize= 00550331
board_init_r:2584 +++ NetBootFileXferSize = 5571377
raspi_read: from:1ff0000 len:7f
raspi_erase_write: offs:1ff0000, count:80
raspi_read: from:1ff0000 len:10000
raspi_erase: offs:1ff0000 len:10000
.
raspi_write: to:1ff0000 len:10000
.
raspi_read: from:1ff0000 len:10000
Done!
raspi_erase_write: offs:50000, count:550331
raspi_erase: offs:50000 len:550000
.....................................................................................
raspi_write: to:50000 len:550000
.....................................................................................
raspi_read: from:50000 len:10000
raspi_read: from:60000 len:10000
raspi_read: from:70000 len:10000
raspi_read: from:80000 len:10000
raspi_read: from:90000 len:10000

[...]

raspi_read: from:590000 len:10000
raspi_read: from:5a0000 len:10000
raspi_erase: offs:5a0000 len:10000
.
raspi_write: to:5a0000 len:10000
.
raspi_read: from:5a0000 len:10000
Done!


U-Boot 1.1.4 (Sep  8 2021 - 09:37:10)

Board: Ralink APSoC DRAM:  64 MB
relocate_code Pointer at: 83fb8000
enable ephy clock...done. rf reg 29 = 5
SSC disabled.
******************************
Software System Reset Occurred
******************************
spi_wait_nsec: 29
spi device id: c2 20 19 c2 20 (2019c220)
find flash: MX25L25635E
raspi_read: from:20000 len:1000
raspi_read: from:20000 len:1000
============================================
Baicells UBoot Version: 4.3.0.2
--------------------------------------------
ASIC 7620_MP (Port5<->None)
DRAM component: 512 Mbits DDR, width 16
DRAM bus: 16 bit
Total memory: 64 MBytes
Flash component: 32 MBytes NOR Flash
Date:Sep  8 2021  Time:09:37:10
============================================
icache: sets:512, ways:4, linesz:32 ,total:65536
dcache: sets:256, ways:4, linesz:32 ,total:32768

 ##### The CPU freq = 580 MHZ ####
 estimate memory size =64 Mbytes
raspi_read: from:1ff0000 len:80
raspi_read: from:1ff0000 len:80
raspi_read: from:50000 len:40
raspi_read: from:1050000 len:40

=================================================
Check image validation:
Image1 Header Magic Number --> OK
Image2 Header Magic Number --> OK
Image1 Header Checksum --> OK
Image2 Header Checksum --> OK
raspi_read: from:50000 len:54a074
crc_tmp = 1826644150 crchdr1 = 1826644150 image1_len = 5546100
Image1 Data Checksum --> OK
raspi_read: from:1050000 len:ae89ec
crc_tmp = 3738783975 crchdr2 = 3738783975 image2_len = 11438572
Image2 Data Checksum --> OK

Image1: OK Image2: OK  current bootpart:bootPart=master

=================================================

Please choose the operation:
   1: Load system code to SDRAM via TFTP.
   2: Load system code then write to Flash via TFTP.
   3: Boot system code via Flash (default).
   4: Entr boot command line interface.
   7: Load Boot Loader code then write to Flash via Serial.
   9: Load Boot Loader code then write to Flash via TFTP.                                                                                                                                                                                                              0
raspi_read: from:1ff0000 len:80

3: System Boot system code via Flash.
## Booting image1 at bc050000 ...
raspi_read: from:50000 len:40
   Image Name:   MIPS OpenWrt Linux-5.15.142
   Image Type:   MIPS Linux Kernel Image (lzma compressed)
   Data Size:    2283622 Bytes =  2.2 MB
   Load Address: 80000000
   Entry Point:  80000000
raspi_read: from:50040 len:22d866
   Verifying Checksum ... OK
   Uncompressing Kernel Image ... OK
No initrd
## Transferring control to Linux (at address 80000000) ...
raspi_read: from:1ff0000 len:80
bootpart is bootPart=master 1
cmdline= bootPart=master bootver=4.3.0.2
## Giving linux memsize in MB, 64

Starting kernel ...

[    0.000000] Linux version 5.15.142 (cody@meh) (mipsel-openwrt-linux-musl-gcc (OpenWrt GCC 12.3.0 r24664-85f59c8e27) 12.3.0, GNU ld (GNU Binutils) 2.40.0) #0 Fri Dec 15 15:13:02 2023
[    0.000000] Board has DDR2
[    0.000000] Analog PMU set to hw control
[    0.000000] Digital PMU set to hw control
[    0.000000] SoC Type: MediaTek MT7620A ver:2 eco:6
[    0.000000] printk: bootconsole [early0] enabled
[    0.000000] CPU0 revision is: 00019650 (MIPS 24KEc)
[    0.000000] MIPS: machine is Baicells EG7035
[    0.000000] Initrd not found or empty - disabling initrd
[    0.000000] Primary instruction cache 64kB, VIPT, 4-way, linesize 32 bytes.
[    0.000000] Primary data cache 32kB, 4-way, PIPT, no aliases, linesize 32 bytes
[    0.000000] Zone ranges:
[    0.000000]   Normal   [mem 0x0000000000000000-0x0000000003ffffff]
[    0.000000] Movable zone start for each node
[    0.000000] Early memory node ranges
[    0.000000]   node   0: [mem 0x0000000000000000-0x0000000003ffffff]
[    0.000000] Initmem setup node 0 [mem 0x0000000000000000-0x0000000003ffffff]
[    0.000000] Built 1 zonelists, mobility grouping on.  Total pages: 16240
[    0.000000] Kernel command line: console=ttyS0,57600 rootfstype=squashfs,jffs2
[    0.000000] Dentry cache hash table entries: 8192 (order: 3, 32768 bytes, linear)
[    0.000000] Inode-cache hash table entries: 4096 (order: 2, 16384 bytes, linear)
[    0.000000] Writing ErrCtl register=0004c6ac
[    0.000000] Readback ErrCtl register=0004c6ac
[    0.000000] mem auto-init: stack:off, heap alloc:off, heap free:off
[    0.000000] Memory: 56084K/65536K available (5518K kernel code, 605K rwdata, 1196K rodata, 1184K init, 215K bss, 9452K reserved, 0K cma-reserved)
[    0.000000] SLUB: HWalign=32, Order=0-3, MinObjects=0, CPUs=1, Nodes=1
[    0.000000] NR_IRQS: 256
[    0.000000] CPU Clock: 580MHz
[    0.000000] clocksource: systick: mask: 0xffff max_cycles: 0xffff, max_idle_ns: 583261500 ns
[    0.000000] systick: enable autosleep mode
[    0.000000] systick: running - mult: 214748, shift: 32
[    0.000000] clocksource: MIPS: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 6590553264 ns
[    0.000002] sched_clock: 32 bits at 290MHz, resolution 3ns, wraps every 7405115902ns
[    0.015566] Calibrating delay loop... 385.84 BogoMIPS (lpj=1929216)
[    0.087802] pid_max: default: 32768 minimum: 301
[    0.098143] Mount-cache hash table entries: 1024 (order: 0, 4096 bytes, linear)
[    0.112572] Mountpoint-cache hash table entries: 1024 (order: 0, 4096 bytes, linear)
[    0.137577] clocksource: jiffies: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 19112604462750000 ns
[    0.157085] futex hash table entries: 256 (order: -1, 3072 bytes, linear)
[    0.170800] pinctrl core: initialized pinctrl subsystem
[    0.182591] NET: Registered PF_NETLINK/PF_ROUTE protocol family
[    0.199267] rt2880-pinmux pinctrl: there is not valid maps for state default
[    0.227437] rt2880_gpio 10000600.gpio: registering 24 gpios
[    0.238508] rt2880_gpio 10000600.gpio: registering 24 irq handlers
[    0.252856] clocksource: Switched to clocksource systick
[    0.265043] NET: Registered PF_INET protocol family
[    0.274982] IP idents hash table entries: 2048 (order: 2, 16384 bytes, linear)
[    0.290179] tcp_listen_portaddr_hash hash table entries: 1024 (order: 0, 4096 bytes, linear)
[    0.306948] Table-perturb hash table entries: 65536 (order: 6, 262144 bytes, linear)
[    0.322270] TCP established hash table entries: 1024 (order: 0, 4096 bytes, linear)
[    0.337516] TCP bind hash table entries: 1024 (order: 0, 4096 bytes, linear)
[    0.351532] TCP: Hash tables configured (established 1024 bind 1024)
[    0.364357] UDP hash table entries: 256 (order: 0, 4096 bytes, linear)
[    0.377290] UDP-Lite hash table entries: 256 (order: 0, 4096 bytes, linear)
[    0.391640] NET: Registered PF_UNIX/PF_LOCAL protocol family
[    0.402888] PCI: CLS 0 bytes, default 32
[    0.410849] rt-timer 10000100.timer: maximum frequency is 1220Hz
[    0.428055] workingset: timestamp_bits=14 max_order=14 bucket_order=0
[    0.448041] squashfs: version 4.0 (2009/01/31) Phillip Lougher
[    0.459576] jffs2: version 2.2 (NAND) (SUMMARY) (LZMA) (RTIME) (CMODE_PRIORITY) (c) 2001-2006 Red Hat, Inc.
[    0.482447] Serial: 8250/16550 driver, 2 ports, IRQ sharing disabled
[    0.496374] printk: console [ttyS0] disabled
[    0.504908] 10000c00.uartlite: ttyS0 at MMIO 0x10000c00 (irq = 20, base_baud = 2500000) is a Palmchip BK-3103
[    0.524562] printk: console [ttyS0] enabled
[    0.524562] printk: console [ttyS0] enabled
[    0.541131] printk: bootconsole [early0] disabled
[    0.541131] printk: bootconsole [early0] disabled
[    0.573644] spi spi0.0: force spi mode3
[    0.584153] spi-nor spi0.0: mx25l25635e (32768 Kbytes)
[    0.594613] 8 fixed-partitions partitions found on MTD device spi0.0
[    0.607307] Creating 8 MTD partitions on "spi0.0":
[    0.616879] 0x000000000000-0x000000020000 : "u-boot"
[    0.628301] 0x000000020000-0x000000040000 : "u-boot-env"
[    0.642213] 0x000000040000-0x000000050000 : "factory"
[    0.653916] 0x000000050000-0x000000f00000 : "firmware"
[    0.667491] 0x000000f00000-0x000001000000 : "log"
[    0.678412] 0x000001050000-0x000001ff0000 : "image2"
[    0.691684] 0x000001ff0000-0x000002000000 : "user-data"
[    0.703818] 0x000001000000-0x000001050000 : "customization"
[    0.748460] gsw: setting port4 to ephy mode
[    0.757037] mtk_soc_eth 10100000.ethernet: generated random MAC address 02:94:f1:d5:87:27
[    0.773384] mtk_soc_eth 10100000.ethernet: mdio-bus disabled
[    0.784887] mtk_soc_eth 10100000.ethernet: loaded mt7620 driver
[    0.797597] mtk_soc_eth 10100000.ethernet eth0: mediatek frame engine at 0xb0100000, irq 5
[    0.814766] rt2880_wdt 10000120.watchdog: Initialized
[    0.826743] NET: Registered PF_INET6 protocol family
[    0.843300] Segment Routing with IPv6
[    0.850701] In-situ OAM (IOAM) with IPv6
[    0.858732] NET: Registered PF_PACKET protocol family
[    0.868960] 8021q: 802.1Q VLAN Support v1.8
[    0.882748] /dev/root: Can't open blockdev
[    0.891024] VFS: Cannot open root device "(null)" or unknown-block(0,0): error -6
[    0.905960] Please append a correct "root=" boot option; here are the available partitions:
[    0.922628] 1f00             128 mtdblock0
[    0.922643]  (driver?)
[    0.935679] 1f01             128 mtdblock1
[    0.935692]  (driver?)
[    0.948724] 1f02              64 mtdblock2
[    0.948739]  (driver?)
[    0.961761] 1f03           15040 mtdblock3
[    0.961775]  (driver?)
[    0.974807] 1f04            1024 mtdblock4
[    0.974821]  (driver?)
[    0.987846] 1f05           16000 mtdblock5
[    0.987860]  (driver?)
[    1.000880] 1f06              64 mtdblock6
[    1.000893]  (driver?)
[    1.013925] 1f07             320 mtdblock7
[    1.013939]  (driver?)
[    1.026960] Kernel panic - not syncing: VFS: Unable to mount root fs on unknown-block(0,0)
[    1.043430] Rebooting in 1 seconds..
```
