# FAQ

## General questions

### Does this run on a Pi Zero (non WiFi)?
Most of the payloads do, but not the HID backdoor (if you'd enable USB networking for this payload, it would be possible to connect to the backdoor shell, but 127.0.0.1 attacks have been out of scope for this project).
An example for a more sophisticated payload running without WiFi is the "HID frontdoor": [video link](https://www.youtube.com/watch?v=MI8DFlKLHBk)


### Does this run on other ARM devices?
P4wnP1 uses several features specific to raspbian. Among others: 
- the RNDIS ratepatch is specific to the raspbian RNDIS kernel module
- the LED interaction is based on raspbian FS interface
- USB OTG detection is based on raspbian DEBUG FS intergration
- the USB gadget stack is only tested against the UDC of the rapberry 

But everyone is free to port this to other devices.

### Should I change passwords ?
Sure... and don't forget the WiFi password ;-)

### The setup isn't easy, will there be a prebuilt image?
No. But I'm going to rework `INSTALL.md` to give some guidance on how to get the Pi Zero ready.

### What's the purpose of "John the Ripper (JtR)" and "Responder.py"
The package came from an idea in early development stage. There's an attack mentioned in the README, which allows to steal NetNTLMv2 hashes from locked Windows boxes (Rob ‘MUBIX’ Fuller: “Snagging creds from locked machines”). This was in fact the first payload which got implemented on P4wnP1 (Rsponder.py is there for the same reason, as it is needed to carry out the attack). So the plan was to develop a "LOCKPICKER" for Windows hosts with weak credentials, doing this:
1. Steal the NetNTLMv2 hash with the 'Mubix' approach
2. Fetch the hash from Responder.db and hand it over to John the Ripper
3. Try to crack the hash till P4wnP1 is shut down or SUCCESS.
4. On SUCESS, use the HID keyboard to type out the plain password and unlock the machine.

So if you ask "Where is the lock picker?" ... It has never been finished, for multiple reasons.
- The chance for the "Snagging creds attack" to succeed is very low, since Microsoft patched the issue.
- The third party application I found vulnerable to this was patched, too (see `README.md` for reference).
- Implementing the lock picker isn't a real challenge and there's a whole lot of other work to do on P4wnP1. So feel free to contribute.

Anyway, both tools could come in handy on various payloads (even ones I couldn't even imagine today). The JtR which could be fetched from the the raspbain repo with `apt-get install john` isn't really feature rich. It only supports a basic set of hash types. The package included in P4wnP1 is the JtR Jumbo version, which can handle a ton of hash types. It was compiled on the Pi itself. The hash rate for the mentioned NetNTLMv2 hashes on the Pi Zero W is about 100.000 hashes per second. This means if you have a dictionary with 100000 passwords and your victim chooses one of them, cracking takes a second at max.
The Responder.py version is a slightly modified one, allowing to respond probe requests of Microsoft Windows to check its "online status" in such a manner, that the system believes it has internet access. This could come in handy in several network attacks.

## HID backdoor payload

### What's the difference between this and a ordinary BadUSB ? 

A really good question. Karsten Nohl was the first using the term "BadUSB". What he described (or showed) was that ordinary commercial USB devices could be reprogrammed to make them act differently. Thus it was possible to mod a normal USB flashdrive (with PHISON controller) into an USB keyboard, running keyboard attacks. But the concept wasn't all about keyboards. In fact one could reprogram a vulnerable USB device to be everything, as long as it is defined in the USB specs (and one is able to reverse and modify a firmware based on an Intel 8051 derivate).
So the more precise question should be: "What's the difference between this and an ordinary RubberDucky attack ?"

### What's the difference between this and an ordinary RubberDucky attack ?
The backdoor payload does the same like a RubberDucky attack. The difference is that you can launch keyboard attacks from a WiFi based custom SSH shell on demand. The target sees only two HID devices, no other USB hardware.
From the moment you used the `FireStage1` command, things change a bit:
As stated there are two HID devices. The first one is a HID keyboard (used to carry out the keyboard attacks). The second HID device is a GENERIC HID device. The `FireStage1` command uses the keyboard device, to type out code building up a sort of protocol stack. This protocol stack is used to communicate with P4wnP1 via the second HID device. 

To be more precise:
The shell which spawns using the `shell` command doesn't use a network connection, a serial port or any othe communication device. It is based on a pure Human Interface Device. Now try to explain your firewall to block this communication channel (this isn't socket based). Or try to explain your endpoint protection to block USB HID devices and say goodbye to all kind of controllers using this standard. I guess the difference becomes clear!
So to carry out pure keyboard attacks, this kind of "covert channel" isn't needed, you can run them as soon as you SSH'd into P4wnP1 backdoor interface. But once the covert channel is up after issuing `FireStage1` there should be no need to run further keyboard attacks, as long as you know what you're doing. The input to the shell (and other spawned processes) is tunneled through the HID channel.

### Where is the code for the client side payload ?
Here: https://github.com/mame82/P4wnP1_HID_backdoor_client
