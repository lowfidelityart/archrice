#!/bin/bash

set -e  # Exit on error

echo "Updating system..."
sudo pacman -Syu --noconfirm

# Install essential Arch packages
ARCH_PACKAGES=(
    base-devel git curl wget vim neovim htop tmux 
    unzip zip tar gzip make cmake python python-pip
    nodejs npm go rust fish zsh starship ripgrep fzf
    neofetch tldr xorg xorg-xinit stow nnn
    networkmanager iw wpa_supplicant wireless_tools
    lightdm lightdm-webkit2-greeter
)

echo "Installing Arch packages..."
sudo pacman -S --noconfirm --needed "${ARCH_PACKAGES[@]}"

# Enable NetworkManager for Wi-Fi connectivity
echo "Enabling NetworkManager..."
sudo systemctl enable --now NetworkManager

# Install and enable LightDM with WebKit Greeter
sudo systemctl enable lightdm

# Install yay (AUR helper)
if ! command -v yay &>/dev/null; then
    echo "Installing yay..."
    git clone https://aur.archlinux.org/yay-bin.git /tmp/yay-bin
    cd /tmp/yay-bin
    makepkg -si --noconfirm
    cd ~
    rm -rf /tmp/yay-bin
fi

# Install AUR packages
AUR_PACKAGES=(
    nerd-fonts-complete
    tealdeer
    lsd
    cointop
    howdoi
    yewtube
    nvchad-git
    pywal
    chadwm-git
    epic-games-launcher
    lightdm-webkit-theme-aether
)

echo "Installing AUR packages..."
yay -S --noconfirm --needed "${AUR_PACKAGES[@]}"

# Clone and install Suckless software (Backup in case ChadWM fails)
echo "Installing Suckless software..."
SUCKLESS_DIR="$HOME/suckless"
mkdir -p "$SUCKLESS_DIR"
cd "$SUCKLESS_DIR"

SUCKLESS_REPOS=(
    "https://git.suckless.org/dwm"
    "https://git.suckless.org/st"
    "https://git.suckless.org/dmenu"
    "https://git.suckless.org/surf"
)

for REPO in "${SUCKLESS_REPOS[@]}"; do
    REPO_NAME=$(basename "$REPO")
    if [ ! -d "$SUCKLESS_DIR/$REPO_NAME" ]; then
        git clone "$REPO"
        cd "$REPO_NAME"
        make && sudo make install
        cd ..
    else
        echo "$REPO_NAME already installed, skipping..."
    fi
done

# Set up .xinitrc to launch ChadWM first, but fallback to DWM if it fails
echo "Setting up .xinitrc..."
cat <<EOF > ~/.xinitrc
if command -v chadwm &> /dev/null; then
    exec chadwm
else
    echo "ChadWM failed! Falling back to DWM..."
    exec dwm
fi
EOF
chmod +x ~/.xinitrc

# Remove BlackArch if it's already installed
if grep -q "\[blackarch\]" /etc/pacman.conf; then
    echo "Removing BlackArch repository..."
    sudo sed -i '/\[blackarch\]/,+1 d' /etc/pacman.conf
    sudo rm -rf /etc/pacman.d/blackarch-mirrorlist
fi

# Mirror Fix (If Needed)
echo "Checking mirror status..."
if [[ ! -s /etc/pacman.d/mirrorlist ]]; then
    echo "Mirror list is empty, resetting to default mirrors..."
    sudo cp /etc/pacman.d/mirrorlist.backup /etc/pacman.d/mirrorlist
fi

echo "Installation complete! Restart and use 'startx' for ChadWM (or DWM fallback)."
