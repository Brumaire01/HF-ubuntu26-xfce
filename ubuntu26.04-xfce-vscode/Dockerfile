FROM lscr.io/linuxserver/webtop:ubuntu-xfce

# ===== 中文语言支持 & 五笔拼音输入法 =====
ENV LANG=zh_CN.UTF-8 \
    LANGUAGE=zh_CN:zh \
    LC_ALL=zh_CN.UTF-8

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    # 中文语言包
    language-pack-zh-hans \
    language-pack-zh-hans-base \
    # 中文字体
    fonts-noto-cjk \
    fonts-wqy-microhei \
    fonts-wqy-zenhei \
    # 输入法框架 + fcitx5 五笔拼音
    fcitx5 \
    fcitx5-frontend-gtk3 \
    fcitx5-frontend-gtk2 \
    fcitx5-frontend-qt5 \
    fcitx5-chinese-addons \
    fcitx5-table-wubi-large \
    fcitx5-table-wubi98-pinyin \
    fcitx5-config-qt \
    # 中文 locale 支持
    locales \
    && locale-gen zh_CN.UTF-8 \
    && update-locale LANG=zh_CN.UTF-8 LC_ALL=zh_CN.UTF-8 \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# 配置 fcitx5 环境变量（容器内所有用户生效）
RUN echo 'export GTK_IM_MODULE=fcitx5' >> /etc/profile.d/fcitx5.sh && \
    echo 'export QT_IM_MODULE=fcitx5' >> /etc/profile.d/fcitx5.sh && \
    echo 'export XMODIFIERS=@im=fcitx5' >> /etc/profile.d/fcitx5.sh && \
    echo 'export INPUT_METHOD=fcitx5' >> /etc/profile.d/fcitx5.sh && \
    echo 'export LANG=zh_CN.UTF-8' >> /etc/profile.d/fcitx5.sh && \
    chmod +x /etc/profile.d/fcitx5.sh

# 确保环境变量也在 /etc/environment 中生效
RUN echo 'LANG=zh_CN.UTF-8' >> /etc/environment && \
    echo 'LANGUAGE=zh_CN:zh' >> /etc/environment && \
    echo 'LC_ALL=zh_CN.UTF-8' >> /etc/environment && \
    echo 'GTK_IM_MODULE=fcitx5' >> /etc/environment && \
    echo 'QT_IM_MODULE=fcitx5' >> /etc/environment && \
    echo 'XMODIFIERS=@im=fcitx5' >> /etc/environment

# 创建 fcitx5 默认配置目录及启动配置（五笔拼音）
RUN mkdir -p /etc/skel/.config/fcitx5/conf && \
    printf '[Groups/0]\nName=默认\nDefault Layout=us\nDefaultIM=pinyin\n\n[Groups/0/Items/0]\nName=keyboard-us\n\n[Groups/0/Items/1]\nName=pinyin\n\n[GroupOrder]\n0=默认\n' > /etc/skel/.config/fcitx5/profile && \
    printf '[Hotkey]\nTriggerKey=CTRL_SPACE\n\n[Behavior]\nActiveByDefault=True\n' > /etc/skel/.config/fcitx5/config

# 更新系统并安装基础工具
RUN apt-get update \
    && apt-get install -y --no-install-recommends curl wget dante-server unzip ca-certificates \
    # SSH 服务
    openssh-server \
    # debconf 依赖，消除 dpkg 安装时的警告
    dialog apt-utils whiptail \
    # VS Code / Electron 运行时依赖
    libxshmfence1 libnss3 libatk1.0-0t64 libatk-bridge2.0-0t64 \
    libdrm2 libxkbcommon0 libxcomposite1 libxdamage1 libxfixes3 \
    libxrandr2 libgbm1 libpango-1.0-0 libcairo2 libasound2t64 \
    libgtk-3-0t64 libx11-xcb1 libxcb-dri3-0 \
    && apt-get autoremove -y \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# ==================== 安装 Node.js 24 ====================
RUN curl -fsSL https://deb.nodesource.com/setup_24.x | bash - \
    && apt-get install -y nodejs \
    && npm config set registry https://mirrors.cloud.tencent.com/npm/ \
    && rm -rf /var/lib/apt/lists/*

# ==================== 安装 Golang ====================
ENV GO_VERSION=1.24.0
ENV GOROOT=/usr/local/go
ENV GOPATH=/root/go
ENV PATH=$GOROOT/bin:$GOPATH/bin:$PATH

RUN wget -qO /tmp/go.tar.gz https://go.dev/dl/go${GO_VERSION}.linux-amd64.tar.gz \
    && tar -C /usr/local -xzf /tmp/go.tar.gz \
    && rm /tmp/go.tar.gz \
    && go env -w GOPROXY=https://goproxy.cn,direct \
    && go env -w GO111MODULE=on    

# ==================== 安装 Docker CLI ====================
RUN curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg \
    && echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list \
    && apt-get update \
    && apt-get install -y docker-ce-cli \
    && rm -rf /var/lib/apt/lists/*

# ==================== 安装 docker-compose ====================
RUN curl -SL "https://github.com/docker/compose/releases/latest/download/docker-compose-linux-x86_64" \
    -o /usr/local/bin/docker-compose \
    && chmod +x /usr/local/bin/docker-compose

# ==================== 安装 easytier =========================
ENV NETWORK_SECRET="admin@123easytier"

# ==================== SSH 服务配置 ====================
# root 密码自定义（默认 root）
ENV ROOT_PASSWORD="root"

# 配置 sshd：允许 root 登录、密码认证、公钥认证
RUN sed -i \
    -e 's/^#\?PermitRootLogin.*/PermitRootLogin yes/' \
    -e 's/^#\?PasswordAuthentication.*/PasswordAuthentication yes/' \
    -e 's/^#\?PubkeyAuthentication.*/PubkeyAuthentication yes/' \
    /etc/ssh/sshd_config && \
    # 预生成 host keys
    ssh-keygen -A && \
    # 配置 root 免密登录公钥
    mkdir -p /root/.ssh && chmod 700 /root/.ssh && \
    echo 'ssh-rsa You PubkeyAuthentication' > /root/.ssh/authorized_keys && \
    chmod 600 /root/.ssh/authorized_keys

RUN wget -q https://github.com/EasyTier/EasyTier/releases/download/v2.2.4/easytier-linux-x86_64-v2.2.4.zip && \
    unzip easytier-linux-x86_64-v2.2.4.zip && \
    mv easytier-linux-x86_64/easytier-core /usr/local/bin/easytier-core && \
    chmod +x /usr/local/bin/easytier-core && \
    rm -rf easytier-linux-x86_64 easytier-linux-x86_64-v2.2.4.zip

COPY danted.conf /etc/danted.conf

# ==================== 安装 VS Code ====================
# ENV CODE_VERSION=1.124.2-1781225536
# RUN wget -qO /tmp/code.deb "https://update.code.visualstudio.com/${CODE_VERSION}/linux-deb-x64/stable" \
RUN wget -qO /tmp/code.deb "https://vscode.download.prss.microsoft.com/dbazure/download/stable/93cfdd489c3b228840d0f86ec77c3636277c93ea/code_1.125.0-1781601565_amd64.deb" \
    && DEBIAN_FRONTEND=noninteractive dpkg -i /tmp/code.deb || true \
    && apt-get update \
    && apt-get install -f -y --no-install-recommends \
    && rm /tmp/code.deb \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# 创建桌面快捷方式（webtop 桌面可见）
RUN mkdir -p /usr/share/applications && \
    cat > /usr/share/applications/code.desktop << 'EOF'
[Desktop Entry]
Name=Visual Studio Code
Comment=Code Editing. Redefined.
GenericName=Text Editor
Exec=code --no-sandbox --disable-gpu --user-data-dir=/root/.vscode-root %F
Icon=code
Type=Application
StartupNotify=false
Categories=Development;IDE;
MimeType=text/plain;
EOF

# 将快捷方式放入骨架目录，webtop 首次启动时自动复制到用户桌面
RUN mkdir -p /etc/skel/Desktop && \
    cp /usr/share/applications/code.desktop /etc/skel/Desktop/ && \
    chmod +x /etc/skel/Desktop/code.desktop

# 设置 VS Code alias，容器内终端直接输入 code --no-sandbox 启动
RUN echo 'alias code="code --no-sandbox --disable-gpu --user-data-dir=/root/.vscode-root"' >> /etc/skel/.bashrc && \
    echo 'alias code="code --no-sandbox --disable-gpu --user-data-dir=/root/.vscode-root"' >> /etc/profile.d/vscode.sh && \
    chmod +x /etc/profile.d/vscode.sh
    
# ===== 首页 =====
RUN mkdir -p /var/www/html
COPY index.html /var/www/html/index.html

# 7860 端口的 nginx server 配置
COPY proxy-server.conf /defaults/proxy-server.conf

# 覆盖 init-nginx 脚本，在原始逻辑后追加 proxy server 配置到 nginx 主配置中
COPY init-nginx-run /etc/s6-overlay/s6-rc.d/init-nginx/run
RUN chmod +x /etc/s6-overlay/s6-rc.d/init-nginx/run

# easytier + dante 自动启动服务（s6-overlay v3 longrun 服务）
COPY easytier-run /etc/s6-overlay/s6-rc.d/easytier/run
RUN chmod +x /etc/s6-overlay/s6-rc.d/easytier/run && \
    printf 'longrun\n' > /etc/s6-overlay/s6-rc.d/easytier/type && \
    # 将 easytier 加入 user bundle，随容器自动启动
    mkdir -p /etc/s6-overlay/s6-rc.d/user/contents.d && \
    touch /etc/s6-overlay/s6-rc.d/user/contents.d/easytier

# sshd 自动启动服务（s6-overlay v3 longrun 服务）
COPY sshd-run /etc/s6-overlay/s6-rc.d/sshd/run
RUN chmod +x /etc/s6-overlay/s6-rc.d/sshd/run && \
    printf 'longrun\n' > /etc/s6-overlay/s6-rc.d/sshd/type && \
    touch /etc/s6-overlay/s6-rc.d/user/contents.d/sshd

EXPOSE 7860 22
