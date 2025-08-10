
# Workshop: Kubernetes Multi-Cluster Management với Cluster API

## Executive Summary

Workshop này hướng dẫn triển khai và quản lý đa cụm Kubernetes sử dụng **Cluster API** (CAPI), tập trung vào tự động hóa vòng đời cụm và triển khai GitOps với Argo CD. Cluster API giúp tự động hóa việc tạo, nâng cấp, và xóa cụm Kubernetes. Argo CD đảm bảo triển khai ứng dụng nhất quán và có thể kiểm tra được thông qua Git.

**Tổng quan giải pháp:**

- **Cluster API**: Tự động hóa vòng đời cụm Kubernetes trên AWS (EKS).
- **Argo CD**: Triển khai ứng dụng theo GitOps.
- **AWS EKS**: Cung cấp cụm Kubernetes được quản lý.

**Lợi ích:**

- Giảm thời gian quản lý cụm so với phương pháp thủ công.
- Tăng cường tính nhất quán và kiểm soát triển khai ứng dụng.
- Nâng cao kỹ năng quản lý đa cụm Kubernetes và DevOps.

**Chi phí dự kiến:**
- **Hạ tầng AWS**: $1.25
- **Thời gian**: 3h.

**Kết quả mong đợi:**

- Hiểu rõ cách sử dụng Cluster API để quản lý cụm Kubernetes.
- Thành thạo triển khai GitOps với Argo CD.
- Biết cách cấu hình autoscaling cho EC2 và EKS.
- Quản lý kubeconfig và truy cập cụm từ cụm quản lý.
- Áp dụng quy trình CI/CD cho hạ tầng Kubernetes.

---

# 1. Problem Statement

## Current Situation

Việc triển khai và quản lý đa cụm Kubernetes đang trở thành nhu cầu thiết yếu trong các tổ chức hiện đại, đặc biệt khi ứng dụng được phân phối trên nhiều vùng địa lý hoặc môi trường khác nhau. Tuy nhiên, các phương pháp quản lý thủ công thường phức tạp, tốn thời gian, dễ xảy ra lỗi cấu hình, và khó mở rộng.

Cluster API cung cấp một cách tiếp cận tự động hóa vòng đời cụm Kubernetes thông qua các manifest khai báo. Khi kết hợp với Argo CD, các cụm và ứng dụng có thể được triển khai nhất quán, kiểm soát được, và dễ dàng rollback thông qua GitOps.

## Key Challenges

- **Quản lý vòng đời cụm**: Việc tạo, nâng cấp, và xóa cụm Kubernetes thủ công mất nhiều thời gian và dễ sai sót.
- **Thiếu tự động hóa triển khai**: Không có quy trình GitOps rõ ràng dẫn đến cấu hình không nhất quán.
- **Khó mở rộng**: Việc mở rộng cụm theo nhu cầu thực tế đòi hỏi thao tác thủ công và thiếu khả năng tự điều chỉnh.
- **Giám sát và rollback hạn chế**: Không có hệ thống theo dõi trạng thái cụm và ứng dụng một cách tập trung.
- **Chi phí vận hành cao**: Thiếu tối ưu tài nguyên dẫn đến lãng phí chi phí.

## Stakeholder Impact

- **Lập trình viên**: Gặp khó khăn khi triển khai ứng dụng trên nhiều cụm với cấu hình khác nhau.
- **Kỹ sư DevOps**: Thiếu công cụ để tự động hóa và giám sát cụm một cách hiệu quả.
- **Quản lý CNTT**: Lo ngại về chi phí, bảo mật, và khả năng kiểm soát hệ thống phân tán.
- **Doanh nghiệp**: Rủi ro về hiệu suất, bảo mật và chi phí nếu không có giải pháp phù hợp.

## Business Consequences

- Tăng chi phí vận hành do thiếu tự động hóa và tối ưu tài nguyên.
- Giảm hiệu suất triển khai và thời gian đưa sản phẩm ra thị trường.
- Tăng rủi ro bảo mật và vi phạm tuân thủ.
- Giảm khả năng cạnh tranh trong môi trường cloud-native.

---

# 2. Solution Architecture

## Architecture Overview

- **Cluster API**: Tự động hóa vòng đời cụm Kubernetes qua manifest khai báo.            
- **Argo CD**: Triển khai ứng dụng và hạ tầng theo GitOps.                            
- **Kind**: Cụm Kubernetes cục bộ dùng làm cụm quản lý.                            
- **AWS EC2/EKS**: Nơi triển khai các workload cluster.                                   
- **Git Repository**: Lưu trữ manifest cho cụm và ứng dụng, là nguồn duy nhất cho Argo CD.  

## AWS Services Used

- **Amazon EC2**: Triển khai cụm Kubernetes tùy chỉnh với Cluster API.
- **Amazon EKS**: Cụm Kubernetes được quản lý, tích hợp với Cluster API.
- **IAM & CloudFormation**: Cấp quyền cho Cluster API tương tác với AWS.
- **VPC, Subnet, Security Group**: Cấu hình mạng cho các cụm.

## Component Design
### 1. **Management Cluster (Kind)**

- Được khởi tạo cục bộ bằng Kind.
- Cài đặt Cluster API với provider AWS.
- Cài đặt Argo CD để điều phối triển khai GitOps.

### 2. **Workload Cluster (EC2 & EKS)**

- EC2: Tạo cụm với manifest YAML, cấu hình số lượng node, SSH key, v.v.
- EKS: Sử dụng flavor `eks-managedmachinepool` để triển khai node group.
- Cấu hình autoscaling cho cả EC2 và EKS để tối ưu tài nguyên.

### 3. **Argo CD GitOps Flow**

- Argo CD được cấp quyền `cluster-admin`.
- Manifest ứng dụng và cụm được lưu trong Git.
- Argo CD tự động đồng bộ trạng thái từ Git vào cụm.

### 4. **CI/CD cho hạ tầng**

- Mỗi thay đổi trong Git (ví dụ nâng cấp version Kubernetes) sẽ được Argo CD áp dụng tự động.
- Có thể tích hợp thêm các pipeline CI như GitHub Actions để kiểm tra manifest trước khi merge.

## Security Architecture

Giải pháp đảm bảo tính bảo mật ở cả cấp độ cụm và ứng dụng thông qua:

- **Quản lý IAM tập trung**: Sử dụng CloudFormation để tạo IAM roles cho Cluster API tương tác với AWS.
- **Phân quyền Argo CD**: Gán quyền `cluster-admin` cho Argo CD để triển khai tài nguyên một cách có kiểm soát.
- **RBAC cho Autoscaler**: Cấu hình Role và RoleBinding để giới hạn quyền truy cập của Cluster Autoscaler.
- **Bảo mật SSH**: Tạo SSH key riêng cho từng vùng triển khai EC2/EKS, tránh dùng chung key.
- **Kubeconfig bảo mật**: Kubeconfig của EKS được lưu dưới dạng secret trong cụm quản lý, không lộ ra ngoài.

## Scalability Design
### Tự động mở rộng cụm

Giải pháp hỗ trợ mở rộng linh hoạt theo nhu cầu thực tế:

#### EC2 Cluster

- Sử dụng **Cluster Autoscaler** với cấu hình:
  - Tự động scale từ 1 đến 10 node.
  - Cân bằng tải và tránh downtime.
- Triển khai qua Argo CD để đảm bảo đồng bộ với Git.

#### EKS Cluster

- Dùng **Managed Node Group** của AWS:
  - Cấu hình minSize và maxSize trong manifest.
  - AWS tự động điều chỉnh số lượng node.

### Nâng cấp cụm

- Cập nhật version Kubernetes trong manifest.
- Argo CD tự động đồng bộ và triển khai rolling update.
- Đảm bảo không gián đoạn dịch vụ trong quá trình nâng cấp.

### Architecture Diagram
![architecture-diagram](./architecture-diagram.png)

---

# 3. Technical Implementation

## Implementation Phases

1. **Khởi tạo cụm quản lý**:
   - Tạo cụm Kind.
   - Cài đặt Cluster API với provider AWS.
   - Thiết lập IAM và biến môi trường.

2. **Tạo cụm workload**:
   - Tạo manifest cho cụm EC2 và EKS.
   - Cài đặt Argo CD trên cụm quản lý.
   - Triển khai cụm qua Argo CD bằng manifest kiểu `Application`.

3. **Nâng cấp và mở rộng cụm**:
   - Cập nhật phiên bản Kubernetes trong manifest.
   - Cấu hình Cluster Autoscaler cho EC2 và EKS.

---

# 4. Timeline & Milestones

## Key Milestones

1. Cụm quản lý Cluster API được khởi tạo.
2. Cụm EC2 và EKS được triển khai qua Argo CD.
3. Cluster Autoscaler hoạt động trên các cụm.
4. Nâng cấp cụm thành công qua GitOps.

---

## 5. Budget Estimation

### Infrastructure Costs

| Thành phần        | Dịch vụ AWS       | Chi phí ước tính |
|------------------|-------------------|------------------|
| EKS              | 3 nodes `t3.medium` | ~$0.37           |
| EC2              | 6 nodes `t3.medium` | ~$0.75           |
| Management EC2   | 1 node `t3.medium`  | ~$0.12           |
| IAM, VPC         | Included            | ~$0.00           |


**Tổng chi phí workshop 3 giờ: ~**🟩 **$1.25 USD**

---

## 6. Risk Assessment

### Risk Matrix

| Rủi ro                        | Tác động     | Xác suất     | Giảm thiểu                                      |
|------------------------------|--------------|--------------|-------------------------------------------------|
| Management cluster lỗi       | Cao          | Thấp         | Kiểm tra cấu hình, dùng AMI backup              |
| Giao tiếp liên cụm thất bại  | Cao          | Trung bình   | Kiểm tra Cilium và Security Groups              |
| Lỗi chính sách Kyverno       | Trung bình   | Trung bình   | Áp dụng trước trong môi trường test             |
| Thiếu thời gian triển khai   | Trung bình   | Trung bình   | Dự phòng 30 phút, chuẩn bị tài liệu bổ sung     |


### Contingency Plans

- **Khôi phục cluster** từ S3 hoặc AMI backup.
- **Xóa và tái tạo workload clusters** bằng `clusterctl`.
- **Sử dụng cấu hình mẫu** từ tài liệu chính thức nếu gặp lỗi.
- **Cung cấp tài liệu tự học** nếu workshop vượt thời lượng.

---

## 7. Expected Outcomes

### Success Metrics

- Cụm quản lý và workload được triển khai thành công.
- Argo CD đồng bộ hóa trạng thái cụm và ứng dụng.
- Cilium Cluster Mesh hoạt động ổn định giữa các cụm.
- Kyverno áp dụng chính sách bảo mật đúng như kỳ vọng.
- API phản hồi < 200ms trong bài test hiệu năng.

### Business Benefits

- **Ngắn hạn (0–6 tháng)**: Tự động hóa quản lý cụm, giảm lỗi cấu hình.
- **Trung hạn (6–18 tháng)**: Tối ưu chi phí, hỗ trợ mở rộng linh hoạt.
- **Dài hạn (18+ tháng)**: Tăng khả năng cạnh tranh với chiến lược cloud-native.

### Technical Improvements

- Thành thạo Cluster API, Argo CD, Cilium, Karmada, Kyverno.
- Hiểu rõ kiến trúc đa cụm và cách triển khai GitOps.
- Áp dụng chính sách bảo mật và autoscaling hiệu quả.

### Long-term Value

- Người tham gia có thể áp dụng vào dự án thực tế.
- Doanh nghiệp xây dựng được nền tảng Kubernetes đa cụm đáng tin cậy, tiết kiệm chi phí.

---

# Appendices

## A. Technical Specifications
- **EC2 Cluster**: 3 control plane nodes + 3 worker nodes (`t3.medium`)  
- **EKS Cluster**: Kubernetes v1.31, sử dụng flavor `eks-managedmachinepool`
- **Cluster API**: Phiên bản mới nhất, provider AWS.
- **Argo CD**: Cài đặt trên cụm quản lý (Kind), cấp quyền `cluster-admin` 


## B. Cost Calculations
| Hạng mục              | Chi phí ước tính     |
|------------------------|----------------------|
| **AWS EC2 Instances**  | ~$1.00 cho 3h sử dụng `t3.medium` |
| **EKS Cluster**        | ~$0.25 cho 3h sử dụng node group nhỏ |
| **Tổng cộng**          | **~$1.25** cho toàn bộ workshop |


## C. Architecture Diagrams

![architecture-diagram](./architecture-diagram.png)

## D. References

- [Cluster API Documentation](https://cluster-api.sigs.k8s.io/)
- [AWS EKS Documentation](https://docs.aws.amazon.com/eks/)
- [Argo CD Documentation](https://argo-cd.readthedocs.io/)
- [Docker Documentation](https://docs.docker.com/)
- [Kind Documentation](https://kind.sigs.k8s.io/)
- [Kubernetes Documentation](https://kubernetes.io/docs/home/)
