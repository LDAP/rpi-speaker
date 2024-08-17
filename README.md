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
- Install rustup (needed for librespot)
```sh
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```
- Install librespot for spotify
```sh
git clone https://github.com/librespot-org/librespot
cd librespot
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
