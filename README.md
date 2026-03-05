# Atlas ERP Config Repository

## Giới thiệu

Đây là repository chứa tập trung tất cả các file cấu hình (configuration) cho hệ thống Atlas ERP. Tất cả các service sẽ lấy cấu hình từ đây thông qua Spring Cloud Config Server.

## Cấu trúc

```
atlas-erp-config-repo/
├── application.properties              # Cấu hình chung cho tất cả services (Redis + Kafka)
├── iam-service.properties              # Cấu hình riêng cho IAM Service
├── hr-service.properties                # Cấu hình riêng cho HR Service
├── inventory-service.properties         # Cấu hình riêng cho Inventory Service
├── organization-service.properties      # Cấu hình riêng cho Organization Service
├── production-service.properties        # Cấu hình riêng cho Production Service
├── accounting-service.properties        # Cấu hình riêng cho Accounting Service
├── payroll-service.properties           # Cấu hình riêng cho Payroll Service
├── reporting-service.properties         # Cấu hình riêng cho Reporting Service
├── workflow-service.properties          # Cấu hình riêng cho Workflow Service
└── log-ingest-service.properties        # Cấu hình riêng cho Log Ingest Service
```

## Cấu hình chung (application.properties)

### Redis - Dùng chung cho tất cả services

```properties
spring.data.redis.host=helping-labrador-22731.upstash.io
spring.data.redis.port=6379
spring.data.redis.password=<password>
spring.data.redis.ssl.enabled=true
```

### Kafka - Dùng chung cho tất cả services

```properties
spring.kafka.bootstrap-servers=kafka-2d3930ef-student-337e.g.aivencloud.com:24786
spring.kafka.properties.security.protocol=SSL
spring.kafka.properties.ssl.keystore.location=C:/kafka-cert/kafka.keystore.p12
spring.kafka.properties.ssl.keystore.password=changeit
spring.kafka.properties.ssl.keystore.type=PKCS12
spring.kafka.properties.ssl.truststore.location=C:/kafka-cert/kafka.truststore.jks
spring.kafka.properties.ssl.truststore.password=changeit
spring.kafka.properties.ssl.truststore.type=PKCS12

# Producer
spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer
spring.kafka.producer.value-serializer=org.apache.kafka.common.serialization.StringSerializer
spring.kafka.producer.acks=all
spring.kafka.producer.retries=3

# Consumer
spring.kafka.consumer.auto-offset-reset=earliest
spring.kafka.consumer.key-deserializer=org.apache.kafka.common.serialization.StringDeserializer
spring.kafka.consumer.value-deserializer=org.apache.kafka.common.serialization.StringDeserializer

# Disable SASL
spring.kafka.properties.sasl.mechanism=
spring.kafka.properties.sasl.jaas.config=
```

## Cách sử dụng

### 1. Khởi động Config Server

```bash
cd config-server
./mvnw spring-boot:run
```

Config Server sẽ chạy tại: http://localhost:8888

### 2. Các service kết nối đến Config Server

Mỗi service cần có cấu hình sau trong `application.properties`:

```properties
spring.application.name=<service-name>
spring.config.import=optional:configserver:http://config-server:8888
```

### 3. Truy cấp cấu hình

- Cấu hình default: `http://localhost:8888/<service-name>/default`
- Cấu hình dev: `http://localhost:8888/<service-name>/dev`
- Cấu hình prod: `http://localhost:8888/<service-name>/prod`

## Danh sách Services

| Service | Database | Port | Kafka Group |
|---------|----------|------|-------------|
| iam-service | iam_db | - | iam-service-group |
| hr-service | hr_db | 8080 | hr-service-group |
| inventory-service | inventory_db | - | inventory-service-group |
| organization-service | organization_db | 8080 | organization-service-group |
| production-service | production_db | - | production-service-group |
| accounting-service | accounting_db | - | accounting-service-group |
| payroll-service | payroll_db | - | payroll-service-group |
| reporting-service | reporting_db | - | reporting-service-group |
| workflow-service | workflow_db | 8080 | workflow-service-group |
| log-ingest-service | MongoDB | 8080 | log-ingest-service-group |

## Ghi chú

- Cấu hình Redis và Kafka được đặt trong `application.properties` để dùng chung cho tất cả services
- Mỗi service có file cấu hình riêng chứa các thông tin đặc thù (database name, server port, kafka group-id)
- Các service sử dụng `@Value` hoặc `@ConfigurationProperties` để inject các giá trị cấu hình

