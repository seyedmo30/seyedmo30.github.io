
# 

### چرا ECDH لازم بود و چی شد

ما می‌خواستیم بدنهٔ درخواست بین کلاینت و سرور طوری رمز بشه که حتی اگر کسی ترافیک رو شنود کنه، نتونه متن رو بخونه. با ECDH (الگوریتم تبادل کلید بر پایه‌ی بیضوی) کلاینت و سرور بدون نیاز به جداول احراز هویت، یک کلید موقت مشترک می‌سازند. آن کلید را با یک KDF تبدیل به کلید AEAD کردیم و متن را رمز و امضا (authenticated) کردیم — نتیجه: محرمانگی و یکپارچگی پیام تا وقتی کلید منقضی نشده برقرار است.



### mTLS (Mutual TLS)

mTLS چیست و چرا همیشه ضروری نیست

mTLS یعنی TLS دوطرفه: هم سرور هم کلاینت در مرحله‌ی TLS یک گواهی معتبر نشان می‌دهند و هر دو همدیگر را تایید می‌کنند. این جلوی MITM فعال را قوی می‌گیرد زیرا مهاجم باید گواهی و کلید خصوصی معتبری داشته باشد. با این حال، در برخی سناریوها لازم نیست همیشه از mTLS استفاده شود: اگر بتوانیم public key سرور را به‌صورت امن (out-of-band یا از طریق CA/TLS) به کلاینت تحویل دهیم و/یا شبکه داخلی مطمئن باشد، ترکیب ECDH اپلیکیشن‌لِیر + TLS ساده یا کنترل شبکه اغلب برای بسیاری موارد کافی است — mTLS اضافه‌امنیتی است ولی پیچیدگی و مدیریت گواهی‌ها را بالا می‌برد، مخصوصاً در محیط توسعه.


### terminology

+ AEAD (Authenticated Encryption with Associated Data): هم رمز و هم صحت پیام را تضمین می‌کند.

+ Nonce: عدد/بایت تصادفی که برای یکتا بودن رمز هر پیام لازم است.

+ Ephemeral key: کلید موقت (برای هر درخواست یا دورهٔ کوتاه) که امنیت forward secrecy می‌دهد.

+ KDF / HKDF: تابعی که از shared secret یک کلید متقارن امن می‌سازد.

## example ECDH + AEAD

### server

جزئیات server — چه کارها انجام شد و چرا

تولید/بارگذاری کلید بلندمدت
سرور در startup یک زوج کلید X25519 ساخت یا از فایل/سِکریت لود می‌کند. public این برای کلاینت‌ها قابل خواندن است.

endpoint /pubkey
فقط public key را برمی‌گرداند تا کلاینت‌ها بتوانند ECDH انجام بدهند.

دریافت هدرها در /secure
سرور هدرهای X-Ephemeral-Pub, X-Key-Exp, X-Request-Id, X-Client-IP را می‌خواند. اگر نبودند درخواست را رد می‌کند.

اعتبارسنجی expiry
مقدار X-Key-Exp (unix seconds) را پارس می‌کند:

اگر الان بعد از expiry باشد → reject ("key expired").

اگر expiry خیلی دورتر از allowed (مثلاً بیش از ۶۰ ثانیه) باشد → reject (برای جلوگیری از جعل کلاینتی که expiry بالایی می‌فرسته).

جلوگیری از replay
از یک مپ ساده (یا در prod از Redis) استفاده می‌کنیم: اگر reqID دیده شده باشد درخواست رد می‌شود. این جلوگیری می‌کند کسی ciphertext را دوباره ارسال کند.

تطبیق آی‌پی
سرور r.RemoteAddr را می‌گیرد (معمولاً ip:port) و با X-Client-IP مقایسه می‌کند. اگر mismatch بود، reject — مگر اینکه remote loopback باشد و سرور را برای تست محلی شل کرده باشیم (همان اصلاح پیشنهادی).

مشتق کلید و بازگشایی

ephemeralPub را base64-decode می‌کند، سپس shared = serverPriv.ECDH(ephemeralPub) و همان KDF را اعمال می‌کند تا کلید 32 بایتی بدست آید.

AAD را همانطور که کلاینت ساخت (method|path|clientIP|reqID) می‌سازد.

ciphertext را می‌خواند، نانس را جدا می‌کند، و aead.Open(nil, nonce, ct, aad) اجرا می‌کند. اگر tag نادرست باشد یا AAD فرق کند بازگشایی شکست می‌خورد و خطا برمی‌گردد.

پاسخ به کلاینت
در صورت موفقیت، سرور payload را پردازش می‌کند (در مثال ما لاگ می‌کند) و 200 OK برمی‌گرداند.

```go


// server.go
// Run: go run server.go
package main

import (
	"crypto/ecdh"
	"crypto/rand"
	"crypto/sha256"
	"encoding/base64"
	"errors"
	"fmt"
	"io"
	"log"
	"net"
	"net/http"
	"strings"
	"sync"
	"time"

	"golang.org/x/crypto/chacha20poly1305" // this is in x/crypto but vendored in Go toolchain; if unavailable, use "crypto/cipher" + "crypto/aes"
)

const (
	HeaderEphemeralPub = "X-Ephemeral-Pub"
	HeaderKeyExp       = "X-Key-Exp"    // unix seconds
	HeaderReqID        = "X-Request-Id" // random request id
	HeaderClientIP     = "X-Client-IP"  // IP that client believes is source (used for verification)
)

var (
	serverPriv *ecdh.PrivateKey
	serverPub  []byte

	// replay map: requestID -> expiry time
	replayMu sync.Mutex
	replay   = map[string]time.Time{}
)

func main() {
	// generate server long-term X25519 keypair
	curve := ecdh.X25519()
	priv, err := curve.GenerateKey(rand.Reader)
	if err != nil {
		log.Fatalf("failed to generate server key: %v", err)
	}
	serverPriv = priv
	serverPub = priv.PublicKey().Bytes()
	log.Printf("server public key (base64): %s\n", base64.RawURLEncoding.EncodeToString(serverPub))

	// cleanup goroutine for replay map
	go func() {
		for {
			time.Sleep(30 * time.Second)
			now := time.Now()
			replayMu.Lock()
			for k, t := range replay {
				if t.Before(now) {
					delete(replay, k)
				}
			}
			replayMu.Unlock()
		}
	}()

	http.HandleFunc("/pubkey", handlePubKey)
	http.HandleFunc("/secure", handleSecure)

	addr := ":8086"
	log.Printf("listening on %s", addr)
	log.Fatal(http.ListenAndServe(addr, nil))
}

func handlePubKey(w http.ResponseWriter, r *http.Request) {
	w.Header().Set("Content-Type", "text/plain")
	_, _ = w.Write([]byte(base64.RawURLEncoding.EncodeToString(serverPub)))
}

func handleSecure(w http.ResponseWriter, r *http.Request) {
	plaintext, err := serverDecryptRequest(r)
	if err != nil {
		http.Error(w, err.Error(), http.StatusBadRequest)
		log.Printf("decrypt error: %v", err)
		return
	}
	// Successful decrypt
	log.Printf("DECRYPTED payload: %s", string(plaintext))
	w.WriteHeader(http.StatusOK)
	_, _ = w.Write([]byte("ok"))
}

func serverDecryptRequest(r *http.Request) ([]byte, error) {
	ephemB64 := r.Header.Get(HeaderEphemeralPub)
	expStr := r.Header.Get(HeaderKeyExp)
	reqID := r.Header.Get(HeaderReqID)
	clientIPHdr := r.Header.Get(HeaderClientIP)

	if ephemB64 == "" || expStr == "" || reqID == "" || clientIPHdr == "" {
		return nil, errors.New("missing required encryption headers")
	}
	ephemPubBytes, err := base64.RawURLEncoding.DecodeString(ephemB64)
	if err != nil {
		return nil, fmt.Errorf("bad ephemeral pub: %w", err)
	}

	// parse expiry
	var expUnix int64
	_, err = fmt.Sscanf(expStr, "%d", &expUnix)
	if err != nil {
		return nil, fmt.Errorf("bad expiry header: %w", err)
	}
	expTime := time.Unix(expUnix, 0)
	now := time.Now()
	if now.After(expTime) {
		return nil, errors.New("key expired")
	}
	// enforce maximum 60s from now
	if expTime.Sub(now) > 60*time.Second {
		return nil, errors.New("expiry too far in future")
	}

	// simple replay protection
	replayMu.Lock()
	if t, ok := replay[reqID]; ok && t.After(time.Now()) {
		replayMu.Unlock()
		return nil, errors.New("replay detected: request id seen")
	}
	// mark seen for 120s
	replay[reqID] = time.Now().Add(120 * time.Second)
	replayMu.Unlock()

	// client IP verification: compare header to r.RemoteAddr IP
	clientIPRemote := extractIPFromRemoteAddr(r.RemoteAddr)
	if clientIPHdr != clientIPRemote {
		// you can be stricter (reject) or log — we'll reject for demo
		return nil, fmt.Errorf("client IP mismatch: header=%s remote=%s", clientIPHdr, clientIPRemote)
	}

	// derive symmetric key
	key, err := deriveKey(serverPriv, ephemPubBytes)
	if err != nil {
		return nil, fmt.Errorf("derive key: %w", err)
	}

	// read body
	ct, err := io.ReadAll(r.Body)
	if err != nil {
		return nil, err
	}

	// build AAD: method|path|clientIP|reqID
	aad := []byte(buildAAD(r.Method, r.URL.Path, clientIPRemote, reqID))

	plain, err := decryptWithAEAD(key, aad, ct)
	if err != nil {
		return nil, fmt.Errorf("aead open failed: %w", err)
	}
	return plain, nil
}

func extractIPFromRemoteAddr(remote string) string {
	// remote is usually "ip:port"
	if strings.Contains(remote, ":") {
		host, _, err := net.SplitHostPort(remote)
		if err == nil {
			return host
		}
	}
	return remote
}

func deriveKey(priv *ecdh.PrivateKey, peerPubBytes []byte) ([]byte, error) {
	curve := ecdh.X25519()
	peerPub, err := curve.NewPublicKey(peerPubBytes)
	if err != nil {
		return nil, err
	}
	shared, err := priv.ECDH(peerPub)
	if err != nil {
		return nil, err
	}
	// simple KDF: SHA256(shared || context)
	info := []byte("x25519-chacha20poly1305-v1")
	h := sha256.Sum256(append(shared, info...))
	return h[:], nil
}

func buildAAD(method, path, clientIP, reqID string) string {
	return fmt.Sprintf("%s|%s|%s|%s", method, path, clientIP, reqID)
}

func decryptWithAEAD(key, aad, payload []byte) ([]byte, error) {
	aead, err := chacha20poly1305.New(key)
	if err != nil {
		return nil, err
	}
	if len(payload) < chacha20poly1305.NonceSize {
		return nil, errors.New("payload too short")
	}
	nonce := payload[:chacha20poly1305.NonceSize]
	ct := payload[chacha20poly1305.NonceSize:]
	plain, err := aead.Open(nil, nonce, ct, aad)
	if err != nil {
		return nil, err
	}
	return plain, nil
}

```


### client


جزئیات client — چه چیزهایی انجام شد و چرا

گرفتن public key سرور
کلاینت از GET /pubkey، public key سرور را گرفت (base64). این‌ها برای مشتق shared secret لازم‌اند.

ساختن کلید اپهِمرال
با ecdh.X25519().GenerateKey() یک private/ public اپهِمرال ساختیم. این اپهِمرال فقط برای این درخواست (یا تا مدت کوتاهی) معتبر است.

مشتق کلید سیمِتریک با ECDH
shared = ephemeralPriv.ECDH(serverPub) — این shared بین کلاینت و سرور مساوی خواهد بود. سپس از SHA256 (یا HKDF پیشنهادشده) روی shared + context استفاده می‌کنیم تا کلید 32 بایتی برای ChaCha20-Poly1305 بسازیم.

ساختن AAD
AAD = method|path|clientIP|requestID — این داده‌ها در محاسبه tag احراز هویت AEAD شرکت می‌کنند، یعنی تغییر در این موارد باعث می‌شود بازگشایی (Open) ناموفق شود. این باعث می‌شود نه تنها متن رمزنگاری شود، بلکه اطلاعات مهمی مانند آدرس مسیر/روش/آی‌پی به ciphertext «متصل» شوند.

رمزنگاری AEAD

یک نانس ۱۲ بایتی تصادفی تولید می‌کنیم.

ciphertext = nonce || aead.Seal(nil, nonce, plaintext, aad)
نانس باید برای یک کلید خاص هر بار متفاوت باشد؛ ما نانس را همراه ciphertext می‌فرستیم.

هدرها و ارسال
کلاینت هدرها را می‌گذارد:

X-Ephemeral-Pub = base64(ephemeralPub)

X-Key-Exp = unix timestamp (now + 60s)

X-Request-Id = شناسه یکتا (برای جلوگیری از تکرار)

X-Client-IP = آی‌پی که کلاینت فکر می‌کند سرور خواهد دید (این همان قسمت بود که تو محلی mismatch داشت)


```go

// client.go
// Run: go run client.go
package main

import (
	"crypto/ecdh"
	"crypto/rand"
	"crypto/sha256"
	"encoding/base64"
	"fmt"
	"io"
	"log"
	"math/big"
	"net"
	"net/http"
	"net/url"
	"strings"
	"time"

	"golang.org/x/crypto/chacha20poly1305"
)

const (
	HeaderEphemeralPub = "X-Ephemeral-Pub"
	HeaderKeyExp       = "X-Key-Exp"
	HeaderReqID        = "X-Request-Id"
	HeaderClientIP     = "X-Client-IP"
)

func main() {
	serverURL := "http://127.0.0.1:8086"
	pubKey, err := fetchServerPubKey(serverURL + "/pubkey")
	if err != nil {
		log.Fatalf("fetch pubkey: %v", err)
	}
	log.Printf("got server pubkey: %s", base64.RawURLEncoding.EncodeToString(pubKey))

	// Prepare plaintext
	plaintext := []byte(`{"message":"hello from client","time":"` + time.Now().Format(time.RFC3339) + `"}`)

	// get outbound IP (what the server will likely see)
	myIP := chooseClientIPForServer(serverURL)

	log.Printf("client outbound IP: %s", myIP)

	// request id
	reqID := generateReqID()

	// ephemeral key
	curve := ecdh.X25519()
	ephemPriv, err := curve.GenerateKey(rand.Reader)
	if err != nil {
		log.Fatalf("generate ephemeral: %v", err)
	}
	ephemPub := ephemPriv.PublicKey().Bytes()

	// derive symmetric key
	key, err := deriveKey(ephemPriv, pubKey)
	if err != nil {
		log.Fatalf("derive key: %v", err)
	}

	// build AAD and encrypt
	aad := []byte(buildAAD("POST", "/secure", myIP, reqID))
	ct, err := encryptWithAEAD(key, aad, plaintext)
	if err != nil {
		log.Fatalf("encrypt: %v", err)
	}

	// expiry 60s
	exp := time.Now().Add(60 * time.Second).Unix()

	// build request
	client := &http.Client{Timeout: 10 * time.Second}
	req, err := http.NewRequest("POST", serverURL+"/secure", strings.NewReader(string(ct)))
	if err != nil {
		log.Fatalf("new req: %v", err)
	}
	req.Header.Set(HeaderEphemeralPub, base64.RawURLEncoding.EncodeToString(ephemPub))
	req.Header.Set(HeaderKeyExp, fmt.Sprintf("%d", exp))
	req.Header.Set(HeaderReqID, reqID)
	req.Header.Set(HeaderClientIP, myIP)
	req.Header.Set("Content-Type", "application/octet-stream")

	resp, err := client.Do(req)
	if err != nil {
		log.Fatalf("post error: %v", err)
	}
	defer resp.Body.Close()
	b, _ := io.ReadAll(resp.Body)
	log.Printf("server status: %s, body: %s", resp.Status, string(b))
}

func fetchServerPubKey(url string) ([]byte, error) {
	client := &http.Client{Timeout: 5 * time.Second}
	resp, err := client.Get(url)
	if err != nil {
		return nil, err
	}
	defer resp.Body.Close()
	b, err := io.ReadAll(resp.Body)
	if err != nil {
		return nil, err
	}
	decoded, err := base64.RawURLEncoding.DecodeString(strings.TrimSpace(string(b)))
	if err != nil {
		return nil, err
	}
	return decoded, nil
}

func deriveKey(priv *ecdh.PrivateKey, peerPubBytes []byte) ([]byte, error) {
	curve := ecdh.X25519()
	peerPub, err := curve.NewPublicKey(peerPubBytes)
	if err != nil {
		return nil, err
	}
	shared, err := priv.ECDH(peerPub)
	if err != nil {
		return nil, err
	}
	info := []byte("x25519-chacha20poly1305-v1")
	h := sha256.Sum256(append(shared, info...))
	return h[:], nil
}

func encryptWithAEAD(key, aad, plaintext []byte) ([]byte, error) {
	aead, err := chacha20poly1305.New(key)
	if err != nil {
		return nil, err
	}
	nonce := make([]byte, chacha20poly1305.NonceSize)
	if _, err := rand.Read(nonce); err != nil {
		return nil, err
	}
	ct := aead.Seal(nil, nonce, plaintext, aad)
	return append(nonce, ct...), nil
}

func buildAAD(method, path, clientIP, reqID string) string {
	return fmt.Sprintf("%s|%s|%s|%s", method, path, clientIP, reqID)
}

func getOutboundIP() (string, error) {
	// dial UDP to a public IP (no data sent) to learn local IP
	conn, err := net.Dial("udp", "8.8.8.8:80")
	if err != nil {
		return "", err
	}
	defer conn.Close()
	localAddr := conn.LocalAddr().(*net.UDPAddr)
	return localAddr.IP.String(), nil
}

func generateReqID() string {
	// random hex-ish id
	n, _ := rand.Int(rand.Reader, big.NewInt(1<<62))
	return fmt.Sprintf("%x", n.Uint64())
}

// new helper
func chooseClientIPForServer(serverURL string) string {
	u, err := url.Parse(serverURL)
	if err == nil {
		host := u.Hostname()
		if host == "localhost" || host == "127.0.0.1" || host == "[::1]" {
			return "127.0.0.1"
		}
	}
	// fallback to outbound IP for non-localhost servers
	ip, err := getOutboundIP()
	if err != nil {
		return "127.0.0.1"
	}
	return ip
}

```