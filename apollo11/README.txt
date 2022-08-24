================================================================================
Exploring M17-related data formats using NASA Apollo 11 Audio
================================================================================
--------------------------------------------------------------------------------
Permission to use NASA source material

Reference: https://www.nasa.gov/multimedia/guidelines/index.html

Still Images, Audio Recordings, Video, and Related Computer Files for
Non-Commercial Use

NASA content - images, audio, video, and computer files used in the rendition
of 3-dimensional models, such as texture maps and polygon data in any format -
generally are not subject to copyright in the United States. You may use this
material for educational or informational purposes, including photo
collections, textbooks, public exhibits, computer graphical simulations and
Internet Web pages. This general permission extends to personal Web pages.

News outlets, schools, and text-book authors may use NASA content without
needing explicit permission, subject to compliance with these guidelines. NASA
content used in a factual manner that does not imply endorsement may be used
without needing explicit permission. NASA should be acknowledged as the source
of the material. NASA occasionally uses copyright-protected material of third
parties with permission on its website. Those images will be marked identified
as copyright protected with the name of the copyright holder. NASA's use does
not convey any rights to others to use the same material. Those wishing to use
copyright protected material of third parties must contact the copyright holder
directly.

NASA has extensive image and video  galleries online, including historic
images, current missions, astronomy pictures, Earth images and ways to search
for NASA images. Generally, each mission and program has a video and image
collection on the topic page. For example, Space Station videos can be found at
https://www.nasa.gov/mission_pages/station/videos/index.html. Content can also
be found on our extensive social media channels.  

For questions about specific images, please call 202-358-1900. For questions
about specific video, please call 202-358-0309.

--------------------------------------------------------------------------------
Download of source material in wav format

Reference: https://www.nasa.gov/mission_pages/apollo/apollo11_audio.html

$ wget https://www.nasa.gov/62282main_countdown_launch.wav
$ mv 62282main_countdown_launch.wav apollo11_countdown.wav

--------------------------------------------------------------------------------
Conversion from wav to aud

Reference: https://tarxvf.tech/blog/m17_quick_start_2021_08_transmit_hackrf

Note: 
  - Uses sox to normalize audio levels and attenuate a bit too
  - Uses '-t raw' to tell sox to not put a file header onto the output

$ sox apollo11_countdown.wav -c 1 -b 16 -e signed-integer -r 8k -t raw - \
  norm gain -3 > apollo11_countdown.aud 

--------------------------------------------------------------------------------
Creating a shorter aud file

Notes:
  - I want a shorter .aud file so I can see preamble more often
  - The aud file format is two byte samples at 8000 samples per second
  - Therefore each second of data is 16000 bytes of data
  - To skip 30 seconds of data we need to skip 480 blocks of size 1000
  - We can use 'dd' command with 'skip' to skip over unwanted audio at start
  - We can use 'dd' command with 'count' to truncate to a desired length
  - We can use 'dd' block size of 1000 for easier math
  - Can do the math for 30 seconds via bash: "echo $((30*2*8000/1000))"
  - Trial and error results in the following sequence:

$ dd if=apollo11_countdown.aud bs=1000 skip=448 |
  dd of=apollo11_3210.aud bs=1000 count=70

Playback:

$ play -c 1 -b 16 -e signed-integer -r 8k -t raw apollo11_3210.aud 

You should hear "3, 2, 1, 0".

--------------------------------------------------------------------------------
Note on tools to create rrc, sym and bin files

I am assuming your version of m17_mod supports --rrc, --sym, --bin flags to
control output formats.

One such version is at https://github.com/n1ai/m17-cxx-demod

--------------------------------------------------------------------------------
Creating bin file from shorter aud file 

Reference: https://tarxvf.tech/blog/m17_quick_start_2021_08_transmit_hackrf
Reference: https://github.com/mobilinkd/m17-cxx-demod.git
Reference: https://spec.m17project.org/appendix/file-formats

Notes:
  - bin format is M17 symbols packed two bits per symbol (dibits), 4 symbols
    per byte (+3 = 01, +1 = 00, -1 = 10, -3 = 11) with the MSB first. These 
    are unsigned 8-bit values at 4800 symbols per second, which is 4 symbols 
    per byte at 1200 bytes per second.

$ cat apollo11_3210.aud | m17-mod -S N0CALL --bin > apollo11_3210.bin

--------------------------------------------------------------------------------
Creating sym file from shorter aud file 

References: See previous section

Notes:
  - sym format is M17 symbols (+3, +1, -1, -3) encoded as signed 8-bit values 
    at the rate of 4800 symbols per second.
$ cat apollo11_3210.aud | m17-mod -S N0CALL --sym > apollo11_3210.sym --------------------------------------------------------------------------------
Creating rrc file from shorter aud file 

References: See previous section

Notes:
  - rrc format is RRC-filtered and scaled M17 symbols. In order to
    generate a reasonable RRC waveform, the symbol rate (4800 symbols per
    second) is upsampled by a factor of 10 to an RRC sample rate of 48000
    samples per second, then the upsampled symbols are passed through the
    RRC filter. The output samples of the RRC filter are multiplied by
    7168 to fit within a signed 16-bit LE representation (e.g. a +3 value
    would be +21504).

$ cat apollo11_3210.aud | m17-mod -S N0CALL --rrc > apollo11_3210.rrc

--------------------------------------------------------------------------------
Print sizes of rrc and bin files, and the ratio of their sizes

$ bsz=`ls -l apollo11_3210.bin | awk '{print $5}'`
$ rsz=`ls -l apollo11_3210.rrc | awk '{print $5}'`
$ echo $bsz $rsz | awk '{printf "%d %d %.1f\n", $1, $2, $2*1.0/$1}'

Notes:
  - My output is "5436 434880 80.0"
  - This shows the rrc file is 80 times bigger than the bin file
  - bin file is symbols packed four to a byte 
  - rrc file is one symbol in two bytes
  - This explains a factor of 8
  - rrc file is also upsampled by 10x to generate the rrc waveform
  - This explains a factor of 80

--------------------------------------------------------------------------------
Recording of reception of full countdown file

The gzip-compressed IQ format wave file of apollo_countdown.bin as
send by the m17-gnuradio flowgraph by hackrf to rtl-sdr dongle and
sdrplusplus has been added.

It is compressed because it's size was 556M uncompressed but 85M
compressed.  The size is large because sdrplusplus recorded 2.4M
of spectrum.

After decompressing with gunzip, sox should show the following info:

$ sox --i apollo_countdown_m17_iq.wav 
Input File     : 'apollo_countdown_m17_iq.wav'
Channels       : 2
Sample Rate    : 2.4e+06
Precision      : 16-bit
Duration       : 00:01:00.65 = 145557248 samples ~ 4548.66 CDDA sectors
File Size      : 582M
Bit Rate       : 76.8M
Sample Encoding: 16-bit Signed Integer PCM


