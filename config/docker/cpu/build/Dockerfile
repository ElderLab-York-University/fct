# Copyright (C) 2025 Helio Perroni Filho (xperroni@gmail.com)
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#         http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# Build script for Docker image.
#
# Author: Helio Perroni Filho

# Base image for the OS, update as convenient.
FROM ubuntu:24.04

USER root
WORKDIR /
SHELL ["/bin/bash", "-xo", "pipefail", "-c"]

# Build settings.
ARG uid
ARG gid
ARG DEBIAN_FRONTEND=noninteractive
ARG TZ
ENV TZ=$TZ

RUN <<EOF
# Update the package index and upgrade existing packages.
apt update && apt upgrade -y

# Install basic utilities.
apt install -y \
  nano

# Install system packages.
apt install -y \
  dumb-init \
  openjdk-21-jre \
  python-is-python3 \
  python3-pip \
  python3-venv

# Clean the APT cache and index.
apt clean && rm -rf /var/cache/apt/archives /var/lib/apt/lists
EOF

# Create ordinary user with passwordless sudo permission.
RUN <<EOF
userdel -r ubuntu # Remove the default ubuntu user.

groupadd -g $gid user
useradd -u $uid -g $gid -G dialout,plugdev,video -m -s /bin/bash user
echo 'user ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers
EOF

# Set up bash for the ordinary user.
COPY --chown=user:user user.bashrc /home/user/.bashrc

# Setup dumb-init as the init process.
# See: https://github.com/Yelp/dumb-init
ENTRYPOINT ["/usr/bin/dumb-init", "--"]

# Set up the ordinary user as the image default.
USER user
WORKDIR /home/user
CMD ["/bin/bash"]

RUN <<EOF
# Create a Python virtual environment to hold packages through PyPI.
python -m venv --system-site-packages pypi-packages
source pypi-packages/bin/activate

# Install additional Python packages.
pip install --upgrade pip
pip install \
  nlopt \
  opencv-python \
  pyyaml
EOF
