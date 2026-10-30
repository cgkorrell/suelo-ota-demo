# suelo-ota-demo
SUELO STAGE 2 - INTERNET-PULL OTA (SPARE ESP32 ONLY)
==================================================
Goal: a real internet-hosted firmware update from TEST_B_NET to TEST_C_NET,
triggered by changing a public GitHub file rather than pushing from your LAN.

PREREQUISITES
- Spare ESP32 board ONLY, no compressor/pump/relay wired to it.
- Arduino IDE 2.x, esp32 by Espressif Systems 3.3.12 as previously used.
- 4 MB ESP32, selected Board ESP32 Dev Module, Flash Size 4MB, OTA-capable
  Partition Scheme: Default 4MB with spiffs (1.2 MB APP/1.5 MB SPIFFS).
- GitHub account to create one temporary PUBLIC repository named suelo-ota-demo.
- Marin Wi-Fi SSID and password. Never put credentials on GitHub.

SETUP (all details below)
1. Create GitHub repository at https://github.com/new. Name EXACTLY
   suelo-ota-demo, visibility Public, initialize with README, default branch main.
2. Open SUELO_OTA_STAGE2_TEST_B_BOOTSTRAP/SUELO_OTA_STAGE2_TEST_B_BOOTSTRAP.ino
   Replace THREE placeholders: GitHub username, Marin Wi-Fi SSID and password.
   This B bootstrap sketch will store Wi-Fi credentials in NVS, so later public
   OTA binaries do NOT contain the password. DO NOT upload B source or its
   binary to GitHub. If the board loses NVS, flash B again via USB.
3. Connect the second ESP32 to USB. Select USB port COM4 (or actual COM port),
   correct board and partition, and Upload B. View at 115200 baud.
   Confirm: FIRMWARE=TEST_B_NET | WiFi=CONNECTED | IP=...
   Before manifest exists, GitHub 404 errors are expected on 90s checks.
4. Open SUELO_OTA_STAGE2_TEST_C_INTERNET/SUELO_OTA_STAGE2_TEST_C_INTERNET.ino
   Replace ONLY GitHub username. Do NOT add Wi-Fi SSID or password to Test C.
   Compile C, but DO NOT upload it by USB.
5. Arduino IDE Sketch > Export Compiled Binary. In the Test C sketch folder,
   identify the application .bin (not the .with_bootloader.bin or .bootloader.bin,
   .partitions.bin, or .elf). The correct .bin should be around 1 MB.
   Copy or rename that application-only .bin to TEST_C_NET.bin.
6. In PUBLIC GitHub repo: Add file > Upload files > upload TEST_C_NET.bin,
   commit to the main branch. Do NOT publish Test B or Wi-Fi credentials.
7. In public GitHub repo: Add file > Create new file > manifest.txt.
   Paste the 2 lines from manifest_TEMPLATE.txt, replacing YOUR_GITHUB_USERNAME
   with your actual GitHub username. Commit manifest.txt to main.
   The resulting raw URL must be:
   https://raw.githubusercontent.com/YOUR_GITHUB_USERNAME/suelo-ota-demo/main/manifest.txt
   Browse to this raw URL and confirm it displays those two lines.
8. Keep ESP32 powered by USB; use COM4 Serial Monitor at 115200. After GitHub
   cache propagation and next periodic check (~90s), the board should log:
      INTERNET_OTA: NEW firmware TEST_B_NET -> TEST_C_NET
      INTERNET_OTA: FLASH WRITE STARTED
   It will reboot. Serial Monitor should then show FIRMWARE=TEST_C_NET.
   New TEST_C reads Wi-Fi credentials from NVS and checks manifest on its own.
9. To demonstrate truly outside the LAN, use a phone with Wi-Fi OFF (cellular)
   to publish a future manifest change while ESP remains on Marin Wi-Fi.
   Do not publish another version until we build and test its .bin.

IMPORTANT SAFETY / SECURITY LIMITS
- THIS DEMO DISABLES TLS CERTIFICATE VERIFICATION (setInsecure) and has NO
  firmware signature or authenticated release approval. It automatically
  installs any matching versioned .bin in that public repository when changed.
- Never connect either sketch to the AWG, compressor, pump, battery or actuators.
- Never use this updater on remotely deployed equipment. It is only for
  proving the mechanics of internet delivery on the standalone spare board.
- The TEST_C public binary contains NO Wi-Fi password. TEST_B source DOES;
  keep TEST_B and its compiled binary private.
- Because the public .bin has no secrets, it may be hosted publicly for this
  controlled proof of concept. Firmware itself will not be authenticated.
- An OTA update uses the inactive flash application partition, not the SD card.
- If the ESP32 is bricked or NVS is erased, recover locally by USB flash.
- Firmware size must fit selected app partition (~1.2 MB). No OTA on no-OTA
  schemes. Do not change partition scheme between B and C.
- Do NOT select network port for viewing Serial Monitor; use USB COM port.

KNOWN TEST LIMITATIONS
- GitHub may cache raw files for several minutes. The next poll can be 90s
  later. If 404 continues, open both URLs in a browser and confirm exact names.
- If upload fails, check HTTP response, printed HTTPUpdate error and actual
  application-only binary file (not .with_bootloader).
- A public repository is used for easy demo hosting; NOT a production OTA plan.
- This source is not compiled/hardware-tested in this environment. User must
  compile both sketches with their installed ESP32 core and report errors.
