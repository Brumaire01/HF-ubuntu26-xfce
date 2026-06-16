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
    && apt-get install -y --no-install-recommends curl wget ca-certificates \
    && apt-get autoremove -y \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# ==================== 安装 Node.js 24 ====================
RUN curl -fsSL https://deb.nodesource.com/setup_24.x | bash - \
    && apt-get install -y nodejs \
    && rm -rf /var/lib/apt/lists/*

# ==================== 安装 Golang ====================
ENV GO_VERSION=1.24.0
ENV GOROOT=/usr/local/go
ENV GOPATH=/root/go
ENV PATH=$GOROOT/bin:$GOPATH/bin:$PATH

RUN wget -qO /tmp/go.tar.gz https://go.dev/dl/go${GO_VERSION}.linux-amd64.tar.gz \
    && tar -C /usr/local -xzf /tmp/go.tar.gz \
    && rm /tmp/go.tar.gz \
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
        
# ===== 首页 =====
RUN mkdir -p /var/www/html
COPY index.html /var/www/html/index.html

# 7860 端口的 nginx server 配置
COPY proxy-server.conf /defaults/proxy-server.conf

# 覆盖 init-nginx 脚本，在原始逻辑后追加 proxy server 配置到 nginx 主配置中
COPY init-nginx-run /etc/s6-overlay/s6-rc.d/init-nginx/run
RUN chmod +x /etc/s6-overlay/s6-rc.d/init-nginx/run

EXPOSE 7860
