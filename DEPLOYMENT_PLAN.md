
# Atlas ERP - Kế hoạch Triển khai Spring Cloud Config

## Tổng quan

Spring Cloud Config Server được triển khai để tập trung hóa cấu hình cho tất cả các service trong hệ thống Atlas ERP. Tất cả các file cấu hình được lưu trữ trong `atlas-erp-config-repo` và được phục vụ thông qua Config Server.

## Kiến trúc

```
┌─────────────────────────────────────────────────────────────┐
│                     Config Server                           │
│                    (Port: 8888)                              │
│                    config-server/                           │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                atlas-erp-config-repo/                       │
│  ┌─────────────────┐  ┌─────────────────┐                   │
│  │ application.properties │  │ Kafka + Redis │ (chung)      │
│  └─────────────────┘  └─────────────────┘                   │
│  ┌─────────────────┐  ┌─────────────────┐                   │
│  │ iam-service.properties    │ (database)   │               │
│  │ hr-service.properties      │ (database)   │               │
│  │ ...                      │ (database)   │               │
│  └─────────────────┘  ┌─────────────────┘                   │
└─────────────────────────────────────────────────────────────┘
                              │
        ┌─────────┬─────────┬─────────┬─────────┐
        ▼         ▼         ▼         ▼         ▼
    ┌───────┐ ┌───────┐ ┌───────┐ ┌───────┐ ┌───────┐
    │  IAM  │ │  HR   │ │Inventory│ │ Org  │ │ ...   │
    │Service│ │Service│ │Service │ │Service│ │Service│
    └───────┘ └───────┘ └───────┘ └───────┘ └───────┘
```

## Cấu hình Chung

### Redis (Dùng chung)
```
Host: helping-labrador-22731.upstash.io
Port: 6379
SSL: enabled
```

### Kafka (Dùng chung)
```
Bootstrap Servers: kafka-2d3930ef-student-337e.g.aivencloud.com:24786
Security Protocol: SSL
```

## Các bước Triển khai

### Bước 1: Khởi động Config Server

```bash
cd config-server
./mvnw spring-boot:run
```

Config Server sẽ chạy tại: `http://localhost:8888`

### Bước 2: Khởi động các Services

Sau khi Config Server khởi động, có thể khởi động các services:

```bash
# IAM Service
cd iam-service && ./mvnw spring-boot:run

# HR Service
cd hr-service && ./mvnw spring-boot:run

# Inventory Service
cd inventory-service && ./mvnw spring-boot:run

# Organization Service
cd organization-service && ./mvnw spring-boot:run

# Production Service
cd production-service && ./mvnw spring-boot:run

# Accounting Service
cd accounting-service && ./mvnw spring-boot:run

# Payroll Service
cd payroll-service && ./mvnw spring-boot:run

# Reporting Service
cd reporting-service && ./mvnw spring-boot:run

# Workflow Service
cd workflow-service && ./mvnw spring-boot:run

# Log Ingest Service
cd log-ingest-service && ./mvnw spring-boot:run
```

## Cấu hình trong từng Service

Mỗi service có file `application.properties` với nội dung tối thiểu:

```properties
spring.application.name=<service-name>
spring.config.import=optional:configserver:http://config-server:8888
```

## Danh sách Services và Cấu hình

| Service | Database | Server Port | Kafka Group |
|---------|----------|-------------|-------------|
| iam-service | iam_db | random | iam-service-group |
| hr-service | hr_db | 8080 | hr-service-group |
| inventory-service | inventory_db | random | inventory-service-group |
| organization-service | organization_db | 8080 | organization-service-group |
| production-service | production_db | random | production-service-group |
| accounting-service | accounting_db | random | accounting-service-group |
| payroll-service | payroll_db | random | payroll-service-group |
| reporting-service | reporting_db | random | reporting-service-group |
| workflow-service | workflow_db | 8080 | workflow-service-group |
| log-ingest-service | MongoDB | 8080 | log-ingest-service-group |

## Test Config Server

Kiểm tra Config Server hoạt động:

```bash
# Lấy cấu hình iam-service
curl http://localhost:8888/iam-service/default

# Lấy cấu hình hr-service
curl http://localhost:8888/hr-service/default
```

## Lưu ý quan trọng

1. **Thứ tự khởi động**: Config Server phải khởi động trước các services
2. **Cấu hình chung**: Redis và Kafka được định nghĩa trong `application.properties` của config-repo
3. **Cấu hình riêng**: Database và các cấu hình đặc thù của từng service nằm trong file `<service-name>.properties`
4. **MongoDB**: log-ingest-service sử dụng MongoDB thay vì PostgreSQL

## Troubleshooting

### Config Server không khởi động được
- Kiểm tra đường dẫn đến `atlas-erp-config-repo` trong `config-server/application.properties`
- Đảm bảo thư mục config-repo tồn tại và có quyền đọc

### Service không kết nối được đến Config Server
- Kiểm tra Config Server đang chạy
- Kiểm tra URL trong `spring.config.import`
- Kiểm tra network connectivity

### Cấu hình không được áp dụng
- Kiểm tra tên file config trong config-repo khớp với `spring.application.name`
- Kiểm tra label (branch) trong config-server


