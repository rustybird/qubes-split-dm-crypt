# _Split dm-crypt_ for Qubes OS


**Isolates device-mapper based secondary storage encryption (i.e. not
the root filesystem) and LUKS header processing to DisposableVMs.**

Instead of directly attaching an encrypted LUKS partition from a source
VM such as sys-usb to a destination VM and decrypting it there, it works
like this:

1. The encrypted partition is attached from the source VM to an offline
   _device DisposableVM_ configured not to parse its content in any way:
   The kernel partition scanners, udev probes, and UDisks handling are
   disabled.

2. From there, the LUKS header is sent to a (short-lived) offline
   _header DisposableVM_ prompting for the password, and the encryption
   key is sent back to the device DisposableVM, which validates that it
   received an AES-XTS or Adiantum key and creates the dm-crypt mapping.

3. Finally, the decrypted partition is attached from the device
   DisposableVM to the destination VM.

**If the destination VM is compromised, it does not know the password or
encryption key. It also cannot easily exfiltrate decrypted data to the
disk in a form that would allow an attacker who seizes the disk contents
later to read it.** (But see below for caveats.)


## Usage

```
qvm-block-split attach|at|a [--ro] [<opt>...]     [<dst>] <src>:<dev>
                detach|dt|d                               <src>:<dev>

                overwrite-everything-with-random          <src>:<dev>
                overwrite-header-with-random              <src>:<dev>
                overwrite-header-with-format [<opt>...]   <src>:<dev>
                overwrite-header-with-shell  [<opt>...]   <src>:<dev>
                modify-header-with-shell     [<opt>...]   <src>:<dev>

The <dst> argument defaults to yet another DisposableVM.

<opt> can be: --key-file=[<vm>:]<path>
              --type=luks1 to override the default --type=luks2
```

As seen above, **the `qvm-block-split` attach/detach commands accept a
subset of the familiar `qvm-block` syntax**, and some other commands are
included:

- Fully overwrite a device with random data

- Overwrite just the LUKS header with random data

- Format a new LUKS device


## Remaining attacks

- After detaching, the password and/or key will linger in more RAM
  locations than without _Split dm-crypt_. Until there is a way to wipe
  the DisposableVMs' memory, and `qvm-block-split` is modified not to
  pass the key through dom0's memory, **power off your computer when
  memory forensics is a concern.**

- If both the destination VM and the source VM/disk are compromised,
  they could establish a covert channel using e.g. read and write access
  patterns, slowly saving some amount of decrypted data to the disk.

- If the source VM/disk is compromised and successfully exploits the
  header DisposableVM using a malicious LUKS header, a known key could
  be sent to the device DisposableVM and used to present malicious
  device content to the destination VM to potentially exploit it as
  well. **Be suspicious if you do not see the expected filesystem data
  in the destination VM. Or simply use a DisposableVM as the destination
  VM.**

- **Don't forget to overwrite your disk with random data before creating
  a LUKS volume on it.** Otherwise, a compromised destination VM could
  trivially save decrypted data to the disk in its free space, by
  encoding each bit as an unmodified (still empty or in some other way
  nonrandom-looking) or modified (random-looking) 16 byte AES block or
  in case of Adiantum a 512/4096 byte block depending on the dm-crypt
  sector size.


## Installation

1. Copy `vm/` to a DisposableVM Template's _TemplateVM_ (e.g.
   `fedora-XX`) - not to the DisposableVM Template _itself_ (e.g.
   `fedora-XX-dvm`).

   Inspect the code, and `sudo make install`. Shut down the TemplateVM
   when finished.

2. Copy `dom0/bin/qvm-block-split` to dom0, e.g. into `~/bin/`, inspect
   the code extra carefully, and `chmod +x` the script.

3. Either make your DisposableVM Template from step 1 the system-wide
   default:

        qubes-prefs default_dispvm fedora-XX-dvm

   Or just let _Split dm-crypt_ know what it is:

        echo TEMPLATE_FOR_DISPVMS=fedora-XX-dvm >/etc/split-dm-crypt.conf


## Safety warning

The code's error handling is strict, and I haven't experienced any data
loss during development. Nevertheless, please **ensure you have a backup
of all drives that are connected to your computer.**


## Redistribution

_Split dm-crypt_ is under public domain equivalent license, see the
LICENSE-0BSD file for details.
