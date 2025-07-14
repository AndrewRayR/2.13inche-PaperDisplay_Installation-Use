# 2.13inch e-Paper HAT V2 Setup Guide for Raspberry Pi OS Lite

## What You'll Need
- Raspberry Pi (any model with 40-pin GPIO header)
- 2.13inch e-Paper HAT V2 (Waveshare)
- MicroSD card (8GB or larger)
- Power supply for your Pi
- Computer for initial setup

## Initial Setup

### 1. Flash Raspberry Pi OS Lite (Headless Setup)
- Download Raspberry Pi Imager from the official website
- Flash Raspberry Pi OS Lite to your SD card
- **Enable SSH**: In the imager, click the gear icon → Enable SSH → Use password authentication
- **Configure WiFi**: Set your WiFi credentials in the imager settings
- **Set username/password**: Create a user account (e.g., username: `pi`)
- Eject SD card and insert into Pi, then power on

### 2. Connect via SSH
Find your Pi's IP address (check your router or use network scanner), then connect:

```bash
ssh pi@192.168.1.xxx
# Replace with your Pi's actual IP address
```

**Alternative IP discovery methods:**
```bash
# From another Linux/Mac machine on same network
nmap -sn 192.168.1.0/24 | grep -i raspberry

# Or use hostname (if available)
ssh pi@raspberrypi.local
```

### 3. Initial Configuration via SSH
Once connected, run the configuration tool:
```bash
sudo raspi-config
```

**Essential settings to configure:**
- **System Options** → **Password** → Change default password
- **System Options** → **Hostname** → Set a custom hostname (optional)
- **Localisation Options** → **Timezone** → Set your timezone
- **Advanced Options** → **Expand Filesystem** → Ensure full SD card usage

### 2. Enable SPI Interface (via SSH)
The e-Paper HAT requires SPI communication:

```bash
sudo raspi-config
```

- Navigate to: `Interfacing Options` → `SPI` → `Yes`
- **Important**: Select `<Finish>` and choose `<Yes>` when asked to reboot
- Wait for reboot, then reconnect via SSH:
```bash
ssh pi@192.168.1.xxx
```

### 3. Update System (via SSH)
```bash
sudo apt update
sudo apt upgrade -y
```

**Note**: This may take several minutes over SSH. You can safely leave the terminal open and wait for completion.

## Hardware Connection

### Pin Connections (HAT Version)
**Power down your Pi before connecting the HAT:**
```bash
sudo shutdown -h now
```

Wait for the Pi to fully power down, then:
1. Disconnect power from your Pi
2. Carefully align and attach the HAT to the 40-pin GPIO header
3. Ensure the HAT is firmly seated
4. Reconnect power and wait for boot
5. Reconnect via SSH

The HAT version connects directly to the 40-pin GPIO header - no wiring needed!

**Important**: The HAT uses these GPIO pins:
- **VCC**: 3.3V
- **GND**: Ground
- **DIN**: GPIO 10 (SPI0 MOSI)
- **CLK**: GPIO 11 (SPI0 SCLK)
- **CS**: GPIO 8 (SPI0 CE0)
- **DC**: GPIO 25
- **RST**: GPIO 17
- **BUSY**: GPIO 24

Simply attach the HAT to your Pi's GPIO header - it's designed to fit perfectly.

## Software Installation (All via SSH)

### 1. Install Required Packages
```bash
sudo apt install python3-pip python3-pil python3-numpy git -y
```

### 2. Install BCM2835 Library (C Library - Optional)
**This step is optional but recommended for better performance:**
```bash
cd ~
wget http://www.airspayce.com/mikem/bcm2835/bcm2835-1.71.tar.gz
tar zxvf bcm2835-1.71.tar.gz
cd bcm2835-1.71/
./configure
make
sudo make check
sudo make install
cd ~
```

### 3. Install WiringPi (Optional)
```bash
sudo apt install wiringpi -y
```

### 4. Install Python Libraries
```bash
pip3 install RPi.GPIO
pip3 install spidev
```

**Verify installation:**
```bash
python3 -c "import RPi.GPIO, spidev; print('Libraries installed successfully')"
```

## Get the Demo Code (via SSH)

### 1. Clone Waveshare Repository
```bash
cd ~
git clone https://github.com/waveshare/e-Paper.git
cd e-Paper
```

### 2. Navigate to Python Examples
```bash
cd RaspberryPi_JetsonNano/python/examples
```

### 3. List Available Examples
```bash
ls -la *2in13_V2*
```

You should see files like:
- `epd_2in13_V2_test.py`
- Other example scripts

## Basic Usage Examples (via SSH)

### 1. Display Test Pattern
```bash
cd ~/e-Paper/RaspberryPi_JetsonNano/python/examples
python3 epd_2in13_V2_test.py
```

**Expected output:**
- The script will print initialization messages
- Display should show test patterns, text, and images
- Process takes about 10-15 seconds

### 2. Create Custom Python Script
Create a new file using nano editor:
```bash
cd ~
nano hello_epaper.py
```

Copy and paste this code (use Ctrl+Shift+V in most SSH clients):

```python
#!/usr/bin/python3
import sys
import os
picdir = os.path.join(os.path.dirname(os.path.dirname(os.path.realpath(__file__))), 'pic')
libdir = os.path.join(os.path.dirname(os.path.dirname(os.path.realpath(__file__))), 'lib')
if os.path.exists(libdir):
    sys.path.append(libdir)

import logging
from waveshare_epd import epd2in13_V2
import time
from PIL import Image,ImageDraw,ImageFont

logging.basicConfig(level=logging.DEBUG)

try:
    epd = epd2in13_V2.EPD()
    epd.init(epd.FULL_UPDATE)
    epd.Clear(0xFF)
    
    # Create image
    image = Image.new('1', (epd.width, epd.height), 255)
    draw = ImageDraw.Draw(image)
    
    # Add text
    font = ImageFont.load_default()
    draw.text((10, 10), 'Hello World!', font = font, fill = 0)
    draw.text((10, 30), 'e-Paper Display', font = font, fill = 0)
    
    epd.display(epd.getbuffer(image))
    
    epd.sleep()
    
except IOError as e:
    print(e)
    
except KeyboardInterrupt:    
    print("Interrupted")
    epd2in13_V2.epdconfig.module_exit()
    exit()
```

Save and exit nano (Ctrl+X, Y, Enter), then run:
```bash
python3 hello_epaper.py
```

### 3. File Transfer Options
**To transfer images or files to your Pi:**

**Option A: Using SCP (from your computer):**
```bash
scp your_image.png pi@192.168.1.xxx:~/
```

**Option B: Using wget (on Pi):**
```bash
wget https://example.com/image.png -O ~/image.png
```

**Option C: Using nano to create files directly:**
```bash
nano myfile.txt
```

## Display Specifications

### 2.13inch e-Paper HAT V2 Details
- **Resolution**: 250×122 pixels
- **Display Colors**: Black, White
- **Interface**: SPI
- **Viewing Angle**: >170°
- **Refresh Time**: 2s (full refresh)
- **Power Consumption**: 
  - Operating: ~26.4mW
  - Standby: <0.017mW
- **Operating Temperature**: 0°C ~ 50°C

## Troubleshooting

### SSH Connection Issues

1. **Cannot connect to Pi**
   ```bash
   # Check if Pi is on network
   ping raspberrypi.local
   # Or scan for Pi's IP
   nmap -sn 192.168.1.0/24
   ```

2. **SSH connection refused**
   ```bash
   # SSH might not be enabled - need physical access to enable it
   # Or check if SSH is running
   sudo systemctl status ssh
   sudo systemctl enable ssh
   sudo systemctl start ssh
   ```

3. **WiFi not connecting**
   ```bash
   # Check WiFi status
   sudo iwconfig
   # Reconfigure WiFi
   sudo raspi-config
   # Navigate to System Options → Wireless LAN
   ```

### Common e-Paper Issues

1. **"No module named 'waveshare_epd'"**
   ```bash
   # Make sure you're in the right directory
   cd ~/e-Paper/RaspberryPi_JetsonNano/python/examples
   # Check if files exist
   ls -la ../lib/waveshare_epd/
   ```

2. **SPI Interface Not Working**
   ```bash
   # Check if SPI is enabled
   lsmod | grep spi
   # Should show spi_bcm2835
   # If not, re-enable SPI
   sudo raspi-config
   ```

3. **Display Not Updating**
   - Check HAT connection (requires physical access)
   - Verify SPI is enabled
   - Try power cycling the Pi:
   ```bash
   sudo reboot
   ```

4. **Permission Errors**
   ```bash
   # Make script executable
   chmod +x your_script.py
   # Or run with sudo
   sudo python3 your_script.py
   ```

### Remote Debugging Tips

1. **Monitor system logs**
   ```bash
   # View recent system messages
   sudo dmesg | tail -20
   # Monitor logs in real-time
   sudo journalctl -f
   ```

2. **Check GPIO status**
   ```bash
   # Install gpio tools
   sudo apt install gpiod -y
   # Check pin status
   gpioinfo
   ```

3. **Test SPI without display**
   ```bash
   # Install spi-tools
   sudo apt install spi-tools -y
   # Test SPI interface
   spi-config -d /dev/spidev0.0 -q
   ```

## Performance Tips

### 1. Optimize Refresh Strategy
- Use partial refresh for text updates when possible
- Full refresh only when necessary (every 10-20 partial refreshes)
- Implement double buffering for smoother animations

### 2. Image Optimization
```python
# Convert images to 1-bit for better performance
image = image.convert('1')
```

### 3. Power Management
```python
# Always put display to sleep when not in use
epd.sleep()
```

## Advanced Features

### 1. Partial Refresh
```python
epd.init(epd.PART_UPDATE)
epd.displayPartBaseImage(epd.getbuffer(image))
# Update only changed portions
epd.displayPart(epd.getbuffer(new_image))
```

### 2. Custom Fonts
```python
# Use custom fonts
font = ImageFont.truetype('/usr/share/fonts/truetype/dejavu/DejaVuSans.ttf', 12)
draw.text((10, 10), 'Custom Font!', font=font, fill=0)
```

## Project Ideas

1. **Weather Station Display**
2. **System Monitor** (CPU, Memory, Temperature)
3. **News Ticker**
4. **Digital Clock**
5. **IoT Status Dashboard**
6. **E-book Reader**
7. **Smart Home Control Panel**

## Additional Resources

- [Waveshare e-Paper Wiki](https://www.waveshare.com/wiki/2.13inch_e-Paper_HAT)
- [Official GitHub Repository](https://github.com/waveshare/e-Paper)
- [Raspberry Pi GPIO Pinout](https://pinout.xyz/)

## Quick Commands Reference

### SSH Connection
```bash
# Connect to Pi
ssh pi@192.168.1.xxx

# Find Pi on network
nmap -sn 192.168.1.0/24 | grep -i raspberry

# Copy files to Pi
scp file.txt pi@192.168.1.xxx:~/

# Copy files from Pi
scp pi@192.168.1.xxx:~/file.txt ./
```

### System Setup
```bash
# Enable SPI
sudo raspi-config

# Install dependencies
sudo apt install python3-pip python3-pil python3-numpy git -y

# Install Python libraries
pip3 install RPi.GPIO spidev

# Clone repository
git clone https://github.com/waveshare/e-Paper.git

# Run test
cd e-Paper/RaspberryPi_JetsonNano/python/examples
python3 epd_2in13_V2_test.py

# Check SPI
lsmod | grep spi

# System info
cat /proc/cpuinfo | grep Model
```

### File Management via SSH
```bash
# List files
ls -la

# Create directory
mkdir foldername

# Edit file
nano filename.py

# Make executable
chmod +x filename.py

# View file content
cat filename.py

# Download file from internet
wget https://example.com/file.txt
```

---

**Note**: This HAT is designed specifically for the Raspberry Pi's 40-pin GPIO header. The direct connection makes setup much easier compared to breadboard wiring, and the included mounting hardware ensures a secure connection.
