```mermaid
flowchart TD
A["Danh sách lô"]
B["Mạng dữ liệu di động"]
C["Wifi"]
D["Dữ liệu offline"]



%"Trường hợp offline và chưa có dữ liệu offline"%

E["Cần online để tải dữ liệu lần đầu"]

B --> |"Không"| E
C --> |"Không"| E
D --> |"Không"| E

%"Trường hợp offline và đã có dữ liệu offline"%

F["Hiển thị dữ liệu offline"]

B --> |"Không"| F
C --> |"Không"| F
D --> |"Có"| F
```
