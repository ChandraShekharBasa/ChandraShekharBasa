Insecure Cryptography - Weak Algorithm Use

package medicalclinic.services.impl;

import medicalclinic.exceptions.EncryptionException;
import medicalclinic.services.AesEncryptionService;

import javax.crypto.Cipher;
import javax.crypto.spec.SecretKeySpec;
import javax.ejb.Singleton;
import java.nio.charset.StandardCharsets;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import java.util.Arrays;
import java.util.Base64;

@Singleton
public class AesEncryptionServiceImpl implements AesEncryptionService {

    private static final String ALGORITHM = "AES";

    @Override
    public String encrypt(String text, String secretKey) throws EncryptionException {
        try {
            SecretKeySpec secretKeySpec = prepareSecreteKey(secretKey);
            Cipher cipher = Cipher.getInstance(ALGORITHM);
            cipher.init(Cipher.ENCRYPT_MODE, secretKeySpec);
            return Base64.getEncoder()
                    .encodeToString(cipher.doFinal(text.getBytes(StandardCharsets.UTF_8)));
        } catch (Exception e) {
            throw new EncryptionException("Encrypt exception.", e);
        }
    }

    @Override
    public String decrypt(String text, String secretKey) throws EncryptionException {
        try {
            SecretKeySpec secretKeySpec = prepareSecreteKey(secretKey);
            Cipher cipher = Cipher.getInstance(ALGORITHM);
            cipher.init(Cipher.DECRYPT_MODE, secretKeySpec);
            return new String(cipher.doFinal(Base64.getDecoder().decode(text)));
        } catch (Exception e) {
            throw new EncryptionException("Decrypt exception.", e);
        }
    }

    public SecretKeySpec prepareSecreteKey(String myKey) throws NoSuchAlgorithmException {
        byte[] key = myKey.getBytes(StandardCharsets.UTF_8);
        MessageDigest sha = MessageDigest.getInstance("SHA-1");
        key = sha.digest(key);
        key = Arrays.copyOf(key, 16);

        return new SecretKeySpec(key, ALGORITHM);
    }
}
