# aztec
Aztec is a privacy-first Layer 2 on Ethereum. It supports smart contracts with both private &amp; public state and private &amp; public execution.
High level view

## Tổng quan hệ thống Aztec
- Người dùng tương tác với Aztec qua Aztec.js (giống như Web3.js hoặc Ethers.js).
- Private functions (hàm riêng tư) được thực thi trên thiết bị người dùng thông qua PXE.
- Public functions (hàm công khai) được thực thi bởi Public VM chạy trên các node mạng Aztec.
- Các giao dịch được rollup lại và gửi lên Ethereum để xác minh tính hợp lệ.

## 🔐 Private vs Public Execution
- Private execution: Thực hiện trên thiết bị người dùng thông qua PXE – môi trường thực thi riêng tư.
- Public execution: Thực hiện bởi các node mạng qua Public VM (Aztec VM).
- Luồng xử lý giao dịch: Private → Public (một chiều).
→ Private có thể gọi Public, nhưng Public không thể gọi ngược lại Private.

##🧠 PXE (Private Execution Environment)
- Chạy phía client (trình duyệt hoặc Node.js).
- Quản lý khóa, notes, tạo proof.
- Là phần của thư viện aztec.js.
- Không biết gì về Public VM.

### ⚙️ Public VM (Aztec VM)
- Tương tự EVM.
- Chạy trên các node.
- Không biết gì về PXE.

## 🌲 Trạng thái riêng tư và công khai
### Private state:
- Dùng mô hình UTXO với các "notes".
- Notes được lưu trong cây UTXO, và khi bị hủy sẽ tạo nullifier.
- Nullifiers lưu trong nullifier tree.
- Public state:
- Giống như Ethereum – sổ cái công khai.
- Dữ liệu lưu trong public data tree.

## 👛 Tài khoản và khóa
### Account abstraction:
- Mỗi tài khoản là một smart contract.
- Cho phép tùy biến xác thực, nonce, phí.
- Ba cặp khóa cho mỗi tài khoản:
- Nullifier key pair – dùng để hủy note.
- Incoming viewing key – mã hóa note cho người nhận.
- Outgoing viewing key – mã hóa note cho người gửi.
- Không có sẵn khóa ký – lập trình viên phải định nghĩa trong account contract.

## ✍️ Ngôn ngữ Noir
- DSL (domain-specific language) cho viết smart contract ZK.
- Có thể dùng để viết circuits xác minh onchain hoặc offchain.
- Noir được dùng để phát triển contract cho Aztec.
