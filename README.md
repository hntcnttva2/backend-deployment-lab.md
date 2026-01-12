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
- Ý nghĩa từng phần
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
- Ý nghĩa từng phần
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

# 🧩 3 — CORE INFRASTRUCTURE
