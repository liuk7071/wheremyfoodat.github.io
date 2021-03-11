# Funny Anti-Piracy and Anti-Emulator Measures in Games

Piracy has always been a recurring issue in the world of gaming. A lot of people don't want to, or can't, pay the amount gaming companies ask for. [Or maybe they don't want to pay almost full price for a game released in 2014.](https://www.amazon.com/Captain-Toad-Treasure-Tracker-Nintendo-Switch/dp/B07BBMV8MY) Of course companies have historically never been fans of having their software illegally obtained, and as such have turned to [DRM (Digital Rights Management).](https://en.wikipedia.org/wiki/Digital_rights_management) While modern games are known to have some pretty wacky DRM measures, retro games aren't that far behind ; in fact, anti-piracy measures have been a thing since at least the 80s. Let's take a look at some iconic and some less iconic occurences of anti-piracy or anti-emulator tricks from a couple of older systems.

# Gameboy
![Imgur](https://imgur.com/HMQy7UB.png)

Probably not the first console you expected to see here, right? We're starting off with the original Gameboy, released in 1989. Admittedly, Gameboy games aren't really known for their DRM or anything, since it most often does not exist. Did you know however, that the program that's always displayed during bootup (called a bootrom) is an anti-piracy check itself?

![Imgur](https://imgur.com/wssqLfT.png)
As you can see from the above code, taken from [this disassembly of the bootrom on Github](https://gist.github.com/drhelius/6063288), the Gameboy bootrom actually validates part of the cartridge (where the classic "Nintendo" logo should be stored), and hangs if it doesn't match with what it expects.

This should be a really good and effective anti-piracy measure, right? Not really, all you have to do to bypass it is make a proper header for your ROM. Good first try though.

# SNES
Just a year after the Gameboy's release, in 1990, Nintendo graced humanity with the SNES, a lovely console that's very peculiar at the same time. Let's start with some context first. SNES games, much like the NES and Gameboy, contain some extra hardware inside the cartridge, such as extra RAM or ROM banks. 
<br>
There is however, something else that's peculiar about SNES cartridges... A lot of games have coprocessors in their cartridges! The SNES already has 2 processors - a 65816, which is the main processor on the system, and a Sony SPC700, a secondary 6502-like, fully programmable CPU that handles audio! Not only can it do mixing (at an extremely slow rate, but still possible), it also controls the entire audio subsystem.

As if that wasn't enough, you can add extra CPUs via the cartridge! An iconic example of this is the [SuperFX/GSU chip](https://problemkaputt.de/fullsnes.htm#snescartgsunprogrammablerisccpuakasuperfxmariochip10games) used by iconic games such as Star Fox. Hell, a commercial game even uses an ARM(assume to be ARMv3) chip in its mapper! That moment when your coprocessor is faster than your main CPU...

Obviously with the insane amount of different mapper chips, as well as the whole coprocessor situation, bootlegging SNES games becomes more difficult. Which brings us to our first form of anti-piracy

![Imgur](https://imgur.com/DYGxULk.png)
<p style="text-align: center;">An epic emulation fail</p>

This is actually part of Super Metroid's 2-stage anti-piracy/emulation check. Super Metroid has its own unique mapper chip. When booting, the game tries to validate that the correct amount of SRAM is present on the cartridge (8KB) and that it's properly mirrored. A bootleg cartridge or an bad emulator (_sigh_) would fail this check and cause the game to hang! And this behavior is present in a bunch of stupid games that have their own unique mappers... 

As I mentioned before this is part of Super Metroid's 2-stage piracy check... The first part is checking the console's region via a Pixel Processing Unit (PPU) register! 

![Imgur](https://imgur.com/Z29ww9E.png)

SNES developers were doing what current-day developers are doing all the way back in 1994.

Let's see some more original uses for anti-piracy on the SNES

# Earthbound - Pioneering anti-piracy on the SNES
Earthbound too performs the classic anti-piracy checks that Super Metroid does too. It checks the console region and displays a screen saying "This game is not designed for your Super Famicom or Super NES." if the region is PAL. It will also try to verify that its mapper chip only has 8KB of SRAM in a similar fashion to Super Metroid, and if not, it freezes with an anti-piracy screen.

![Imgur](https://imgur.com/npHysRC.png)
<br>
Gottem

What happens if we try to get into the game via hax though? They thought of that too. Well, get ready to play the hardest game you've ever played then
<br>
![Imgur](https://imgur.com/AkoTdjt.png)

Instead of just adding more anti-piracy checks that just don't let you play the game, they went out of their way to make the game as unpleasant as possible. The enemy spawn rate is ridiculous, there's enemies in areas they don't appear, entering certain places makes the game freeze...  And if you somehow manage to get to the final boss... The game will delete your save file and freeze in the middle of the fight. Ouch.

And finally, we've got the SNES CIC lookout chip, a security chip that cartridges are required to have, containing a 4-bit CPU with a small built-in ROM. An identical chip is located inside the SNES. These 2 chips exchange bitstreams of data between them, and if the received data does not match or the transmission timings are not very good... A RESET signal is sent to the console. RIP. Oh did I mention the NES followed a similar premise too?

[Info on the NES](https://wiki.nesdev.com/w/index.php/CIC_lockout_chip)
<br>
[Info on the SNES](https://problemkaputt.de/fullsnes.htm#snescartridgeciclockoutchip)

As if that weren't enough, there's a lot of different CIC versions, all of which can be found in the links above

# N64
Of course Nintendo couldn't live without a CIC chip... So, we've got a bunch of them, again...

![Imgur](https://imgur.com/wrQyrjB.png)
[Visit the bootcode documentation this image is from here](https://www.retroreversing.com/n64bootcode)

Every cartridge also has its own bootcode, which is accompanied by a matching CIC chip. Most cartridges use the CIC chips CIC-NUS-7101/CIC-NUS-6102 along with the bootcode from the site above. As if that weren't enough, there's more.

Let's take a look at a couple games.

![Imgur](https://imgur.com/RoS9MaQ.png)

Aside from ruining a perfectly good game series, Banjo Tooie is also paranoid about piracy. First off, it tries to verify that its save chip is correct; it needs to be 2KiB EEPROM, otherwise it'll get stuck in a "NO CONTROLLER" screen.

[Oh also it uh... communicates with its CIC chip and uses the responses to decrypt game assets... 268 times.](https://tcrf.net/Banjo-Tooie#Anti-Piracy) It has more piracy checks than it had sales.

Next on, let's move to a good game.

![Imgur](https://imgur.com/KMxirqz.png)

This adorable dinosaur game is also evil, but way less so. To make sure the cartridge has not been tampered with, it verifies part of the bootcode at around address $A0000164 and compares it with what it's supposed to be. If this anti-piracy check fails, it gets stuck on this title screen. Admittedly it's a very cute screen, I wouldn't mind getting stuck here.

This is a measure you're very unlikely to trigger however, there's really not much of a point to it.

# Gameboy Advance
DRM on the Gameboy Advance is also somewhat straight forward. The iconic BIOS bootup sequence, in a similar manner to the Gameboy bootrom, verifies the cartridge's legitimacy, which is once again, easily fixed by just making a proper cartridge header

![Imgur](https://imgur.com/4L98KNO.png)

There is however a very notable example of anti-piracy/anti-emulation that can't go unmentioned. The GBA Classic NES series.

![Imgur](https://imgur.com/TyvQvIe.png)

For overpriced remasters of NES games, this series took DRM to a whole other level for the GBA. Let's break some of the things these games do

- These are the only GBA cartridges where the ROM is mirrored. On other carts, there's very specific behavior for reading from Out of Bounds ROM addresses, documented [here](https://problemkaputt.github.io/gbatek.htm#gbaunpredictablethings). However, the ROM in these cartridges is mirrored - on a bootleg cartridge or an emulator which doesn't support this, joypad input becomes utterly broken, making the game not-so-enjoyable
- They rely on an edge case in the `STM` instruction to set up DMA registers
- They execute a lot of code out of VRAM, as it is relatively fast to access. This is not really hard to handle at all, it's just a tad funny.
- These games also overwrite instructions which have already been prefetched by the CPU, as Yet Another Anti-Emulation Measure. If this is not taken into account, they break

Of course, some games implement anti-emulation by accident. [Check out this article about the atrocities committed by Hello Kitty, as an example](https://mgba.io/2020/01/25/infinite-loop-holy-grail/).

And remember guys, don't pirate games
![Imgur](https://imgur.com/ILlZrhR.png)
<br>
Super Mario All-Stars on the SNES, teaching us important life lessons