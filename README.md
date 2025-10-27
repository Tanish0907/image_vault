
```markdown
# Image Vault

Image Vault is a tool for encrypting and decrypting image files. It allows you to securely split images into multiple encrypted parts and then reassemble them. The project leverages AES encryption to ensure the security of your image files.

## Features

- Encrypt images by splitting them into multiple parts.
- Decrypt images and reassemble them from encrypted parts.
- Supports customizable number of parts for splitting.
- Uses AES encryption for secure data handling.

## Requirements

- Python 3.x
- Required Python packages:
  - `cryptography`
  - `pillow`
  - `click`

You can install the required packages using pip:

```bash
pip install cryptography pillow click
```

## Usage

The main script provides a command-line interface with options for encryption and decryption.

### Encrypting a Folder of Images

To encrypt a folder of images, use the `--encrypt` flag. You can specify the input path, output path, key, and number of parts for splitting:

```bash
fs --encrypt --input_path ./path/to/images --output_path ./path/to/output --key your_key --parts 4
```

### Decrypting a Folder of Encrypted Images

To decrypt a folder of encrypted images, use the `--decrypt` flag. You only need to specify the input path and the key:

```bash
fs --decrypt --input_path ./path/to/encrypted --key your_key
```

### Command Line Options

- `--encrypt`: Use this flag for encryption.
- `--decrypt`: Use this flag for decryption.
- `--input_path`: Path to the input directory. Default is `./`.
- `--output_path`: Path to the output directory. Default is `./op`.
- `--key`: Encryption/Decryption key. Default is `1234`.
- `--parts`: Number of parts to split the file into for encryption. Default is `1`.

## Example

Encrypting an image folder:

```bash
python script.py --encrypt --input_path ./test_images --output_path ./encrypted_images --key mysecretkey --parts 4
```

Decrypting an image folder:

```bash
python script.py --decrypt --input_path ./encrypted_images --key mysecretkey
```

## Implementation Details

### Key Generation

A key is generated from a seed token using SHA-256 hashing:

```python
def key_gen(seed):
    seed = bytes(seed, "utf-8")
    seed_hash = hashlib.sha256()
    seed_hash.update(seed)
    return seed_hash.hexdigest()
```

### Encryption

Images are split into parts, each part is padded, and then encrypted using AES in CBC mode:

```python
def encrypt_file(f_name, file_number, input_file, seed_token):
    key = seed_token.ljust(32)[:32].encode('utf-8')
    iv = os.urandom(16)
    cipher = Cipher(algorithms.AES(key), modes.CBC(iv), backend=default_backend())
    encryptor = cipher.encryptor()
    data = input_file
    padder = padding.PKCS7(algorithms.AES.block_size).padder()
    padded_data = padder.update(data) + padder.finalize()
    encrypted_data = encryptor.update(padded_data) + encryptor.finalize()
    hex_data = binascii.hexlify(iv + encrypted_data).decode('utf-8')
    with open(f'{f_name}{file_number}', 'w') as output_file:
        output_file.write(hex_data)
    print(f"Encrypted data written to {f_name}{file_number}")
```

### Decryption

The encrypted parts are read, decrypted, and reassembled into the original image:

```python
def decrypt_file(input_file, seed_token):
    key = seed_token.ljust(32)[:32].encode('utf-8')
    hex_data = input_file
    encrypted_data_with_iv = binascii.unhexlify(hex_data)
    iv = encrypted_data_with_iv[:16]
    encrypted_data = encrypted_data_with_iv[16:]
    cipher = Cipher(algorithms.AES(key), modes.CBC(iv), backend=default_backend())
    decryptor = cipher.decryptor()
    padded_data = decryptor.update(encrypted_data) + decryptor.finalize()
    unpadder = padding.PKCS7(algorithms.AES.block_size).unpadder()
    data = unpadder.update(padded_data) + unpadder.finalize()
    print("Decrypted data")
    return data
```

