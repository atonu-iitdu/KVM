# Enabling and Using Serial Console on KVM Virtual Machines

This guide explains how to verify, configure, and access the serial console of a KVM virtual machine (VM). The serial console is useful for debugging, detailed boot logging, and system recovery.

---

## 1. Check if Serial Console is Present (from Host)

Login to the **KVM host** as `root` and run:

```bash
# virsh dumpxml dev-kvm-01 | grep console
    <console type='pty' tty='/dev/pts/1'>
    </console>
```

or

```bash
# virsh ttyconsole dev-kvm-01
/dev/pts/1
```

✅ If you see `/dev/pts/*`, a serial console is already present.

---

## 2. Verify Kernel Boot Command Line (from Guest)

Login to the **guest VM** as `root` and check:

```bash
# cat /proc/cmdline
BOOT_IMAGE=(hd0,msdos1)/vmlinuz-5.14.0-503.40.1.el9_5.x86_64 root=/dev/mapper/Kube_Cluster-root ro rd.lvm.lv=Kube_Cluster/root crashkernel=1G-4G:192M,4G-64G:256M,64G-:512M
```

🔍 If you see `rhgb quiet`, the boot logs are hidden. Remove them for detailed console logs.

---

## 3. Remove `rhgb quiet` Parameters (from Guest)

```bash
# grubby --update-kernel=ALL --remove-args='rhgb quiet'
```

This ensures full boot logs are visible on the console.

---

## 4. Detect Available Serial Console in Guest

```bash
# dmesg | grep tty
[    0.039092] printk: legacy console [tty0] enabled
[    0.274372] 00:00: ttyS0 at I/O 0x3f8 (irq = 4, base_baud = 115200) is a 16550A
```

✅ Here `ttyS0` is available at baud rate **115200**.

---

## 5. Add Serial Console to Kernel Boot Parameters (from Guest)

Add both VGA (`tty0`) and serial (`ttyS0`) consoles:

```bash
# grubby --update-kernel=ALL --args='console=tty0 console=ttyS0,115200'
```

Then reboot:

```bash
# reboot
```

---

## 6. Connect to Serial Console (from Host)

List running VMs:

```bash
# virsh list
 Id   Name                        State
-------------------------------------------
 1    KMaster-02_192.168.122.22   running
 2    guest                       running
```

Connect to the guest console:

```bash
# virsh console 2
Connected to domain 'guest'
Escape character is ^] (Ctrl + ])
```

You should now see the login prompt:

```text
worker-3 login: root
Password:
Last login: Wed Sep 24 15:08:09 from 192.168.122.1
[root@guest ~]#
```

---

## 7. Notes

- Use `Ctrl + ]` to **exit** the console.
- If console doesn’t show logs, ensure kernel arguments include `console=ttyS0,115200`.
- Recommended baud speed is **115200**.

---

✅ With this setup, you can view detailed boot logs and login directly to the VM through the **serial console** from the host.
