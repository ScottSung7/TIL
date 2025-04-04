-- 코드 (IV 없는 버전)
```java
@Component
public class JavaCryptoUtil implements PasswordEncoder {
    // IV(Initialization Vector, 초기화 백터) 미적용. (ECB 모드 사용 중)
    private final String encryptionAlgorithm = "AES";

    @Value("${encryption.key}")
    private String key;

    @Override
    public String encrypt(String password) {
        return encrypting(password, setMode(Cipher.ENCRYPT_MODE));
    }

    @Override
    public boolean checkPassword(String password, String encryptedPassword) {
        return password.equals(decrypt(encryptedPassword));
    }

    private String decrypt(String encryptedPassword) {
        return decrypting(encryptedPassword, setMode(Cipher.DECRYPT_MODE));
    }

    private SecretKey getKey() {
        return new SecretKeySpec(key.getBytes(), 0, byteSizeCheck(key), encryptionAlgorithm);
    }

    private int byteSizeCheck(String key) {
        if (key.getBytes().length != 16) throw new JavaCryptoException(BYTE_SIZE_LENGTH_WRONG);
        return 16;
    }

    private Cipher setMode(int mode) {
        try {
            Cipher cipher;
            String padding = "PKCS5Padding";

            String mode_ECB = "ECB";
            cipher = Cipher.getInstance(encryptionAlgorithm + "/" + mode_ECB + "/" + padding);
            cipher.init(mode, getKey());
            return cipher;
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

//getInstance -> NoSuchPaddingException, NoSuchAlgorithmException
//init -> InvalidAlgorithmParameterExceprtion, InvalidKeyException
//doFinal -> IllegalBlockSizeException, BadPaddingException
```
```java
package com.company.member.infrastructure.encryption.aes256.exception.ErrorCode;

import lombok.Getter;
import lombok.RequiredArgsConstructor;

@Getter
@RequiredArgsConstructor
public enum JavaCryptoErrorCode {

    JAVA_ENCRIPTION_ERROR("자바 암호화 에러"),
    JAVA_DECRIPTION_ERROR("자바 복호화 에러"),

    JAVA_CRYPTO_IV_NOT_FOUND("IV값이 DB에 존재하지 않습니다."),
    NO_ID_FOR_IV("IV값을 저장하기 위한 ID값이 존재하지 않습니다."),

    BYTE_SIZE_LENGTH_WRONG("키값의 바이트 사이즈 길이는 16바이트이어야 합니다."),
    BAD_PADDING("패딩이 올바르지 않습니다."),
    NO_SUCH_ALGORITHM("알고리즘이 존재하지 않습니다."),
    NO_SUCH_PADDING("패딩이 존재하지 않습니다."),
    INVALID_ALGORITHM_PARAMETER("알고리즘 파라미터가 올바르지 않습니다."),
    INVALID_KEY("키값이 올바르지 않습니다."),
    ILLEGAL_BLOCK_SIZE("블록 사이즈가 올바르지 않습니다."),
    IV_NOT_FOUND("IV값이 존재하지 않습니다. 비밀번호가 암호화 되어 있지 않을 수 있습니다.");


    private final String deatil;
}
```
