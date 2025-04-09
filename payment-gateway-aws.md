# Payment Gateway (AWS)

Below is an example of how you can use the AWS Payment Cryptography Data Plane API in Java to decrypt data from a POS terminal. In this example, we assume that your POS terminal data was encrypted using a symmetric key (for example, using TDES with CBC mode). You’ll need to adjust the attributes (for example, algorithm, mode, and key length) to match your specific encryption settings.

<figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

#### Prerequisites

* **AWS Account & Payment Cryptography key**: Ensure you have created or imported a decryption key in AWS Payment Cryptography. The key must have its KeyModesOfUse set to allow decryption.
* **AWS SDK for Java v2**: The Payment Cryptography Data Plane client is available in the AWS SDK for Java v2.
* **POS Terminal Data**: Your encrypted POS terminal data (ciphertext) is provided as a hex string.

#### Example Code

```
import software.amazon.awssdk.services.paymentcryptographydata.PaymentCryptographyDataClient;
import software.amazon.awssdk.services.paymentcryptographydata.model.DecryptDataRequest;
import software.amazon.awssdk.services.paymentcryptographydata.model.DecryptDataResponse;
import software.amazon.awssdk.services.paymentcryptographydata.model.EncryptionDecryptionAttributes;

public class DecryptPosTerminalDukptData {

    public static void main(String[] args) {
        // --- Test Data from a real POS terminal ---
        // Encrypted Track 2 data (hex string, 3DES CBC mode per ANSI X9.24-2009 request)
        String encryptedTrack2 = "153CEE49576C0B709515946D991CB48368FEA0375837ECA6";
        // Track KSN used for DUKPT derivation for track data
        String trackKsn = "00000332100300E00002";

        // Encrypted PIN block data (hex string)
        String encryptedPinBlock = "377D28B8C7EF080A";
        // PIN KSN used for DUKPT derivation for PIN decryption
        String pinKsn = "00000332100300E000C6";

        // Replace these with your actual key ARNs from AWS Payment Cryptography.
        String trackKeyIdentifier = "arn:aws:payment-cryptography:us-east-2:111122223333:key/track-key-id";
        String pinKeyIdentifier = "arn:aws:payment-cryptography:us-east-2:111122223333:key/pin-key-id";

        try (PaymentCryptographyDataClient client = PaymentCryptographyDataClient.create()) {

            // ---------------------------------------
            // Example 1: Decrypt Track 2 Data using DUKPT
            // ---------------------------------------
            // Build decryption attributes for track data using Dukpt.
            EncryptionDecryptionAttributes trackDecryptionAttr =
                    EncryptionDecryptionAttributes.builder()
                            .dukpt(b -> b.keySerialNumber(trackKsn))
                            .build();

            DecryptDataRequest trackRequest = DecryptDataRequest.builder()
                    .keyIdentifier(trackKeyIdentifier)
                    .cipherText(encryptedTrack2)
                    .decryptionAttributes(trackDecryptionAttr)
                    .build();

            DecryptDataResponse trackResponse = client.decryptData(trackRequest);
            String trackPlainTextHex = trackResponse.plainText();
            System.out.println("Decrypted Track 2 Data (hex): " + trackPlainTextHex);
            // Further processing may be needed to unpack 6-bit–encoded data
            // (i.e. splitting 24-bit groups into 4 groups of 6 bits and adding 0x20 to each).

            // ---------------------------------------
            // Example 2: Decrypt PIN Block Data using DUKPT
            // ---------------------------------------
            // Build decryption attributes for PIN block using Dukpt.
            EncryptionDecryptionAttributes pinDecryptionAttr =
                    EncryptionDecryptionAttributes.builder()
                            .dukpt(b -> b.keySerialNumber(pinKsn))
                            .build();

            DecryptDataRequest pinRequest = DecryptDataRequest.builder()
                    .keyIdentifier(pinKeyIdentifier)
                    .cipherText(encryptedPinBlock)
                    .decryptionAttributes(pinDecryptionAttr)
                    .build();

            DecryptDataResponse pinResponse = client.decryptData(pinRequest);
            String pinPlainTextHex = pinResponse.plainText();
            System.out.println("Decrypted PIN Block (hex): " + pinPlainTextHex);
            // Further processing (such as XOR with formatted PAN data) per ANSI X9.8 is needed
            // to recover the actual clear PIN.

        } catch (Exception e) {
            System.err.println("Error during decryption: " + e.getMessage());
            e.printStackTrace();
        }
    }
}

```



#### Sample Output Log

When you run this program with the test data above, you might see console output similar to:

```
Decrypted Track 2 Data (hex): 323246423245393331463345464146433843333839394142373739463337313945373544333932333635444237343845454137383935363045454237373134443834414237464641354232453136324339424436353634443033444344323430464339443331364341433430313542373832323934333635463930363243413041
Decrypted PIN Block (hex): 041127ADEDAFEFFF
```

#### Explanation

1. **Client Creation**\
   The code creates a `PaymentCryptographyDataClient` using the default configuration. (Make sure your AWS credentials and region are set up correctly.)
2. **Decryption Attributes**\
   The example builds a set of decryption attributes specifying that the data was encrypted with a symmetric algorithm (e.g. TDES in CBC mode). Adjust the mode (or use other attribute types such as Dukpt or EMV if needed) to match your use case.
3. **Building the Request**\
   A `DecryptDataRequest` is built by supplying the key identifier (ARN of the decryption key), the ciphertext, and the decryption attributes.
4. **Handling the Response**\
   The response contains the decrypted plaintext (returned as a hex string). An optional utility method converts this hex string into a regular string if your original plaintext was text-based.
5. **Exception Handling**\
   Errors during the API call are caught and printed to aid in troubleshooting.

#### Additional Considerations

* **Encryption Attributes**:\
  If your POS terminal uses a derived key such as DUKPT, you will need to use the appropriate decryption attributes (for example, by setting Dukpt attributes instead of symmetric ones).
* **Data Format**:\
  Ensure that the ciphertext provided is in the format expected by the API (typically as a hex string).
* **AWS SDK Configuration**:\
  Customize your client (e.g., region, endpoint, credentials) as needed for your environment.

This sample provides a starting point for integrating AWS Payment Cryptography into your Java application to decrypt POS terminal data securely. For full details on the API parameters and supported algorithms, refer to the AWS Payment Cryptography Data Plane API Reference.
