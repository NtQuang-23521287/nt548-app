# NT548 - DevOps Lab 3

## 1. Giới thiệu

Đây là ứng dụng microservice đơn giản được xây dựng bằng **Node.js** và **Express** nhằm phục vụ bài thực hành DevOps.

Ứng dụng trả về nội dung:

```text
NT548 DevOps Lab 3
```

Hệ thống được triển khai theo mô hình CI/CD với:
- GitHub
- Jenkins
- SonarQube
- Docker
- Docker Hub
- Kubernetes/K3s

---

# 2. Yêu cầu môi trường

Trước khi chạy project, cần cài đặt các công cụ sau:

| Công cụ | Phiên bản đề nghị |
|---|---|
| Node.js | >= 18 |
| npm | mới nhất |
| Docker | mới nhất |
| kubectl | mới nhất |
| K3s / Kubernetes | tùy chọn |
| Jenkins | tùy chọn |
| SonarQube | tùy chọn |

---

## 2.1. Kiểm tra môi trường

Kiểm tra Node.js:

```bash
node -v
npm -v
```

Kiểm tra Docker:

```bash
docker -v
```

Kiểm tra kubectl:

```bash
kubectl version --client
```

Kiểm tra Kubernetes/K3s:

```bash
kubectl get nodes
```

Nếu cluster hoạt động bình thường sẽ hiển thị trạng thái `Ready`.

---

# 3. Clone source code

Clone repository:

```bash
git clone <repository-url>
```

Di chuyển vào thư mục project:

```bash
cd nt548-app
```

---

# 4. Chạy ứng dụng ở môi trường local

## 4.1. Cài đặt dependency

```bash
npm install
```

---

## 4.2. Khởi động ứng dụng

```bash
npm start
```

Ứng dụng sẽ chạy tại:

```text
http://localhost:3000
```

---

## 4.3. Kiểm tra kết quả

Mở trình duyệt:

```text
http://localhost:3000
```

Hoặc dùng curl:

```bash
curl http://localhost:3000
```

Kết quả mong đợi:

```text
NT548 DevOps Lab 3
```

---

# 5. Chạy ứng dụng bằng Docker

## 5.1. Build Docker image

```bash
docker build -t nt548-app:local .
```

---

## 5.2. Chạy Docker container

```bash
docker run --rm -p 3000:3000 nt548-app:local
```

Ứng dụng sẽ được expose tại:

```text
http://localhost:3000
```

---

## 5.3. Kiểm tra kết quả

```bash
curl http://localhost:3000
```

Kết quả:

```text
NT548 DevOps Lab 3
```

---

# 6. Push Docker image lên Docker Hub

## 6.1. Đăng nhập Docker Hub

```bash
docker login
```

---

## 6.2. Tag image

```bash
docker tag nt548-app:local ntquang2012/nt548-app:latest
```

---

## 6.3. Push image

```bash
docker push ntquang2012/nt548-app:latest
```

Sau khi push thành công, image sẽ xuất hiện trên Docker Hub repository.

---

# 7. Triển khai ứng dụng lên Kubernetes/K3s

## 7.1. Cấu trúc thư mục Kubernetes

```text
k8s/
├── deployment.yaml
└── service.yaml
```

Trong đó:

- `deployment.yaml`
  - Tạo Deployment cho ứng dụng
  - Sử dụng image từ Docker Hub

- `service.yaml`
  - Expose ứng dụng bằng NodePort

---

## 7.2. Deploy ứng dụng

```bash
kubectl apply -f k8s/
```

---

## 7.3. Kiểm tra deployment

Kiểm tra Deployment:

```bash
kubectl get deployments
```

Kiểm tra Pod:

```bash
kubectl get pods
```

Pod cần ở trạng thái:

```text
Running
```

Ví dụ:

```text
NAME                          READY   STATUS    RESTARTS   AGE
nt548-app-xxxxxxxxxx-xxxxx    1/1     Running   0          1m
```

---

## 7.4. Kiểm tra Service

```bash
kubectl get svc
```

Kết quả mong đợi:

```text
NAME            TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
nt548-service   NodePort   10.x.x.x        <none>        3000:30080/TCP   1m
```

---

# 8. Kiểm tra kết quả triển khai

## 8.1. Nếu chạy local Kubernetes

Truy cập:

```text
http://localhost:30080
```

---

## 8.2. Nếu chạy trên server/K3s

Lấy IP node:

```bash
kubectl get nodes -o wide
```

Sau đó truy cập:

```text
http://<NODE_IP>:30080
```

Ví dụ:

```text
http://13.212.144.68:30080
```

---

## 8.3. Kết quả mong đợi

Trang web hiển thị:

```text
NT548 DevOps Lab 3
```

---

# 9. Kiểm tra log ứng dụng

Xem log Pod:

```bash
kubectl logs -l app=nt548-app
```

Kết quả mong đợi:

```text
Server running on port 3000
```

---

# 10. Debug Kubernetes

Nếu gặp lỗi, sử dụng các lệnh sau:

## Kiểm tra Deployment

```bash
kubectl describe deployment nt548-app
```

---

## Kiểm tra Service

```bash
kubectl describe svc nt548-service
```

---

## Kiểm tra Pod

```bash
kubectl describe pod -l app=nt548-app
```

---

# 11. Jenkins CI/CD Pipeline

Project sử dụng Jenkins Pipeline để tự động hóa quy trình CI/CD.

Pipeline gồm các stage:

1. Checkout source code
2. SonarQube Scan
3. Docker Build
4. Docker Push
5. Deploy to K3s

---

## 11.1. Yêu cầu trước khi chạy Jenkins Pipeline

Cần cấu hình:

### Jenkins Credentials

| Credential ID | Loại |
|---|---|
| sonar-token | Secret text |
| dockerhub-creds | Username with password |

---

### Jenkins Tools

Jenkins container cần có:
- Docker
- kubectl
- SonarScanner

SonarScanner cần nằm tại:

```text
/opt/sonar-scanner/bin/sonar-scanner
```

---

## 11.2. Chạy Pipeline

Sau khi push code lên GitHub:

```bash
git add .
git commit -m "Update source code"
git push
```

Jenkins sẽ tự động:
- pull source code
- scan SonarQube
- build Docker image
- push image lên Docker Hub
- deploy ứng dụng lên K3s

---

# 12. Cấu trúc project

```text
.
├── app.js
├── Dockerfile
├── Jenkinsfile
├── package.json
└── k8s
    ├── deployment.yaml
    └── service.yaml
```

---

# 13. Ý nghĩa các file

| File | Chức năng |
|---|---|
| app.js | Source code ứng dụng |
| package.json | Quản lý dependency Node.js |
| Dockerfile | Build Docker image |
| Jenkinsfile | Định nghĩa CI/CD pipeline |
| deployment.yaml | Kubernetes Deployment |
| service.yaml | Kubernetes Service |

---

# 14. Dọn dẹp tài nguyên

## Xóa Kubernetes resources

```bash
kubectl delete -f k8s/
```

---

## Xóa Docker image local

```bash
docker rmi nt548-app:local
```

---

# 15. Công nghệ sử dụng

| Công nghệ | Vai trò |
|---|---|
| Node.js | Backend application |
| Express | Web framework |
| Docker | Containerization |
| Docker Hub | Container Registry |
| Jenkins | CI/CD |
| SonarQube | Static Code Analysis |
| Kubernetes/K3s | Container Orchestration |

```