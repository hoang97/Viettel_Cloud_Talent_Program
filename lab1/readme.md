# Deploy OpenStack with Kolla-Ansible

Editor: **Do Bao Hoang**

---
## Table of contents
[I. Introduction](#intro)
- [1. OpenStack](#openstack)
- [2. Kolla](#kolla)
- [3. Ansible](#ansible)

[II. System requirements](#requirements)

[III. Deployment](#deployment)

[IV. Encountered Errors](#errors)

[V. References](#references)

---
## I. Introduction <a name='intro'></a>

### 1. OpenStack <a name='openstack'></a>
**OpenStack** là một platform điện toán đám mây nguồn mở hỗ trợ cả `public clouds` và `private clouds`. Nó cung cấp giải pháp xây dựng hạ tầng điện toán đám mây đơn giản, có khả năng mở rộng và nhiều tính năng phong phú.

<div align="center">
  <img width="300" src="imgs/openstack_logo.png" alt="OpenStack logo">
</div>

<div align="center">
  <i>Pic. 1 - OpenStack logo</i>
</div>

Ban đầu, **OpenStack** được phát triển bởi **NASA** và **Rackspace**, phiên bản đầu tiên vào năm 2010. Định hướng của họ từ khi mới bắt đầu là tạo ra một dự án nguồn mở mà mọi người có thể sử dụng hoặc đóng góp. OpenStack dưới chuẩn `Apache License 2.0,` vì thế phiên bản đầu tiên đã phát triển rộng rãi trong cộng đồng được hỗ trợ bởi hơn 12000 cộng tác viên trên gần 130 quốc gia, và hơn 150 công ty bao gồm *Redhat*, *Canonical*, *IBM*, *AT&T*, *Cisco*, *Intel*, *PayPal*, *Comcast* và một nhiều cái tên khác. 

**OpenStack** được phát triển theo chu kỳ 6 tháng. Sau mỗi bản phát hành thử nghiệm sẽ có các bản phát hành ổn định (stable released).

*Table 1 - OpenStack Last 5 Releases*
| Series | Status | Initial Release Date |
|--------|--------|----------------------|
| 2023.1 Antelope | Development | 2023-03-22 estimated (schedule) |
| Zed | Maintained | 2022-10-05 | 
| Yoga | Maintained | 2022-03-30 |
| Xena | Maintained | 2021-10-06 |
| Wallaby | Maintained | 2021-04-14 |

**OpenStack** không phải là một dự án đơn lẻ mà là một nhóm các dự án nguồn mở nhằm mục đích cung cấp các dịch vụ cloud hoàn chỉnh. **OpenStack** chứa nhiều thành phần:

- **OpenStack compute** (`Nova`): là module quản lý và cung cấp máy ảo. Nó hỗ trợ nhiều hypervisors gồm KVM, QEMU, LXC, XenServer... Compute là một công cụ mạnh mẽ mà có thể điều khiển toàn bộ các công việc: networking, CPU, storage, memory, tạo, điều khiển và xóa bỏ máy ảo, security, access control.
- **OpenStack Image Service** (`Glance`): là module quản lý các disk image ảo. Glance hỗ trợ các ảnh Raw, Hyper-V (VHD), VirtualBox (VDI), Qemu (qcow2) và VMWare (VMDK, OVF). Bạn có thể thực hiện: cập nhật thêm các virtual disk images, cấu hình các public và private image và điều khiển việc truy cập vào chúng, và tất nhiên là có thể tạo và xóa chúng.
- **OpenStack Object Storage** (`Cinder`): dùng để quản lý lưu trữ. Nó là một hệ thống lưu trữ phân tán cho quản lý tất cả các dạng của lưu trữ như: archives, user data, virtual machine image … Có nhiều lớp redundancy và sự nhân bản được thực hiện tự động, do đó khi có node bị lỗi thì cũng không làm mất dữ liệu, và việc phục hồi được thực hiện tự động.
- **Identity Server** (`Keystone`): quản lý xác thực cho user và projects.
- **OpenStack Netwok** (`Neutron`): là thành phần quản lý network cho các máy ảo. Cung cấp chức năng network as a service. Đây là hệ thống có các tính chất pluggable, scalable và API-driven.
- **OpenStack dashboard** (`Horizon`): cung cấp cho người quản trị cũng như người dùng giao diện đồ họa để truy cập, cung cấp và tự động tài nguyên cloud. Việc thiết kế có thể mở rộng giúp dễ dàng thêm vào các sản phẩm cũng như dịch vụ ngoài như billing, monitoring và các công cụ giám sát khác.

### 2. Kolla <a name='kolla'></a>

**Openstack Kolla** là Project hay công cụ sử dụng để triển khai, vận hành **Openstack**. **Kolla** được phát hành từ phiên bản `Kilo` và chính thức trở thành Project Openstack tại phiên bản `Liberty`.

Với ý tưởng của **Project Kolla** là triển khai **Openstack** trong môi trường `Container`, tự động triển khai **Openstack** bằng Kolla Ansible. Qua đó chỉ với 1 vài thao tác, chúng ta đã có môi trường **Openstack** để sử dụng. Hơn nữa, **Project Kolla** cũng cung cấp sẵn các giải pháp về giám sát, HA, Rolling Upgrades … cho **Openstack**

<div align="center">
  <img width="300" src="imgs/kolla_logo.png" alt="OpenStack logo">
</div>

<div align="center">
  <i>Pic. 2 - Kolla logo</i>
</div>

**Kolla** gồm có 3 dự án chính:

- `Kolla-ansible`: triển khai các container của Kolla bằng Ansible
- `Kayobe`: triển khai OpenStack control plane trên máy bare metal sử dụng Kolla-ansible và Bifrost
- `Ansible-collection-kolla`: cung cấp các ứng dụng Ansible phổ biến được sử dụng bởi các dự án Kolla

**Kolla** cũng có các phiên bản stable tương ứng với **OpenStack**.

*Table 2 - Kolla and OpenStack Last Releases*
| Kolla Series | OpenStack Series |
|--------------|------------------|
| `15.1.x.x` | `lastest` |
| `15.0.x.x` | Zed |
| `14.x.x` | Yoga | 
| `13.x.x` | Xena | 
| `12.x.x` | Wallaby | 

### 3. Ansible <a name='ansible'></a>

**Ansible** là một trong những công cụ quản lý cấu hình hiện đại, nó tạo điều kiện thuận lợi cho công việc cài đặt, quản lý và bảo trì các server từ xa, với thiết kế tối giản giúp người dùng cài đặt và chạy nhanh chóng.

<div align="center">
  <img width="300" src="imgs/ansible_logo.png" alt="OpenStack logo">
</div>

<div align="center">
  <i>Pic. 3 - Ansible logo</i>
</div>

**Ansible** sử dụng kiến trúc `agentless` không cần đến agent để giao tiếp với các máy khác. Cơ bản nhất là giao tiếp thông qua các giao thức `WinRM` trên Windows, `SSH` trên Linux hoặc giao tiếp qua chính `API` của thiết bị đó cung cấp.

Để hiểu và áp dụng **Ansible** một cách hiệu quả, các bạn cần nắm rõ một số thuật ngữ được sử dụng như: ***Controller Machine***, ***Inventory***, ***Playbook***, ***Task***, ***Module***, ***Role***, ***Play***, ***Facts***, ***Handlers***, ***Variables***, ***Conditions***.

**Ansible** có 2 dự án cộng đồng chính: `Ansible community package` và `Ansible-core`

*Table 3 - Compare Ansible community project*
| Ansible community package | ansible-core |
|---------------------------|--------------|
| Uses new versioning (2.10, then 3.0.0) | Continues “classic Ansible” versioning (2.11, then 2.12) |
| Follows semantic versioning rules | Does not use semantic versioning |
| Maintains only one version at a time | Maintains latest version plus two older versions | 
| Includes language, runtime, and selected Collections | Includes language, runtime, and builtin plugins | 
| Developed and maintained in Collection repositories | Developed and maintained in ansible/ansible repository | 



---
## II. System requirements <a name='requirements'></a>

Trong bài thực hành này mình sẽ sử dụng VM Ubuntu (Ubuntu 22.04 LSTM) chạy trên Virtual Box. 
Để cài đặt phiên bản Kolla-ansible `15.1.0` cùng OpenStack `lastest` theo [hướng dẫn](https://docs.openstack.org/kolla-ansible/latest/user/quickstart.html#top) 

- Suggesting
    - 8GB RAM
    - 2 network interfaces
    - 2 disks (atleast 40GB disk space)
    - 4 cores cpu
- Personal VM
    - 4GB RAM
    - 2 network interfaces
    - 2 disks (40GB - 40GB)
    - 4 cores cpu

---
## III. Deployment <a name='deployment'></a>

---
## IV. Encountered Errors
<a name='errors'></a>

- MariaDB liveness error
- Not create volume group for Cinder LVM
- Re-generate password after changing config file
- Remove container before re-deploy OpenStack
- Permission denied

---
## V. References
<a name='references'></a>
https://docs.openstack.org/kolla-ansible/latest/user/quickstart.html

https://news.cloud365.vn/openstack-kolla-phan-1-huong-dan-cai-dat-openstack-train-all-in-one-bang-kolla-ansible/

https://docs.openstack.org/kolla/latest/