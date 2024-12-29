
package com.scw.airplaneshop.component;

import lombok.extern.log4j.Log4j;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.stereotype.Component;

import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;

@Log4j
@Component
public class CustomPasswordEncoder
        implements PasswordEncoder {

    @Override
    public String encode(CharSequence rawPassword) {
        String encodedPassword = null;
        try {
            MessageDigest messageDigest = MessageDigest.getInstance("MD5");
            messageDigest.update(rawPassword.toString().getBytes());
            byte[] bytes = messageDigest.digest();
            StringBuilder stringBuilder = new StringBuilder();
            for (int i=0; i< bytes.length ; ++i) {
                stringBuilder.append(
                        Integer.toString(
                                (bytes[i] & 0xff) + 0x100, 16).substring(1));
            }
            encodedPassword = stringBuilder.toString();
        } catch (NoSuchAlgorithmException e) {
           log.error(e.getMessage());
        }
        return encodedPassword;
    }

    @Override
    public boolean matches(CharSequence rawPassword,
                           String encodedPassword) {
        if (rawPassword != null && encodedPassword != null
                && encodedPassword.equals(encode(rawPassword))) {
            return true;
        }
        return false;
    }
}
