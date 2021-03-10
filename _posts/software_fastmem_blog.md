# Explaining software fastmem by example - Featuring PS1 rants

One of the most important components of a system is the bus. What's a bus? It's a magic little thing that handles where exactly a memory access will read from/write to. For example, let's assume we're making our simple Gameboy emulator, shall we?

The CPU has specific instructions to read and write to memory. For the Gameboy for example, you can write the contents of the "A" register into address 0xC000 by just simply doing `ld [0xC000], A`
But *where* exactly will writing to 0xC000 write too? Let's look at the Gameboy memory map.

![Imgur](https://imgur.com/4BI2GLG.png)
As you can see in this image, addresses 0xC000 to 0xDFFF are mapped to "Work RAM bank 0". But what makes writes to this address actually go to this address? That's the job of the bus. It takes an address and redirects the read/write to the appropriate location. To emulate this, one can easily make a function like this

```rs
/// Our function to read a byte from an address
fn readByte (address: u16) -> u8 {
    match (address) {
        0..=0x7FFF => readROM(address), // if 0 <= address <= 0x7FFF, return a value from ROM
        0x8000..=0x9FFF => readVRAM(address), // similarly here
        0xA000..=0xBFFF => readExternalRAM(address),
        0xC000..=0xDFFF => readWRAM(address),
        0xE000..=0xFDFF => panic!("Read from Echo"), // echo RAM is actually very simple to emulate, but it's not here because I don't want to tire you with stupid gameboy stuff
        ... and so on
    }
}
```

# What's a fastmem and why do we care?