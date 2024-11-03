# OpenPGP Related Work

## Introduction
OpenPGP is an open standard that allows us to encrypt and decrypt files and communications. It functions as a protocol that defines how encryption and decryption should be performed to ensure compatibility across different software implementations. By using OpenPGP, users can secure their data and maintain privacy.

## GnuPG
GnuPG (GNU Privacy Guard) is an open-source implementation of the OpenPGP standard. It is part of the GNU Project and provides a free and secure way to encrypt data and create digital signatures. GnuPG supports multiple encryption algorithms and is widely used for secure communication and data protection.

## How to install

### In Windows
1. **Download Gpg4win**:
    - Visit the official Gpg4win download page at [Gpg4win](https://gpg4win.org/download.html).
    - Download the installer for Gpg4win.

2. **Run the Installer**:
    - Double-click the downloaded installer file to start the installation process.
    - Follow the on-screen instructions. During installation, you can choose the components you want to install, such as GnuPG, Kleopatra, and GpgOL.

3. **Verify Installation**:
    - Open Command Prompt and type `gpg --version` to verify that Gpg4win (GnuPG) has been installed correctly.

### In Ubuntu
1. **Open Terminal**:
    - You can open the terminal by pressing `Ctrl + Alt + T`.

2. **Update Package List**:
    - Run the following command to update the package list:
      ```bash
      sudo apt-get update
      ```

3. **Install GnuPG**:
    - Install GnuPG by running the following command:
      ```bash
      sudo apt-get install gnupg
      ```

4. **Verify Installation**:
    - After the installation is complete, you can verify it by typing the following command in the terminal:
      ```bash
      gpg --version
      ```


# Using YubiKey for Encryption and Decryption

## Step 1: Install Required Software

1. **GPG**: Ensure GPG is installed and added to the system's PATH environment variable.
2. **YubiKey Management Tool**: Install the YubiKey management tool [YubiKey Manager](https://www.yubico.com/products/services-software/download/yubikey-manager/).

## Step 2: Generate GPG Key

1. Open the command prompt and generate a new GPG key:
    ```sh
    gpg --full-generate-key
    ```

2. Select the key type (typically RSA and RSA).
3. Set the key length (recommended 4096 bits).
4. Set the key expiration date (according to your needs).
5. Enter your name, email address, and comment.
6. Set a passphrase for the key.

## Step 3: Configure YubiKey

1. Open the YubiKey management tool and insert the YubiKey into the USB port.
2. Select **Applications** > **PIV**, then click **Configure PIV**.
3. Set the PIN and management password for the PIV application.

## Step 4: Import GPG Key to YubiKey

1. List the generated GPG keys and note the key ID:
    ```sh
    gpg --list-secret-keys
    ```

2. Edit the generated GPG key to allow exporting to YubiKey:
    ```sh
    gpg --edit-key <your-key-ID>
    ```

3. In the GPG edit interface, execute the following commands:
    ```sh
    toggle
    keytocard
    save
    ```

4. After moving the key to the YubiKey, save and exit.

## Step 5: Encrypt and Decrypt Using YubiKey

1. **Encrypt a file**:
    ```sh
    gpg --encrypt --recipient <your-email-address> <filename>
    ```

2. **Decrypt a file**:
    ```sh
    gpg --decrypt <encrypted-filename> > <decrypted-filename>
    ```

## Step 6: Test YubiKey

1. Insert the YubiKey, open the command prompt, and test encryption:
    ```sh
    echo "This is a test" | gpg --encrypt --armor --recipient <your-email-address>
    ```

2. Test decryption:
    ```sh
    echo "encrypted content" | gpg --decrypt
    ```

## Important Notes

1. **Backup Keys**: Ensure you back up your keys to a secure location before importing them to the YubiKey.
2. **Manage PIN**: Set and remember the PIN and management password for your YubiKey.
3. **Security**: Keep your YubiKey secure to prevent loss or unauthorized use.

