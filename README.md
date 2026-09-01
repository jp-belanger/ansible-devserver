# ansible

## Setup yay

```shell
# Install dependencies
sudo pacman -Syu --needed git base-devel

# Clone and build yay
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si

# Clean up
cd .. && rm -rf yay
```

## Setup ansible

```shell
sudo pacman -S --needed ansible

ansible-galaxy collection install kewlfft.aur

ansible-playbook local.yml -t install --ask-become-pass
```
