# Compile Drivers for Coral Edge TPU on Alpine Linux

This guide describes how to compile and install the kernel modules required for using the Google Coral Edge TPU on Alpine Linux. It is intended for advanced users and hobbyists who want to use Coral PCIe devices on custom Alpine-based systems.

## Requirements:
Make sure you have the following installed on your build machine:
- `git`
- `docker`
- `docker-compose`

## Building the Kernel Modules
```bash
git clone https://github.com/sebastianzehner/alpine-coral-tpu.git
cd alpine-coral-tpu
make
```
This will create `apex.ko` and `gasket.ko` in the `./output` directory.

## Installing the Kernel Modules
Copy the kernel modules into the appropriate directory for your kernel version:
```bash
cp output/*.ko /lib/modules/<kernel-version>/
depmod
modprobe gasket
modprobe apex
```

To load the modules automatically at boot, add them to `/etc/modules`:
```bash
grep gasket /etc/modules || echo "gasket" >> /etc/modules
grep apex /etc/modules || echo "apex" >> /etc/modules
```

## Example
Assuming you compiled the modules locally and want to deploy them to a target system running Alpine 3.22.1 with the `linux-lts` kernel version `6.12.56-0`:

### On the build machine:
```bash
scp output/*.ko root@192.168.7.151:/lib/modules/6.12.56-0-lts/
```

### On the target system:
```bash
depmod
modprobe gasket
modprobe apex

# Optional: Ensure modules are loaded on boot
grep gasket /etc/modules || echo "gasket" >> /etc/modules
grep apex /etc/modules || echo "apex" >> /etc/modules
```

## Customisation
Keep in mind that simply changing the kernel version might not be enough due to version pinning of other installed libraries. If needed, refer to the Alpine package index for updated versions:
https://pkgs.alpinelinux.org/packages?branch=v3.22

### Environment Variables
You can override these environment variables to match your target setup:
| Variable         | Default     | Description                   |
|------------------|-------------|-------------------------------|
| `KERNEL_VERSION` | `6.12.56`   | Kernel version to compile for |
| `KERNEL_RELEASE` | `0`         | Release number                |
| `KERNEL_VARIANT` | `lts`       | Options: `lts`, `virt`        |

## Disclaimer

I'm not a professional developer - just a hobbyist sharing my personal setup.  
This build is provided as-is, with no guarantees that it will work for you.  
If something breaks, you're on your own - but feel free to explore, adapt, and improve!
