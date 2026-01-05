# 🏥 Realtime Messaging Website for Hospitals

Hệ thống nhắn tin real-time dành cho bệnh viện với quản lý tài khoản tập trung, cho phép bác sĩ và bệnh nhân giao tiếp hiệu quả trong môi trường y tế.

[![Node.js](https://img.shields.io/badge/Node.js-18+-green.svg)](https://nodejs.org/)
[![MongoDB](https://img.shields.io/badge/MongoDB-6.0+-green.svg)](https://www.mongodb.com/)
[![Docker](https://img.shields.io/badge/Docker-Ready-blue.svg)](https://www.docker.com/)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

## 📋 Mục lục

- [Giới thiệu](#-giới-thiệu)
- [Tính năng](#-tính-năng)
- [Kiến trúc hệ thống](#-kiến-trúc-hệ-thống)
- [Công nghệ sử dụng](#-công-nghệ-sử-dụng)
- [Cài đặt](#-cài-đặt)
- [Sử dụng](#-sử-dụng)
- [API Documentation](#-api-documentation)
- [Deployment](#-deployment)
- [Testing](#-testing)
- [Troubleshooting](#-troubleshooting)
- [Contributing](#-contributing)

## 🎯 Giới thiệu

Hệ thống nhắn tin real-time cho bệnh viện là một giải pháp toàn diện cho phép:

- **Quản lý tập trung**: Chỉ admin có quyền tạo và quản lý tài khoản
- **Giao tiếp real-time**: Chat trực tiếp giữa bác sĩ và bệnh nhân
- **Bảo mật cao**: JWT authentication và role-based access control
- **Scalable**: Kiến trúc microservices với Docker containerization
- **Notifications**: Hệ thống thông báo real-time qua Socket.IO

## ✨ Tính năng

### 👨‍💼 Quản trị viên (Admin)

- ✅ **Quản lý tài khoản tập trung**: Chỉ admin mới có quyền tạo tài khoản
- ✅ **Tạo tài khoản bác sĩ**: Thêm bác sĩ mới với thông tin chuyên khoa, giấy phép hành nghề
- ✅ **Tạo tài khoản bệnh nhân**: Thêm bệnh nhân mới với thông tin cá nhân đầy đủ
- ✅ **Gán bác sĩ cho bệnh nhân**: Quản lý mối quan hệ bác sĩ-bệnh nhân
- ✅ **Xóa tài khoản**: Xóa tài khoản không còn sử dụng
- ✅ **Thống kê hệ thống**: Xem tổng quan về người dùng và hoạt động
- ✅ **Dashboard quản trị**: Giao diện quản lý trực quan và dễ sử dụng

### 👨‍⚕️ Bác sĩ

- ✅ **Chat với bệnh nhân**: Nhắn tin trực tiếp với bệnh nhân được gán
- ✅ **Chat với đồng nghiệp**: Tạo phòng chat với bác sĩ khác để trao đổi
- ✅ **Quản lý bệnh nhân**: Xem danh sách bệnh nhân được gán
- ✅ **Hoàn thành tư vấn**: Đánh dấu kết thúc cuộc tư vấn
- ✅ **Lịch sử chat**: Xem lại các cuộc trò chuyện trước đó
- ✅ **Thông báo real-time**: Nhận thông báo khi có tin nhắn mới

### 👤 Bệnh nhân

- ✅ **Chat với bác sĩ**: Chỉ có thể chat với bác sĩ được gán
- ✅ **Cập nhật thông tin**: Chỉnh sửa thông tin cá nhân
- ✅ **Xem lịch sử chat**: Theo dõi các cuộc trò chuyện trước đó
- ✅ **Thông báo real-time**: Nhận thông báo khi bác sĩ trả lời

## 🏗️ Kiến trúc hệ thống

Hệ thống được xây dựng theo kiến trúc **microservices** với các service độc lập:

```
┌─────────────────────────────────────────────────────────────┐
│                        Frontend (Nginx)                      │
│              Port: 8080 (HTTP) / 443 (HTTPS)                 │
│  - Admin Dashboard  - Doctor Chat  - Patient Chat           │
└───────────────────────────┬───────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                    API Gateway (Fastify)                     │
│                        Port: 3000                            │
│  - Request Routing  - Load Balancing  - Rate Limiting       │
└───────────┬───────────────┬───────────────┬─────────────────┘
            │               │               │
            ▼               ▼               ▼
    ┌──────────────┐ ┌──────────────┐ ┌──────────────────┐
    │ User Service │ │ Chat Service │ │Notification Svc  │
    │  Port: 3001  │ │  Port: 3002  │ │   Port: 3003     │
    │              │ │              │ │                  │
    │ - Auth       │ │ - Real-time  │ │ - Push Notif     │
    │ - User Mgmt  │ │   Chat       │ │ - Email Alerts   │
    │ - RBAC       │ │ - Rooms      │ │ - Redis Queue    │
    │ - Email      │ │ - Messages   │ │                  │
    └──────┬───────┘ └──────┬───────┘ └────────┬─────────┘
           │                │                  │
           └────────────────┼──────────────────┘
                            │
            ┌───────────────┴───────────────┐
            │                               │
            ▼                               ▼
    ┌──────────────┐              ┌──────────────┐
    │   MongoDB    │              │    Redis     │
    │  Port: 27017 │              │  Port: 6379  │
    │              │              │              │
    │ - User Data  │              │ - Cache      │
    │ - Chat Data  │              │ - Queue      │
    │ - Messages   │              │ - Sessions   │
    └──────────────┘              └──────────────┘
```

### Service Details

#### 1. **API Gateway** (Port 3000)
- Entry point cho tất cả requests
- Route requests đến các microservices
- Load balancing và rate limiting
- CORS handling

#### 2. **User Service** (Port 3001)
- Authentication và Authorization
- User management (CRUD operations)
- Role-based access control (Admin, Doctor, Patient)
- Email service (password reset, notifications)
- JWT token generation và validation

#### 3. **Chat Service** (Port 3002)
- Real-time messaging với Socket.IO
- Chat room management
- Message history
- Doctor-patient và doctor-doctor rooms
- File upload support

#### 4. **Notification Service** (Port 3003)
- Push notifications
- Email notifications
- Redis-based queue system
- Real-time notification delivery

#### 5. **Frontend** (Port 8080)
- Static file serving với Nginx
- Admin dashboard
- Doctor chat interface
- Patient chat interface
- Responsive design

## 🛠️ Công nghệ sử dụng

### Backend

| Technology | Version | Purpose |
|------------|---------|---------|
| **Node.js** | 18+ | Runtime environment |
| **Express.js** | 4.18.2 / 5.1.0 | Web framework |
| **Fastify** | 5.3.3 | API Gateway framework |
| **Socket.IO** | 4.8.1 | Real-time communication |
| **MongoDB** | 6.0+ | Database |
| **Mongoose** | 6.9.1 | ODM for MongoDB |
| **Redis** | Latest | Caching & queue |
| **JWT** | 9.0.0 | Authentication |
| **bcrypt** | 5.1.0 | Password hashing |
| **Nodemailer** | 6.9.1 | Email service |

### Frontend

| Technology | Purpose |
|------------|---------|
| **HTML5** | Markup |
| **CSS3** | Styling |
| **JavaScript (ES6+)** | Client-side logic |
| **Socket.IO Client** | Real-time communication |
| **Nginx** | Web server |

### DevOps

| Technology | Purpose |
|------------|---------|
| **Docker** | Containerization |
| **Docker Compose** | Multi-container orchestration |
| **Nginx** | Reverse proxy & static serving |

## 📦 Cài đặt

### Yêu cầu hệ thống

- **Node.js**: >= 18.x
- **Docker**: >= 20.x (khuyến nghị)
- **Docker Compose**: >= 2.x
- **MongoDB**: 6.0+ (hoặc sử dụng Docker)
- **Redis**: Latest (hoặc sử dụng Docker)
- **Git**: Latest

### Cài đặt với Docker (Khuyến nghị)

#### 1. Clone repository

```bash
git clone https://github.com/baoo694/Realtime-messaging-website-for-hospitals..git
cd Realtime-messaging-website-for-hospitals.
```

#### 2. Cấu hình môi trường

Tạo file `.env` trong thư mục `user-service/`:

```env
PORT=3001
MONGODB_URI=mongodb://mongo:27017/chat-app
JWT_SECRET=your_super_secret_jwt_key_change_in_production
GMAIL_USER=your_email@gmail.com
GMAIL_PASS=your_app_password
```

**Lưu ý**: Để sử dụng Gmail, bạn cần:
1. Bật 2-Factor Authentication
2. Tạo App Password tại [Google Account Settings](https://myaccount.google.com/apppasswords)

#### 3. Khởi động với Docker Compose

```bash
# Build và khởi động tất cả services
docker-compose up -d

# Xem logs
docker-compose logs -f

# Xem logs của service cụ thể
docker-compose logs -f user-service
docker-compose logs -f chat-service
docker-compose logs -f api-gateway
```

#### 4. Tạo tài khoản admin đầu tiên

```bash
# Cài đặt dependencies (nếu chưa có)
npm install

# Chạy script tạo admin
node create_sample_users.js
```

Hoặc sử dụng API trực tiếp:

```bash
curl -X POST http://localhost:3000/api/users/create-initial-admin \
  -H "Content-Type: application/json" \
  -d '{
    "username": "admin",
    "email": "admin@hospital.com",
    "password": "admin123"
  }'
```

**Thông tin đăng nhập mặc định:**
- **Email**: `admin@hospital.com`
- **Password**: `admin123`

### Cài đặt thủ công (Development)

#### 1. Clone và cài đặt dependencies

```bash
git clone https://github.com/baoo694/Realtime-messaging-website-for-hospitals..git
cd Realtime-messaging-website-for-hospitals.

# Cài đặt root dependencies
npm install

# Cài đặt dependencies cho từng service
cd user-service && npm install && cd ..
cd chat-service && npm install && cd ..
cd notification-service && npm install && cd ..
cd api-gateway && npm install && cd ..
```

#### 2. Cấu hình môi trường

Tạo file `.env` trong mỗi service:

**user-service/.env:**
```env
PORT=3001
MONGODB_URI=mongodb://localhost:27017/hospital_chat
JWT_SECRET=your_jwt_secret_key
GMAIL_USER=your_email@gmail.com
GMAIL_PASS=your_app_password
```

**chat-service/.env:**
```env
PORT=3002
MONGODB_URI=mongodb://localhost:27017/hospital_chat
JWT_SECRET=your_jwt_secret_key
```

**notification-service/.env:**
```env
PORT=3003
REDIS_URL=redis://localhost:6379
```

**api-gateway/.env:**
```env
PORT=3000
USER_SERVICE_URL=http://localhost:3001
CHAT_SERVICE_URL=http://localhost:3002
NOTIFICATION_SERVICE_URL=http://localhost:3003
```

#### 3. Khởi động MongoDB và Redis

```bash
# MongoDB
mongod

# Redis (terminal khác)
redis-server
```

#### 4. Khởi động các services

```bash
# Terminal 1 - User Service
cd user-service && npm start

# Terminal 2 - Chat Service
cd chat-service && npm start

# Terminal 3 - Notification Service
cd notification-service && npm start

# Terminal 4 - API Gateway
cd api-gateway && npm start
```

#### 5. Khởi động Frontend

```bash
# Terminal 5 - Frontend (với Nginx)
cd front-end
docker build -t frontend .
docker run -d -p 8080:80 --name frontend frontend

# Hoặc sử dụng local server (cho development)
cd front-end
python3 -m http.server 8080
# hoặc
npx serve -p 8080
```

## 🚀 Sử dụng

### Truy cập ứng dụng

- **Frontend**: http://localhost:8080
- **API Gateway**: http://localhost:3000
- **Admin Dashboard**: http://localhost:8080/admin
- **Doctor Chat**: http://localhost:8080/chat
- **Patient Chat**: http://localhost:8080/patient

### Quy trình sử dụng

#### 1. Đăng nhập Admin

1. Truy cập http://localhost:8080
2. Đăng nhập với tài khoản admin
3. Chuyển hướng đến admin dashboard

#### 2. Tạo tài khoản bác sĩ

1. Vào tab "Quản lý Bác sĩ"
2. Click "Thêm Bác sĩ"
3. Điền thông tin:
   - Username
   - Email
   - Password
   - Chuyên khoa (Specialization)
   - Số giấy phép (License Number)
   - Khoa (Department)
   - Số điện thoại

#### 3. Tạo tài khoản bệnh nhân

1. Vào tab "Quản lý Bệnh nhân"
2. Click "Thêm Bệnh nhân"
3. Điền thông tin:
   - Username
   - Email
   - Password
   - Ngày sinh
   - Số điện thoại
   - Địa chỉ
   - Liên hệ khẩn cấp

#### 4. Gán bác sĩ cho bệnh nhân

1. Vào tab "Gán Bác sĩ"
2. Chọn bệnh nhân từ dropdown
3. Chọn bác sĩ từ dropdown
4. Click "Gán Bác sĩ"

#### 5. Chat giữa bác sĩ và bệnh nhân

1. Bác sĩ đăng nhập → Xem danh sách bệnh nhân → Chọn bệnh nhân để chat
2. Bệnh nhân đăng nhập → Tự động kết nối với bác sĩ được gán → Bắt đầu chat

## 📡 API Documentation

### Base URL

```
http://localhost:3000/api
```

### Authentication

Tất cả các API (trừ login và create-initial-admin) yêu cầu JWT token trong header:

```
Authorization: Bearer <token>
```

### User Service Endpoints

#### Authentication

```http
POST /api/users/login
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "password123"
}

Response:
{
  "token": "jwt_token_here",
  "user": { ... }
}
```

```http
POST /api/users/create-initial-admin
Content-Type: application/json

{
  "username": "admin",
  "email": "admin@hospital.com",
  "password": "admin123"
}
```

#### Admin Endpoints (Requires Admin Role)

```http
POST /api/users/create-doctor
Authorization: Bearer <admin_token>
Content-Type: application/json

{
  "username": "doctor1",
  "email": "doctor1@hospital.com",
  "password": "doctor123",
  "specialization": "Cardiology",
  "licenseNumber": "DOC12345",
  "department": "Heart",
  "phoneNumber": "0123456789"
}
```

```http
POST /api/users/create-patient
Authorization: Bearer <admin_token>
Content-Type: application/json

{
  "username": "patient1",
  "email": "patient1@hospital.com",
  "password": "patient123",
  "dateOfBirth": "1990-01-01",
  "phoneNumber": "0987654321",
  "address": "123 Main St",
  "emergencyContact": "0123456789"
}
```

```http
POST /api/users/assign-doctor
Authorization: Bearer <admin_token>
Content-Type: application/json

{
  "patientId": "patient_id_here",
  "doctorId": "doctor_id_here"
}
}
```

```http
GET /api/users/doctors
Authorization: Bearer <admin_token>
```

```http
GET /api/users/patients
Authorization: Bearer <admin_token>
```

```http
DELETE /api/users/users/:userId
Authorization: Bearer <admin_token>
```

#### User Management

```http
GET /api/users/doctor/patients
Authorization: Bearer <doctor_token>
```

```http
GET /api/users/patient/doctor
Authorization: Bearer <patient_token>
```

```http
PUT /api/users/profile
Authorization: Bearer <token>
Content-Type: application/json

{
  "phoneNumber": "new_phone",
  "address": "new_address"
}
```

### Chat Service Endpoints

```http
POST /api/chat/rooms
Authorization: Bearer <token>
Content-Type: application/json

{
  "participants": ["user_id_1", "user_id_2"],
  "type": "doctor-doctor" | "doctor-patient"
}
```

```http
GET /api/chat/rooms
Authorization: Bearer <token>
```

```http
GET /api/chat/rooms/:roomId/messages
Authorization: Bearer <token>
```

```http
POST /api/chat/rooms/:roomId/messages
Authorization: Bearer <token>
Content-Type: application/json

{
  "content": "Message content",
  "type": "text" | "file"
}
```

```http
GET /api/chat/doctor/rooms
Authorization: Bearer <doctor_token>
```

```http
GET /api/chat/patient/rooms
Authorization: Bearer <patient_token>
```

```http
POST /api/chat/doctor-patient-room
Authorization: Bearer <doctor_token>
Content-Type: application/json

{
  "patientId": "patient_id_here"
}
```

```http
POST /api/chat/doctor-doctor-room
Authorization: Bearer <doctor_token>
Content-Type: application/json

{
  "doctorId": "doctor_id_here"
}
```

```http
PUT /api/chat/rooms/:roomId/complete
Authorization: Bearer <doctor_token>
```

```http
GET /api/chat/stats
Authorization: Bearer <admin_token>
```

### Notification Service Endpoints

```http
POST /api/notifications/send
Authorization: Bearer <token>
Content-Type: application/json

{
  "userId": "user_id_here",
  "message": "Notification message",
  "type": "info" | "warning" | "error"
}
```

```http
GET /api/notifications/user/:userId
Authorization: Bearer <token>
```

```http
DELETE /api/notifications/:notificationId
Authorization: Bearer <token>
```

### Health Check Endpoints

```http
GET /api/health
GET /api/users/health
GET /api/chat/health
GET /api/notifications/health
```

## 🚢 Deployment

### Production với Docker Compose

1. **Cấu hình environment variables** cho production
2. **Setup SSL certificates** (Let's Encrypt)
3. **Configure reverse proxy** (Nginx)
4. **Setup monitoring** (Prometheus, Grafana)

```bash
# Build production images
docker-compose -f docker-compose.prod.yml build

# Deploy
docker-compose -f docker-compose.prod.yml up -d
```

### Environment Variables cho Production

```env
# Security
JWT_SECRET=<strong_random_secret>
NODE_ENV=production

# Database
MONGODB_URI=mongodb://mongo-cluster:27017/hospital_chat
REDIS_URL=redis://redis-cluster:6379

# Email
GMAIL_USER=your_production_email@gmail.com
GMAIL_PASS=your_app_password

# URLs
USER_SERVICE_URL=http://user-service:3001
CHAT_SERVICE_URL=http://chat-service:3002
NOTIFICATION_SERVICE_URL=http://notification-service:3003
```

### Docker Production Best Practices

1. **Multi-stage builds** để giảm image size
2. **Health checks** cho tất cả containers
3. **Resource limits** (CPU, memory)
4. **Logging** với centralized log management
5. **Backup** MongoDB và Redis định kỳ
6. **Monitoring** với health check endpoints

## 🧪 Testing

### Unit Tests

```bash
# User Service
cd user-service && npm test

# Chat Service
cd chat-service && npm test

# Notification Service
cd notification-service && npm test
```

### Integration Tests

```bash
# Run all integration tests
npm run test:integration
```

### Manual Testing

1. **Test Authentication Flow**
   - Login với các role khác nhau
   - Test JWT token expiration
   - Test unauthorized access

2. **Test Chat Functionality**
   - Tạo room
   - Gửi messages
   - Test real-time delivery
   - Test file upload

3. **Test Admin Functions**
   - Tạo users
   - Gán bác sĩ-bệnh nhân
   - Xóa users

## 🔧 Troubleshooting

### Common Issues

#### 1. Không thể tạo admin đầu tiên

**Nguyên nhân**: Service chưa khởi động hoặc MongoDB chưa kết nối

**Giải pháp**:
```bash
# Kiểm tra service có đang chạy không
curl http://localhost:3000/api/health

# Kiểm tra MongoDB connection
docker-compose logs user-service

# Kiểm tra MongoDB container
docker-compose ps mongo
```

#### 2. Không thể đăng nhập

**Nguyên nhân**: 
- Email/password sai
- Tài khoản chưa được tạo bởi admin
- JWT_SECRET không khớp

**Giải pháp**:
- Kiểm tra email và password
- Đảm bảo tài khoản đã được admin tạo
- Kiểm tra JWT_SECRET trong `.env` file

#### 3. Chat không hoạt động

**Nguyên nhân**: 
- Socket.IO connection failed
- Redis chưa chạy
- CORS configuration

**Giải pháp**:
```bash
# Kiểm tra Redis
docker-compose ps redis
docker-compose logs redis

# Kiểm tra Socket.IO connection trong browser console
# Kiểm tra CORS settings trong chat-service
```

#### 4. Email không gửi được

**Nguyên nhân**: 
- Gmail credentials sai
- 2FA chưa bật
- App password chưa được tạo

**Giải pháp**:
1. Bật 2-Factor Authentication
2. Tạo App Password tại [Google Account Settings](https://myaccount.google.com/apppasswords)
3. Sử dụng App Password thay vì mật khẩu thường

#### 5. Services không kết nối được với nhau

**Nguyên nhân**: 
- Network configuration trong Docker
- Service URLs không đúng

**Giải pháp**:
```bash
# Kiểm tra Docker network
docker network ls
docker network inspect <network_name>

# Kiểm tra service URLs trong .env
# Trong Docker: sử dụng service names (user-service, chat-service)
# Trong local: sử dụng localhost
```

### Logs

```bash
# Xem logs của tất cả services
docker-compose logs -f

# Xem logs của service cụ thể
docker-compose logs -f user-service
docker-compose logs -f chat-service
docker-compose logs -f api-gateway
docker-compose logs -f notification-service

# Xem logs với timestamp
docker-compose logs -f --timestamps
```

### Database Issues

```bash
# Kết nối MongoDB
docker exec -it mongo mongosh

# Kiểm tra databases
show dbs

# Kiểm tra collections
use chat-app
show collections

# Backup database
docker exec mongo mongodump --out /data/backup
```

## 🤝 Contributing

Chúng tôi hoan nghênh mọi đóng góp! Vui lòng làm theo các bước sau:

1. **Fork** repository
2. **Create** feature branch (`git checkout -b feature/amazing-feature`)
3. **Commit** changes (`git commit -m 'Add amazing feature'`)
4. **Push** to branch (`git push origin feature/amazing-feature`)
5. **Open** Pull Request

### Coding Standards

- Sử dụng ESLint cho code formatting
- Viết tests cho các tính năng mới
- Cập nhật documentation
- Follow RESTful API conventions
- Sử dụng meaningful commit messages

## 📝 Changelog

### Version 2.0.0 (Current)

- ✅ Hệ thống admin tập trung
- ✅ Chỉ admin mới có quyền tạo tài khoản
- ✅ Giao diện admin dashboard
- ✅ Quản lý bác sĩ và bệnh nhân
- ✅ Gán bác sĩ cho bệnh nhân
- ✅ Xóa tài khoản
- ✅ Thống kê hệ thống
- ✅ Docker containerization
- ✅ Microservices architecture

### Version 1.0.0

- ✅ Chat real-time
- ✅ Authentication
- ✅ Role-based access
- ✅ Doctor-patient relationships

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 📞 Support & Contact

- **Repository**: [github.com/baoo694/Realtime-messaging-website-for-hospitals.](https://github.com/baoo694/Realtime-messaging-website-for-hospitals.)
- **Issues**: [GitHub Issues](https://github.com/baoo694/Realtime-messaging-website-for-hospitals./issues)

## 🙏 Acknowledgments

- Socket.IO team cho real-time communication
- MongoDB team cho database solution
- Fastify team cho high-performance framework
- Tất cả contributors đã đóng góp cho dự án

---

**⚠️ Lưu ý**: Đây là hệ thống demo/educational, không nên sử dụng trong môi trường production mà không có các biện pháp bảo mật bổ sung và audit security.

**🔒 Security**: Đảm bảo thay đổi tất cả default passwords và secrets trước khi deploy production.

