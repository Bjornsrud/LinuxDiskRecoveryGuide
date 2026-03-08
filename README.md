# Linux Hard Disk Recovery Guide

A practical step-by-step guide for rescuing data from old hard disks using Linux tools such as **ddrescue**, **TestDisk**, **PhotoRec**, **exiftool**, and standard shell utilities.

The guide focuses on a safe recovery workflow:

1. Identify the disk
2. Create a full disk image
3. Work only on the image
4. Recover files using filesystem or signature recovery
5. Filter and organize the recovered files

It is particularly useful when working with:

- Old laptop or desktop hard drives
- Disks removed from legacy systems
- Drives that may have been reformatted or partially corrupted
- Large PhotoRec recovery outputs that require filtering

The goal is to document a **repeatable Linux-based recovery workflow** using widely available open-source tools.

---

## Contents

The repository currently contains:

[hard-disk-recovery-guide.md](https://github.com/Bjornsrud/LinuxDiskRecoveryGuide/blob/main/hard-disk-recovery-guide.md)  
A detailed guide describing the full recovery process.

Topics include:

- Disk identification
- Safe imaging with `ddrescue`
- Filesystem recovery with `TestDisk`
- Raw file recovery with `PhotoRec`
- Sorting large recovery results with `find` and `exiftool`
- Filtering recovered images by size, metadata, and timestamps

---

## Intended Audience

This guide is intended for:

- Linux users
- Developers or system administrators
- Digital archaeology / personal data recovery
- Anyone recovering files from old disks

It assumes basic familiarity with the Linux command line.

---

## Safety Notice

Data recovery can be risky if done incorrectly.

This guide is provided for educational purposes only.

**Use the information at your own risk.**

The author takes **no responsibility for data loss, hardware damage, or any other consequences resulting from the use of this guide.**

Always ensure you understand each command before executing it.

---

## License

This project is released under the **MIT License**.

See the `LICENSE` file for details.

---

## Contributing

Improvements, corrections, and suggestions are welcome.  
If you notice errors or have ideas for improving the workflow, feel free to open an issue or submit a pull request.

---

## Acknowledgements

This guide relies on the excellent open-source tools developed by the Linux and data recovery communities, including:

- GNU ddrescue
- TestDisk / PhotoRec
- ExifTool
- GNU coreutils
