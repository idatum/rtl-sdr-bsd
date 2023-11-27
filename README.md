# rtl-sdr-bsd includes NetBSD 10 patches for rtl-sdr and dump1090-fa repos.
The goal of this experiment is to build and run [rtl-sdr](https://github.com/osmocom/rtl-sdr) tools (specifically rtl_tcp) and [dump1090](https://github.com/flightaware/dump1090) (aka dump1090-fa) on a Pine64 Rock64 running [NetBSD 10](http://netbsd.org/releases/formal-10/NetBSD-10.0.html).

My Rock64 is positioned closest to antennas I have mounted outside of my house. It has 2 RTL-SDR USB devices attached to the USB3 port, one for ADS-B and the second as a source for [rtl_433](https://github.com/merbanan/rtl_433) ISM band sensor decoding (e.g. weather station).

For rtl_433, I was unable to run this on NetBSD. It is related to the use of ``rtlsdr_read_async``. With this rtl-sdr.patch applied, I can build rtl_tcp on the Rock64 and remotely connect a separate Linux device running ``rtl_433 -d rtl_tcp:[rock64]``  to use the rtl_tcp protocol. The patch adds a ``-S`` option to specify using ``rtlsdr_read_sync`` in a pthread e.g. ``rtl_tcp -d 0 -S -a [rock64]``. This has been working successfully in my testing across a 5ghz WiFi connection.

For ADS-B, the dump1090-fa.patch patch allows me to build dump1090-fa on the Rock64. This just patches the Makefile. Running this on the Rock64, I have both [ADS-B Exchange](https://www.adsbexchange.com/) and [Piaware](https://www.flightaware.com/adsb) running in Docker containers that connect remotely. This has also been successful in my testing, but I'm not clear why. It still uses ``rtlsdr_read_async`` but I don't see any issues until I try to terminate the process.

#### More details
To get started creating the rtl_433 patch, I first looked at the patches included with the [pkgsrc](https://pkgsrc.org/) ham/rtl-sdr package. I then applied those patches to the cloned rtl-sdr Github repo (tagged at v2.0.1). I then made changes to rtl_tcp. Applying the patches from pkgsrc explains why rtl_fm is also modied in my patch.

For dump1090-fa, the Github repo was tagged at v9.0.

To use, clone each repo and apply the patch separately for each (``patch -p1``). You need gmake for dump1090-fa and cmake for rtl-sdr.

#### TODO
I chose to patch only rtl_tcp for now since I'm fine running rtl_433 on another Linux device (e.g. in Docker). I patched rtl_tcp by adding a pthread that essentially uses that thread instance how ``rtlsdr_read_async`` was used unpatched. I can also similarly experiment with pthreads in other rtl-sdr tools or even rtl_433 directly.

Even though dump1090-fa uses ``rtlsdr_read_async``, I didn't have to patch any source to run it. I just patched the Makefile. I need to understand why having a pthread to handle ``rtlsdr_read_async`` and handling the fifo dequeue on the main process thread is stable on NetBSD 10.

I also noticed I can run rtl_adsb unpatched and there might be some learnings there as well.

Ultimately, from what I understand, there needs to be improvements to usb(4) on NetBSD 10 to make these RTL-SDR programs work unpatched.
