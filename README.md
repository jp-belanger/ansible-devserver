# ansible

## Setup yay

```shell
# Install dependencies
sudo pacman -S --needed git base-devel

# Clone and build yay
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si

# Clean up
cd .. && rm -rf yay
```

## Setup ansible

```shell
sudo pacman -Sy ansible

ansible-galaxy collection install kewlfft.aur

ansible-playbook local.yml -t install --ask-become-pass --ask-vault-pass
```
