all: clean out.bin

out.bin:
	cp -f rom.asm out.asm
	asf2mc8 -l -o out.asm
	rm -f out.asm
	aslink out -i
	srec_cat out.ihx -intel -offset -0xe000 -o out.bin -binary

program: out.bin
	srec_cat out.bin -binary -o out.dphex -intel -address-length=2 -line-length=44 -crlf
	dprom /dev/tty.usbserial-FT0G8F41 out.dphex

clean:
	rm -f out.*
