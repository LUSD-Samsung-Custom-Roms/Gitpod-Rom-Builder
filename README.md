# Gitpod Rom Builder - LineageOS 16.0 for Samsung Galaxy Tab 3 8.0 (SM-T310)

This repository provides an automated configuration and guide to build an unofficial build of **LineageOS 16.0 (Android 9.0 Pie)** for the **Samsung Galaxy Tab 3 8.0 Wi-Fi (lt01wifi)** entirely within cloud-based **Gitpod** workspaces.

---

## ⚠️ Important Workspace Warnings & Disclaimers

Before starting your build inside a cloud machine, you must keep Gitpod's environmental constraints in mind:

*   **Timeout Restrictions**: Gitpod's standard free tier workspaces automatically time out and shut down after **30 minutes of user inactivity**. You must keep your browser tab active or interact with the terminal during the `repo sync` and compilation stages to prevent data loss.
*   **Disk Limits**: Ensure your Gitpod workspace configuration provides at least **100GB to 200GB** of space, as syncing Android 9.0 and building it requires a massive storage footprint.
*   **Compilation Specs**: Performance depends heavily on the resources assigned to your dynamic Gitpod container. 

---

## 🛠️ Step 1: Initialize Workspace & Environment Setup

When you boot into your Gitpod workspace, your Linux terminal needs essential build dependencies, toolchains, and environment variables configured.

Run this block to update the system packages and install the Android compilation dependencies:

```bash
# Update package lists
sudo apt-get update

# Install build dependencies, OpenJDK 8, and toolchains required for Pie/Lineage 16
sudo apt-get install -y bc bison build-essential ccache curl flex g++-multilib \
gcc-multilib git gnupg gperf imagemagick lib32readline-dev lib32z1-dev liblz4-tool \
libncurses5 libncurses5-dev libsdl1.2-dev libssl-dev libxml2 libxml2-utils lzop \
pngcrush rsync schedtool squashfs-tools xsltproc zip zlib1g-dev openjdk-8-jdk

# Ensure standard C formatting rules apply to avoid syntax environment crashes
export LC_ALL=C
```

---

## 🚀 Step 2: Configure Git Identity

The Google `repo` tool requires a global Git name and email to properly manage local sync commits. Set your identity using placeholder data or your GitHub info:

```bash
git config --global user.email "lukestot0103@gmail.com"
git config --global user.name "Gitpod Rom Builder"
```

---

## 📦 Step 3: Setup Google Repo Tool

Create your local bin directory, pull down the secure Google source tracking application, and append it to your terminal environment execution path:

```bash
# Create local bin path and project workspace directories
mkdir -p ~/bin
mkdir -p ~/android/lineage

# Download and permission repo tool
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo

# Export paths so the binary is globally accessible across this terminal instance
export PATH=~/bin:$PATH
```

---

## 🗃️ Step 4: Initialize Manifests & Sync Source Tree

Navigate directly to your designated Android directory to initialize the official LineageOS 16.0 source tree, set up your localized device manifest modifications, and start downloading files.

### 1. Initialize Base Tree
```bash
cd ~/android/lineage
repo init -u https://github.com/LineageOS/android.git -b lineage-16.0 --git-lfs
```

### 2. Configure Device Manifest (`lt01wifi`)
Because the `lt01wifi` is officially unsupported on the Lineage 16.0 tree, you must create a local manifest targeting community developer branches (such as my device and kernel trees). Create the local manifests directory:

```bash
mkdir -p .repo/local_manifests
```
### Use my manifest:
```
wget -O .repo/local_manifests/lusd1.xml https://raw.githubusercontent.com/LUSD-Samsung-Custom-Roms/android/refs/heads/lineage-16.0/lusd1.xml
```
### 3. Sync the Source Repositories
Start the download process. This downloads roughly 100GB+ of data and takes an extended period depending on Gitpod's network bandwidth link:

```bash
repo sync -c -j$(nproc --all) --force-sync --no-clone-bundle --no-tags
```

---

## ⚡ Step 5: Accelerate Builds Using Ccache

To drastically shorten build iterations or keep files optimized within your container cache, utilize compiler caching rules:

```bash
export USE_CCACHE=1
export CCACHE_EXEC=/usr/bin/ccache
ccache -M 50G
```

---

## 🏗️ Step 6: Start Compilation

Initialize the environment injection scripts, instruct the pre-build parameters to recognize the `lt01wifi` structural layout, and build using `brunch`:

```bash
# Initialize build environments
. build/envsetup.sh

# Target configuration injection 
breakfast lt01wifi

# Execute final ROM compilation
brunch lt01wifi
```

---

## 📥 Step 7: Retrieve Your Flashable ROM File

Once the terminal prints a successful build execution message without core block fatal errors, your flashable zip package along with custom recovery files will be securely waiting inside this path:

```text
~/android/lineage/out/target/product/lt01wifi/
```

### Deployment Instructions
1. Download the generated `.zip` installation archive from Gitpod to your local desktop machine.
2. Transfer it via USB or SD Card to your **Samsung Galaxy Tab 3 8.0**.
3. Reboot your device into custom recovery mode (e.g., TWRP Recovery).
4. Perform a complete **Factory Reset** (Wipe Data, System, Cache, and Dalvik).
5. Select and flash the LineageOS 16.0 installation `.zip`. 
6. *(Optional)* Flash an Android 9.0 (Pie) compatible **OpenGApps** package if you require Google Play Services functionality.
7. Reboot system!
