#
#  Makefile for building the Zet BIOS in Linux
#  Copyright (C) 2010  Zeus Gomez Marmolejo <zeus@aluzina.org>
#  Copyright (C) 2010  Donna Polehn <dpolehn@verizon.net>
#
#  This file is part of the Zet processor. This program is free software;
#  you can redistribute it and/or modify it under the terms of the GNU
#  General Public License as published by the Free Software Foundation;
#  either version 3, or (at your option) any later version.
#
#  Zet is distrubuted in the hope that it will be useful, but WITHOUT
#  ANY WARRANTY; without even the implied warranty of MERCHANTABILITY
#  or FITNESS FOR A PARTICULAR PURPOSE. See the GNU General Public
#  License for more details.
#
#  You should have received a copy of the GNU General Public License
#  along with Zet; see the file COPYING. If not, see
#  <http://www.gnu.org/licenses/>.
#

#
#  ZET BIOS BUILD MAKE - Linux version - 10-10-2010
#  ----------------------------------------------------------------------------
#
#  This is the make file for the Zet Rom BIOS and VGA BIOS Builds.
#  To build it, you will need the Open Watcom compiler. If you have
#  it installed on your machine, then you should be able to just type
#  wmake and watcom should build it. There will be a small number of warning
#  messages, this is normal.
#

# File extension in Linux
O = .o
E =

# Current directory (for executing programs)
CDIR = ./

all: bios.rom ../../cores/flash/bios.dat zetbios.dat

!include "common.inc"

vga_vers.asm: vga.asm
	bash -c "sed 's/(git)/$$(git describe)/g' vga.asm > vga_vers.asm"

zetbios_vers.c: zetbios.c
	bash -c "sed 's/(git)/$$(git describe)/g' zetbios.c | sed 's/(build-date)/$$(date -R)/g' | sed 's/(commit-date)/$$(git log -1 --pretty=format:%cD)/g' > zetbios_vers.c"

#
# This section combines all 3 rom sections built below, into one 128K ROM
# ready to be installed. When everything is done running and if all goes well
# you should have a file BIOS.ROM of exactly 128K, this is your bios rom to
# download into your DE1 board.
#
bios.rom: vgabios.rom ff.rom zetbios.rom
	cat vgabios.rom ff.rom zetbios.rom > $@

# BIOS.DAT for Modelsim simulation, we use hexdump to convert it to a row
# of bytes
../../cores/flash/bios.dat: bios.rom
	hexdump -v -e '1/1 "%02X"' -e '"\n"' bios.rom > ../../cores/flash/bios.dat

# HEX ROM MAKE ---------------------------------------------------------------
# This section converts the bios rom binary file into a hex file that the
# DE0 loader program can use to program the Flash Ram on the DE0 board.
# ----------------------------------------------------------------------------
bios.hex: bios.rom hexer zetbios.dat
	$(CDIR)hexer bios.rom

# DAT SHADOW ROM MAKE --------------------------------------------------------
# This section converts the last 256 bytes of the bios rom to a dat file (hex)
# that needs to be included in the Quartus project build. The last 256 bytes
# of the ROM is the shadow boot rom section.
# ----------------------------------------------------------------------------
zetbios.dat: zetbios.rom mifer
	$(CDIR)mifer zetbios.rom
	cp -f zetbios.dat ../../cores/flash/bootrom.dat

#
# This section compiles and links the biossums, vgasums and ff utilities.
# The biossums is used to computer the checksum byte which is then tacked
# onto the end of zetbios.rom. The vgasums program scans the vgabios.rom
# file, computes the checksum which is added to the end. It also pads the
# file out to a full 32k. The ff utility just makes a 32k file of 0xFF which
# is used to pad it to a full 64k segment. After all that, both rom files will
# be 64K in length and ready to be programed into the flash.
#
# The C option flags for this build are standard console build since
# these utilities will run on your PC not on the ZET
#

ff: ../tools/ff.c
	gcc ../tools/ff.c -o ff

biossums: ../tools/biossums.c
	gcc ../tools/biossums.c -o biossums

vgasums: ../tools/vgasums.c
	gcc ../tools/vgasums.c -o vgasums

hexer: ../tools/hexer.c
	gcc ../tools/hexer.c -o hexer

mifer: ../tools/mifer.c
	gcc ../tools/mifer.c -o mifer

clean:
	rm -f *.dat *.hex *.o *.obj *.sym *.rom *.exe *.err *.map *.flash zetbios_vers.c vga_vers.asm ff biossums vgasums mifer hexer

#
#  End of make.
#
