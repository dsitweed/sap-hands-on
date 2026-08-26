Ok, Tanaka. Mình sẽ lập **roadmap học Ansible + Jenkins** kèm **thời gian dự kiến** và **bài tập thực hành**. Mình sẽ tách thành 2 phần: **Ansible** → **Jenkins**, theo trình tự hợp lý để bạn học **thực tế DevOps**, không nhầm lẫn với Kotlin.

---

# **Roadmap học Ansible + Jenkins cho người mới**

## **Phần 1: Ansible (3–4 tuần)**

**Mục tiêu:** Biết viết playbook cơ bản → triển khai môi trường phức tạp → kết hợp CI/CD.

### **Tuần 1: Cơ bản Ansible**

* **Nội dung học:**

  1. Khái niệm: Control node, Managed node, Inventory, Playbook, Module.
  2. Cài đặt Ansible trên Linux/Mac.
  3. Viết playbook đầu tiên:

     * Ping các server
     * Chạy lệnh đơn giản (`shell`, `command`)
  4. Inventory file, host groups
* **Bài tập:**

  * Viết playbook ping tất cả server trong inventory.
  * Viết playbook chạy lệnh `uptime` trên tất cả host.

---

### **Tuần 2: Quản lý file và package**

* **Nội dung học:**

  1. Module `file`, `copy`, `template`
  2. Module `yum` / `apt` / `package` để cài phần mềm
  3. Variables, Loops, Conditions
  4. Handlers (restart service khi config thay đổi)
* **Bài tập:**

  * Viết playbook copy file config từ local → server.
  * Viết playbook cài nginx hoặc apache trên nhiều server, nếu đã cài rồi thì skip.
  * Viết playbook restart service nếu file config thay đổi.

---

### **Tuần 3: Roles và cấu trúc dự án**

* **Nội dung học:**

  1. Tạo role: `role/nginx`, `role/mysql`
  2. Sử dụng `defaults`, `vars`, `tasks`, `handlers`, `templates`
  3. Include / Import Playbook
* **Bài tập:**

  * Tạo role `nginx`:

    * Cài nginx
    * Copy config
    * Restart service
  * Viết playbook chính include role `nginx`.
  * Tạo role `user` để tạo user mới trên server.

---

### **Tuần 4: Advanced & Debug**

* **Nội dung học:**

  1. Facts, Register
  2. Error handling (`block`, `rescue`, `always`)
  3. Dynamic inventory (ví dụ từ AWS / GCP)
  4. Kết hợp Ansible với Git: pull config từ repo
* **Bài tập:**

  * Viết playbook kiểm tra OS version và cài package tương ứng.
  * Viết playbook deploy app nhỏ từ Git repo → server.
  * Tạo dynamic inventory file cho 2 server ảo (VM/local docker).

---

## **Phần 2: Jenkins (2–3 tuần)**

**Mục tiêu:** Biết cài, tạo pipeline, kết hợp với Ansible deploy.

### **Tuần 1: Cơ bản Jenkins**

* **Nội dung học:**

  1. Cài Jenkins (local / Docker)
  2. Khái niệm: Job, Pipeline, Node, Executor
  3. Tạo job freestyle đơn giản
  4. Quản lý plugin cơ bản
* **Bài tập:**

  * Tạo job build project dummy (script echo “Hello Jenkins”)
  * Thêm email notification khi job thất bại
  * Tạo 2 node agent và chạy job trên node khác

---

### **Tuần 2: Pipeline & CI/CD**

* **Nội dung học:**

  1. Declarative pipeline
  2. Steps, Stages, Environment variables
  3. Integration Git (GitHub / GitLab)
  4. Trigger build bằng webhook
* **Bài tập:**

  * Viết pipeline pull code từ GitHub → chạy script test → báo kết quả
  * Pipeline có stage: build → test → deploy
  * Trigger pipeline tự động khi push code

---

### **Tuần 3: Jenkins + Ansible (Deploy thực tế)**

* **Nội dung học:**

  1. Kết hợp Jenkins với Ansible
  2. Pipeline gọi playbook deploy app
  3. Quản lý secrets / credentials (Jenkins Credentials)
  4. Kết hợp multi-environment: dev / staging / prod
* **Bài tập:**

  * Viết pipeline deploy app dummy lên 2 server ảo bằng Ansible
  * Pipeline rollback nếu deploy fail
  * Pipeline với input manual approval trước khi deploy prod

---

## **Lưu ý thực hành**

* Tạo môi trường **VM hoặc Docker** để test playbook + pipeline.
* Thực hành càng nhiều playbook + pipeline → càng nhanh nắm DevOps workflow.
* Mỗi bài tập nên tự viết từ 0, đừng copy paste, để nhớ cú pháp và logic.

---

Mình có thể vẽ cho bạn **sơ đồ timeline trực quan 6–7 tuần**, ai học theo sẽ biết tuần nào làm gì, tuần nào deploy thử server, tuần nào kết hợp Jenkins + Ansible luôn.

Bạn có muốn mình vẽ sơ đồ này không?

SAP: 3–5h mỗi ngày (ưu tiên)
Ansible / Jenkins: 1–2h mỗi ngày, thực hành lab
Cuối tuần: 2h tổng hợp, kiểm tra playbook & pipeline