
# YubiKey Manager Related Work

## Introduction
YubiKey Manager (ykman) is a command line tool for configuring YubiKeys. It allows users to manage various features of their YubiKeys, such as configuring slots, setting up OTP, FIDO, and PIV applications, and more. YubiKey Manager provides a powerful and flexible way to control your YubiKey's settings and capabilities.

## How to Install

### In Windows
1. **Download YubiKey Manager**:
   - Visit the official YubiKey Manager release page on GitHub at [YubiKey Manager Releases](https://github.com/Yubico/yubikey-manager/releases).
   - Download the installer for Windows (usually a `.exe` file).

2. **Run the Installer**:
   - Double-click the downloaded installer file to start the installation process.
   - Follow the on-screen instructions to complete the installation.
   - You may need to add installed path to environment variable PATH

3. **Verify Installation**:
   - Open Command Prompt and type `ykman --version` to verify that YubiKey Manager has been installed correctly.

### In Ubuntu
1. **Open Terminal**:
   - You can open the terminal by pressing `Ctrl + Alt + T`.

2. **Update Package List**:
   - Run the following command to update the package list:
     ```bash
     sudo apt-get update
     ```

3. **Install Dependencies**:
   - Install the required dependencies by running:
     ```bash
     sudo apt-get install libusb-1.0-0-dev libudev-dev python3-pip
     ```

4. **Install YubiKey Manager**:
   - Install YubiKey Manager using pip by running:
     ```bash
     pip3 install yubikey-manager
     ```

5. **Verify Installation**:
   - After the installation is complete, you can verify it by typing the following command in the terminal:
     ```bash
     ykman --version
     ```

## Basic Usage

### List Connected YubiKeys
To list all connected YubiKeys, use the following command:
```bash
ykman list
```

### Managing PIV (Personal Identity Verification)
To generate a new key pair for PIV, use:
```bash
ykman piv generate-key --slot 9a --algorithm RSA2048 public_key.pem
```
To import a certificate, use:
```bash
ykman piv import-certificate 9a certificate.pem
```

### Managing FIDO (Fast Identity Online)
To manage FIDO applications on your YubiKey, you can use:
```bash
ykman fido info
```

### Setting up OTP (One-Time Password)
To configure an OTP slot, use:
```bash
ykman otp static --slot 1
```

## Setting FIDO GPG PIN

### Set FIDO PIN
To set a new FIDO PIN, use the following command:
```bash
ykman fido access change-pin
```
Follow the prompts to enter your current PIN (if any) and set a new PIN.

### Reset FIDO PIN
If you need to reset your FIDO PIN (e.g., if you forgot your current PIN), use the following command:
```bash
ykman fido reset
```
**Warning**: This will reset your FIDO application and delete all credentials stored in it. Use this command with caution.

## Resetting GPG PIN

### Reset GPG PIN
To reset the GPG PIN on your YubiKey, you will need to use the `gpg` command line tool. Here are the steps:

1. **Open Terminal or Command Prompt**:
   - Ensure your YubiKey is connected to your computer.

2. **Access GPG Card Interface**:
   - Run the following command to access the GPG card interface:
     ```bash
     gpg --card-edit
     ```

3. **Admin Mode**:
   - Enter admin mode by typing:
     ```bash
     admin
     ```

4. **Change PIN**:
   - Change the user PIN by typing:
     ```bash
     passwd
     ```
   - Follow the prompts to change the user PIN and the admin PIN.

5. **Quit**:
   - After changing the PINs, type `quit` to exit the GPG card interface.

This setup allows you to utilize YubiKey Manager and manage your YubiKey effectively on both Windows and Ubuntu platforms.
