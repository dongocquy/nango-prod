# Nango Production — Docker Compose

Triển khai [Nango](https://www.nango.dev) (v0.70.6) trên Docker sử dụng image `dongocquy/nango-server:hosted`.

## Yêu cầu

- Docker & Docker Compose v2
- RAM tối thiểu 2GB

## Cài đặt nhanh

```bash
# 1. Clone repo
git clone https://github.com/dongocquy/nango-prod.git
cd nango-prod

# 2. Tạo file .env từ template
cp .env.example .env

# 3. Sinh encryption key (bắt buộc)
sed -i "s|<GENERATE-WITH-OPENSSL>|$(openssl rand -base64 32)|" .env

# 4. Khởi động
docker compose -f docker-compose.prod.yml up -d
```

Truy cập:
- **Dashboard + API:** http://localhost:3003
- **Connect UI:** http://localhost:3009

## Cấu hình Cloudflare Tunnel

Để expose ra internet qua domain riêng, dùng Cloudflare Tunnel:

1. Vào [Cloudflare Zero Trust](https://one.dash.cloudflare.com) → Networks → Tunnels → Create a tunnel
2. Cài `cloudflared` trên máy chạy Docker:
   ```bash
   cloudflared tunnel run --token <TOKEN>
   ```
3. Thêm Public Hostname trong tunnel:
   | Subdomain | Domain | Type | URL |
   |-----------|--------|------|-----|
   | `api-nango` | `<domain>` | HTTP | `localhost:3003` |
   | `connect` | `<domain>` | HTTP | `localhost:3009` |
4. Cập nhật `.env`:
   ```ini
   NANGO_SERVER_URL=https://api-nango.<domain>
   NANGO_PUBLIC_SERVER_URL=https://api-nango.<domain>
   NANGO_PUBLIC_CONNECT_URL=https://connect.<domain>
   ```
5. Restart: `docker compose -f docker-compose.prod.yml up -d`

## Các lệnh thường dùng

```bash
# Xem logs
docker compose -f docker-compose.prod.yml logs -f nango-server

# Xem trạng thái
docker compose -f docker-compose.prod.yml ps

# Khởi động lại
docker compose -f docker-compose.prod.yml restart nango-server

# Dừng tất cả
docker compose -f docker-compose.prod.yml down

# Dừng và xóa data (cẩn thận!)
docker compose -f docker-compose.prod.yml down -v
rm -rf nango-data
```

## Cấu trúc

```
nango-prod/
├── .env.example              # Template biến môi trường
├── docker-compose.prod.yml   # Docker Compose production
├── .env                      # Biến môi trường của bạn (đã gitignore)
└── nango-data/               # Dữ liệu PostgreSQL (đã gitignore)
```

## Services

| Service | Port | Image |
|---------|------|-------|
| nango-db | 5432 | `postgres:16.0-alpine` |
| nango-redis | 6379 | `redis:7.2.4` |
| nango-server | 3003, 3009 | `dongocquy/nango-server:hosted` |

## Biến môi trường quan trọng

| Biến | Ý nghĩa | Mặc định |
|------|---------|----------|
| `NANGO_ENCRYPTION_KEY` | Key mã hóa database (bắt buộc) | — |
| `NANGO_SERVER_URL` | URL API server | `http://localhost:3003` |
| `NANGO_PUBLIC_SERVER_URL` | URL public cho webapp | `http://localhost:3003` |
| `NANGO_PUBLIC_CONNECT_URL` | URL cho Connect UI | `http://localhost:3009` |
| `NANGO_DASHBOARD_USERNAME` | Username basic auth (tùy chọn) | — |
| `NANGO_DASHBOARD_PASSWORD` | Password basic auth (tùy chọn) | — |
| `FLAG_AUTH_ENABLED` | Bật login/signup UI | `false` |

## Image

Image được build từ [nangohq/nango-server](https://hub.docker.com/r/nangohq/nango-server) và push lên Docker Hub cá nhân để đảm bảo availability.

```bash
docker pull dongocquy/nango-server:hosted
```
