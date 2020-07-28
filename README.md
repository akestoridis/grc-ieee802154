# grc-ieee802154

A collection of GNU Radio Companion flow graphs for the inspection of IEEE 802.15.4-based networks


## Description of the GRC flow graphs

The following GRC flow graphs were developed on [Debian 10.3](https://cdimage.debian.org/mirror/cdimage/archive/10.3.0/amd64/iso-cd/) using [GNU Radio 3.7](https://github.com/gnuradio/gnuradio/tree/maint-3.7), with the [gr-foo](https://github.com/bastibl/gr-foo/tree/maint-3.7) and [gr-ieee802-15-4](https://github.com/bastibl/gr-ieee802-15-4/tree/maint-3.7) modules, and a [USRP N210](https://www.ettus.com/all-products/un210-kit/) with an [SBX daughterboard](https://www.ettus.com/all-products/sbx/).

1. **ieee802154_transceiver.grc**: A simplified version of Bastian Bloessl's GRC flow graph for an IEEE 802.15.4 transceiver.

2. **ieee802154_uhd_to_cf32.grc**: A simple GRC flow graph that captures I/Q samples from a USRP for offline analysis.

3. **ieee802154_cf32_to_pcap.grc**: A simple GRC flow graph that demodulates previously captured I/Q samples.


## Instructions

* The following set of instructions was compiled using information from the following sources:
	* <https://files.ettus.com/manual/page_build_guide.html>
	* <https://kb.ettus.com/Building_and_Installing_the_USRP_Open-Source_Toolchain_(UHD_and_GNU_Radio)_on_Linux>
	* <https://files.ettus.com/manual/page_usrp2.html>
	* <https://files.ettus.com/manual/page_calibration.html>
	* <https://www.gnuradio.org/doc/doxygen/build_guide.html>
	* <https://wiki.gnuradio.org/index.php/InstallingGR>
	* <https://wiki.gnuradio.org/index.php/GNURadioCompanion>
	* <https://wiki.gnuradio.org/index.php/UbuntuInstall>
	* <https://www.wime-project.net/installation/>
	* <https://www.youtube.com/watch?v=V-oI7ss2zvs>

* Install Git in order to checkout the repositories:
```
$ sudo apt update
$ sudo apt install git
```

* Install the dependencies of UHD:
```
$ sudo apt update
$ sudo apt install build-essential cmake doxygen libboost-all-dev libusb-1.0-0-dev python-docutils python-mako python-requests python-six
```

* Build and install UHD:
```
$ git clone https://github.com/EttusResearch/uhd.git
$ cd uhd/
$ git checkout v3.14.1.1
$ cd host/
$ mkdir build
$ cd build/
$ cmake ../
$ make
$ make test
$ sudo make install
$ cd ../../../
```

* Add `libuhd.so` in the library path for dynamic linking by writing the following at the end of the `~/.bashrc` file:
```
# libuhd.so
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib
```

* Execute the content of the `~/.bashrc` file and update the list of shared libraries:
```
$ source ~/.bashrc
$ sudo ldconfig
```

* Test whether the `uhd_find_devices` command can be found or not:
```
$ uhd_find_devices
```

* Create a group called `usrp` that includes the user:
```
$ sudo groupadd usrp
$ sudo usermod -aG usrp $USER
```

* To enable real-time scheduling, add the following line in the `/etc/security/limits.conf` file:
```
@usrp            -       rtprio          99
```

* Reboot the computer:
```
$ sudo reboot
```

* Download UHD FPGA images:
```
$ sudo uhd_images_downloader
```

* Create a connection profile for the Ethernet interface that will connect to the USRP with 192.168.10.1 as its static IP address and 255.255.255.0 as its netmask.

* Connect the USRP to the Ethernet interface and then execute the following commands to make sure that you can communicate with it:
```
$ uhd_find_devices
$ uhd_usrp_probe
```

* If the `uhd_usrp_probe` command generated any configuration warnings, edit the `/etc/sysctl.conf` file accordingly, e.g.:
```
# USRP
net.core.rmem_max=50000000
net.core.wmem_max=2500000
```

* If the `/etc/sysctl.config` file was edited, refresh the system configuration:
```
$ sudo sysctl -p
```

* Load the appropriate image, by providing the type and IP address of the USRP, and then let it reboot, e.g.:
```
$ uhd_image_loader --args="type=usrp2,addr=192.168.10.5,reset"
```

* The USRP should be available shortly after it finishes its reboot process:
```
$ uhd_find_devices
$ uhd_usrp_probe
```

* To minimize IQ imbalance and DC offset of your USRP, disconnect any external hardware from the RF antenna ports and execute the following commands to calibrate the daughterboard:
```
$ uhd_cal_rx_iq_balance --verbose
$ uhd_cal_tx_iq_balance --verbose
$ uhd_cal_tx_dc_offset --verbose
```

* Disconnect from the USRP and restore the connection profile.

* Reboot the computer:
```
$ sudo reboot
```

* Install the dependencies of GNU Radio:
```
$ sudo apt update
$ sudo apt install cmake doxygen g++ git libboost-all-dev libcanberra-gtk-module libcomedi-dev libcppunit-dev libfftw3-dev libgsl-dev libqt4-opengl-dev libqwt-dev libsdl1.2-dev libusb-1.0-0-dev libzmq3-dev pkg-config python-sip-dev python-cheetah python-dev python-gtk2 python-lxml python-mako python-numpy python-qt4 python-sphinx python-wxgtk3.0 swig
```

* Build and install GNU Radio:
```
$ git clone --recursive https://github.com/gnuradio/gnuradio.git
$ cd gnuradio/
$ git checkout maint-3.7
$ git submodule update --init --recursive
$ mkdir build
$ cd build/
$ cmake ../
$ make
$ make test
$ sudo make install
$ sudo ldconfig
$ cd ../../
```

* Make sure that GNU Radio was successfully installed:
```
$ gnuradio-config-info --version
$ gnuradio-config-info --prefix
$ gnuradio-config-info --enabled-components
```

* Test a simple GNU Radio flow graph that does not require any SDR:
```
$ python gnuradio/gr-audio/examples/python/dial_tone.py
```

* Make sure that the GNU Radio Companion (GRC) tool was successfully installed:
```
$ gnuradio-companion
```

* Install the icons, mime type, and menu items bundled with GRC:
```
$ sudo /usr/local/libexec/gnuradio/grc_setup_freedesktop install
```

* Build and install gr-foo:
```
$ git clone https://github.com/bastibl/gr-foo.git
$ cd gr-foo/
$ git checkout maint-3.7
$ mkdir build
$ cd build/
$ cmake ../
$ make
$ sudo make install
$ sudo ldconfig
$ cd ../../
```

* Build and install gr-ieee802-15-4:
```
$ git clone https://github.com/bastibl/gr-ieee802-15-4.git
$ cd gr-ieee802-15-4/
$ git checkout maint-3.7
$ mkdir build
$ cd build/
$ cmake ../
$ make
$ sudo make install
$ sudo ldconfig
$ cd ../../
```

* To use the best architecture for the Vector-Optimized Library of Kernels (VOLK) function, execute the following command:
```
$ volk_profile
```

* Open the `gr-ieee802-15-4/examples/ieee802_15_4_CSS_PHY.grc` file with gnuradio-companion to generate and execute the flow graph in order to build the CSS PHY layer as an hierarchical block.

* Open the `gr-ieee802-15-4/examples/ieee802_15_4_OQPSK_PHY.grc` file with gnuradio-companion to generate and execute the flow graph in order to build the O-QPSK PHY layer as an hierarchical block.

* Open the `gr-ieee802-15-4/examples/transceiver_CSS_loopback.grc` file to test the installation without any additional hardware by generating and executing the flow graph.

* Install Wireshark:
```
$ sudo apt update
$ sudo apt install wireshark
```

* Connect to the USRP using its Ethernet profile.

* Connect appropriate antennas to the USRP, open the `gr-ieee802-15-4/examples/transceiver_OQPSK.grc` file, and do the following:
	* Simply generating and executing the flow graph should work because initially it uses the loopback interface.
	* Disable the `Packet Pad` block.
	* Enable the `UHD: USRP Source` block.
	* Set `Ch0: Enable DC Offset Correction` and `Ch0: Enable IQ Imbalance Correction` in `UHD: USRP Source`->`Properties`->`FE Corrections` equal to `True`.
	* Enable the `UHD: USRP Sink` block.
	* Generating and executing the flow graph should still work.
	* Enable the `Wireshark Connector` block.
	* Enable the `File Sink` block.
	* Generating and executing the flow graph should still work, which should generate the `/tmp/sensor.pcap` file that should contain captured packets with `Hello World!` payload.
	* Disable the `Message Strobe` block.
	* After generating and executing the flow graph, you should be able to capture packets from operational IEEE 802.15.4 networks by tuning to the appropriate channel.
	* Delete the connection from `IEEE802.15.4 OQPSK PHY` to `QT GUI Frequency Sink`.
	* Connect `UHD: USRP Source` to `QT GUI Frequency Sink`.
	* By generating and executing the flow graph, you should be able to see the FFT of the Rx PHY interface.
	* Disable the `QT GUI Frequency Sink`.
	* Create a `QT GUI Waterfall Sink`, with `Bandwidth (Hz)` equal to `4e6`
	* Connect `UHD: USRP Source` to `QT GUI Waterfall Sink`.
	* Generating and executing the flow graph should now generate a spectrogram of the Rx PHY interface.
	* Enable the `Socket PDU` block.
	* Enable the `Packet Pad` block.
	* Disable the `UHD: USRP Source` block.
	* Try to transmit a simple message by generating and executing the flow graph, followed by the following commands:
```
$ python3
>>> import socket
>>> sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
>>> message = "Hello World!"
>>> sock.sendto(message.encode(), ("127.0.0.1", 52001))
>>> exit()
```

* Terminate the execution of the flow graph and then do the following:
	* The generated `/tmp/sensor.pcap` file should contain a packet with "Hello World!" in its payload after executing the above commands.
	* An independent IEEE 802.15.4 sniffer should have also been able to capture the packet after tuning to the appropriate channel.

* Disconnect from the USRP and restore the connection profile.

* Install the latest version of Scapy:
```
$ sudo apt update
$ sudo apt install python3-pip
$ sudo pip3 install scapy
```

* Reconnect to the USRP with its Ethernet profile, reopen the modified `gr-ieee802-15-4/examples/transceiver_OQPSK.grc` file, and do the following:
	* Disable the `RIME Stack` block.
	* Disable the `IEEE802.15.4 MAC` block.
	* Connect `Socket PDU` to `IEEE802.15.4 OQPSK PHY`.
	* Try to transmit an IEEE 802.15.4 packet using Scapy (v2.4.2+) by generating and executing the flow graph, followed by the following commands:
```
$ python3
>>> import socket
>>> from scapy.all import *
>>> sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
>>> frame = Dot15d4FCS()/Dot15d4Data(dest_panid=0x4242, dest_addr=0x2323)/"Hello World!"
>>> sock.sendto(bytes(frame), ("127.0.0.1", 52001))
>>> exit()
```

* Terminate the execution of the flow graph and then do the following:
	* An independent IEEE 802.15.4 sniffer should have been able to capture the packet after tuning to the appropriate channel.
	* Try to transmit a Zigbee packet using Scapy (v2.4.2+) by generating and executing the flow graph, followed by the following commands:
```
$ python3
>>> import socket
>>> from scapy.all import *
>>> sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
>>> frame = Dot15d4FCS(fcf_panidcompress=True, fcf_ackreq=True, fcf_srcaddrmode=2)/Dot15d4Data(dest_panid=0x4242, dest_addr=0x2323, src_addr=0x0101)/ZigbeeNWK(discover_route=1, flags='security', destination=0x2323, source=0x0101, radius=30)/ZigbeeSecurityHeader(data='Hello World!')
>>> sock.sendto(bytes(frame), ("127.0.0.1", 52001))
>>> exit()
```

* Terminate the execution of the flow graph and then do the following:
	* An independent IEEE 802.15.4 sniffer should have been able to capture the packet after tuning to the appropriate channel.
	* To turn the flow graph into a simple IEEE 802.15.4 sniffer, enable the `UHD: USRP Source` and disable the `Packet Pad` and `Socket PDU` blocks.

* If you were able to follow these instructions without any errors, then you should be able to generate and execute the flow graphs of this repository.


## Publication

These GRC flow graphs were used in the following publication:

* D.-G. Akestoridis, M. Harishankar, M. Weber, and P. Tague, "Zigator: Analyzing the security of Zigbee-enabled smart homes," in _Proceedings of the 13th ACM Conference on Security and Privacy in Wireless and Mobile Networks (WiSec)_, 2020, pp. 77--88. DOI: [10.1145/3395351.3399363](https://doi.org/10.1145/3395351.3399363).


## License

Copyright (C) 2020 Dimitrios-Georgios Akestoridis

This project is licensed under the terms of the GNU General Public License version 3 or later (GPL-3.0-or-later).
