
@'
# LAB3 - Threats / Threat Detection

- Ho ten: Dao Quoc Trung
- MSSV: 1150080121
- Lop: 11_DH_THMT
- Lab: LAB3
- Moi truong: Windows 11 x64 tren VMware Workstation


## 1. Cach dung moi truong
- Su dung may ao Windows 11 x64.
- PowerShell chay voi quyen Administrator.
- Sysmon duoc cai dat va nap cau hinh sysmon-lab.xml.
- Sysinternals Autoruns duoc su dung de kiem tra persistence.
- Python duoc su dung de tao HTTP listener cuc bo tren 127.0.0.1:8080.

## 2. Cac tinh huong da thuc hien
- Thu thap baseline he thong.
- Kiem tra Defender/EICAR.
- Ghi nhan telemetry bang Sysmon.
- Tao HTTP listener chi tren loopback 127.0.0.1:8080.
- Kiem tra process so huu cong 8080.
- Kiem tra persistence bang Autoruns.
- Tao persistence lanh LAB3_Notepad de quan sat.
- Xoa LAB3_Notepad sau khi kiem tra.

## 3. Ket qua
- Baseline: PASS
- Defender/EICAR: PASS
- Sysmon ProcessCreate: PASS
- HTTP listener 127.0.0.1:8080: PASS
- Sysmon NetworkConnect: PASS
- Autoruns persistence: PASS
- Cleanup LAB3_Notepad: PASS

## 4. Loi gap phai va cach khac phuc
- Ban dau Sysmon64.exe khong ton tai trong C:\LAB3\Tools\Sysmon.
  Khac phuc: tai va giai nen Sysmon vao dung thu muc.
- Autorunsc64.exe ban dau chua co.
  Khac phuc: tai Sysinternals Autoruns va giai nen vao C:\LAB3\Tools\Autoruns.
- Khi tao evidence_sha256.csv, PowerShell co gang hash chinh file dang duoc tao.
  Khac phuc: loai evidence_sha256.csv khoi danh sach file can hash.
- Persistence LAB3_Notepad duoc xoa sau khi hoan thanh kiem tra.

## 5. Evidence
Cac output/log da lam sach nam trong thu muc output/.
Danh sach SHA256 nam trong evidence_sha256.csv.

## 6. Luu y
Repository khong chua installer, executable Sysinternals/Wireshark/Python
hoac file bi Windows Defender quarantine.
'@ | Set-Content "C:\LAB3\SUBMIT\LAB3\README.md" -Encoding UTF8
