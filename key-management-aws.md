# Key Management (AWS)

#### Export symmetric keys using a pre-established key exchange key (TR-31) <a href="#keys-export-tr31" id="keys-export-tr31"></a>

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

When exchanging multiple keys or supporting key rotation, you typically first exchange an initial key encryption key (KEK) using paper key components or, with AWS Payment Cryptography, using [TR-34](https://docs.aws.amazon.com/payment-cryptography/latest/userguide/keys-export.html#keys-export-tr34). After establishing a KEK, you can use it to transport subsequent keys, including other KEKs. We support this key exchange using ANSI TR-31, which is widely supported by HSM vendors.

1.  **Set up your Key Encryption Key (KEK)**

    Make sure you have already exchanged your KEK and have the keyARN (or keyAlias) available.
2.  **Create your key on AWS Payment Cryptography**

    Create your key if it doesn't already exist. Alternatively, you can create the key on your other system and use the import command.
3.  **Export your key from AWS Payment Cryptography**

    When exporting in TR-31 format, specify the key you want to export and the wrapping key to use.

    **Example – Exporting a key using TR31 key block**

    ```
    ws payment-cryptography export-key \
    --key-material='{"Tr31KeyBlock": \
    { "WrappingKeyIdentifier": "arn:aws:payment-cryptography:us-east-2:111122223333:key/ov6icy4ryas4zcza" }}' \
    --export-key-identifier arn:aws:payment-cryptography:us-east-2:111122223333:key/5rplquuwozodpwsp
    ```

    ```
    {
      "WrappedKey": {
        "KeyCheckValue": "73C263",
        "KeyCheckValueAlgorithm": "ANSI_X9_24",
        "KeyMaterial": "D0144K0AB00E0000A24D3ACF3005F30A6E31D533E07F2E1B17A2A003B338B1E79E5B3AD4FBF7850FACF9A3784489581A543C84816C8D3542AE888CE6D4EDDFD09C39957B131617BC",
        "WrappedKeyMaterialFormat": "TR31_KEY_BLOCK"
      }
    }
    ```
4.  **Import the key to your system**

    Use your system's import key implementation to import the key.

```java
// import tr31 keyblock generated with AWS cloud HSM
pos.sendTR31Key(tr31KeyBlock);
```

