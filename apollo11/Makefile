M17_MOD = /usr/local/bin/m17-mod
CALLSIGN = N0CAL
MOD_OUT = \
     apollo11_countdown.bin \
     apollo11_3210.bin \
     apollo11_countdown.sym \
     apollo11_3210.sym \
     apollo11_countdown.rrc \
     apollo11_3210.rrc

all: download long_aud short_aud $(MOD_OUT)

# tests that your version of m17_mod uses '--bin' to produce bin output
# one such version is at https://github.com/n1ai/m17-cxx-demod
%.bin: %.aud
	@echo aud2bin target: $@    # prints the target name
	@echo aud2bin prereq: $<    # the first prerequisite listed
	if [ `$(M17_MOD) --help |& grep -c -e --bin` -gt 0 ]; \
	then $(M17_MOD) -S $(CALLSIGN) --bin < $< > $@; \
	else echo NOTICE: $(M17_MOD) does not support --bin flag; \
	fi
	ls -l $< $@

# tests that your version of m17_mod uses '--sym' to produce sym output
%.sym: %.aud
	@echo aud2sym target: $@    # prints the target name
	@echo aud2sym prereq: $<    # the first prerequisite listed
	if [ `$(M17_MOD) --help |& grep -c -e --sym` -gt 0 ]; \
	then $(M17_MOD) -S $(CALLSIGN) --sym < $< > $@; \
	else echo NOTICE: $(M17_MOD) does not support --sym flag; \
	fi
	ls -l $< $@

# tests that your version of m17_mod uses '--rrc' to produce rrc output
%.rrc: %.aud
	@echo aud2rrc target: $@    # prints the target name
	@echo aud2rrc prereq: $<    # the first prerequisite listed
	if [ `$(M17_MOD) --help |& grep -c -e --rrc` -gt 0 ]; \
	then $(M17_MOD) -S $(CALLSIGN) --rrc < $< > $@; \
	else echo NOTICE: $(M17_MOD) does not support --rrc flag; \
	fi
	ls -l $< $@

download: 
	wget https://www.nasa.gov/62282main_countdown_launch.wav
	mv 62282main_countdown_launch.wav apollo11_countdown.wav

long_aud: apollo11_countdown.wav
	sox apollo11_countdown.wav -c 1 -b 16 -e signed-integer -r 8k -t raw - \
		norm gain -3 > apollo11_countdown.aud 

short_aud: apollo11_countdown.aud
	dd if=apollo11_countdown.aud bs=1000 skip=448 | \
	  dd of=apollo11_3210.aud bs=1000 count=70

long_aud_play: apollo11_countdown.aud
	play -c 1 -b 16 -e signed-integer -r 8k -t raw apollo11_countdown.aud 

short_aud_play: apollo11_3210.aud
	play -c 1 -b 16 -e signed-integer -r 8k -t raw apollo11_3210.aud 

apollo11_countdown.bin: apollo11_countdown.aud

apollo11_3210.bin: apollo11_3210.aud

apollo11_countdown.sym: apollo11_countdown.aud

apollo11_3210.sym: apollo11_3210.aud

apollo11_countdown.rrc: apollo11_countdown.aud

apollo11_3210.rrc: apollo11_3210.aud

install:
	mkdir -p /usr/local/share/m17
	install -m 0666 -t /usr/local/share/m17 \
		apollo11_countdown.aud apollo11_3210.aud \
		$(MOD_OUT)
	ls -l /usr/local/share/m17

clean:
	rm -f \
		apollo11_countdown.wav \
		apollo11_countdown.aud apollo11_3210.aud \
		$(MOD_OUT)

