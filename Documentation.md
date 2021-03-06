## 0. Preface

### Thanks 

This project is based on the work of:

1. [Holguer A Becerra (spanish)], [google translated version] 
    
    HDMI (video) output using a DE0 Nano Development Kit.

2. [FPGA4FUN] 
    
    Creating HDMI video using an FPGA.

3. [chipos81/charcole] 

    HDMI output for NeoGeo.

4. [Public.Resource.Org] 
    
    HDMI [1.3a][HDMI1.3a] and [1.4][HDMI1.4] specification can be easily found on the internet, but [CEA Standard CEA-861-D][EIA-CEA-861-D], to which is refered many times in the HDMI specification, was harder to find. [IEC Standard 60958-3][IEC-60958-3] (Digital Audio Interface: Part 3 Consumer Applications), which is needed to understand how digital audio is embedded into HDMI, I found only here.

### What's working?

Dreamcast 480p (VGA) output via HDMI with embedded 44.1kHz audio (no A/D conversion) directly from dreamcast's video/audio subsystems.

### What's missing?

1. 480i support
2. Upscaling
3. OSD
4. EDID support

## 1. Hardware Modifications

Luckily, the dreamcast uses an external video DAC (IC401), so we can tap into the signals here:

*Video DAC on Schematic:*

![Schematic][IC401schematic]

- **VSync** (VSYNC) (pin 52)
- **HSync** (HSYNC) (pin 53)
- **VClk** (VCLK) (pin 54) 

    Video clock: double the pixel clock (54Mhz for 480p), because RGB values are transmitted in two clock cycles. (See below)

- **D0-D11** (pin 56, pins 1-11)

    From [Dreamcast Hardware Specification Outline (page 37)][dc-hso]:

    > From HOLLY, the 24bit RGB image information (each RGB 8bit) is sent in units of 12 bit to the Digital-Video-Encoder (video DAC/encoder). The original 24 bit image data is divided into 12bit(RGB [11:0]) of MSB (RGB [11:0] = R [7:0], G[7:4]) and 12 bit of LSB (RGB [11:0] = G [3:0], B [7:0]). Then it is sent to the 54 Hz clock (which is double the 27Hz VGA pixel-clock) in a synchronised manner where it alternates between MSB and LSB. For details refer to a separate specification design document.

![Dreamcast video][DCvideo]

For the audio part we can tap into the audio DAC ([PCM1725][PCM1725]) (IC303). The sampling rate is 44.1kHz.

*Audio DAC on Schematic:*

![Schematic][IC303]

- **LRC** (DLRCK) (pin 1) Audio LR Clock (LVTTL)
- **AData** (DSD) (pin 2) Audio Data (LVTTL)
- **BCK** (DBCK) (pin 3) Audio Clock (LVTTL)
- **AClk** (DSCK) (pin 14) 256Fs audio clock signal (LVTTL)

*Soldering points on Dreamcast mainboard:*

![Photo][IC401solderPoints]

*Kynar wire soldered to video DAC:*

![Photo][IC401photo]

It's quite tricky to solder kynar wire directly to the video DAC, because the round wire tends to slip between the legs of the chip, but with a relatively steady hand - mine is not :) - it should be manageable. Lots of flux is the key. Soldering to the audio DAC is be much easier, because of it's larger pitch :)

It should be possible to design a flatflex cable which is soldered directly to the VideoDAC - like the [UltraHDMI] is doing ([UltraHDMI Flatflex])

To enable "VGA" output, you either have to plugin any dreamcast VGA cable, or pull pins 6 and 7 of the dreamcast video connector to ground (this should be done with a switch, to keep 480i capability).

## 2. Video 
    
The dreamcast is generating 720x480p (not VGA) according to [EIA-CEA-861-D][EIA-CEA-861-D] "720x480p @59.94/60 Hz (Formats 2 & 3)" (chapter 4.5), not "640x480p @59.94/60 Hz (Format 1)" (chapter 4.2), but uses only 640 pixels of the possible 720.

## 3. Audio

When tapping into audio, I highly recommend to solder 4 kynar wires to ground and create twisted cables for the audio signals. Without that I experienced major interference from the video lines.
    
## 4. HDMI

Currently I am creating the HDMI output directly via the FPGA's LVDS outputs and a simple LVDS to TMDS converter and a breadboard HDMI plug, which is far from optimal, but working ok for 480p. 

According to [Holguer A Becerra][Holguer A Becerra (spanish)] (top of site) it should even work without any level conversion "since most CML (similar to TMDS) receivers already have AC / DC coupling", but without I didn't had any luck on my AV receiver.

*LVDS to TMDS Converter:*

![LVDS to TMDS Converter][LVDS2TMDS] 

*LVDS to TMDS Converter on breadboard:*

![LVDS to TMDS Converter on breadboard][LVDS2TMDS-breadboard]

For a real product a real HDMI transmitter should be used (e.g. [ADV7513][ADV7513]).

## 5. FPGA

I'm using a [DE0 Nano SOC][de0nanosoc], which is quite overpowered for this job (Cyclone V). It should be possible to use at least a Cyclone III or even a Cyclone II.

The project was recently updated to Altera Quartus Prime 16.1.2 (now Intel), which is available for free from their [website][Quartus]. Just open FPGA/DCx.qpf project and you should be able to compile it.

*Project Photo:*

![Project Photo][Overview]

---

[Quartus]: https://www.altera.com/products/design-software/fpga-design/quartus-prime/overview.html
[de0nanosoc]: http://www.terasic.com.tw/cgi-bin/page/archive.pl?Language=English&No=941
[UltraHDMI]: http://ultrahdmi.retroactive.be/
[UltraHDMI Flatflex]: http://cdn3.bigcommerce.com/s-c7bpm05/product_images/theme_images/ultrahdmi_carousel_2.png?t=1478293813
[Holguer A Becerra (spanish)]: https://sites.google.com/site/ece31289upb/practicas-de-clase/practica-4-sincronizadores/hdmi_de0-nano
[google translated version]: https://translate.google.com/translate?sl=es&tl=en&js=y&prev=_t&hl=de&ie=UTF-8&u=https%3A%2F%2Fsites.google.com%2Fsite%2Fece31289upb%2Fpracticas-de-clase%2Fpractica-4-sincronizadores%2Fhdmi_de0-nano&edit-text=
[FPGA4FUN]: http://fpga4fun.com/HDMI.html
[chipos81/charcole]: https://github.com/charcole/NeoGeoHDMI
[Public.Resource.Org]: https://law.resource.org/pub/12tables.html
[Technical Details]: https://rawgit.com/chriz2600/DreamcastHDMI/master/assets/index.html
[Video Details Link]: https://rawgit.com/chriz2600/DreamcastHDMI/master/assets/video.html
[IC401schematic]: https://github.com/chriz2600/DreamcastHDMI/raw/master/assets/VideoDAConSchematic.png
[IC401photo]: https://github.com/chriz2600/DreamcastHDMI/raw/master/assets/VideoDAC3.JPG
[IC401solderPoints]: https://github.com/chriz2600/DreamcastHDMI/raw/master/assets/VideoDACSolderingPoints.png
[DCvideo]: https://github.com/chriz2600/DreamcastHDMI/raw/master/assets/dc-video.png
[dc-hso]: https://github.com/chriz2600/DreamcastHDMI/raw/master/Documents/Dreamcast_Hardware_Specification_Outline.pdf
[ADV7513]: https://github.com/chriz2600/DreamcastHDMI/raw/master/Documents/Datasheets/ADV7513.pdf
[PCM1725]: https://github.com/chriz2600/DreamcastHDMI/raw/master/Documents/Datasheets/pcm1725.pdf
[LVDS2TMDS]: https://github.com/chriz2600/DreamcastHDMI/raw/master/assets/LVDS2TMDS.png
[LVDS2TMDS-breadboard]: https://github.com/chriz2600/DreamcastHDMI/raw/master/assets/LVDS2TMDS2.JPG
[IC303]: https://github.com/chriz2600/DreamcastHDMI/raw/master/assets/IC303.png
[Overview]: https://github.com/chriz2600/DreamcastHDMI/raw/master/assets/Overview.JPG
[CoreEP4CE6-Oscillator]: https://github.com/chriz2600/DreamcastHDMI/raw/master/assets/Waveshare-CoreEP4CE6.png

[HDMI1.3a]: https://github.com/chriz2600/DreamcastHDMI/raw/master/Documents/Specs/HDMISpecification13a.pdf
[HDMI1.4]: https://github.com/chriz2600/DreamcastHDMI/raw/master/Documents/Specs/HDMI-Specification-1.4.pdf
[EIA-CEA-861-D]: https://github.com/chriz2600/DreamcastHDMI/raw/master/Documents/Specs/EIA-CEA-861-D.pdf
[IEC-60958-3]: https://github.com/chriz2600/DreamcastHDMI/raw/master/Documents/Specs/is.iec.60958.3.2003.pdf

[CoreEP4CE6]: http://www.waveshare.com/wiki/CoreEP4CE6
[AliCoreEP4CE6]: https://www.aliexpress.com/item/Waveshare-Altera-Cyclone-Board-CoreEP4CE6-EP4CE6E22C8N-EP4CE6-ALTERA-Cyclone-IV-CPLD-FPGA-Development-Core-Board-Full/32643916772.html
[FPGA-CycloneIV]: https://github.com/chriz2600/DreamcastHDMI/tree/v0.1/FPGA-CycloneIV
[FPGA-CycloneIV-ADV7513]: https://github.com/chriz2600/DreamcastHDMI/tree/v0.1/FPGA-CycloneIV-ADV7513

[ADV7513]: http://www.analog.com/en/products/audio-video/analoghdmidvi-interfaces/analog-hdmidvi-display-interfaces/adv7513.html
[ADV7513p]: https://github.com/chriz2600/ADV7513

[LVDS2TMDSboard]: https://github.com/chriz2600/LVDS2TMDS