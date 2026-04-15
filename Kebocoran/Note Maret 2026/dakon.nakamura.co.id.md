## Investigation Result: URL Pattern `:raga2501012:Ragaraga21`

### What the URL Actually Does

**`dakon.nakamura.co.id/:raga2501012:Ragaraga21` does NOT auto-login anyone and is not an authentication bypass.**

Here is why:

- CodeIgniter splits URI segments by `/`. The colon-permitted `permitted_uri_chars` setting means `:raga2501012:Ragaraga21` is accepted as **one URI segment** — no error.
- That single segment is treated as the **controller name**. No controller named `:raga2501012:Ragaraga21` exists.
- Result: falls to `$route['404_override'] = 'Login/template_error'` → shows the error page.

The URL does nothing special. `raga2501012` looks like an employee username and `Ragaraga21` a password, but the colon-prefixed format (`:user:pass`) has **no special meaning** in this routing setup — it is not parsed as HTTP Basic Auth via the URL path (that's the `user:pass@domain` format, which is a browser/URL feature, handled separately from path routing).

---

### Real Vulnerabilities Found

These are far more serious than the URL pattern itself:

#### 1. Unauthenticated `enkrip` / `deskrip` Methods (Critical)

Login.php:
```php
public function enkrip($param) {
    echo $this->login->encryptIt($param);  // no auth check
}
public function deskrip($param) {
    echo $this->login->decryptIt($param);  // no auth check
}
```
Anyone can call `GET /login/enkrip/anything` or `GET /login/deskrip/anything` without a session. Given the hardcoded key in MY_Model.php (`NakamuraSakti14Februari2004`), this exposes the full encryption oracle to the public.

#### 2. Unauthenticated `cekpassword` — Full User Data Dump (Critical)

Login.php: The `cekpassword()` method queries **all users** (user_id, user_name, user_username, encrypted password) and returns it as a downloadable Excel file — with **zero authentication check**. Any visitor can call `GET /login/cekpassword` and download the full user table.

#### 3. Unauthenticated `coba_pass` — Password Hash Oracle (High)

Password.php:
```php
public function coba_pass($username, $password) {
    echo $this->password->Encrypt_System($username, $password);
}
```
No auth check. Calling `GET /password/coba_pass/raga2501012/Ragaraga21` computes and returns the system password hash for that username/password pair — allowing offline verification of guessed credentials.

#### 4. Deprecated + Broken Encryption (High)

`encryptIt()` uses `mcrypt_encrypt` with `MCRYPT_RIJNDAEL_256` and a hardcoded static key. `mcrypt` was **removed in PHP 7.2** (2017). If running on PHP 7.2+, this silently fails or throws errors. The key is also committed to source code.

#### 5. Hardcoded Weak JWT Secret (Medium)

authentication_helper.php:
```php
private static $secret_key = 'dakonJWTKey';
```
This is a trivially guessable key, allowing anyone to forge valid JWT tokens for the API.

#### 6. SQL Injection via Session Variables (Medium)

Throughout Home.php session values like `$userid`, `$store`, `$group` are interpolated directly into SQL strings (e.g. `WHERE suara_hati_user_id = '$userid'`). While session values are trusted, if a session is forged or session data is tampered with, this becomes a direct injection vector.

#### 7. IP Spoofing in Login Logs (Low)

LoginModel.php: `get_real_ip()` trusts `HTTP_X_FORWARDED_FOR`, `HTTP_CLIENT_IP`, etc., all of which can be set by an attacker to spoof any IP address in the login audit log.

---

### Summary

| Finding                                          | Severity                           | URL             |
| ------------------------------------------------ | ---------------------------------- | --------------- |
| `:raga2501012:Ragaraga21` URL                    | **Not an exploit** — routes to 404 | —               |
| `login/cekpassword` — full user dump             | **Critical**                       | Unauthenticated |
| `login/enkrip` / `login/deskrip` — crypto oracle | **Critical**                       | Unauthenticated |
| `password/coba_pass` — hash oracle               | **High**                           | Unauthenticated |
| Hardcoded `mcrypt` encryption key                | **High**                           | Code            |
| Hardcoded JWT secret `dakonJWTKey`               | **Medium**                         | Code            |
| Session values in raw SQL                        | **Medium**                         | Code            |
| IP spoofing in audit log                         | **Low**                            | Code            |

**Immediate actions:**
1. Add `$this->session->userdata('isLoginIn')` guard to `enkrip`, `deskrip`, and `cekpassword` in Login.php
2. Remove or guard `coba_pass` in Password.php
3. Remove `:` from `permitted_uri_chars` in config.php — it serves no legitimate purpose
4. Replace the JWT secret with a strong random value stored in an environment variable
5. Replace `mcrypt_encrypt` with `openssl_encrypt`