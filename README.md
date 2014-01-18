AM335x remoteproc firmware for driving WS28xx RGB LEDS
=======================================================

WS2812 Datasheet: http://www.adafruit.com/datasheets/WS2812.pdf

WS2801 Datasheet: http://www.adafruit.com/datasheets/WS2801.pdf

Koen's Evil Vendor Kernel: https://github.com/beagleboard/kernel/tree/3.8

Universe Pin Mappings on Beagebone Black/White:

* 0  -> P8\_45 (PRU1 R30\_0)
* 1  -> P8\_46 (PRU1 R30\_1)
* 2  -> P8\_43 (PRU1 R30\_2)
* 3  -> P8\_44 (PRU1 R30\_3)
* 4  -> P8\_41 (PRU1 R30\_4)
* 5  -> P8\_42 (PRU1 R30\_5)
* 6  -> P8\_39 (PRU1 R30\_6)
* 7  -> P8\_40 (PRU1 R30\_7)
* 8  -> P8\_27 (PRU1 R30\_8)
* 9  -> P8\_29 (PRU1 R30\_9)
* 10 -> P8\_28 (PRU1 R30\_10)
* 11 -> P8\_30 (PRU1 R30\_11) *(clock line for WS2801)*

"Benchmarks" using WS281x chipset on frames per second using spidev universe count and at 256 slots

* 1 -> 127 FPS
* ...
* 10 -> 100 FPS
* ...
* 12 -> 96 FPS

Example usage *(low speed virtio serial usage)*:

*NOTE: Virtio serial doesn't support termios so no flow control, and you'll need a hacked up minicom*

		 $ cp ws28xx-pru1-ws281x /lib/firmware/ws28xx-pru1
		 $ cp ws28xx-pru0 /lib/firmware/ws28xx-pru0
		 $ echo BB-BONE-PRU-05 > /sys/devices/bone\_capemgr.\*/slots 
		 $ minicom -D /dev/vport0p0

		 PRU#0> ?
		 Help
		  s <universe>              select universe 0-11
		       ws2801 (firmware) -> 0-10
		       ws281x (firmware) -> 0-11
		  b 	                    blanks slots 0-255
		  m <val>                   max number of slots per universe 0-255
		  w <num> <v1>.<v2>.<v3>    write 24-bit GRB value to slot number
		  l                         latch data out the PRU1

		 PRU#0> m 30
		 PRU#0> b
		 PRU#0> w 0 ff.00.00
		 PRU#0> w 1 00.ff.00
		 PRU#0> w 2 00.00.ff
		 ...
		 PRU#0> s 1
		 PRU#1> b
		 PRU#1> w 0 ff.ff.00
		 PRU#1> w 1 00.ff.ff
		 PRU#1> w 2 ff.ff.ff
		 ...
		 PRU#1> l

	         ** Blinky Lights! **

Examples usage *(HIGH speed ioctl/spidev usage)*:

		$ cat /proc/misc | grep pru_leds
		 59 pru_leds
		$ mknod /dev/pruleds0.0 c 10 59
		$ echo "m 30" > /dev/pruleds0.0

		** Install OLA and use the examples/spidev-pru.conf **


Important Notes:

* Disable HDMI out on BeagleBone Black to free up PRU pins
 * "optargs=capemgr.disable\_partno=BB-BONELT-HDMI,BB-BONELT-HDMIN" in /boot/uEnv.txt
* Blanking only has to be called once per universe unless you are changing slot count
* Disabling unused slots will increase FPS and reduce the data written
* After latching data updating values is locked till the transaction completes
 * To avoid double buffering we have to be sure all data is written
 * Accessing the same PRU shared memory will stall one or both PRUs
