all: clean out.bin

out.bin:
	asf2mc8 -l -o+out rom.asm
	aslink -i out
	srec_cat out.hex -intel -offset -0xe000 -o out.bin -binary

program: out.bin
	srec_cat out.bin -binary -o out.dphex -intel -address-length=2 -line-length=44 -crlf
	dprom /dev/tty.usbserial-FT0G8F41 out.dphex

clean:
	rm -f out.*
