package com.csw.cybersport.auth.service.impl;

import com.csw.cybersport.auth.exception.CryptoException;

import org.apache.commons.codec.binary.Base64;
import org.apache.commons.lang3.StringUtils;

import javax.crypto.*;
import javax.crypto.spec.IvParameterSpec;
import javax.crypto.spec.SecretKeySpec;
import javax.enterprise.context.RequestScoped;
import javax.inject.Inject;
import javax.jms.JMSException;
import java.security.InvalidAlgorithmParameterException;
import java.security.InvalidKeyException;
import java.security.NoSuchAlgorithmException;
import java.security.SecureRandom;
import java.util.Arrays;

import static java.nio.charset.StandardCharsets.UTF_8;

@RequestScoped
public class CryptoService {

    private static final String AES = "AES";

    private final static int IV_LENGTH = 16;

    private static final String TRANSFORMATION = "AES/CBC/PKCS5Padding";

    @Inject
    private LogService logService;

    @Inject
    private SecretKey secretKey;

    public String encrypt(String value) throws CryptoException, JMSException {
        if (StringUtils.isBlank(value)) {
            return "";
        }
        byte[] encoded = value.getBytes(UTF_8);
        return Base64.encodeBase64String(encrypt(encoded));
    }

    public String decrypt(String value) throws CryptoException, JMSException {
        if (StringUtils.isBlank(value)) {
            return "";
        }
        byte[] cipherText = Base64.decodeBase64(value);
        return new String(decrypt(cipherText));
    }

    private byte[] encrypt(byte[] value) throws CryptoException, JMSException {
        try {
            byte[] initVector = new byte[IV_LENGTH];
            (new SecureRandom()).nextBytes(initVector);
            SecretKeySpec keySpec = new SecretKeySpec(secretKey.getEncoded(),
                    AES);
            Cipher cipher = Cipher.getInstance(TRANSFORMATION);
            cipher.init(Cipher.ENCRYPT_MODE, keySpec, new IvParameterSpec(initVector));

            byte[] cipherText = new byte[initVector.length
                    + cipher.getOutputSize(value.length)];
            System.arraycopy(initVector, 0, cipherText, 0,
                    initVector.length);
            cipher.doFinal(value, 0, value.length, cipherText,
                    initVector.length);
            return cipherText;
        } catch (ShortBufferException | IllegalBlockSizeException
                | BadPaddingException | InvalidKeyException
                | InvalidAlgorithmParameterException
                | NoSuchAlgorithmException | NoSuchPaddingException
                | IllegalArgumentException | IllegalStateException e) {
            logService.error("Encryption error: " + e.getMessage());
            throw new CryptoException();
        }
    }

    private byte[] decrypt(byte[] value) throws CryptoException, JMSException {
        try {
            byte[] initVector = Arrays.copyOfRange(value, 0,
                    IV_LENGTH);
            SecretKeySpec keySpec = new SecretKeySpec(
                    secretKey.getEncoded(), AES);

            Cipher cipher = Cipher.getInstance(TRANSFORMATION);
            cipher.init(Cipher.DECRYPT_MODE, keySpec, new IvParameterSpec(initVector));

            return cipher.doFinal(value, IV_LENGTH, value.length - IV_LENGTH);
        } catch (IllegalBlockSizeException | BadPaddingException
                | InvalidKeyException | InvalidAlgorithmParameterException
                | NoSuchAlgorithmException
                | NoSuchPaddingException
                | IllegalArgumentException | IllegalStateException e) {
            logService.error("Decryption failed : " + e.getMessage());
            throw new CryptoException();
        }
    }

}
