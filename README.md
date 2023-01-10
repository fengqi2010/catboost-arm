# catboost-arm

## 编译catboost

### 3.7.4
```bash
# 需要在x86_64架构的机器上执行，在aarch64执行会失败
# docker挂载目录
mkdir wheel_out
cd wheel_out
# 需要指定platform
docker run -it --platform=linux/amd64 -v $(pwd)/:/wheel_out -e PYTHON_VSERSION=3.7.4 -e TAG=v1.1.1 debian:11
# 以上命令在容器外执行

# 以下命令在容器内执行
# 替换镜像源
sed -i -E 's/(deb|security).debian.org/mirrors.tuna.tsinghua.edu.cn/g' /etc/apt/sources.list
# sed -i -E 's/(deb|security).debian.org/mirrors.huaweicloud.com/g' /etc/apt/sources.list
apt update && \
	apt install -y make build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev \
	wget curl llvm libncursesw5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev git
# 安装pyenv
curl -s -S -L https://raw.githubusercontent.com/pyenv/pyenv-installer/master/bin/pyenv-installer | bash
# 使用代理下载catboost
# curl -s -S -L https://ghproxy.com/https://raw.githubusercontent.com/pyenv/pyenv-installer/master/bin/pyenv-installer | bash
echo 'export PATH="$HOME/.pyenv/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(pyenv init -)"' >> ~/.bashrc
echo 'eval "$(pyenv virtualenv-init -)"' >> ~/.bashrc
exec "$SHELL"
pyenv install -v $PYTHON_VSERSION
pyenv virtualenv $PYTHON_VSERSION catboost_build_$PYTHON_VSERSION
pyenv activate catboost_build_$PYTHON_VSERSION
# npm nodejs
curl -sS -L https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list
apt update
apt install -y nodejs npm
npm config set registry https://registry.npmmirror.com
npm install rimraf yarn -g
# 下载catboost
git clone -b $TAG https://github.com/catboost/catboost.git
# 使用代理下载catboost
# git clone -b v1.1.1 https://ghproxy.com/https://github.com/catboost/catboost.git
cd catboost
curl -sS -L https://ghproxy.com/https://raw.githubusercontent.com/fengqi5866/catboost-arm/main/arm64-v1.1.1.patch | git apply

cd catboost/python-package/catboost
python3 ../mk_wheel.py --target-platform=default-linux-aarch64 -DPYTHON_BIN=/root/.pyenv/versions/$PYTHON_VSERSION/bin/python3 -DPYTHON_CONFIG=/root/.pyenv/versions/$PYTHON_VSERSION/bin/python3-config -DHAVE_CUDA=no
mv ../*.whl /wheel_out
```

### 其他版本

```bash
# 3.7.16 3.8.16 3.9.16 3.10.9
export PYTHON_VSERSION=3.8.16
pyenv install -v $PYTHON_VSERSION
pyenv virtualenv $PYTHON_VSERSION catboost_build_$PYTHON_VSERSION
pyenv activate catboost_build_$PYTHON_VSERSION
python3 ../mk_wheel.py --target-platform=default-linux-aarch64 -DPYTHON_BIN=/root/.pyenv/versions/$PYTHON_VSERSION/bin/python3 -DPYTHON_CONFIG=/root/.pyenv/versions/$PYTHON_VSERSION/bin/python3-config -DHAVE_CUDA=no
```

## 测试

```bash
# 需要在arm64架构的机器上执行
git clone https://ghproxy.com/https://github.com/fengqi5866/catboost-arm.git && cd catboost-arm
docker run -it --platform=linux/arm64 -v $(pwd)/:/wheel_out python:3.7.4-buster bash
# 以上命令在容器外执行

# 以下命令在容器内执行
pip3 install /wheel_out/catboost-1.1.1-cp37-none-linux_aarch64.whl -i https://pypi.tuna.tsinghua.edu.cn/simple
python3 test_catboost.py
```