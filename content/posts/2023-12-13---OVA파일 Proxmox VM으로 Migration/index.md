---
title: Import OVA as Proxmox VM
date: "2023-12-13"
template: "post"
draft: false
slug: "/posts/Import-OVA-as-Proxmox-VM"
category: "Proxmox"
tags:
  - "Proxmox"
  - "OVA"
  - "Migration"
  - "VirtualBox"
description: "VirtualBox, VMWare등의 가상화 프로그램에서 Proxmox로 VM을 이전하기 위한 방법 중 하나입니다."
---

OVA는 Open Viratual Applicance 파일로, VirtualBox, VMWare 등의 가상화 프로그램에서 가상 머신(VM)과 관련된 다양한 파일을 저장하는데 사용합니다. 이 파일을 이용해서 가상화 디스크를 불러오고 VM을 시작할 수 있도록 만들어 줍니다. 그러나 Proxmox에서는 OVA 파일을 사용하지 않습니다. 그렇기 때문에 OVA에서 정의된 VM을 사용할 수가 없습니다. 따라서 이번 포스트에서는 OVA파일이 포함하고 있는 VM 디스크 파일을 이용해 저장된 VM을 사용할 수 있도록 하였습니다.

- [Create VM](#create-vm)
- [Delete associated disk](#delete-associated-disk)
- [Import OVA](#import-ova)
- [Assign OVA to VM](#assign-ova-to-vm)

## Create VM
먼저 평소대로 Proxmox에서 VM을 만드는데, 설치 드라이브만 제외하고 만듭니다.

![Create: Virtual Machine](./media/image.png)

## Delete associated disk
VM 생성이 완료가 되면 VM Hardware를 들어가게 되면 아까 VM 생성 마법사를 통해 선택했던 Hard Disk가 있습니다. 이 디스크는 OVA에 있는 디스크로 대체할 거기 때문에 상단에 있는 Detach를 한뒤 삭제하도록 합니다.

![Alt text](./media/image-2.png)

## Import OVA
다음 단계는 OVA 디스크를 가져온 뒤 VM에 할당하는 작업입니다. OVA 파일을 Proxmox 서버로 복사한 뒤 OVA 파일의 압축을 풀고, 그 안에 있는 VM disk(vmdk)를 VM에 할당할 것입니다.

OVA 파일 안에는 아래와 같이 3개의 파일이 있는데 이 중에서 vmdk확장자로 되어 있는 파일을 사용합니다. 압축 해제는 리눅스에서 tar 명령어나 윈도우에서 압축 프로그램을 이용하면 됩니다.

![Alt text](./media/image-3.png)

OVA 파일을 압축 해제 한 후에는, vmdk 파일을 아래와 같은 명령어를 통해 qemu 디스크로 불러옵니다.

    # qm importdisk [VM번호] [디스크파일이름].vmdk local-lvm -format qcow2
![Alt text](./media/image-4.png)

Import 과정이 다 끝나고 난 뒤에 위에서 생성한 VM의 Hardware 속성을 보게 되면 import한 디스크가 있을 것입니다.
![Alt text](./media/image-5.png)

## Assign OVA to VM
이제 Proxmox에 디스크를 가져왔지만 사용할 수 있는 단계는 아닙니다. 디스크가 VM에 할당된 것은 아니기 때문에 할당하는 작업을 해줘야 합니다. 가져온 디스크를 선택해서 Edit을 클릭한다. 그러면 디스크를 추가할 수 있는 마법사가 나오게 됩니다. 여기서 Bus/Device에서 VM의 디스크 접근 방식을 선택해야 합니다. 보통은 SCSI를 선택하면 되지만 본인이 이전 VM에서 사용했던 방식을 선택하면 됩니다. 선택한 후에 Add를 눌러서 추가합니다.

![Alt text](./media/image-6.png)

디스크를 추가한 후에는 VM의 부팅 순서를 변경해야 합니다. Options에 Boot Order를 Edit하게 되면 아래와 같이 ide2와 net0 device들만 활성화 되어 있습니다. 방금 추가한 디스크를 사용하기 위해 scsi0 디바이스를 Enable하고 부팅 순서의 최우선순위로 설정하겠습니다.

![Alt text](./media/image-7.png)
![Alt text](./media/image-8.png)

이제 서버를 시작해 보면 OVA 파일이 성공적으로 Proxmox로 이전된 것을 확인할 수 있습니다.
![Alt text](./media/image-9.png)