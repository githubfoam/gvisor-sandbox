# -*- mode: ruby -*-
# vi: set ft=ruby :
$ubuntu_docker_script = <<SCRIPT
# Get Docker Engine - Community for Ubuntu
# https://docs.docker.com/install/linux/docker-ce/ubuntu/
sudo apt-get remove docker docker-engine docker.io containerd runc
sudo apt-get update
sudo apt-get install \
apt-transport-https \
ca-certificates \
curl \
software-properties-common -y
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo apt-key fingerprint 0EBFCD88
sudo add-apt-repository \
"deb [arch=amd64] https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) \
stable"
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io -y
echo "==================================================================="
sudo docker --version
echo "==================================================================="
# Manage Docker as a non-root user
# https://docs.docker.com/install/linux/linux-postinstall/
sudo usermod -aG docker vagrant # add user to the docker group
sudo systemctl enable docker
SCRIPT
$gvisor_script = <<SCRIPT
DIST=release
sudo apt-get update && sudo apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
curl -fsSL https://gvisor.dev/archive.key | sudo apt-key add -
sudo add-apt-repository "deb https://storage.googleapis.com/gvisor/releases ${DIST} main"
sudo apt-get update && sudo apt-get install -y runsc
sudo runsc install
sudo systemctl restart docker
sudo docker run --runtime=runsc --rm hello-world
sudo docker run --runtime=runsc -it ubuntu dmesg
SCRIPT
$bazel_script = <<SCRIPT
# Install bazel
sudo curl https://bazel.build/bazel-release.pub.gpg | sudo apt-key add -
echo "deb [arch=amd64] https://storage.googleapis.com/bazel-apt stable jdk1.8" | sudo tee /etc/apt/sources.list.d/bazel.list
sudo apt-get update && sudo apt-get install bazel -y
# # Install gvisor
# wget https://storage.googleapis.com/gvisor/releases/nightly/latest/runsc
# wget https://storage.googleapis.com/gvisor/releases/nightly/latest/runsc.sha512
# sha512sum -c runsc.sha512
# sudo mv runsc /usr/local/bin
# sudo chown root:root /usr/local/bin/runsc
# sudo chmod 0755 /usr/local/bin/runsc
# # configure Docker to use runsc by adding a runtime entry to Docker configuration
# cat <<EOF | sudo tee /etc/docker/daemon.json
# {
#     "runtimes": {
#         "runsc": {
#             "path": "/usr/local/bin/runsc"
#         }
#     }
# }
# EOF
# sudo systemctl restart docker
# sudo docker run --runtime=runsc --rm hello-world
SCRIPT
BOX_IMAGE = "bento/ubuntu-18.04"
HOSTNAME = "gvisor-sandbox"
Vagrant.configure("2") do |config|
  config.vm.box = BOX_IMAGE
  config.vm.box_check_update = false
  config.vm.hostname = HOSTNAME
  config.vm.network "private_network", ip: "192.168.50.10"
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "1024"
    vb.cpus = "2"
  end
  # config.vm.provision "shell", inline: $ubuntu_docker_script, privileged: false
  config.vm.provision "shell", inline: <<-SHELL
  echo "===================================================================================="
                            hostnamectl status
  echo "===================================================================================="
  echo "         \   ^__^                                                                  "
  echo "          \  (oo)\_______                                                          "
  echo "             (__)\       )\/\                                                      "
  echo "                 ||----w |                                                         "
  echo "                 ||     ||                                                         "
  SHELL
end
