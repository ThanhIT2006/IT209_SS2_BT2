# Bài 2: Khởi tạo User thường và thiết lập đặc quyền quản trị

## 1. Mục tiêu

Bài thực hành nhằm tạo một tài khoản người dùng thường có tên `devops` trên Ubuntu Server thay vì sử dụng trực tiếp tài khoản `root`.

Các mục tiêu chính:

* Tạo user `devops`.
* Thêm `devops` vào nhóm `sudo`.
* Sao chép SSH Key từ tài khoản `root` sang `devops`.
* Thiết lập quyền chính xác cho thư mục `.ssh` và file `authorized_keys`.
* Kiểm tra khả năng đăng nhập SSH bằng user `devops`.
* Kiểm tra quyền quản trị thông qua lệnh `sudo whoami`.

---

## 2. Môi trường thực hiện

* Hệ điều hành: Ubuntu Server
* Máy chủ: DigitalOcean Droplet
* User ban đầu: `root`
* User cần tạo: `devops`

---

## 3. Tạo user `devops`

Đăng nhập vào Droplet bằng tài khoản `root` và thực hiện:

```bash
adduser devops
```

Hệ thống yêu cầu nhập mật khẩu cho user `devops`.

Sau khi nhập mật khẩu, Ubuntu hỏi thêm các thông tin:

```text
Full Name []:
Room Number []:
Work Phone []:
Home Phone []:
Other []:
```

Có thể nhấn `Enter` để bỏ qua các thông tin này.

Cuối cùng hệ thống hỏi:

```text
Is the information correct? [Y/n]
```

Nhập:

```text
Y
```

Kiểm tra user đã được tạo:

```bash
id devops
```

Kết quả mong đợi có dạng:

```text
uid=1001(devops) gid=1001(devops) groups=1001(devops)
```

---

## 4. Thêm `devops` vào nhóm sudo

Thực hiện:

```bash
usermod -aG sudo devops
```

Kiểm tra:

```bash
groups devops
```

Kết quả mong đợi:

```text
devops : devops sudo
```

Như vậy user `devops` đã được thêm vào nhóm `sudo` và có thể thực hiện các lệnh quản trị thông qua `sudo`.

---

## 5. Sao chép SSH Key từ root sang devops

SSH Key của tài khoản `root` nằm trong:

```text
/root/.ssh
```

Sử dụng `rsync` để sao chép cấu hình SSH:

```bash
sudo rsync --archive --chown=devops:devops /root/.ssh /home/devops/
```

Lệnh trên thực hiện:

* `--archive`: giữ nguyên cấu trúc và thuộc tính file khi sao chép.
* `--chown=devops:devops`: thay đổi owner và group của file sau khi sao chép.
* `/root/.ssh`: thư mục SSH của root.
* `/home/devops/`: thư mục home của user `devops`.

Sau khi thực hiện, kiểm tra:

```bash
ls -la /home/devops/
```

Kết quả cần có:

```text
.ssh
```

---

## 6. Kiểm tra quyền thư mục `.ssh`

Kiểm tra:

```bash
ls -ld /home/devops/.ssh
```

Thư mục `.ssh` nên có quyền:

```text
drwx------ 
```

Tương ứng với quyền `700`.

Nếu cần thiết lập lại quyền:

```bash
chmod 700 /home/devops/.ssh
```

Đảm bảo owner:

```bash
chown -R devops:devops /home/devops/.ssh
```

---

## 7. Phân quyền file `authorized_keys`

Kiểm tra:

```bash
ls -l /home/devops/.ssh/authorized_keys
```

File `authorized_keys` nên có quyền:

```text
-rw-------
```

Tương ứng với quyền `600`.

Thiết lập quyền:

```bash
chmod 600 /home/devops/.ssh/authorized_keys
```

Đảm bảo owner:

```bash
chown devops:devops /home/devops/.ssh/authorized_keys
```

Kiểm tra lại:

```bash
ls -la /home/devops/.ssh
```

Kết quả mong đợi:

```text
drwx------  devops devops .ssh
-rw-------  devops devops authorized_keys
```

---

## 8. Kiểm tra quyền của thư mục Home

Kiểm tra:

```bash
ls -ld /home/devops
```

Đảm bảo thư mục thuộc về user `devops`.

Nếu cần sửa:

```bash
chown devops:devops /home/devops
```

---

## 9. Kiểm tra SSH từ máy cá nhân

Từ máy tính cá nhân, sử dụng private key tương ứng với public key đã được lưu trong:

```text
/home/devops/.ssh/authorized_keys
```

Thực hiện:

```bash
ssh -i /path/to/private_key devops@<IP_ADDRESS_DROPLET>
```

Trong đó:

* `/path/to/private_key`: đường dẫn đến private SSH key trên máy cá nhân.
* `<IP_ADDRESS_DROPLET>`: địa chỉ IP của DigitalOcean Droplet.

Ví dụ:

```bash
ssh -i ~/.ssh/id_ed25519 devops@<IP_ADDRESS_DROPLET>
```

Nếu đăng nhập thành công, Terminal sẽ chuyển sang user:

```text
devops@ubuntu-server:~$
```

Kiểm tra user hiện tại:

```bash
whoami
```

Kết quả:

```text
devops
```

---

## 10. Kiểm tra quyền sudo

Sau khi đăng nhập bằng `devops`, thực hiện:

```bash
sudo whoami
```

Hệ thống có thể yêu cầu nhập mật khẩu của user `devops`:

```text
[sudo] password for devops:
```

Nhập mật khẩu đã tạo ở bước `adduser`.

Nếu cấu hình thành công, kết quả:

```text
root
```

Điều này chứng minh rằng:

* Đang đăng nhập bằng user thường `devops`.
* `devops` thuộc nhóm `sudo`.
* `devops` có thể thực hiện các thao tác quản trị thông qua `sudo`.

---

## 11. Log Terminal kiểm tra

### Kiểm tra user

```bash
$ whoami
devops
```

### Kiểm tra nhóm

```bash
$ groups
devops sudo
```

### Kiểm tra sudo

```bash
$ sudo whoami
[sudo] password for devops:
root
```

### Kiểm tra SSH Key

```bash
$ ls -la ~/.ssh
```

Kết quả:

```text
drwx------ 2 devops devops 4096 ... .
-rw------- 1 devops devops  ... authorized_keys
```