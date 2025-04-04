-- 코드
```java
@Component
public class JavaCryptoUtil_IV {

    private String encryptionAlgorithm = "AES";

    private final String mode_ECB = "ECB";
    private final String mode_CBC = "CBC";
    private final String padding = "PKCS5Padding";

    @Autowired
    private JavaCryptoIVRepository javaCryptoIVRepository;

    @Value("${encryption.key}")
    private String key;
    //ECB -> ID로 한번 암호화하면 같은 값이 나옴

    public String encrypt(String password, Long id) {
        byte[] iv = saveIv(id);
        return encrypting(password, setMode(Cipher.ENCRYPT_MODE, iv));
    }

    public String decrypt(String encryptedPassword, Long id) {
        byte[] iv = javaCryptoIVRepository.get(id).orElseThrow(
                () -> new JavaCryptoException(JavaCryptoErrorCode.IV_NOT_FOUND)
        ).getIv();
        return decrypting(encryptedPassword, setMode(Cipher.DECRYPT_MODE, iv));
    }

    //Private Methods
    private SecretKey getKey() {
        return new SecretKeySpec(key.getBytes(), 0, byteSizeCheck(key), encryptionAlgorithm);
    }

    private byte[] getIv() {
        byte[] iv = new byte[byteSizeCheck(key)];
        new SecureRandom().nextBytes(iv);
        return iv;
    }

    private byte[] saveIv(Long id) {
        byte[] iv = getIv();
        javaCryptoIVRepository.save(JavaCryptoIVEntity.createJavaCryptoIV(id, iv));
        return iv;
    }

    private int byteSizeCheck(String key) {
        if (key.getBytes().length != 16) throw new JavaCryptoException(BYTE_SIZE_LENGTH_WRONG);
        return 16;
    }

    private Cipher setMode(int mode, byte[] iv) {
        try {
            Cipher cipher;
            if (iv != null) {
                cipher = Cipher.getInstance(encryptionAlgorithm + "/" + mode_CBC + "/" + padding);
                cipher.init(mode, getKey(), new IvParameterSpec(iv));
                return cipher;
            } else {
                cipher = Cipher.getInstance(encryptionAlgorithm + "/" + mode_ECB + "/" + padding);
                cipher.init(mode, getKey());
                return cipher;
            }
        } catch (InvalidAlgorithmParameterException e) {
            throw new JavaCryptoException(INVALID_ALGORITHM_PARAMETER);
        } catch (NoSuchPaddingException e) {
            throw new JavaCryptoException(NO_SUCH_PADDING);
        } catch (NoSuchAlgorithmException e) {
            throw new JavaCryptoException(NO_SUCH_ALGORITHM);
        } catch (InvalidKeyException e) {
            throw new JavaCryptoException(INVALID_KEY);
        }
    }

    private String encrypting(String password, Cipher cipher) {
        try {
            byte[] encrypted = cipher.doFinal(password.getBytes(StandardCharsets.UTF_8));
            return new String(Base64.getEncoder().encode(encrypted));
        } catch (IllegalBlockSizeException e) {
            throw new JavaCryptoException(JavaCryptoErrorCode.ILLEGAL_BLOCK_SIZE);
        } catch (BadPaddingException e) {
            throw new JavaCryptoException(JavaCryptoErrorCode.BAD_PADDING);
        }
    }

    private String decrypting(String encryptedPassword, Cipher cipher) {
        try {
            byte[] decrypted = cipher.doFinal(Base64.getDecoder().decode(encryptedPassword));
            return new String(decrypted, StandardCharsets.UTF_8);
        } catch (IllegalBlockSizeException e) {
            throw new JavaCryptoException(JavaCryptoErrorCode.ILLEGAL_BLOCK_SIZE);
        } catch (BadPaddingException e) {
            throw new JavaCryptoException(JavaCryptoErrorCode.BAD_PADDING);
        }
    }


}
```
```java
package com.company.member.infrastructure.encryption.aes256.db.entity;

import com.company.member.infrastructure.encryption.aes256.exception.JavaCryptoException;
import jakarta.persistence.Entity;
import jakarta.persistence.Id;
import jakarta.persistence.Lob;
import jakarta.persistence.Table;
import lombok.*;

import static com.company.member.infrastructure.encryption.aes256.exception.ErrorCode.JavaCryptoErrorCode.NO_ID_FOR_IV;

@Entity
@Getter
@Builder(access = AccessLevel.PROTECTED)
@AllArgsConstructor(access = AccessLevel.PROTECTED)
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Table(name = "java_crypto_iv")
public class JavaCryptoIVEntity {

    @Id
    private Long id;

    @Lob
    private byte[] iv;

    public static JavaCryptoIVEntity createJavaCryptoIV(Long id, byte[] iv) {
        if (id == null && id < 1) {
            throw new JavaCryptoException(NO_ID_FOR_IV);
        }

        return JavaCryptoIVEntity.builder()
                .id(id)
                .iv(iv)
                .build();
    }

}
```
