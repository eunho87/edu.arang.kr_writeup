## auth-bypass-basic

```python
@app.route("/getflag")    
def getflag():
    if not sessionCheck(loginCheck=True):
        return flask.redirect(flask.url_for("login"))

    if getmyinfo(flask.session["login_info"]["userseq"])["balance"] > 1000000000:
        return FLAG
```

계좌의 잔액이 10억보다 많으면 플래그가 출력되는 구조이다

```python
@app.route("/transfer", methods=["GET","POST"])
...
if amount > nowamount:
...
if amount > 1000000000:
....
param = (nowamount-amount, from_address)
```

해당 부분들을 보면 하한에 대한 검증이 없다는 점을 알 수 있다. 즉 amount에 음수값을 넣으면 이체 받는 계좌의 잔액을 10억보다 크게 만들 수 있다.

`/transfer` 에서 `-2000000000` 을 이체하게 되면 아래와 같은 alert 구문이 뜨면서 flag를 획득 할 수 있다.

![](images/34.png)

flag{5aa1bb182e9da650c9bb}
