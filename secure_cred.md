Here's a non-interactive version of the file encryption/decryption utility for credentials:

```python
from cryptography.fernet import Fernet
import os

class CredentialCrypto:
    def __init__(self, key_file='secret.key'):
        self.key_file = key_file
        self.key = self._load_or_generate_key()

    def _load_or_generate_key(self):
        """Load existing key or generate a new one if none exists"""
        if os.path.exists(self.key_file):
            with open(self.key_file, 'rb') as f:
                return f.read()
        else:
            key = Fernet.generate_key()
            with open(self.key_file, 'wb') as f:
                f.write(key)
            return key

    def encrypt_file(self, input_file, output_file=None):
        """Encrypt a file containing credentials"""
        if output_file is None:
            output_file = input_file + '.enc'
            
        fernet = Fernet(self.key)
        
        with open(input_file, 'rb') as f:
            original_data = f.read()
        
        encrypted_data = fernet.encrypt(original_data)
        
        with open(output_file, 'wb') as f:
            f.write(encrypted_data)
        
        return output_file

    def decrypt_file(self, input_file, output_file=None):
        """Decrypt a file containing credentials"""
        if output_file is None:
            output_file = input_file.replace('.enc', '') if input_file.endswith('.enc') else input_file + '.dec'
            
        fernet = Fernet(self.key)
        
        with open(input_file, 'rb') as f:
            encrypted_data = f.read()
        
        decrypted_data = fernet.decrypt(encrypted_data)
        
        with open(output_file, 'wb') as f:
            f.write(decrypted_data)
        
        return output_file

    def encrypt_credentials(self, username, password, output_file='credentials.enc'):
        """Directly encrypt credentials without needing a file"""
        fernet = Fernet(self.key)
        credential_data = f"username={username}\npassword={password}".encode()
        encrypted_data = fernet.encrypt(credential_data)
        
        with open(output_file, 'wb') as f:
            f.write(encrypted_data)
        
        return output_file

    def decrypt_credentials(self, input_file='credentials.enc'):
        """Directly decrypt credentials and return as dictionary"""
        fernet = Fernet(self.key)
        
        with open(input_file, 'rb') as f:
            encrypted_data = f.read()
        
        decrypted_data = fernet.decrypt(encrypted_data).decode()
        credentials = {}
        for line in decrypted_data.splitlines():
            key, value = line.split('=', 1)
            credentials[key.strip()] = value.strip()
        
        return credentials
```

### Example Usage:

```python
# Example 1: Encrypt/decrypt a credentials file
crypto = CredentialCrypto('my_secret.key')

# Encrypt a file containing credentials
encrypted_file = crypto.encrypt_file('plain_credentials.txt')

# Decrypt the file
decrypted_file = crypto.decrypt_file(encrypted_file)

# Example 2: Direct credential handling
# Encrypt credentials directly
crypto.encrypt_credentials('admin', 's3cr3tP@ss', 'prod_credentials.enc')

# Decrypt and use credentials
credentials = crypto.decrypt_credentials('prod_credentials.enc')
print(f"Username: {credentials['username']}")
print(f"Password: {credentials['password']}")

# Example 3: Using in a script that needs credentials
def connect_to_service():
    crypto = CredentialCrypto('service.key')
    try:
        creds = crypto.decrypt_credentials('service_creds.enc')
        # Use creds['username'] and creds['password'] to authenticate
        print("Connected successfully!")
    except Exception as e:
        print(f"Failed to decrypt credentials: {e}")
```

### Key Features:

1. **Non-interactive**: All operations are performed through method calls
2. **Flexible Input/Output**: Can work with files or direct credential strings
3. **Automatic Key Management**: Generates key if none exists
4. **Simple Integration**: Easy to incorporate into existing scripts
5. **Dictionary Output**: Returns credentials in a easy-to-use dictionary format

### Security Recommendations:

1. Set strict file permissions on the key file: `chmod 600 secret.key`
2. Never commit the key file or encrypted credentials to version control
3. For production use, consider adding file permission checks
4. The decrypted credentials exist in memory - be mindful of this in long-running processes

This implementation provides a clean, programmatic way to handle credential encryption/decryption without any user interaction.
