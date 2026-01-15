# 🧩 1 — NESTJS BASE & BOILERPLATE
## 🔹 Tạo NestJS Base Project
- Bước 1: Tạo project
```c
nest new `project name`
```
- Bước 2: Chạy thử 
```c
cd `project name`
npm run start:dev
```
- Bước 3: Mở trình duyệt: `http://localhost:3000` thấy `Hello World!` là đã tạo thành công một project nestjs base.
## 🔹 Boilerplate là gì?
### Khái niệm
Boilerplate là bộ khung dự án được xây dựng sẵn, bao gồm cấu trúc thư mục, cấu hình cơ bản, các module phổ biến, giúp tạo dự án mới nhanh và đồng nhất.

### Tại sao cần Boilerplate?
- Tiết kiệm thời gian
- Chuẩn hoá kiến trúc
- Dễ bảo trì
- Dễ mở rộng

### NestJS Boilerplate
NestJS boilerplate thường bao gồm:
- Module structure
- Config env
- Database connection
- Authentication
- Logging
- Docker setup
## 🔹 Project Structure
- Cấu trúc thư mục
```c
src/
├─ app.controller.spec.ts
├─ app.controller.ts
├─ app.module.ts
├─ app.service.ts
├─ main.ts
```
| File | Chức năng |
|------|---------|
| main.ts | Entry point của ứng dụng |
| app.module.ts | Root module |
| app.controller.ts | Nhận request từ client |
| app.service.ts | Xử lý nghiệp vụ |
| app.controller.spec.ts | Unit test cho controller |

- Test file trong NestJS

NestJS sử dụng **Jest** cho testing.

Các file có đuôi `.spec.ts` là file test tự động.  
Ví dụ: `app.controller.spec.ts` dùng để test `app.controller.ts`.

Các file test này rất quan trọng khi xây dựng hệ thống CI/CD, giúp đảm bảo code không bị lỗi khi deploy.

## 🔹 Thiết kế Structure Boilerplate
- Structure
```c
src/
├─ modules/
│   ├─ auth/
│   │   ├─ controller/
|   │   │   ├─ auth.controller.ts
|   │   │   ├─ auth.controller.spec.ts
│   │   ├─ dto/
│   │   ├─ entity/
│   │   ├─ repository/
|   │   │   ├─ auth.repository.ts
│   │   ├─ service/
|   │   │   ├─ auth.service.spec.ts
|   │   │   ├─ auth.service.ts
│   │   ├─ auth.module.ts
│   ├─ users/
│   │   ├─ controller/
|   │   │   ├─ users.controller.ts
|   │   │   ├─ users.controller.spec.ts
│   │   ├─ dto/
│   │   ├─ entity/
│   │   ├─ repository/
|   │   │   ├─ users.repository.ts
│   │   ├─ service/
|   │   │   ├─ users.service.spec.ts
|   │   │   ├─ users.service.ts
│   │   ├─ users.module.ts
│   └─ ...
├─ common/
│   ├─ guards/
│   ├─ filters/
│   ├─ interceptors/
│   ├─ decorators/
│   └─ pipes/
├─ config/
│   ├─ app.config.ts
│   ├─ database.config.ts
│   └─ env.validation.ts
├─ app.module.ts
└─ main.ts
```
| Thư mục | Chức năng |
|--------|---------|
| modules | Business logic theo chức năng |
| common | Code dùng chung toàn hệ thống |
| config | Quản lý cấu hình & môi trường |
| main.ts | Entry point |


# 🧩 2 — ARCHITECTURE & MODULE DESIGN

## 🔹 Module Architecture trong NestJS
Mỗi module = 1 domain nghiệp vụ độc lập
Ví dụ: `auth`, `users`, `products`, `orders`, ...
## 🔹 Cấu trúc tiêu chuẩn của 1 module
```c
auth/
 ├─ controller/
 ├─ service/
 ├─ repository/
 ├─ dto/
 ├─ entity/
 └─ auth.module.ts
```
| File | Chức năng |
|--------|---------|
| Controller | Nhận request HTTP, trả response |
| Service | Xử lý nghiệp vụ |
| Repository | Giao tiếp Database |
| Dto | Validate + transform dữ liệu |
| Entity | Schema dữ liệu |
| Module | Kết nối các thành phần |


## 🔹 Luồng xử lý request

Client → Controller → DTO Validation → Service → Repository → Database → Response

## 🔹 Chi tiết từng tầng
### Controller – API Layer
```c
@Controller('auth')
export class AuthController {
  constructor(private readonly authService: AuthService) {}

  @Post('login')
  login(@Body() dto: LoginDto) {
    return this.authService.login(dto);
  }
}
```
### DTO – Validation Layer
```c
export class LoginDto {
  @IsEmail()
  email: string;

  @IsString()
  password: string;
}
```
### Service – Business Logic
```c
@Injectable()
export class AuthService {
  constructor(private readonly authRepo: AuthRepository) {}

  async login(dto: LoginDto) {
    const user = await this.authRepo.findByEmail(dto.email);
    // xử lý nghiệp vụ
    return user;
  }
}
```
### Repository – Data Access Layer
```c
@Injectable()
export class AuthRepository {
  constructor(@InjectModel(User.name) private userModel: Model<User>) {}

  findByEmail(email: string) {
    return this.userModel.findOne({ email });
  }
}
```
### Entity – Database Schema
```c
@Schema()
export class User {
  @Prop() email: string;
  @Prop() password: string;
}
```

# 🧩 3 — DOCKER & CONTAINERIZATION
## 🔹 Docker là gì?

| Thành phần | Vai trò                                   |
| ---------- | ----------------------------------------- |
| Docker     | Công cụ đóng gói ứng dụng thành container |
| Image      | Bản build của app                         |
| Container  | App đang chạy                             |
| Dockerfile | Kịch bản build image                      |

- Lợi ích:
 - Chạy giống nhau trên mọi máy
 - Không phụ thuộc môi trường local
 - Deploy nhanh, ổn định
 - Tạo nền tảng cho CI/CD
## 🔹 Tạo Dockerfile cho NestJS
- Tạo `Dockerfile` (đặt ở root project). Dockerfile là bản thiết kế mô tả cách tạo môi trường chạy app và cách khởi động app.
```c
# Sử dụng image Node.js phiên bản 18
FROM node:18-alpine

# Tạo và đặt thư mục làm việc trong container là /app
WORKDIR /app

# Copy package.json và package-lock.json từ máy host vào container
# Chỉ copy file này trước để tận dụng cache khi build
COPY package*.json ./

# Cài toàn bộ dependency của project vào container
RUN npm install

# Copy toàn bộ source code từ project vào container
COPY . .

# Build project NestJS (tạo thư mục dist/)
RUN npm run build

# Mở cổng 3000 trong container (NestJS mặc định chạy port này)
EXPOSE 3000

# Lệnh chạy khi container khởi động
# Chạy file đã build trong thư mục dist
CMD ["node", "dist/main.js"]

```
- Tạo `.dockerignore`. Mục đích không copy những file không cần thiết vào image → build nhanh, image nhẹ.
```c
node_modules
dist
.git
.env
```

- Build Docker Image
Tại root project:
```c
docker build -t backend-base .
```
Kiểm tra image:
```c
docker images
```
- Tạo file môi trường `.env`
```c
PORT=3000
DB_URI=mongodb://localhost:27017/backend-base
JWT_SECRET=supersecret
```
- Chạy backend bằng Docker
```c
docker run -p 3000:3000 --env-file .env backend-base
```

- Tạo file `docker-compose.yml`. NestJS chạy local – MongoDB chạy Docker

 ```c
version: '3.9'

services:
  mongo:
    image: mongo:6
    container_name: dev-mongo
    ports:
      - "27017:27017"
    volumes:
      - mongo_data:/data/db

volumes:
  mongo_data:

```

- Chạy toàn bộ hệ thống
```c
docker compose up --build
```

- Kết nối MongoDB Compass

```c
mongodb://localhost:27017
```


# 🧩 4 — CI/CD & DEPLOYMENT
## 🔹 CI/CD là gì?
- CI (Continuous Integration): Tự động build + test mỗi khi có code mới
- CD (Continuous Deployment): Tự động deploy code lên server
- Mục tiêu: Mỗi lần git push → hệ thống tự build – tự test – tự deploy mà không cần thao tác tay.

## 🔹 Kiến trúc triển khai

```c
Developer → GitHub → GitHub Actions (CI/CD)
                     │
                     ▼
                VPS (Production Server)
                     │
                     ▼
                 Docker Containers

```

# 🧩 5 — REVERSE PROXY & SSL (Traefik / Nginx)

## 🔹 Reverse Proxy là gì?
Reverse Proxy là một server trung gian đứng giữa người dùng và backend server.
- Vai trò:
 - Nhận request từ Internet
 - Quyết định request đó gửi đến backend nào
 - Áp dụng bảo mật, SSL, limit, redirect...
- Lợi ích:
 - Che giấu backend thật
 - Bảo mật hệ thống
 - Dễ mở rộng
 - Tăng hiệu suất
 - Quản lý SSL tập trung
## 🔹 Traefik là gì?
Traefik là một modern reverse proxy & load balancer, tối ưu cho containerized environment
- Điểm mạnh:
 - Tự động phát hiện container
 - Tự động tạo route
 - Tự động cấp SSL với Let's Encrypt
 - Cấu hình bằng Docker labels (rất tiện)
### Cấu trúc hoạt động của Traefik
Traefik hoạt động theo mô hình:
```c
Người dùng
   ↓
EntryPoints (80 / 443)
   ↓
Routers (đọc domain / path)
   ↓
Middlewares (bảo mật, redirect, limit…)
   ↓
Services (Node.js / API backend)
```
| Thành phần      | Vai trò                                           |
| --------------- | ------------------------------------------------- |
| **EntryPoints** | Cổng Traefik lắng nghe (HTTP, HTTPS)              |
| **Routers**     | Quyết định request đi về service nào              |
| **Services**    | Backend (Node.js, API…)                           |
| **Middlewares** | Các luật xử lý request (HTTPS, auth, rate limit…) |

## 🔹 Triển khai Traefik + NodeJS bằng Docker
Trong file `docker-compose.yml` 
```c
version: '3.9'

services:

  traefik:
    image: traefik:v2.10
    container_name: traefik
    command:
      - "--api.insecure=true"
      - "--providers.docker=true"
      - "--providers.docker.exposedbydefault=false"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.websecure.address=:443"
      - "--certificatesresolvers.myresolver.acme.tlschallenge=true"
      - "--certificatesresolvers.myresolver.acme.email=your_email@example.com"
      - "--certificatesresolvers.myresolver.acme.storage=/letsencrypt/acme.json"
    ports:
      - "8080:80"
      - "8443:443"
      - "9000:8080"
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
      - "./letsencrypt:/letsencrypt"

  app:
    image: node:18
    container_name: node-app
    working_dir: /usr/src/app
    volumes:
      - .:/usr/src/app
    command: npm run start:prod
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.node.rule=Host(`localhost`) || Host(`localhost:8080`)"
      - "traefik.http.routers.node.entrypoints=web"
      - "traefik.http.services.node.loadbalancer.server.port=3000"

  mongo:
    image: mongo:6
    container_name: dev-mongo
    volumes:
      - mongo_data:/data/db

volumes:
  mongo_data:

```

Khi truy cập `http://localhost:8080` Traefik sẽ xử lý:

- Nhận request tại EntryPoint web :80
- Router node khớp rule Host(localhost)
- Forward request tới Service node
- Service chuyển request vào NodeJS container port 3000

## 🔹 SSL là gì?
SSL là cơ chế mã hóa dữ liệu giữa client và server. Mọi dữ liệu (password, token, API, cookie…) đều được mã hóa,
ngăn chặn nghe lén, giả mạo, tấn công man-in-the-middle.
## 🔹 SSL trong kiến trúc hệ thống
```c
Browser
   ↓ HTTPS
Traefik (Terminate SSL)
   ↓ HTTP
NodeJS
   ↓
MongoDB
```
## 🔹 Cấu hình SSL trong dự án
- Traefik
```c
--certificatesresolvers.myresolver.acme.tlschallenge=true
--certificatesresolvers.myresolver.acme.email=your_email@example.com
--certificatesresolvers.myresolver.acme.storage=/letsencrypt/acme.json
```
- App
```c
- "traefik.http.routers.app-secure.entrypoints=websecure"
- "traefik.http.routers.app-secure.tls.certresolver=myresolver"
```
- Redirect HTTP → HTTPS
```c
- "traefik.http.middlewares.forcehttps.redirectscheme.scheme=https"
- "traefik.http.routers.app.middlewares=forcehttps"
```


# 🧩 6 — MONITORING & OPERATIONS


## 🔹 Monitoring là gì?
Monitoring là việc theo dõi toàn bộ hệ thống theo thời gian thực. Nếu không monitoring → hệ thống chết mà không biết.
| Thứ theo dõi     | Vì sao                 |
| ---------------- | ---------------------- |
| CPU / RAM / Disk | Biết server có quá tải |
| Network          | Phát hiện nghẽn        |
| Request / Error  | Biết backend đang lỗi  |
| Container health | Biết service nào chết  |
| Latency          | Biết hệ thống có chậm  |

🔹 Vì sao backend bắt buộc phải có Monitoring?

Khi hệ thống chạy thật:

- Người dùng có thể bị lỗi
- Server có thể quá tải
- Container có thể crash
- Database có thể nghẽn

## 🔹 Grafana là gì?
Grafana là nền tảng quan sát & dashboard phổ biến nhất hiện nay.
Nó hiển thị:

- CPU / RAM / Disk
- Network traffic
- Request, error rate
- Container status
- Database metrics

## 🔹 Coolify là gì?
Coolify là nền tảng quản lý & deploy server chạy trên VPS.

Coolify cho phép:

- Deploy project từ GitHub
- Quản lý Docker
- Theo dõi CPU / RAM / logs
- Auto SSL
- Restart container
- Quản lý database

## 🔹 Kiến trúc hệ thống có Monitoring
```c
Client
   ↓
Traefik / Nginx
   ↓
NodeJS App
   ↓
Database

↑
Prometheus → Grafana
↑
Coolify quản lý toàn bộ server
```
