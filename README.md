# 🛠️ EC2 Instance Recovery: Lost PEM File 🔐

**Author**: Niharika B.  
**Description**: Step-by-step guide to regain SSH access to an EC2 instance when the associated `.pem` key is lost by mounting its volume to a helper instance and injecting a new key.

---

## 🧩 Problem Statement

When a PEM key is lost for an EC2 instance, SSH access is no longer possible. But since the volume still contains your data, you can recover access by:

- 🔌 Detaching the root volume
- 🔧 Mounting it on a helper EC2 instance
- 🛠️ Adding a new public key to `authorized_keys`
- 🔁 Reattaching the volume to the original instance

---

## 🧰 Prerequisites

- ✅ AWS Console access
- ✅ Helper EC2 instance (with working PEM, e.g., `kkk.pem`)
- ✅ New key pair generated (`kkk.pem`)
- ✅ Basic Linux and EC2 knowledge

---

## 🔁 Recovery Steps

### 1️⃣ Stop the Target EC2 Instance

📍 Go to **AWS Console > EC2 > Instances** → Select the instance (with lost PEM) → Click **Stop**

---

### 2️⃣ Detach the Root Volume

📍 Go to **Elastic Block Store > Volumes**:

1. 🔍 Find the volume attached to the stopped instance (check `Attachment Info`)
2. ❌ Click **Actions > Detach volume**

---

### 3️⃣ Attach the Volume to a Helper Instance

1. 🔄 Select the detached volume
2. 📎 Click **Attach volume**
3. 🧩 Choose your helper instance from the dropdown
4. 💡 Set device name as `/dev/sdf` or `/dev/xvdf` (AWS may rename it internally)

---

### 4️⃣ SSH into the Helper Instance

```bash
ssh -i "kkk.pem" ec2-user@<Helper_Instance_Public_IP>
```

---

### 5️⃣ Mount the Volume

```bash
sudo mkdir /mnt/recover
lsblk  # Find the new volume, e.g., /dev/xvdf1
sudo file -s /dev/xvdf1  # Ensure it's XFS
sudo xfs_repair /dev/xvdf1  # If mounting fails initially
sudo mount -t xfs /dev/xvdf1 /mnt/recover
```

---

### 6️⃣ Replace `authorized_keys`

You can copy the helper instance’s public key or a new one:

```bash
cat ~/.ssh/authorized_keys  # On helper instance
# Copy the content
sudo nano /mnt/recover/home/ec2-user/.ssh/authorized_keys  # Paste the key here
```

Or if `.ssh/authorized_keys` exists in helper instance:

```bash
sudo cp ~/.ssh/authorized_keys /mnt/recover/home/ec2-user/.ssh/authorized_keys
```
> 🛑 Make sure the recovered volume is not mounted as read-only. Remount with write permissions if needed.

---

### 7️⃣ Unmount and Reattach

```bash
sudo umount /mnt/recover
```

🔁 Go to AWS Console:

1. Go to **Volumes** → Select the volume → **Detach** from helper instance
2. Reattach it back to original instance as `/dev/xvda` (or the same device name it had before)

---

### 8️⃣ Start the Old Instance

▶️ Start the original instance. Now try SSH again using the **new PEM** (e.g., `kkk.pem`):

```bash
ssh -i "kkk.pem" ec2-user@<Old_Instance_Public_IP>
```

✅ You should regain access to the instance.

---

## 🎉 Success

You’ve successfully regained access to your EC2 instance without the original PEM file by injecting a new SSH public key into the volume. 🎯

---

## 📎 Notes

- Always create a backup of your `.pem` files.
- Use IAM roles and Systems Manager (SSM) for future access to avoid PEM dependency.

---

## 📚 Resources

- [AWS EC2 User Guide](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/)
- [XFS Repair](https://linux.die.net/man/8/xfs_repair)

---

🔐 _Happy Hacking, Buddy!_ 🚀

