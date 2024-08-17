## Raspberry Pi Bluetooth Speaker with Spotify Connect and AirPlay

- Install Raspberry Pi OS (Lite)
- Enable autologin using `raspi-config`
- Optionally: Add the dt_overlay= line of your DAC to the config.txt
- Often the instructions disable the internal audio. Don't do that if you want to use it. We are later outputing to all outputs simultanously.
- Install Pipewire, Wireplumber and Avahi
```sh
sudo apt install pipewire pipewire-alsa pipewire-pulse wireplumber avahi-daemon libpulse-dev
```
- Optionally: use ~/.config/wireplumber.. to rename sinks
- Use ~/.config/pipewire.conf to add a combine sink to simultanously output to all alsa sinks
- Make the new combined sink the default sink
```sh
wpctl status
# Seach for the new sink and remember the ID
wpctl set-default <ID>
# Now might also be a good moment to increase the volume of all outputs to 1.
# Get the IDs from above and run the following command for each output.
wpctl set-volume <ID> 1
```
- Install rustup (needed for librespot)
```sh
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```
- Install librespot for spotify
```sh
# First we need to increase the swap size
sudo dphys-swapfile swapoff
nano /etc/dphys-swapfile
# Change CONF_SWAPSIZE to something like 2048
sudo dphys-swapfile setup
sudo dphys-swapfile swapon
git clone https://github.com/librespot-org/librespot
cd librespot
# The next command builds librespot and can take up to 1 hour
cargo build --release librespot --no-default-features --features "pulseaudio-backend"
# Add the service file from .config/systemd/user/librespot and adapt the path
systemctl --user enable librespot
```
- Install shairport-sync for AirPlay
```sh
git clone https://github.com/mikebrady/shairport-sync
cd shairport-sync
sudo apt install --no-install-recommends build-essential git autoconf automake libtool \
    libpopt-dev libconfig-dev libasound2-dev avahi-daemon libavahi-client-dev libssl-dev libsoxr-dev
autoreconf -fi
./configure --sysconfdir=/etc --with-pa --with-soxr --with-avahi --with-ssl=openssl --with-systemd
make
sudo make install
# Add the service file from .config/systemd/user/shairport and adapt the path
systemctl --user enable shairport
```
- Add `bt-pair-agent/bt-pair-agent.service` and adapt the path. This autostarts a bluetooth handler that accepts all connections.
- Run `sudo sed -i 's/#JustWorksRepairing.*/JustWorksRepairing = always/' /etc/bluetooth/main.conf`
