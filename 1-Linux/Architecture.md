## OS Architecture
* **Bootloader:** Code fragment executed at startup to boot the OS (GRUB in Parrot Linux).
* **OS Kernel:** Core component managing system hardware and I/O resources.
* **Daemons:** Background services handling tasks like scheduling, networking, and printing.
* **OS Shell:** Command-line interpreter between user and kernel (Bash, Zsh, Fish...).

## File System Hierarchy

/ (Root)
├── bin    -> Essential command binaries
├── boot   -> Static files of the boot loader
├── dev    -> Device files
├── etc    -> Host-specific system configuration
├── home   -> User home directories
├── lib    -> Essential shared libraries and kernel modules
├── media  -> Mount point for removable media
├── mnt    -> Mount point for a temporarily mounted filesystem
├── opt    -> Add-on application software packages
├── proc   -> Kernel and process information virtual filesystem
├── root   -> Home directory for the root user
├── run    -> Run-time variable data
├── sys    -> Kernel and system information virtual filesystem
├── tmp    -> Temporary files
├── usr    -> Secondary hierarchy
└── var    -> Variable data




